# Twinfront Command — 技術架構（Architecture）

- 版本：v0.2｜2026-11-21
- 文件角色：系統邊界、模擬契約、效能與測試策略的單一來源。耐久技術決策見 [`adr/`](adr/)。
- 狀態：方向已定，細節仍是候選；沒有標【已決】的選擇都可以被 ADR 取代。
- 前提：Web 優先、單機、無後端、無帳號；目標裝置包含 iPad Safari 與普通筆電。

## 1. 目標與限制

| 目標 | 具體含義 |
|---|---|
| 可重播 | 同種子、同指令、同規則版本 → 同事件、同結果、同最終快照 |
| 可測試 | 模擬核心不依賴 DOM、瀏覽器或牆鐘；可在 Node 與瀏覽器跑同一套 fixtures |
| 低門檻 | 普通內顯與 iPad 可玩；不需要高階 GPU |
| 低維運 | 靜態檔案即可部署；沒有伺服器、資料庫、帳號或第三方分析 |
| 可移植 | 規則與內容資料可被未來的 Roblox 切片參考；不承諾共用程式碼 |
| 可理解 | 前端只讀快照與事件；UI 不直接修改模擬狀態 |

**主要限制：**

- 單機意味著沒有伺服器權威；反作弊與 PvP 不在 MVP。
- 10 Hz 模擬與 60 Hz 渲染之間需要插值；不能在渲染層偷改模擬。
- iPad Safari 的記憶體、觸控與背景分頁行為與桌面不同，必須真機驗收。
- 瀏覽器分頁被切到背景時，不追趕 tick；恢復後維持暫停，避免一次補算大量時間。

## 2. 系統形狀

```
鍵盤／觸控 ──┐
             ├──> Command queue ──> packages/sim ──> Events + Snapshot
AI 指令源 ───┘                         ↑
                                       │
                              packages/content（JSON schema + 資料表）
                                       │
                                  apps/web（Canvas 2D）
                                       │
                                  畫面與 UI
```

- `packages/sim` 是唯一的規則權威；它不知道現在是即時或回合制，也不知道畫面長什麼樣。
- `apps/web` 負責收集輸入、決定何時推進 tick、插值與渲染。
- `packages/content` 提供兵種、建築、研究、數值與文字 ID；模擬與前端都讀同一份資料。
- `fixtures` 保存種子、指令、預期事件與最終快照，用於重播與跨平台對齊。
- AI 是「指令來源」之一，不是模擬內部的一部分；AI 產生的指令要進 replay。

## 3. 目標目錄

```
apps/
  web/                 Vite + TypeScript；Canvas 2D；輸入、渲染、UI
packages/
  sim/                 純 TS；固定 10 Hz；種子 RNG；整數定點
  content/             JSON schema、資料表、驗證腳本
fixtures/              種子、指令、預期事件、最終快照
docs/
  adr/                 技術決策
  architecture.md      本文件
```

- 這個目錄是 M0 的目標，不是現在已存在。程式碼 scaffold 由 A01 建立。
- 不在 M0 預先建立空資料夾；等第一個檔案進來再建，避免空目錄與假進度。
- 套件管理與 Node 版本在 scaffold 落地時固定；文件不先假裝有指令。

## 4. 核心契約

### 4.1 型別

```ts
type Side = 'A' | 'B';

interface Command {
  tick: number;
  side: Side;
  seq: number;          // 每方單調遞增
  kind: 'produce' | 'research' | 'build' | 'surrender';
  payload: unknown;     // 由 kind 決定；schema 驗證
}

interface GameEvent {
  tick: number;
  id: number;           // 單調遞增；重播時穩定
  kind: string;         // unit.spawned, unit.destroyed, outpost.destroyed, ...
  entityId?: number;
  payload: unknown;
}

interface Snapshot {
  tick: number;
  factions: unknown;
  entities: unknown;
  result: 'ongoing' | 'A' | 'B' | 'draw';
}

interface Replay {
  rulesVersion: string;
  schemaVersion: string;
  seed: number;
  commands: Command[];   // 包含 AI 指令
  finalHash: string;     // 最終快照的穩定雜湊
}
```

- `Command.tick` 是「生效 tick」，不是按下按鈕的牆鐘時間；UI 在兩個 tick 之間送出的指令，排到下一個 tick。
- `seq` 由每一方單調遞增；同一 tick 的指令以 `(tick, side, seq)` 排序，避免瀏覽器事件順序造成差異。
- `Event.id` 在單局內單調遞增；重播時必須完全一致。
- `finalHash` 用穩定序列化計算；欄位順序、數字格式與陣列排序都要固定。
- 所有 schema 變更都要昇 `schemaVersion`；規則或數值行為變更要昇 `rulesVersion`。

### 4.2 Tick 順序

每個 tick 固定執行：

1. 驗證並套用本 tick 的指令（資源、佇列、冷卻、終局狀態）。
2. 完成研究與生產；生成新單位。
3. 移動、尋路與目標選擇。
4. 產生攻擊意圖，批次結算傷害。
5. 套用傷害、死亡、建築摧毀；處理前哨首次獎勵。
6. 檢查終局條件。
7. 產生事件與快照。

- 這個順序是契約；改變順序可能改變結果，必須昇 `rulesVersion` 並補 fixtures。
- 先驗證指令再結算，避免「資源不夠但已生成」或「終局後仍下單」。
- 傷害批次結算，避免單位陣列走訪順序影響結果。
- 死亡實體在該 tick 內標記，結算後移除；事件仍保留 `entityId`。

### 4.3 決定性

- 模擬內禁止 `Math.random`、`Date.now`、`performance.now`、DOM、網路與瀏覽器 API。
- 隨機使用種子 RNG；RNG 的呼叫順序是規則的一部分，不能因 UI 或平台改變。
- 位置、生命、傷害、計時使用整數或整數定點；避免 `Math.sin`、`Math.sqrt` 等平台差異。若需要距離，使用平方距離或自訂整數 sqrt。
- 排序一律使用完整鍵：`entityId`、`tick`、`side`、`seq`；不要依賴引擎的預設排序穩定性。
- 時間只存在於 tick 編號；牆鐘時間只屬於 `apps/web` 的排程器。
- AI 的即時版本可以有自己的啟發式，但 replay 記錄的是它產生的指令，不是重新跑 AI。

### 4.4 節奏層（重要）

模擬不知道「即時」或「回合」。`apps/web` 的 pacing scheduler 決定何時呼叫 `step()`：

| 模式 | 行為 |
|---|---|
| 即時可暫停 | 以 10 Hz 累加器推進；暫停時停止累加；背景分頁自動暫停，不追趕 |
| 同步回合 | 規劃階段不推進 tick；執行階段一次推進固定 N 個 tick（例如 5 秒＝50 tick） |

這個分層讓 DEC-02 的 A/B 測試不需要重寫模擬，只需要換排程器與 UI。若 W2 選擇同步回合，`packages/sim` 不變。

### 4.5 快照與插值

- 模擬每 tick 產生完整快照；前端保留最近數個快照，以固定延遲（候選 100 ms）插值渲染。
- 渲染不直接讀可變的模擬物件；只讀唯讀快照。
- 若渲染落後，丟棄舊快照而不是無限追趕；寧可視覺短暫落後，不要阻塞模擬。
- 背景分頁回來時，顯示暫停狀態；不要一次補算背景期間的 tick。

## 5. 渲染與前端

| 項目 | 候選做法 | 狀態 |
|---|---|---|
| 渲染器 | Canvas 2D；精靈與俯視格線 | 【暫定】ADR-0001 |
| 3D | 不在 MVP；只有原型比較證明收益才考慮 Three.js | 【不做】MVP |
| 圖層 | 地形 → 建築 → 單位 → 特效 → UI | 【暫定】 |
| 縮放 | DPR 上限 2；低畫質降到 1 | 【暫定】 |
| 相機 | 固定俯視角；不旋轉；可平移／縮放？MVP 固定全圖 | 【待決】 |
| 輸入 | 指標事件統一處理滑鼠與觸控；主要操作不依賴 hover | 【已決】 |
| UI | HTML/CSS 覆蓋層或 Canvas UI？MVP 優先用 HTML/CSS 做可存取按鈕 | 【暫定】 |
| 音訊 | MVP 暫定無；若加入，Web Audio＋靜音 | 【待決】 |

- UI 不直接修改 sim；所有操作都轉成 `Command`。
- 主要按鈕與單位選取要有足夠觸控面積；不能要求右鍵、hover 或鍵盤快捷鍵。
- 低畫質只降渲染負載（DPR、粒子、裝飾、更新頻率），不得改變模擬、相剋或結果。
- 所有畫面文字都從字串表讀取，不寫死在圖或程式碼；首發語言待 DEC-07。

## 6. 內容與資料

### 6.1 ID 與 schema

- ID 使用穩定、可讀的命名：`unit.infantry`、`unit.tank`、`unit.artillery`、`unit.scout`、`tower.fixed`、`outpost.standard`、`hq.standard`、`research.armor`、`research.industry`。
- 每個項目有 `domain`（`ground | air | sea`）、`role`、`cost`、`buildTime`、`damageTags` 等欄位；實際 schema 由 A02 定義。
- `schemaVersion` 管資料形狀；`rulesVersion` 管規則與數值行為。兩者不要混用。
- 內容資料是數值的唯一來源；`game-design.md` 只保留摘要與候選值。

### 6.2 CI 驗證

- 重複 ID、缺引用、負值、未知欄位、缺必填欄位 → 失敗。
- 每個單位／建築／研究都要有可讀名稱與圖示 ID；缺圖示時用 placeholder，不讓 UI 空白。
- 平衡變更要附變更前後與理由；CI 只保證 schema，不保證好玩。

## 7. 存檔與設定

- 只存：音量／畫質／減少動態、教學是否看過、最近一場結算摘要、lane 預設選擇。
- 不存：中途戰局、完整 replay、玩家身分。
- 使用版本化 key；讀到舊版或損壞資料時，回到預設值並提示一次，不阻止開局。
- 存檔格式變更要補 migration，或明訂不支援舊版；不要默默讓玩家失去設定。
- 瀏覽器清除資料即失去設定；因為沒有帳號與雲端，這是可接受限制，文件要說清楚。

## 8. 測試與品質閘門

| 層級 | 內容 | 何時跑 |
|---|---|---|
| 單元 | 資源、佇列、研究、傷害、終局、排序 | 每個 PR |
| 重播 fixtures | 同種子同指令 → 同事件與 finalHash | 每個 PR |
| 不變量 | 資源不為負、佇列不超限、終局後無指令、實體不重複 | 每個 PR |
| Schema | 內容 JSON 驗證 | 每個 PR |
| 瀏覽器 smoke | 開局、下單、結算、再戰 | 每個 PR |
| 多瀏覽器／種子回歸 | Safari、Chromium、Firefox；固定種子 | Nightly |
| 真機 | M1 iPad、M1 Pro、低階 Windows 內顯 | RC／里程碑 |
| 效能 | 三種場景：典型 36–40、正常 60、壓力 120 | RC／里程碑 |

### 8.1 效能場景

| 場景 | 單位數 | 目的 |
|---|---|---|
| 典型 | 全場 36–40 | 反映預期實戰；UI 與特效也要正常 |
| 正常上限 | 全場 60 | P0 驗收門檻；p95 幀間隔 ≤33.3 ms |
| 壓力 | 全場 120 | 不得崩潰或失去操作；不要求 30 FPS |

- 每台裝置暖機後跑 3 次、每次 10 分鐘；記裝置、OS、瀏覽器、視窗、畫質、溫度情境。
- 量測要分開：渲染幀間隔、模擬 tick CPU、記憶體、載入大小。不要用一個「FPS」數字代表全部。
- 舊原型與新灰盒分開記錄；不要把舊專案數據當成新專案證據。
- CI 的 GPU／VM 數據不算真機證據。

### 8.2 載入預算

- 首局必要下載 ≤5 MiB（壓縮後傳輸）是候選門檻。
- 建議拆分：JS ≤1.5 MiB、美術 ≤2.5 MiB、字型 ≤0.5 MiB、音訊 ≤0.5 MiB；實際以量測為準。
- 音訊與非必要裝飾可延後載入；首局不能等全部素材。

## 9. CI、預覽與版本

- PR：lint、typecheck、schema、sim/replay、單一瀏覽器 smoke。
- Nightly：多瀏覽器、固定種子回歸、依賴掃描。
- RC：真機、完整一局、壓測、存檔損壞恢復、預覽回滾。
- 預覽只出靜態檔；使用不可變版本，保留上一版可回滾。內部部署細節不寫進公開 repo。
- 版本規則：`schemaVersion` 管資料形狀，`rulesVersion` 管規則；破壞性變更要補 migration 或明訂不支援。
- Replay 只保證在同一 `rulesVersion` 內重現；跨版本讀取要顯示警告，不假裝可重現。

## 10. 隱私與安全

- MVP 不載入第三方分析、廣告、追蹤像素或遠端設定。
- 不收集帳號、Email、位置或裝置指紋；真機測試資料以匿名代碼記錄。
- 不在 repo 放 token、內網主機名、私有 repo 連結或個人資料；PR 檢查要抓。
- 依賴要 pin lockfile；新增依賴要有理由，避免供應鏈風險。
- 若未來加入 PvP、雲端存檔或排行榜，需新的架構決策與伺服器權威模型；不是 MVP 的漸進功能。

## 11. 待決技術問題

| ID | 問題 | 目前傾向 | 決策點 |
|---|---|---|---|
| TQ-01 | Canvas 2D 或 Three.js？ | Canvas 2D；3D 只在證明收益後 | ADR-0001 重評 |
| TQ-02 | 單位資料用 ECS、陣列或 SoA？ | 先用簡單陣列＋明確排序；不為架構而架構 | M0 |
| TQ-03 | UI 用 HTML/CSS 或 Canvas？ | HTML/CSS 優先，較易觸控與無障礙 | M1 |
| TQ-04 | 尋路：格線 A* 或固定路徑？ | 雙路固定路徑＋局部閃避；MVP 不做全地圖尋路 | M1 |
| TQ-05 | 音訊要不要進 MVP？ | 暫定不要 | W4 |
| TQ-06 | 字串表格式與翻譯流程？ | JSON 字串表；繁中先行 | DEC-07 |
| TQ-07 | Replay 是否壓縮／存哪？ | 測試 fixtures 存 repo；玩家 replay 不存 | M1 |
| TQ-08 | 是否做 PWA／離線快取？ | MVP 不做；靜態部署即可 | M3 |

## 12. 非目標

- 伺服器權威、真人 PvP、配對、排行榜、雲端存檔。
- 3D 渲染、物理引擎、可破壞地形。
- 關卡編輯器、Mod API、使用者生成內容。
- 帳號、付費、廣告、分析、推播。
- 歷史裝備資料庫或真實戰術模擬。

## 13. 相關決策

- [ADR-0001：Web 優先與 Canvas 2D](adr/0001-web-first-mvp.md)
- [ADR-0002：決定性模擬核心](adr/0002-deterministic-sim-core.md)
- [ADR-0003：單機、無後端 MVP](adr/0003-single-player-no-backend.md)
- [ADR-0004：對局節奏（提議中）](adr/0004-match-pacing-proposed.md)

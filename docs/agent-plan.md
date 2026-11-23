# Frontline Signal（《前線情報官》）｜Agent 實作計畫（Public MIT 版）

> 版本：v0.3｜2026-11-23｜Repo：`frontline-signal`｜已整合情報欺敵方向（ADR-0001）與公開 repo 去識別化規範。

## 01｜專案定位與參考基準

本文件給 coding agents 與整合負責人使用，放置於公開 MIT repo `frontline-signal`（`docs/agent-plan.md`）。需求基準請一律讀取 `docs/product-brief.md` 與 `docs/adr/0001-intel-deception-direction-and-engine.md`；目前 checkpoint 見 `docs/checkpoints/`。本文件不得包含任何私有基礎設施資訊、主機規格、runner 編號或憑證。

| 層級 | 範圍 | 明確不做 |
|---|---|---|
| MVP（Web 單機） | 1v1 對 AI、5–7 區連通地圖、2 座 HQ、2 個補給節點、步兵／裝甲／偵察／火砲、掃描與假接觸、1 條共用產線＋1–2 項研究、雙尺度縮放、結算與統計 | 真人 PvP、多人同步、離線掛機、多陣營科技樹、海空單位、Roblox 上架 |
| P1（驗證後） | 碉堡定案與實裝、更多地圖、架空早期飛機與船艦、更多欺敵行動（隱蔽／反制） | 直接搬運受版權保護的歷史或他作素材 |

## 02｜核心循環與模擬規則（灰盒參數，均可調）

核心循環：開局偵察 → 區域部署與排產 → 自動交戰 → 爭奪補給節點／摧毀前哨 → 掃描或施放誘餌 → 研發或增援 → 攻破 HQ → 結算揭露真相。

| 模組 | 規則與契約 |
|---|---|
| 經濟（REQ-04） | 起始 120 補給；基礎每 10 秒 +10；每控制 1 個補給節點每 10 秒 +8；首次摧毀敵方前哨 +50。普通單位與誘餌擊破不給補給。 |
| 單位與排產（REQ-02） | 步兵 30/5s、偵察 45/6s、裝甲 70/10s、火砲 85/12s。共用生產佇列上限 3 筆待生產訂單（不是場上單位上限）；下單即扣費；人口上限暫定每方 30，計入排隊與存活單位。碉堡保留獨立 ID 命名空間待 DEC-02 定案。 |
| 情報層（REQ-09） | 每個實體對敵方有四種呈現：不可見、未確認接觸、已確認單位、最後已知位置。**掃描**：短暫揭露指定區域真實接觸；**假接觸（誘餌）**：產生未確認標記，具冷卻與數量上限，不可攻擊、不可占點、不可計入擊毀統計。 |
| AI 規則 | AI 與玩家共用經濟與冷卻，**只能讀取自身 `PlayerView`**，禁止存取 `WorldState` 中敵方未曝光實體；AI 必須可能被假接觸吸引調兵。 |
| 終局與統計（REQ-05） | 固定 10 Hz tick 批次結算；同 tick 雙 HQ 歸零判平局；12 分鐘超時依 HQ 血量百分比 → 已控節點數 → 平局判定。終局拒收指令；統計由事件流推導，以 `entityId` 去重，結算才揭露誘餌時間軸。 |

## 03｜工程架構與三層資料分離

公開 repo `frontline-signal`（MIT），採用 TypeScript + Vite。畫面引擎暫定 **Three.js（WebGL 2）**，並保持 `apps/web` 與模擬核心完全解耦，以便第 2 週視需要以 PixiJS 做同快照 2D 對照。

| 目錄 | 職責與邊界 |
|---|---|
| `packages/sim` | 純 TS、零 DOM／牆鐘／渲染依賴。維護 `WorldState`，提供 `step(world, commands)` 與 `buildPlayerView(world, side)`。固定 10 Hz、種子 RNG、整數運算、穩定排序。 |
| `packages/content` | JSON Schema 與資料表（`schemaVersion`, `rulesVersion`）；CI 檢查重複 ID、缺引用、負值與未知欄位。 |
| `apps/web` | 僅讀取 `PlayerView` 與插值快照，渲染 `RenderScene`（遠景 2D 棋子／近景低模 3D）。鍵鼠與觸控共用指令管線，跨尺度保持選取狀態；背景切頁自動暫停。 |
| `fixtures/` | 種子、指令序列、預期 `PlayerView`／事件與終局快照；用於回歸測試、防作弊驗證與未來跨平台對齊。 |
| `docs/adr/` | 記錄玩法方向、引擎選擇、統計契約與平台決策。 |

最小介面契約：`Command(tick, side, seq, kind, payload)`、`Event(tick, id, kind, entityId, payload)`、`WorldState`、`PlayerView(tick, side, visibleUnits, contacts, nodes, resources, intelCooldowns)`、`Replay(rulesVersion, seed, commands)`。

## 04｜效能、真機與公開 CI 策略

以下為候選驗收門檻，非已實測結果。CI 僅負責自動化正確性與基本 smoke；真機效能與觸控驗收獨立進行。

| 項目 | 驗收基準與做法 |
|---|---|
| 測試矩陣（REQ-06） | M1 iPad Safari、M1 Pro 16 GB（Safari／Chromium）、Windows 內顯筆電（型號待補）。記錄 OS、瀏覽器、視窗、DPR 與溫度情境。 |
| 負載門檻 | 正常場景 60 可動單位：渲染幀間隔 p95 ≤ 33.3 ms、模擬 tick CPU p95 ≤ 8 ms；壓力場景 120 單位不得崩潰或失控。首局必要下載 ≤ 5 MiB（壓縮後）。 |
| 雙尺度優化 | 遠景切換 2D 棋子並停送高模繪製；低畫質先限 DPR=1、關閉即時陰影與全螢幕後製、批次繪製重複幾何，不改變模擬規則。 |
| 公開 CI 分層 | PR：lint、typecheck、schema、`packages/sim` 與 `PlayerView` 防作弊測試、單一瀏覽器 smoke。Nightly：多種子回歸與長局穩定性。部署、預覽環境與私有 CI 不在本 repo 範圍。 |

## 05｜Roblox 後續路線（非 MVP）

Roblox 不納入 Web MVP，亦不承諾共用 TypeScript 執行碼；可移植資產為數值表、規則文件、原創素材原檔與 `fixtures/` 回放向量。

| 階段 | 交付與停損條件 |
|---|---|
| 私有技術切片 | 於團隊擁有的 Roblox Experience 以 Luau／Studio 重寫最小規則（單圖、兩兵種、掃描／誘餌、HQ），伺服器權威下發單方視圖，與 Web `fixtures/` 核對結果。 |
| 裝置與效能 | 以 iPad Roblox 客戶端搭配 MicroProfiler 驗證觸控與正常場景 30 FPS。 |
| 發布與合規 | 依 Creator Hub 區分 16+／Trusted Friends 與未滿 16 歲受眾之驗齡、內容問卷、費用／訂閱與平台評估門檻；首版不開自由文字聊天，不使用真實歷史敏感標誌。 |

## 06｜短中長期里程碑

以一位 part-time 主開發搭配 coding agents 估算；W＝週，checkpoint 對應見下表。

| 時段 | 交付 | 出關／停損條件 |
|---|---|---|
| W1（CP1） | 專案骨架、`WorldState`/`PlayerView` 分離、純棋子情報灰盒 | 5–8 人試玩；玩家能分辨已確認／未確認接觸並因情報改變部署。 |
| W2（CP2） | Three.js 雙尺度縮放切片（視需要做 PixiJS 同快照對照）＋真機基準＋2 張目標畫面實機重現 | 拍板 DEC-02（節奏與碉堡）、DEC-03（引擎與效能門檻）與 DEC-07（美術方向）。 |
| W3–6（CP3） | 完整 1v1 對 AI、補給節點、研發、教學、結算時間軸與回放測試 | 新局到結算完整重跑；達真機正常負載門檻；至少 5 人完成教學與對局。 |
| W7–12（CP4） | 美術一致化、觸控打磨、兩輪封測、防雪球平衡、第二地圖候選 | 解決反覆出現的可讀性與雪球問題；未過不擴充海空。 |
| M4–6+ | 二選一：Web 海空擴充 或 Roblox 私有技術切片 | 單線驗證通過後再排另一項；真人 PvP 另立里程碑。 |

## 07｜Agent 工單清單（A01–A08）

標準指令（由 A01 建立，非現成 repo 指令）：`npm ci`、`npm run dev`、`npm run verify`、`npm run test:e2e`、`npm run build`。

| ID／角色 | 依賴 | 交付 | 驗收條件 |
|---|---|---|---|
| A01 Infra | 無 | `frontline-signal` MIT 骨架、`AGENTS.md`、lockfile、公開 PR CI | 乾淨 checkout 可跑 `npm run verify`；不含任何私有環境設定或憑證。 |
| A02 Rules | 無 | `packages/content` schema、ADR-0001 落實、事件與 `PlayerView` 契約 | 無效 ID／負值被拒；提供有效與無效 fixtures。 |
| A03 Sim | A02 | `WorldState`、`buildPlayerView`、經濟、排產、掃描／誘餌、HQ 終局 | 誘餌零傷害／零占點／零擊毀計數；同種子同指令結果一致；AI 無法讀取未曝光敵軍。 |
| A04 Render/UI | A01, A02 | Three.js 雙尺度場景（遠景棋子／近景低模）、觸控指令、暫停 | 全程觸控無需 hover；跨尺度保持選取狀態；支援 mock `PlayerView` 獨立開發。 |
| A05 Gameplay | A03, A04 | 情報驅動 AI、教學、結算統計與誘餌揭露時間軸 | AI 會對未確認接觸做出合理調度；結算統計與事件流 100% 一致。 |
| A06 QA/Perf | A01；A03–05 | 壓測種子（60/120 單位）、真機量測報告、視需要做 PixiJS 2D 對照 | 記錄裝置、瀏覽器、p95 幀間隔與 tick 耗時；不以 CI 數據冒充真機。 |
| A08 Art/Audio | 無（平行） | `docs/art-direction.md` 落實：探索稿 → 目標畫面＋可重現性說明 → 音效與動畫清單 | 目標畫面在 M1 iPad 實機重現且達效能門檻；對外圖像只用實機擷取。 |
| A07 Platform | Web MVP 後 | Roblox 私有切片與差距報告 | 與 Web fixtures 結果一致、iPad 客戶端可完賽；不擅自公開或付費。 |

CP1 做 A01、A02 全部，A03–A06 的灰盒子集（模擬、2D 灰盒前端、AI v0、試玩分析），以及 A08 的探索稿；細項見 `docs/checkpoints/cp1-intel-greybox.md`。

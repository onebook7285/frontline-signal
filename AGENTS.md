# AGENTS.md — coding agents 工作規則

## 開工前必讀（依序）

1. `docs/product-brief.md`：需求基準（REQ／H／DEC ID）。
2. `docs/adr/`：已做的方向與架構決策。
3. `docs/agent-plan.md`：規則參數、架構邊界、工單。
4. 目前 checkpoint 文件（`docs/checkpoints/`）與你被派的工單。

需求以 `docs/product-brief.md` 為唯一基準；其他文件引用 ID，不另外維護一份需求。

## 不可違反的不變式

這些規則都要有自動化測試保護。改到相關程式時，測試必須跟著存在且通過。

1. **模擬純度**：`packages/sim` 零 DOM、零牆鐘（`Date.now`／`performance.now`）、零渲染依賴。只用種子 RNG、整數運算、穩定排序。
2. **決定性**：同 `rulesVersion`＋同種子＋同指令序列 → 完全相同的事件流與終局快照。
3. **情報隔離**：UI 與 AI 只能讀 `PlayerView`。`buildPlayerView(world, side)` 的輸出不得包含對該方未曝光的敵方實體或其真實屬性。
4. **誘餌語意**：假接觸不造成傷害、不占點、不計入擊毀統計、不給補給；結算時才向雙方揭露。
5. **終局**：終局後拒收所有指令；統計由事件流推導並以 `entityId` 去重。

## 標準指令

由 CP1-01（A01）建立；建立前這些指令尚不存在。

| 指令 | 用途 |
|---|---|
| `npm ci` | 安裝相依 |
| `npm run dev` | 本機開發伺服器 |
| `npm run verify` | lint＋typecheck＋schema 檢查＋單元測試（PR 必過） |
| `npm run test:e2e` | 瀏覽器 smoke |
| `npm run build` | 產出靜態檔 |

## 公開 repo 規則

- 本 repo 公開、MIT。**不得**提交：主機名稱、內網位址、runner 或機器規格、帳號、token、金鑰、`.env` 檔。
- 不得提交試玩者個資或原始試玩紀錄；只收去識別化的彙總結果。
- 不得提交來源或授權不明的素材（見 README「素材與授權」）。
- 部署、預覽環境與私有 CI 不在本 repo 管理範圍。

## 交付

- 一張工單一個 branch、一個 PR；PR 描述逐條對照工單驗收條件，附 `npm run verify` 結果。
- 規則參數變動要同步更新 `docs/agent-plan.md`；方向或架構變動另開 ADR。
- 不擴大工單範圍；發現範圍外問題就另開 issue。

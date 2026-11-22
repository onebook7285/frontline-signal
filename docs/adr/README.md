# 架構決策紀錄（ADR）

- 版本：v0.2｜2026-11-21
- 用途：記錄耐久、難以回頭的技術決策，以及當初的替代方案與重評條件。
- 對象：工程、未來的維護者、AI 協作者。

## 什麼時候寫 ADR

- 改變模擬契約、tick 順序、RNG、資料格式、渲染器、部署模型、依賴策略或測試策略。
- 三個月後有人會問「為什麼當初這樣選？」的技術決定。
- 不是每個 PR 都要 ADR；數值微調、文案、局部重構不需要。

## 狀態

| 狀態 | 意思 |
|---|---|
| Proposed | 提議中；尚未拍板，不能當成對外承諾 |
| Accepted | 已採用；要改需開新 ADR 取代 |
| Superseded | 已被新 ADR 取代；保留舊檔 |
| Deprecated | 仍存在但不建議使用 |
| Rejected | 曾被考慮但未採用；保留理由 |

## 編號與規則

- 四位數遞增：`0001`、`0002`…；不要重用編號。
- 檔名：`0001-short-title.md`。
- 一個 ADR 一個決策；不要把多個無關選擇塞進同一篇。
- 已 Accepted 的 ADR 不改結論；要改就開新 ADR，舊的標 `Superseded by ADR-xxxx`。
- 可以補充「實作結果」與「重評日期」，但不要改寫歷史理由。
- 公開 ADR 不寫內網主機、私人 repo、帳號、預算或個人資料。

## 索引

| ADR | 標題 | 狀態 | 日期 |
|---|---|---|---|
| [0001](0001-web-first-mvp.md) | Web 優先，TypeScript＋Vite＋Canvas 2D | Accepted（暫定） | 2026-11-21 |
| [0002](0002-deterministic-sim-core.md) | 決定性模擬核心 | Accepted（暫定） | 2026-11-21 |
| [0003](0003-single-player-no-backend.md) | 單機、無後端 MVP | Accepted（暫定） | 2026-11-21 |
| [0004](0004-match-pacing-proposed.md) | 對局節奏：即時可暫停（提議中） | Proposed | 2026-11-21 |

## 模板

新增 ADR 時複製 [`0000-template.md`](0000-template.md)，填完後更新本索引。

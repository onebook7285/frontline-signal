# 貢獻指南（Contributing）

- 專案階段：Pre-alpha，CP1（情報灰盒）規劃完成、尚無可玩版本。
- 文件語言：繁體中文（台灣用語）為準；英文摘要與翻譯歡迎。
- 授權：程式與文件以 MIT 釋出；素材依 [`ASSETS.md`](ASSETS.md)。

## 現在最有用的貢獻

| 類型 | 可以做什麼 | 從哪裡開始 |
|---|---|---|
| 設計審閱 | 指出規則矛盾、不可測試的主張、玩家會誤解的地方 | 開 issue，引用 `REQ`／`H`／`DEC` 編號 |
| 文件 | 錯字、英文摘要、連結 | 小 PR 即可 |
| 試玩 | CP1 灰盒出來後協助測試 iPad 與桌機 | [`docs/checkpoints/cp1-intel-greybox.md`](docs/checkpoints/cp1-intel-greybox.md) 第 5 節 CP1-08 |
| 程式 | CP1-01 骨架完成後的 sim、web、content | 先開 issue，等維護者確認範圍 |

## Issue 與 Pull Request

- 用既有模板：Bug、Feature、Task、Playtest。驗收條件要可觀察。
- 涉及產品範圍、平台、授權或架構的變更，先開 DEC／ADR（範本：[`docs/adr/0000-template.md`](docs/adr/0000-template.md)）。
- 一個 PR 一件事；標題用 `feat:`、`fix:`、`docs:`、`test:`、`chore:`、`refactor:`；預設 squash merge。
- 規則變更要附前後差異與 replay fixture；改 tick 順序、RNG、傷害結算或 schema 時昇 `rulesVersion`／`schemaVersion`。
- 不變式與標準指令見 [`AGENTS.md`](AGENTS.md)；人類貢獻者同樣適用。

## 素材與內容

- 只送原創、明確授權或公共領域素材，並記入 `ASSETS.md`。
- 內容、符號與暴力尺度依 [`docs/content-policy.md`](docs/content-policy.md)。

## AI 輔助

歡迎使用；你仍對正確性、授權、測試與安全負責。請在 PR 註明 AI 輔助範圍。不要讓 AI 自動合併、發布或改 repo 設定。

## 行為與安全

- 互動遵守 [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)；安全問題依 [`SECURITY.md`](SECURITY.md) 私下回報。
- 這是 part-time 專案，維護者會盡量在兩週內回應。

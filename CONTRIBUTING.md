# 貢獻指南（Contributing）

- 專案階段：設計與灰盒前期；尚無可玩版本。
- 文件語言：以繁體中文（台灣用語）為準；英文摘要、翻譯與名詞校正歡迎。
- 授權：程式與文件以 MIT 釋出；素材依 [`ASSETS.md`](ASSETS.md) 的規則。

## 1. 現在最有用的貢獻

| 類型 | 可以做什麼 | 從哪裡開始 |
|---|---|---|
| 設計審閱 | 指出規則矛盾、不可測試的主張、玩家會誤解的地方 | 開 issue，引用 `REQ`／`DEC`／`DQ` 編號 |
| 文件 | 修正錯字、補英文摘要、改善圖表與連結 | 小 PR 即可，不必先開 issue |
| 試玩 | 灰盒出來後，協助測試 iPad 與桌機 | 看 [`docs/playtesting.md`](docs/playtesting.md) |
| 美術探索 | 可讀的輪廓、陣營識別、placeholder | 看 [`docs/art-direction.md`](docs/art-direction.md) |
| 程式 | M0 scaffold 之後的 sim、web、content | 先開 issue，等維護者確認範圍 |

## 2. 開始之前

1. 讀 [`docs/README.md`](docs/README.md) 的文件地圖與狀態標籤。
2. 搜尋既有 issue，避免重複。
3. 開 issue 說明問題、範圍與驗收；不要直接送大 PR。
4. 涉及產品範圍、平台、授權或架構的變更，先開 DEC／ADR，不要用 PR 偷渡。
5. 涉及內容、歷史符號或素材，先讀 [`docs/content-policy.md`](docs/content-policy.md)。

## 3. Issue

- 用既有模板：Bug、Feature、Task、Playtest。
- 標明影響區域（sim／web／content／art／qa／platform／docs）與大小（S／M／L）。
- 驗收條件要可觀察；不要寫「做得好看」。
- 需要決策的 issue 標 `needs:decision`，連到 DEC／ADR。
- 不要貼 secrets、內網主機名、私人 repo 連結、試玩者個資。

## 4. Pull Request

- 一個 PR 一件事；避免順便重構。
- 先開 issue 再開 PR；錯字、連結、明顯文件修正可直接 PR。
- 標題用 `feat:`、`fix:`、`docs:`、`test:`、`chore:`、`refactor:`。
- 分支：`feat/<topic>`、`fix/<topic>`、`docs/<topic>`、`chore/<topic>`；AI 協作可用 `agent/<tool>/<topic>`。
- 預設 squash merge；PR 標題會成為 main 的 commit 訊息。
- 作者不自行合併公開 PR；由維護者審閱。

### PR 必填

- 摘要與動機。
- 連結的 issue。
- 變更類型與影響區域。
- 測試證據（指令、結果、截圖／錄影）。
- 風險與未測事項。
- 文件／ADR 是否更新。
- 素材來源與授權（若涉及美術、音訊、字型）。
- 是否使用 AI 輔助；若大部分由 AI 產生，請註明。

### 規則變更

- 附變更前後、種子、預期影響。
- 更新 replay fixture；CI 要能重播。
- 改變 tick 順序、RNG、傷害結算或 schema → 昇 `rulesVersion`／`schemaVersion`。
- 不要只貼新數字；說明為什麼改、預期解決什麼。

## 5. 程式風格

- 先以 M0 scaffold 的設定為準；目前沒有既有程式碼可遵循。
- 模擬核心：純 TS、無 DOM／牆鐘／網路；固定 10 Hz；種子 RNG；整數定點；穩定排序。
- 前端：不直接改 sim；所有操作轉成 `Command`；主要操作不依賴 hover。
- 內容：數值放 JSON；ID 穩定；schema 驗證要過。
- 新增依賴要有理由；不要為了小功能引入大套件。
- 註解解釋「為什麼」，不要重述程式在做什麼。

## 6. 測試

- 每個 PR 至少要有相關測試或證據；沒有測試的變更要說明原因。
- 規則變更要有 replay fixture 或單元測試。
- UI 變更要附截圖或短錄影；觸控變更要說明裝置。
- 效能相關變更要附量測方法與裝置；不要用 CI 的 GPU 數據假冒真機。
- 文件 PR 要確認連結可用、沒有私人資訊。

## 7. 素材

- 只送原創、明確授權或公共領域素材。
- 在 PR 與 [`ASSETS.md`](ASSETS.md) 記錄：作者、來源、授權、修改、署名。
- 不送抓取來的遊戲素材、電影截圖、未授權字型或音效包。
- AI 輔助素材要記錄工具與使用條款；授權不明就不用。
- 內容符號、暴力尺度與歷史指涉依 [`docs/content-policy.md`](docs/content-policy.md)。

## 8. 授權

- 送出的程式與文件以 MIT 授權；不需要簽 CLA。
- 素材依 `ASSETS.md` 的授權規則；若你沒有權利授權，不要送。
- 若貢獻涉及第三方程式碼，請在 PR 註明來源與授權，並確認相容。

## 9. AI 輔助

- 歡迎使用 AI 輔助；你仍對正確性、授權、測試與安全負責。
- 不要讓 AI 自動合併、發布或改 repo 設定。
- 大型重構、依賴新增、schema 變更先開 issue。
- 不要把私人區內容、token、主機名、私有 repo 貼進 PR。
- 人類審閱會特別看：幻覺 API、未測路徑、抄來的程式碼、過度重構。

## 10. 行為與內容

- 所有互動遵守 [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)。
- 遊戲內容遵守 [`docs/content-policy.md`](docs/content-policy.md)。
- 不歡迎騷擾、歧視、仇恨符號、真實政治宣傳、盜版素材、未經同意的個資。
- 維護者可移除違規內容或拒絕合併。

## 11. 回覆時間

這是 part-time 的開源專案，不保證即時回覆。維護者會盡量在兩週內回應 issue／PR；若超過，可以禮貌 ping 一次。不要重複開 issue 或直接 tag 所有人。

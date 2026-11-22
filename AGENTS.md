# AGENTS.md

給 AI 編碼助理（以及人類協作者）的 repo 規則。這份文件不授予發布、合併或改 repo 設定的權限。

## 先讀

1. [`docs/README.md`](docs/README.md)：文件地圖、狀態標籤、單一來源。
2. [`docs/product-brief.md`](docs/product-brief.md)：需求與產品決策。
3. [`docs/game-design.md`](docs/game-design.md)：規則與候選數值。
4. [`docs/architecture.md`](docs/architecture.md)：模擬契約與技術邊界。
5. [`docs/ways-of-working.md`](docs/ways-of-working.md)：Issue／PR／ADR 流程。

## 不可協商

- **決定性**：`packages/sim` 不得使用 `Math.random`、`Date.now`、`performance.now`、DOM、網路或平台相關浮點數學。使用種子 RNG、整數定點、穩定排序。
- **單一來源**：不要在多份文件重抄需求表、規則表或里程碑。用 ID 引用。
- **狀態標籤**：把候選值寫成【已決】是錯誤；不確定就用【暫定】、【假設】或【待決】。
- **不偷改範圍**：新增功能、依賴、schema 變更、平台方向，先開 issue／DEC／ADR。
- **不偷改數值**：平衡數值變更要附理由、種子與預期影響；不要只改數字。
- **不留 secrets**：token、`.env`、內網主機名、私有 repo 連結、試玩者個資不得進 repo。
- **不自行發布**：不要 push 到 main、不要合併 PR、不要改 GitHub 設定、不要發布 release，除非人類明確授權。
- **內容政策**：虛構陣營、無真實仇恨符號、無血腥；素材要有授權。

## 程式規則

- `packages/sim`：純 TS、固定 10 Hz、tick 順序固定、Command／Event／Snapshot／Replay 契約。
- `apps/web`：只讀快照與事件；所有操作轉成 `Command`；主要操作不依賴 hover。
- `packages/content`：數值放 JSON；ID 穩定；schema 驗證要過。
- 新增依賴：先問「標準函式庫或既有依賴能不能做？」；不能才加，並在 PR 說明。
- 測試：規則變更補 replay fixture；UI 變更附截圖；效能變更附量測方法。
- 不要大規模重排檔案或格式化不相關的程式；PR 要小。

## 文件規則

- 繁體中文（台灣用語）為準；英文摘要歡迎。
- 日期用 `YYYY-MM-DD`；狀態用【已決】【暫定】【假設】【待決】【不做】。
- 新文件先問「它是不是某類資訊的唯一來源」；不是就別新增。
- 修改需求／決策時，更新 ID 與交叉引用；不要覆蓋歷史。
- HTML／簡報是快照，標日期，不手改。

## 工作流程

1. 讀 issue 與相關文件。
2. 確認範圍、驗收、可改路徑。
3. 在小 PR 中實作；必要時先寫 ADR。
4. 跑測試與 lint（有 scaffold 後）。
5. 在 PR 寫：動機、變更、測試證據、風險、未測事項。
6. 等人類審閱；不要自行合併。

## 不確定時

停下來問。寧可少做一個功能，不要偷偷改規則、範圍或公開承諾。

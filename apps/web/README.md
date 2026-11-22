# apps/web

Web 前端（Vite＋TypeScript＋Canvas 2D）。**尚未建立 scaffold。**

職責：

- 收集鍵盤／觸控輸入，轉成 `Command`。
- 決定何時推進模擬 tick（即時或同步回合）。
- 讀取 `Snapshot` 與 `Event`，做插值與渲染。
- 顯示 UI、教學、結算與設定。

邊界：

- 不直接修改模擬狀態。
- 主要操作不依賴 hover、右鍵或鍵盤。
- 低畫質只降渲染負載，不改模擬與相剋。

詳見 [`docs/architecture.md`](../../docs/architecture.md)。Scaffold 由 M0 的 A01 建立。

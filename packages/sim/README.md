# packages/sim

純 TypeScript 模擬核心。**尚未建立 scaffold。**

職責：

- 固定 10 Hz tick；種子 RNG；整數定點。
- 指令驗證、生產、研究、移動、交戰、傷害批次結算、終局判定。
- 產生 `GameEvent`、`Snapshot` 與可重播的 `Replay`。

邊界：

- 無 DOM、牆鐘、網路、瀏覽器 API。
- 不使用 `Math.random`、`Date.now`、`performance.now`。
- 不依賴渲染或 UI；節奏由 app 層決定。

詳見 [`docs/architecture.md`](../../docs/architecture.md) 與 [`docs/adr/0002-deterministic-sim-core.md`](../../docs/adr/0002-deterministic-sim-core.md)。

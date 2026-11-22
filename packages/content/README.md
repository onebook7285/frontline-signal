# packages/content

遊戲內容資料與 schema。**尚未建立 scaffold。**

職責：

- 兵種、建築、研究、數值的 JSON 資料。
- `schemaVersion` 與 `rulesVersion`。
- CI 驗證：重複 ID、缺引用、負值、未知欄位。

邊界：

- 內容資料是數值的唯一來源；`docs/game-design.md` 只保留摘要。
- ID 穩定、可讀：`unit.infantry`、`tower.fixed`、`research.armor`。
- 不把渲染素材或 UI 文案塞進規則資料；用 ID 引用。

詳見 [`docs/architecture.md`](../../docs/architecture.md) §6。

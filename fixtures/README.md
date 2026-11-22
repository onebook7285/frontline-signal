# fixtures

重播與回歸測試資料。**尚未建立。**

每個 fixture 應包含：

- 種子、`rulesVersion`、`schemaVersion`。
- 指令序列（含 AI 指令）。
- 預期事件與最終快照 `finalHash`。
- 一段簡短說明：這個 fixture 驗證什麼。

用途：

- 同種子、同指令 → 同事件與結果。
- 跨平台／跨瀏覽器規則對齊。
- 規則變更時抓回歸。

詳見 [`docs/architecture.md`](../docs/architecture.md) §8。

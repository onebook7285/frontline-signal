# Frontline Signal（《前線情報官》）

> 一款可在一般電腦與平板瀏覽器上遊玩的情報欺敵型即時戰略遊戲：在可縮放的戰場上偵察、佈下假情報、調度部隊，攻破敵方指揮部。
>
> *A browser-based, intel-and-deception real-time strategy game set in WWII and the early post-war era. Scout, bluff with decoy contacts, redeploy across a zoomable front, and break the enemy HQ.*

**狀態：Pre-alpha／企劃階段**——目前 repo 只有需求、設計決策與開發計畫，尚無可遊玩版本。

## 核心設計

| 支柱 | 說明 |
|---|---|
| 情報不對稱 | 戰場區分「已確認敵軍／未確認接觸／最後已知位置」；你看到的不一定是真的。 |
| 欺敵與反制 | 首版兩個情報行動：**掃描**（揭露指定區域真實接觸）與**假接觸**（製造誘餌標記）。 |
| 區域機動 | 5–7 個連通區域＋橫向換側通道＋補給節點，取代固定推線。 |
| 低微操指揮 | 下令後單位自動執行；只有移動、固守、撤回三種核心指令。 |
| 雙尺度縮放 | 遠景是戰略棋盤棋子，近景是低模 3D 單位；觸控與滑鼠共用同一套操作。 |
| 畫面與聲音 | 「作戰室沙盤」的視覺語言：遠景是桌上地圖與棋子，近景是手繪微縮模型；情報事件有專屬的無線電與電報音效。 |
| 輕量可及 | 目標是不需要高階顯卡：M1 iPad Safari 與內顯筆電都能順跑。 |

**展示即成品**：本專案對外使用的畫面一律是實機擷取；概念圖必須能在實機重現並達到效能門檻，做不到就修改概念圖（見 [美術方向](docs/art-direction.md) 第 1 節）。

## MVP 範圍

- **做**：1v1 對 AI、單張 5–7 區地圖、2 座 HQ、2 個補給節點、步兵／裝甲／偵察／火砲、掃描與假接觸、1 條共用產線＋1–2 項研究、雙尺度縮放、結算與擊毀統計。
- **不做**：真人 PvP、多人同步、離線掛機、多陣營科技樹、海空單位、Roblox 上架。

## 技術方向

TypeScript + Vite 的 monorepo。模擬核心與畫面完全分離，並強制三層資料邊界：

```
WorldState（真實狀態，10 Hz 決定性模擬）
   └─ buildPlayerView(side) → PlayerView（單方可見情報；UI 與 AI 都只能讀這層）
                                  └─ RenderScene（Three.js 雙尺度呈現，暫定）
```

AI 不能讀取 `WorldState` 中未曝光的敵軍——這是測試會檢查的硬性規則，不是約定。引擎選擇與理由見 [ADR-0001](docs/adr/0001-intel-deception-direction-and-engine.md)。

## 文件

| 文件 | 內容 |
|---|---|
| [docs/product-brief.md](docs/product-brief.md) | 需求登錄（REQ）、設計假設（H）、待決策（DEC）——需求的唯一基準 |
| [docs/agent-plan.md](docs/agent-plan.md) | 規則參數、架構、效能門檻、里程碑、工單（給開發者與 coding agents） |
| [docs/art-direction.md](docs/art-direction.md) | 畫面、動畫、音效方向與「展示即成品」規則 |
| [docs/adr/](docs/adr/) | 架構與方向決策紀錄 |
| [docs/checkpoints/](docs/checkpoints/) | 各 checkpoint 的範圍、驗收與出關條件 |
| [AGENTS.md](AGENTS.md) | coding agents 的工作規則與不可違反的不變式 |
| [CONTRIBUTING.md](CONTRIBUTING.md) | 如何參與、issue／PR 規則 |
| [docs/content-policy.md](docs/content-policy.md) | 架空設定、符號、暴力尺度與素材紅線 |

## 路線圖

| Checkpoint | 目標 | 出關條件 |
|---|---|---|
| **CP1 情報灰盒** | 純棋子灰盒，可完整打一局對 AI | 試玩者能分辨已確認／未確認接觸，並因情報改變部署 |
| CP2 雙尺度真機切片 | Three.js 遠近縮放＋真機效能基準＋目標畫面實機重現 | 拍板對局節奏、碉堡、渲染引擎與美術方向 |
| CP3 完整 1v1 | 補給、研發、教學、結算時間軸、回放測試 | 達真機正常負載門檻；5 人完成教學與對局 |
| CP4 封測 | 美術與音效一致化、動畫與觸控打磨、防雪球平衡 | 可讀性與雪球問題收斂；展示圖全部來自實機 |

## 素材與授權

- 程式碼與文件採 [MIT License](LICENSE)。
- 本 repo 只收原創或授權明確、相容於公開散布的素材，並登錄於 [ASSETS.md](ASSETS.md)。**不收**從其他遊戲擷取的模型、貼圖、音效或介面。
- 載具採真實二戰車型並依真實尺寸建模；陣營為架空，不使用真實國旗、軍徽、部隊標記或歷史敏感標誌（見 [內容政策](docs/content-policy.md)）。

# Twinfront Command

**A short-session, lane-based strategy game with auto-combat, built for the browser.**

> Working title. Design phase — there is no playable build yet.
> 目前是設計與灰盒前期，尚無可玩版本。

Twinfront Command is a 1v1 strategy game against AI. You spend supply, choose a lane for each production order, queue units, and invest in research while your forces fight automatically. The target match length is 8–12 minutes. The design is intentionally small: two lanes, two HQs, four unit types, one resource. No account, no backend, no idle progression.

The setting is a fictional mid-20th-century-inspired world. It is not a historical simulation, and it is not affiliated with any real-world military or political organization.

## What it is

- A complete, short match you can finish on an iPad or a laptop.
- A game about **deployment and timing**, not about clicking fast.
- A readable greybox-first project: rules and replayability before art polish.
- An open-source project under the MIT license.

## What it is not

- Not an idle/incremental game. The Chinese word 「放置」 in early notes means **deployment** (placing and ordering units), not offline farming.
- Not a base-builder, a historical wargame, or a realism simulator.
- Not multiplayer in the MVP. No accounts, no cloud saves, no leaderboards, no analytics.
- Not a Roblox game yet. Roblox is a research track after the web MVP.

## Status

| Item | Current state |
|---|---|
| Phase | M0 — foundations, rules, greybox |
| Playable build | None yet |
| Match pacing | Working assumption: real-time with pause; a synchronous-turn variant is being compared in W2 (DEC-02) |
| Platform | Web first; desktop + iPad Safari are the reference devices |
| MVP scope | P0 candidate, not frozen until W2 |
| License | MIT for code and docs; third-party assets keep their own licenses |

## Repository map

```
docs/
  product-brief.md      requirements, evidence, product decisions
  game-design.md        core loop, candidate rules, design questions
  architecture.md       simulation contracts, rendering, testing
  roadmap.md            milestones, workstreams, risks, cut line
  playtesting.md        validation protocol and decision rules
  art-direction.md      visual pillars and MVP asset list
  content-policy.md     fictional setting, violence, asset licensing
  ways-of-working.md    issues, PRs, ADRs, definition of done
  adr/                  durable technical decisions
  research/             time-sensitive research with dates
apps/web/               web client (planned; scaffold lands in M0)
packages/sim/           deterministic simulation core (planned)
packages/content/       rules and content data (planned)
fixtures/               replay and regression fixtures (planned)
```

Start with [`docs/README.md`](docs/README.md) for the document map and status legend.

## How to follow or help

The project is design-phase. The most useful contributions right now are:

- **Design review**: point out unclear rules, contradictions, or untestable claims.
- **Playtesting**: once the greybox exists, 5–8 testers per round on iPad and desktop.
- **Art exploration**: readable silhouettes and faction identity, not historical accuracy.
- **Documentation**: English summaries, typo fixes, and clearer diagrams.

Read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a PR. For conduct and content rules, see [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) and [`docs/content-policy.md`](docs/content-policy.md).

## Design principles

1. **Finish a match in one sitting.** 8–12 minutes is a design constraint, not a marketing claim.
2. **Command, don't micro.** The player makes a few meaningful choices; units execute.
3. **Readability beats realism.** Silhouette, icon, and motion must identify units; color is never the only channel.
4. **One resource, no farming loops.** No repeatable kill bounties, no second currency to patch balance.
5. **Deterministic by construction.** Same seed and commands produce the same result, on every device.
6. **Small enough to finish.** When scope and quality conflict, cut scope.

## License

Code and documentation are released under the [MIT License](LICENSE). Original art and audio assets, when added, will be licensed separately; third-party assets retain their own licenses. See [`ASSETS.md`](ASSETS.md).

## 中文摘要

Twinfront Command（暫譯《雙線指揮》）是一款 8–12 分鐘的雙路策略遊戲：玩家花補給、選路線、排生產、投資研究，單位下場後自動交戰。節奏（即時可暫停或同步回合）仍在 W2 驗證。目標是讓普通筆電與 iPad 都能順跑，不需要高階顯卡，也不做離線掛機。目前只有設計文件，還沒有可玩版本。專案以 MIT 授權公開；原始美術與音訊的授權另行標註。

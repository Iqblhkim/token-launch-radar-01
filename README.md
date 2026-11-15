[![Token Radar](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![Rank](https://make.fun/badge/token-radar-01/Iqblhkim/token-launch-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01)
[![Reward](https://make.fun/badge/token-radar-01/Iqblhkim/token-launch-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01)

# Token Radar — pre-token protocols ranked by implied value

An entry for [make.fun's "Build the best token launch radar" bounty](https://make.fun/bounty/token-radar-01) ($1,325 USDC). It reads the sealed DefiLlama snapshot, finds the protocols whose token does not exist yet, ranks them by an implied launch value, and shows how to farm each one.

## What it does

- **Covers the whole watchlist** (40/40) and adds every rankable protocol below the $50M floor — **178 rows** in all. Rankable means: no token on record, and not an exchange, bridge, liquid-staking or restaking product, wrapped or staked product, or versioned pool. The watchlist's own test is applied to every row, not just to the watchlist.
- **Model radar-2** — one function of the snapshot's own figures, stated in full on the dashboard:
  `implied = round(TVL × 0.25 × growth × maturity × breadth)`, where growth is the clamped 7-day TVL change, maturity is age with no token (0.6–1.0 over 3 years), and breadth is +0.04 per extra chain (up to 1.20×). Top row: Steakhouse Financial, $652.4M implied.
- **Farm plan per protocol** — 7 steps with cost and time, derived from the protocol's category, chains, TVL and age, plus a 30-day checkpoint step and an estimated total cost.
- **Dashboard** — dark, static, no network: search, category / chain / minimum-TVL filters, watchlist-only toggle, sortable columns, 25-row pages, a detail drawer per protocol (model breakdown with a recompute check), a live-token view (24 protocols set aside with their market caps), and the model with a worked example.

## Run it

Node 22.18+ (24 works too), standard library only, nothing to install:

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

It writes `out/board.json`, `out/data.js`, `out/dashboard/`, `out/screenshots/` and `out/dossier.json`. Run it twice: both runs write identical bytes.

## Layout

- `main.ts` — entrypoint: reads the snapshot, builds the board, writes the outputs and the dossier.
- `lib/model.ts` — the radar-2 model, the rankable test, the farm plans, the snapshot parser and the board checks.
- `vendor/makefun-board.ts` — make.fun's dossier library, copied unchanged from the kit.
- `dashboard/` — `index.html`, `app.css`, `app.js` (42,859 bytes in all: the judge reads every line).
- `screenshots/` — the board, a protocol's detail view, and a filtered view, all 1440×900 renders of this dashboard.
- `dev/` — local checks: `verify.mjs` (the gate's seven rows, the dossier rebuild, determinism) and `shots.mjs` (the screenshots plus rendered-page checks against `board.json`).

## License

MIT. See [LICENSE](LICENSE).

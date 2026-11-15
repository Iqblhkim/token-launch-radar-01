# Token Radar — build notes

Entry for make.fun bounty **"Build the best token launch radar"** (`token-radar-01`, $1,325 USDC escrowed, deadline Mon 14 Dec 2026 10:00 UTC). Built 2026-11-16 in `~/workspace/makefun/token-launch-radar-01/`. Node 24.22.1 on this machine (the bounty needs 22.18+; a newer major counts).

## What the program does

`main.ts` (Node, standard library only) reads the sealed snapshot (`/input/snapshot.json`, or `--input`), scores every rankable pre-token protocol with model **radar-2**, and writes `board.json`, `data.js`, `dashboard/`, `screenshots/` and `dossier.json` to `/output` (or `MAKEFUN_OUTPUT_DIR`). The dashboard is a static dark page with no network: search, category / chain / minimum-TVL filters, watchlist-only toggle, sortable columns, 25-row pages, a per-protocol detail drawer (model breakdown + farm plan + estimated cost), a live-token view, and the model with a worked example.

## Design decisions (how this differs from the house starter, model radar-1 / 2-plain)

1. **The early radar applies the watchlist's own test.** The starter ranks early rows by TVL alone, so on this snapshot **39 of its 160 early rows are bridges, liquid-staking / restaking products and wrapped or staked products** (e.g. TxFlow Bridge, STKESOL by SOL Strategies, Bitget SOL, Mantle Restaking, Scroll Bridge) — exactly what the bounty says to exclude. This board applies the full test (no token; category not Bridge / Canonical Bridge / Liquid Staking / Liquid Restaking / Restaked BTC / CEX; no version marker; no whole word wrapped/staked/staking; no exchange-prefixed name) to every row.
2. **Full coverage, no TVL cap on the early radar.** 178 rows: all 40 watchlist + all 138 rankable protocols below the $50M floor. The starter stops at 200 rows with 39 bad ones; this board has 178 clean ones.
3. **The watchlist's line is respected above the floor.** 20 protocols at or above $50M that the watchlist leaves out (Sentora Curator $2.76B, Polymarket International, Circle Gateway, USDCx, Defi Saver Asset Management, …) are **not** ranked. The watchlist was built from DefiLlama records carrying a parent protocol, which this snapshot does not carry, so the house's exclusion of them is followed rather than second-guessed from ten fields.
4. **Model radar-2 = the house model + one stated factor.** `implied = round(TVL × 0.25 × growthMult × maturityMult × breadthMult)`, growth clamped [−60, 120] × 0.5 weight, maturity 0.6 + min(age, 1095)/1095 × 0.4 (unknown age 365d), breadth = 1 + 0.04 × min(chains−1, 5) (1.00–1.20×). Chain breadth is a snapshot figure with real variance here (113 single-chain, 46 with 4+ chains, max 45). The page recomputes every row and checks it against the board's number.
5. **Farm plans: 7 steps, category-specific.** Step 2 (the action) and step 3 (the weekly action) are specialised for all 33 categories the rankable set carries (risk curators, allocators, basis trading, yield aggregators, CDP, options, indexes, payments, privacy, governance, OTC, CeDeFi, card, Anchor/Decentralized BTC, …), capital $50/$100/$150/$250 by TVL, hold 60/45/30 days by age, plus a 30-day checkpoint step (unwind if the points page or TVL went quiet) and a claim-only-from-the-official-link step. The page regenerates each plan from the same rules; the detail drawer shows an estimated total cost.
6. **Dashboard built to be read whole.** Source total 43,241 bytes (cap 65,536), so the judge reads every line. Dark theme, WATCHLIST tags on watchlist rows, hash state (`#q`, `#category`, `#chain`, `#p`) with a `hashchange` listener, pagination (25/page, 8 pages).

## Test results (all run locally against the kit's snapshot)

- `node dev/verify.mjs out` — **18/18 PASS**: board 178 rows (100–4,000), 290,126 bytes; ranks exactly 1..178; implied non-increasing; watchlist 40/40; farm steps 3–12 × 20–600 chars on every row; dashboard 3 files, 42,859 bytes, no `http(s)` string anywhere; screenshots 3, each ≤ 512 KiB, 1440×900; dossier byte-for-byte identical to a fresh `buildDossier` rebuild, **738,760 bytes** (< 1.5 MiB).
- `node dev/shots.mjs` — **ALL PASS, no console/page errors**: board page 1 shows 25 rows (rank 1 Steakhouse Financial), pager "Page 1 of 8 — 178 rows", model panel worked example names the top row; **8 detail views sampled across categories** (Steakhouse, Gauntlet, Curvance, Bittensor dTAO, OnRe, Tori Finance, vfat.io, Monarq) — all 7 farm steps match `board.json` exactly and the page recomputes the board's implied value in every case; filtered view (Lending) shows 14 rows, only Lending; live tab shows 24 live-token protocols. Screenshots: 1-board.png 159,063 B; 2-detail.png 211,114 B; 3-filtered.png 147,135 B (all 1440×900 PNG, in the 300–350 KB target band or below). All three screenshots were **viewed** and read cleanly.
- **Determinism**: two runs into separate dirs produce byte-identical outputs (`diff -r` clean), re-verified after the last edit.
- **Top of the board** (radar-2): 1. Steakhouse Financial $652,363,296 · 2. Gauntlet $388,531,755 · 3. Veda $373.5M · 4. USDD $231.4M · 5. Upshift $134.0M · 10. OpenTrade $55,527,676 · 178. Monarq $727,032. Total implied value $3,350,485,975.
- **Starter baseline** (for comparison): the kit starter run on the same snapshot gives 200 rows, rank 1 Steakhouse $543,636,080 — the difference is the breadth factor (Steakhouse runs on 11 chains → 1.20×) and the cleaned early set.
- Runtime check: `node main.ts` runs on this machine's Node 24.22.1 with no flags; the code uses erasable TypeScript only (interfaces + type annotations, no enums) and the standard library, so it runs under the grader's Node 22 the same way the starter does.

## Transcript / recording situation

**Standard transcript bundle — NOT strict live-record.** This bounty takes the usual makefun bundle (kit CLI `bundle`); there is no `record`-command requirement as in markdown-render-speed-02. This session began 2026-11-16 ~06:13 WIB (2026-11-15 23:13 UTC), **after** the competition opened (2026-11-14T10:00:00Z), so `thinking_in_window` is satisfiable. `thinking_fresh` (the `makefun anchor …` line from kit `init`) applies only to **source-bundle** entries; this entry is for a **GitHub repository** submission (the established flow: repo + `.makefun/owner.json`), so no init/anchor was run — and `init` would have refused anyway, since it commits its own LICENSE into an empty repo while the parent creates the GitHub repo.

Authorship: `main.ts`, `lib/model.ts` and all three `dashboard/` files were **written in this session** (typed via the agent's file writes, then revised after testing — see the hashchange listener, the `watchlist_slugs` fix, the search-placeholder fix). `vendor/makefun-board.ts` is the kit's library **copied unchanged**, exactly as its own header instructs ("Copy it to vendor/makefun-board.ts in your repository"). No starter file was copied in as entry code.

## Files (this workspace = the future repo contents)

- `main.ts`, `lib/model.ts`, `vendor/makefun-board.ts`, `dashboard/index.html`, `dashboard/app.css`, `dashboard/app.js`, `screenshots/1-board.png`, `screenshots/2-detail.png`, `screenshots/3-filtered.png`, `README.md`, `LICENSE` (MIT, Iqblhkim), `BUILD_NOTES.md` (this file)
- `dev/verify.mjs`, `dev/shots.mjs` — local test tools (harmless in the repo; can be left out)
- `out/` — last local build output (regenerated by `main.ts`; not part of the repo)

## What the parent still does (repo + bundle + submit)

1. Create the public GitHub repo `Iqblhkim/token-launch-radar-01` (created after the 2026-11-14 opening) with these files; add `.makefun/owner.json` = `{"wallet": "FwVYoFhusw6aGqn1WWE53qyirPGikWRDWoYF6LcL1DqW"}` with the first commit (repo owner is the user's GitHub login, so the claim file is belt-and-braces); set GitHub About: website `https://make.fun/bounty/token-radar-01`, topic `makefun` (repo_about check).
2. README already carries the three live badges (competition, rank, reward) with owner/repo filled in — verify no `{owner}` placeholder remains.
3. Bundle this session (and its parent session if it worked on the entry) with the kit CLI, `--muse-usage` for the Muse usage rows, then `node cli/makefun.mjs check --kit <kit> --repo <repo> --transcripts transcripts.tar.gz`.
4. Submit: `node cli/makefun.mjs submit --competition token-radar-01 --repo-url https://github.com/Iqblhkim/token-launch-radar-01 --commit <sha> --title "Token Radar" --bundle transcripts.tar.gz --keypair <dedicated wallet keypair>`. Max 5 submissions/UTC day, 10 total; last before the deadline is final. **Do not claim any prize without the user's permission.**

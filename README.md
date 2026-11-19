[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Cmochance/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Cmochance/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

# Token Radar — pre-token protocols ranked by implied value

An entry for make.fun's `token-radar-01` bounty. It reads a sealed DefiLlama snapshot of 758
protocols, finds the ones that record no token and are not a wrapped, staked or bridged product of
a token that already trades, and ranks them by what their launch would be worth — then hands in a
static dashboard with the board, a detail view per protocol and a farm plan for each one.

## The model (radar-2, comps-calibrated)

Every row is priced off the live-token protocols in its own category:

- `multiple` — the category's median mcap/TVL among trading peers; with 5+ peers the category
  median is used, with 3–4 it is blended half-and-half with the global median (0.19x), with
  fewer it is the global median. Clamped to 0.02x–1.5x so one outlier peer never sets the price.
- `growth` — `1 + clamp(change_7d, -60, 120) / 100 × 0.5`
- `maturity` — `0.6 + min(age_days, 1095) / 1095 × 0.4` (age unknown: 365 days)
- `implied = round(tvl_usd × multiple × growth × maturity)`

Confidence follows from the peer count, a known listing age and a reported seven-day change.
Risk flags name what could cut the value: a fast TVL fall, a very young protocol, a single chain,
a small base. No protocol name is special-cased; the dashboard prints the comparables table the
multiple came from, and the page recomputes every row from the snapshot figures.

The board covers the whole 40-protocol watchlist plus all 158 other rankable pre-token protocols
in the snapshot (198 rows). Live-token protocols are set aside with their market caps shown.

## Run it

```sh
# from the repository root, with the sealed snapshot mounted at /input (or --input <path>)
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input /path/to/snapshot.json
```

`main.ts` (Node 22, standard library only) writes `board.json`, `data.js`, `dashboard/`,
`screenshots/` and `dossier.json` to the output directory. Two runs write the same bytes.

Open `out/dashboard/index.html` over HTTP (it reads `../data.js`) — no network requests.

## Layout

- `main.ts` — the program: snapshot in, board + dashboard + dossier out
- `lib/model.ts` — the radar-2 model, farm plans, board build
- `vendor/makefun-board.ts` — the kit's dossier library, byte for byte
- `dashboard/` — the static page (index.html, app.css, app.js; no framework, no network)
- `screenshots/` — board, protocol detail, filtered view
- `board.json`, `dossier.json` — the outputs of the run that produced this commit

## License

MIT. See `LICENSE`.

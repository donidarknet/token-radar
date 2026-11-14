# Token Radar — model radar-2 (make.fun token-radar-01)

The best token launch radar: it reads a sealed DefiLlama snapshot of 758 protocols, works out which
ones record no token yet, and ranks them by what their launch would be worth — each with the steps
to farm it. An AI judge compares the dashboards two at a time; the 10 best share $1,325.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

Node 22.18 or later, standard library only, no install. `main.ts` reads the snapshot, writes
`board.json` (198 ranked rows), `data.js`, copies `dashboard/` and `screenshots/` to the output
directory, and writes `dossier.json` with `buildDossier` from `vendor/makefun-board.ts`.

## The model (radar-2)

Every row is scored by one function of the snapshot's own figures; no protocol name is
special-cased.

- **Multiple, calibrated from the snapshot itself**: the median market-cap-to-TVL ratio of the
  live-token protocols in the same snapshot (here 0.2227 over the 128 with a positive market cap),
  clamped to [0.10, 0.50]. The multiple is the market's own ratio of what a token is worth to the
  TVL that supports it.
- **Growth**: `1 + clamp(change_7d, -80, 120) / 100 × 0.6` — a fall hurts more than a rise helps.
- **Maturity**: peaks in years 1–3 of a protocol's life (`0.55 + 0.45 × min(age, 1095)/1095`),
  then eases 15% for the oldest protocols, which have had the longest to launch and have not.
- **Breadth**: `+5%` per chain, up to 5 chains.
- **Implied value** = `round(tvl_usd × multiple × growth × maturity × breadth)`.

The board covers all 198 rankable pre-token protocols: the 40 watchlist slugs plus 158 early ones
the watchlist's own exclusion rule (wrapped/staked/bridged products, exchanges, versioned pools)
leaves out — including a $2.76B curator the watchlist missed. Live-token protocols are set aside
with their market caps shown, never mixed into the ranking. A protocol records no token; the
record is not proof it has none.

## The dashboard

`dashboard/` is a static page with no network: a sortable, filterable board (search, category,
chain, watchlist-only), a top-three spotlight, a live-token view, a detail drawer for every
protocol with the model's inputs recomputed in the page and a seven-step farm plan with costs and
times, and a panel that states the model in full.

## Files

- `main.ts` — the entrypoint: snapshot in, board + dashboard + dossier out
- `lib/model.ts` — the radar-2 model, the farm plans, the board checks
- `vendor/makefun-board.ts` — make.fun's dossier library, byte for byte the kit's copy
- `dashboard/` — `index.html`, `app.css`, `app.js`
- `screenshots/` — the board, a protocol's detail view, a filtered view

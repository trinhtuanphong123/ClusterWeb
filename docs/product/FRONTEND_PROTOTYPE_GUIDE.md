# docs/product/FRONTEND_PROTOTYPE_GUIDE.md

# Strata — Frontend Prototype Guide (Next.js MVP, for Zed AI)

> Implementation guide to turn the Strata prototype into an MVP Next.js frontend. **Spec, not production code** — no component implementations here. Pair with `DESIGN_SYSTEM.md` (tokens/components), `UI_SPEC.md` (pages/states), and `CHART_SPEC.md` (charts).
>
> **Non-negotiable framing:** the product clusters Vietnamese stocks by current movement behavior; historical retrieval explains/validates; outcome stats are descriptive historical context. No buy/sell/hold language, no prediction, no recommendation. The frontend renders precomputed snapshots — it never runs models.

---

## 1. Stack & principles

- **Next.js (App Router)** + TypeScript. React Server Components for snapshot reads; Client Components only where interaction needs it (tabs, filters, search, segmented controls).
- **Styling:** the design-system CSS token files + component primitives. Use CSS variables (`var(--*)`); do not hardcode colors/spacing. (The prototype uses inline styles bound to tokens — any equivalent approach is fine as long as tokens are the single source.)
- **Read-only data:** all 15 endpoints are GET over precomputed outputs. No mutations, no auth flows in MVP (internal/analyst tool). CORS restricted to the frontend origin.
- **No model execution at request time, ever.** Copy and behavior must reflect "reads precomputed snapshots".

---

## 2. Routes → pages → endpoints

| Route | Page | Primary endpoint(s) |
|---|---|---|
| `/` | Overview | `GET /api/overview`, `GET /api/market/status` |
| `/clusters` | Clusters | `GET /api/clusters/latest?include=profiles` |
| `/clusters/[clusterId]` | Cluster Detail | `GET /api/clusters/{id}`, `/{id}/outcomes`, `/{id}/historical-patterns`, `GET /api/tickers/{rep}/neighbors` |
| `/tickers` | Tickers | `GET /api/tickers?q=&sector=&exchange=&limit=&offset=` |
| `/tickers/[ticker]` | Ticker Detail | `GET /api/tickers/{t}`, `/{t}/history`, `/{t}/current-state`, `/{t}/historical-patterns`, `/{t}/neighbors` |
| `/watchlist` | Watchlist | MVP: derive from `/api/clusters/latest` + `/historical-patterns` (dedicated endpoint later) |
| `/pipeline` | Pipeline Status | `GET /api/pipeline/status` |
| `/methodology` | Methodology | `GET /api/methodology/summary` |
| — (layout) | App shell freshness | `GET /api/market/status` |
| infra | health check | `GET /health` (monitoring only; not a page) |

**Run selection:** cluster endpoints default to the latest **valid** run; support an optional `run_id` query param passthrough for deep links (not required in MVP UI).

---

## 3. Data fetching & caching

- Fetch in Server Components per page; pass typed data to client subcomponents.
- **Caching:** snapshot-backed responses are cacheable with **short TTLs** and revalidate against `latest_snapshot`. Suggested `revalidate`: market status / current-state ~30–60s; overview / clusters / outcomes / patterns ~5 min (tied to run validity); methodology long (rarely changes). `/health` never cached.
- Treat the API `is_stale`, `as_of_date`, and `latest_snapshot` fields as the **source of truth** for freshness display — don't recompute staleness on the client.
- Standard horizons are fixed system-wide: **5d / 20d / 60d**. Do not offer other horizons.

---

## 4. Mock data shape (MVP scaffolding)

Until the API is wired, drive the UI from a typed mock module mirroring the contract. Indicative shapes (descriptive context only — no predicted fields):

```ts
type Profile = { volatility: string; shape?: string; drawdown?: string; volume?: string; mkt_rel?: string };

type Cluster = {
  cluster_id: string;            // "C1"… → color via (n-1)%12+1
  size: number;
  profile: Profile;
  representatives: string[];
  members?: string[];
  spark?: number[];              // normalized shape preview
};

type Member = { ticker: string; company_name: string; sector: string;
  membership_score: number; is_representative: boolean; mkt_rel_20d: number; spark: number[] };

type OutcomeRow = { horizon: "5d"|"20d"|"60d"; n_samples: number;
  fwd_return: { mean:number; median:number; p25:number; p75:number };
  max_drawdown: { mean:number; median:number };
  excess_return_vs_index: { mean:number; median:number }; };       // low_confidence = n_samples < 30

type PatternMatch = { rank:number; match_ticker:string;
  match_window_start:string; match_window_end:string;
  method:"dtw"|"mpdist"|"corr"|"matrix_profile"; window_length:number; similarity:number;
  future_return_5d:number; future_return_20d:number; future_return_60d:number;
  max_drawdown_20d:number; excess_return_20d:number; };

type Ticker = { ticker:string; company_name:string; sector:string; exchange:"HOSE"|"HNX";
  is_active:boolean; current_cluster:{ run_id:number; cluster_id:string; membership_score:number };
  behavior_summary: Profile };

type CurrentState = { ticker:string; as_of:string; last_price:number; intraday_return:number;
  intraday_volatility:number; source_interval:string; is_trading_session:boolean; is_stale:boolean };

type Bar = { date:string; open:number; high:number; low:number; close:number; volume:number };

type Neighbor = { ticker:string; weight:number; same_cluster:boolean; lead?:"leads"|"lags"; lag_days?:number };

type MarketStatus = { as_of_date:string; is_trading_day:boolean;
  day_type:"trading"|"weekend"|"holiday"|"manual_closure";
  last_data_update:string; last_feature_update:string; last_model_update:string;
  latest_cluster_run:number; latest_snapshot:{ id:number; as_of_date:string }; is_stale:boolean };

type Job = { job_name:string; status:"succeeded"|"running"|"skipped"|"failed";
  trigger:"scheduled"|"manual"; started_at:string; ended_at:string|null; error_message:string|null };
```

Keep mock numbers deterministic (seeded) so renders are stable. Mock tickers should use real VN names (HPG, SSI, VIC, VHM, FPT, VCB, …) and exchanges HOSE/HNX. Every outcome/pattern mock must carry the `disclaimer` string.

---

## 5. Cross-cutting UI behavior

- **Freshness everywhere:** sidebar `FreshnessPill` (always), top-bar `run {id}` badge, Overview stale banner, `as-of` on outcome/retrieval/pipeline, "current state stale" on ticker detail. Driven by `is_stale` / `as_of_date`.
- **Disclaimer everywhere it's required:** any outcome or historical-pattern surface renders `DisclaimerNote`; prefer the API `disclaimer` text verbatim.
- **Cluster color identity** is stable across all views — resolve once, reuse.
- **States:** see `UI_SPEC.md` "States" for loading (shell-first + skeletons), empty (`EmptyState`), error (`400/404/503/500` mapping), stale, and non-trading-day behavior. The shell and freshness footer must render even when content fails.
- **Numbers:** mono tabular; signed `+`/`−` (true minus); always show `n = …`; flag `n < 30` as low-confidence.

---

## 6. MVP scope

**In scope (build now):**
- All 8 pages in `UI_SPEC.md`, the app shell, and the design-system components.
- The six chart types in `CHART_SPEC.md` as inline SVG (no charting library).
- Read-through of all 15 endpoints (Watchlist derived for now), with loading/empty/error/stale handling.
- Freshness + disclaimer behavior, sector/search filters, in-page tabs and horizon switch.

**Out of scope (post-MVP):**
- Auth / user accounts / personalization; saved watchlists (MVP watchlist is derived, read-only).
- OHLC candles, zoom/brush, crosshair tooltips (MVP price chart = close line + area).
- Force-directed lead-lag graph (MVP = neighbor list).
- `run_id` history browsing UI, multi-run comparison.
- Real-time streaming (data is snapshot/hourly; short-TTL revalidation is sufficient).
- PDF/report export, theming, i18n.

**Definition of done (MVP):** every page renders from the contract shapes with correct freshness/stale/non-trading-day treatment, the required disclaimers appear on all outcome/pattern surfaces, no prohibited vocabulary anywhere, and all visuals use only Strata tokens and components.

---

## 7. Copy guardrails (lint these)

- **Never render:** buy, sell, hold, signal, target price, forecast, predict/prediction, guaranteed, profit, recommendation, outperform (as advice), bullish/bearish.
- **Always frame outcomes** as historical/descriptive ("historically tended to", "in the analog sample", "descriptive context").
- **Methodology page** must keep the "This dashboard is not" list (price forecaster · recommendation engine · financial advice · real-time trading terminal).
- Consider a simple CI text check against the prohibited list across the rendered copy and mock data.

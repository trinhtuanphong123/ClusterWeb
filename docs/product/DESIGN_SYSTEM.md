# docs/product/DESIGN_SYSTEM.md

# Strata — Design System (frontend reference)

> **What this is:** the visual + interaction contract for the Strata dashboard, distilled from the Cluster Analytics Design System and the finished prototype. Build the Next.js frontend against the tokens and components below — do not invent new colors, type, spacing, or components.
>
> **What the product is:** a serious analytics dashboard that **clusters Vietnamese-listed stocks by current movement behavior**, and attaches **historical pattern retrieval as an explanation/validation layer**. Outcome statistics are **descriptive historical context only**.
>
> **What it is not:** a trading app, a price-prediction system, or a recommendation engine. It never tells the user what to do.

---

## 1. Product framing (binding on all UI copy)

- **Primary task:** current behavior clustering. Everything else supports it.
- **Explanation layer:** historical pattern retrieval explains and validates clusters and tickers. It is never presented as a forecast.
- **Outcome statistics:** always framed as descriptive historical context, always with sample size, always with the disclaimer.
- **Prohibited vocabulary (never render, anywhere):** buy, sell, hold, signal, target price, forecast, predict/prediction, guaranteed, profit, recommendation, outperform (as advice), bullish/bearish.
- **Preferred vocabulary:** behavior cluster, current behavior, movement behavior, historical context, similar/analog pattern, outcome distribution, descriptive statistic, analytical watchlist, decision-support, lead-lag relationship, representative member, membership score, freshness/staleness, as-of date.
- **Disclaimer rule:** every surface that shows outcome statistics OR historical pattern matches must show the descriptive-context disclaimer. Use the `disclaimer` field from the API verbatim when present.

---

## 2. Color tokens

Defined as CSS custom properties (already shipped in the design-system token files). Reference via `var(--*)`; never hardcode hex.

### Neutral ink scale (cool slate — carries the whole UI)
`--ink-950 #0B0F14` · `--ink-900 #11161D` · `--ink-800 #1B232D` · `--ink-700 #2A3542` · `--ink-600 #3D4A5A` · `--ink-500 #566374` · `--ink-400 #788596` · `--ink-300 #9BA6B4` · `--ink-200 #C4CCD6` · `--ink-150 #DBE1E9` · `--ink-100 #E9EDF2` · `--ink-75 #F0F3F7` · `--ink-50 #F5F7FA` · `--ink-25 #FAFBFC` · `--white #FFFFFF`

### Brand accent — steel blue (one disciplined accent, used sparingly)
`--blue-50 #ECF2F9` · `--blue-100 #D6E3F1` · `--blue-500 #2F5E8E` (accent) · `--blue-600 #274E76` (hover) · `--blue-700 #1F3D5C` (active)

### Teal — secondary, reserved for structural relations (lead-lag, neighbors)
`--teal-50 #E8F4F3` · `--teal-500 #1E8079` · `--teal-700 #134F4B`

### Directional outcome — muted, descriptive only (never trading-screen green/red)
`--pos-500 #2E7D5B` (forest) · `--neg-500 #B0573F` (terracotta). Always paired with a label; never standalone.

### Status
fresh = `--pos-500` · stale = `--amber-500 #B7791F` · failed = `--red-500 #B23B3B` · running = `--blue-500` · skipped = `--ink-400`

### Cluster categorical palette (12 mid-tone, mutually distinguishable hues)
`--cluster-1 #3A6EA5` · `-2 #C26B3E` · `-3 #4E8A6B` · `-4 #8A6BB0` · `-5 #C0913B` · `-6 #B5577A` · `-7 #4AA0A8` · `-8 #7B8A4E` · `-9 #9A5B4A` · `-10 #5C6BC0` · `-11 #4F7E91` · `-12 #A06FA0`.
**Rule:** a cluster keeps the same color everywhere (chips, dots, charts, tables). Resolve `Cx → --cluster-((x-1) % 12 + 1)`.

### Semantic aliases (prefer these in components)
`--surface-page (--ink-50)` · `--surface-card (--white)` · `--surface-header (--ink-950)` · `--surface-hover (--ink-50)` · `--text-strong (--ink-950)` · `--text-body (--ink-800)` · `--text-muted (--ink-500)` · `--text-subtle (--ink-400)` · `--text-link (--blue-600)` · `--border-subtle (--ink-100)` · `--border-default (--ink-150)` · `--border-strong (--ink-200)` · `--accent (--blue-500)` · `--ring 0 0 0 3px rgba(47,94,142,0.28)`.

---

## 3. Typography

- **Families:** `--font-sans` = IBM Plex Sans (all UI text); `--font-mono` = IBM Plex Mono (every number, ticker, date, ID, run id, code) with tabular figures `font-feature-settings: "tnum" 1, "lnum" 1`; `--font-serif` = IBM Plex Serif (editorial/methodology prose and pull-quotes only).
- **Scale (compact, data-dense):** 2xs 11 · xs 12 · sm 13 · base 14 (default body) · md 15 · lg 16 · xl 18 · 2xl 22 · 3xl 28 · 4xl 36.
- **Roles:** `--type-h2` (semibold 22), `--type-h4` (semibold 16, panel titles), `--type-body` (14/1.5), `--type-body-sm` (13), `--type-label` (medium 13), `--type-caption` (12), `--type-eyebrow` (semibold 11 UPPERCASE, tracking 0.07em), `--type-serif-lead` (serif 18, lead prose).
- **Casing:** sentence case everywhere (headings, buttons, labels, table headers). The single exception is eyebrow/overline labels, which are UPPERCASE with 0.07em tracking.
- **Numbers:** signed percentages with a true minus sign `−` (U+2212), e.g. `+1.8%`, `−3.2%`. Dates ISO `2026-06-20`. Horizons `5d / 20d / 60d`. Tickers UPPERCASE.

---

## 4. Spacing & layout

- 4px base grid. The product is **dense by design** — analysts scan many rows.
- **App frame:** fixed left sidebar `--sidebar-width 248px` (near-black `--ink-950`) + top bar `--topbar-height 56px` (white surface). Content column `--content-max 1440px`, centered, page padding `--pad-page 24px`.
- **Panels** separate by 16px gaps and hairline borders, not heavy cards.
- **Card padding** `--pad-card 20px` (tables/charts manage their own insets via `padded=false`).
- **Radii:** 4–6px on controls/cards, 3px on chips/cells, pill only for badges/toggles. Nothing heavily rounded.

---

## 5. Borders, elevation, texture

- Borders do the structural work — hairlines (`--ink-100/150`) divide everything.
- Cards: white, 1px border, at most `--shadow-sm`. Reserve `--shadow-lg/xl` for true overlays (menus, dialogs, tooltips).
- **No gradients, no photographic imagery, no illustration, no decorative noise.** Only functional "texture": faint dotted gridlines on charts, hairline grid in matrix views. Empty space is intentional.

---

## 6. Iconography

- **Set:** Lucide (1.5px uniform stroke, rounded joins, 24×24 grid), rendered inline as SVG, size 16–18px in UI / 14px inline with text, stroke inherits `currentColor`. Functional only — never decorative or display-size.
- **Common glyphs:** `layout-grid` (overview), `boxes` (clusters), `list` (tickers), `eye` (watchlist), `activity` (pipeline), `book-open` (methodology), `arrow-right-left` / `git-branch` (lead-lag), `history` (historical patterns), `bar-chart-3` (outcomes/behavior), `clock` / `refresh-cw` (freshness), `alert-triangle` (stale / low-confidence), `search`, `chevron-left/right`, `info`.
- No emoji. No multicolor/3D icons. No hand-drawn SVG.

---

## 7. Motion

- Minimal, quick: transitions 140–200ms on a standard ease. Fades and 2–4px translate-ins only. **No bounce, no spring, no looping decorative animation.** Charts may draw in once on mount. Respect `prefers-reduced-motion`.

---

## 8. Interaction states

- **Hover:** surfaces shift to `--surface-hover`; accent buttons darken to `--accent-hover`; table rows highlight `--ink-50`.
- **Press/active:** one step darker (`--accent-active`); no scale-down except small icon buttons (subtle 0.96).
- **Focus:** always visible — 3px soft steel-blue ring `--ring`; never removed.
- **Selected:** accent-soft fill `--blue-50` + a 2px left or bottom accent rule (sidebar nav, tabs).

---

## 9. Component inventory

Reusable primitives (already in the design system; map 1:1 to React components in the frontend). Props listed are the contract surface the prototype uses.

| Component | Purpose | Key props |
|---|---|---|
| `Button` | Primary control | `variant` (primary/secondary/ghost/danger), `size` (sm/md), `leadingIcon`, `trailingIcon`, `disabled` |
| `IconButton` | Square icon-only action (refresh) | `label` (a11y), `children` (icon) |
| `Badge` | Status / category label | `tone` (neutral/accent/positive/negative/warning/teal), `dot`, `mono` |
| `ClusterDot` | Cluster color identity | `cluster` (id or number), `size`, `label` |
| `OutcomeValue` | Signed return rendered as muted descriptive context | `value` (ratio), `digits`, `size`, `format` (percent/number) |
| `Sparkline` | Tiny normalized shape preview (not a price chart) | `data` (number[]), `stroke`, `fill`, `width`, `height` |
| `StatTile` | Single labelled metric | `label`, `value`, `sub`, `accent`, `align` |
| `DisclaimerNote` | Required descriptive-context framing | `variant` (block/inline), `children` (defaults to standard text) |
| `EmptyState` | No-data state | title/description/icon |
| `FreshnessPill` | First-class staleness indicator | `stale`, `asOf`, `label` |
| `Input` | Text/search field | `value`, `onChange`, `placeholder`, `leadingIcon`, `size` |
| `SegmentedControl` | Compact exclusive switch (horizon, view mode) | `options`, `value`, `onChange`, `size` |
| `Select` | Dropdown (sector/exchange filter) | `value`, `onChange`, `options`, `size` |
| `Card` | Base surface | `padded`, `shadow` |
| `PanelHeader` | Section title row | `eyebrow`, `title`, `description`, `actions`, `divider` |
| `SidebarNav` | Left navigation | `sections`, `value`, `onSelect` |
| `Tabs` | Underline in-page section switch | `tabs` (value/label/count), `value`, `onChange` |

Two app-level composites the prototype adds on top (not generic primitives, but reused): **ProfileChip** (label + value descriptive chip) and **status cards** (icon + eyebrow + mono value + sub + freshness pill) used on Overview.

---

## 10. Freshness as a first-class visual (binding)

Because data is snapshot-based and can be stale (weekends, holidays, hourly cadence), staleness is shown **explicitly everywhere**, never hidden:

- An amber `FreshnessPill` with the `as-of` date in the sidebar footer (always visible).
- A `run {id}` badge in the top bar.
- An amber stale-data banner on Overview when `is_stale`.
- `as-of` timestamps on outcome/retrieval/pipeline surfaces.
- Ticker current-state is labelled stale outside trading sessions.

See `UI_SPEC.md` §"States" for exact loading / empty / error / stale / non-trading-day behavior.

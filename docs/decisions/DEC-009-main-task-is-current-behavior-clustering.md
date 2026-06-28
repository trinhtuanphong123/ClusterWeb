# DEC-009: The Main Task Is Current Behavior Clustering

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The project could be framed in several ways: price prediction, buy/sell signal generation, or behavior analytics. The chosen identity must be unambiguous so future work (and future coding agents) build the right thing. The product is explicitly **not** a prediction or recommendation system.

## Decision

The **primary task is current behavior clustering**: at time `T`, over stock universe `S`, compute behavior representations from rolling windows and cluster stocks by behavior similarity. All other components exist to support this output. The system is **decision-support analytics, not a trading system**.

## Rationale

- **Clear product identity** — "which stocks are currently behaving alike" is the core question the dashboard answers.
- **Avoids forecasting traps** — prices are stochastic, noisy, and non-stationary; the project deliberately does not predict prices or emit signals.
- **Behavior over price level** — clustering on behavior features (returns, normalized paths, volatility, drawdown, volume, market-/sector-relative) yields comparable, scale-free, interpretable groups.
- **Anchors scope** — every pipeline stage (features → windows → clustering) and supporting layer is oriented around producing and explaining clusters.

## Consequences

- Outputs are descriptive, not predictive or prescriptive; the UI must frame them as such.
- Engineering priorities center on cluster quality, stability, and interpretability.
- Supporting layers (historical retrieval, lead-lag, outcome, watchlist) are defined relative to clustering, not as standalone products.

## Alternatives considered

- **Price prediction model** — out of scope by design; misrepresents what is achievable and invites misuse; rejected.
- **Buy/sell recommendation system** — explicitly a non-goal; carries advisory liability and contradicts the analytics framing; rejected.
- **Per-stock charting tool only** — fails to deliver the cross-sectional "who moves alike" insight that is the project's purpose; rejected.


## Related decisions

- DEC-006 — intraday 5-minute is current-state only, not a clustering input.
- DEC-008 — trading-calendar gating for official clustering runs.
- DEC-010 — historical retrieval as the explanation layer for clusters.
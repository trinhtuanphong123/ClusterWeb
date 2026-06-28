# DEC-010: Historical Pattern Retrieval Is the Explanation Layer

- **Status:** Accepted
- **Date:** 2026-06-22
- **Deciders:** Project team

## Context

The system retrieves historically similar windows for stocks/clusters and summarizes what followed them. This capability could be misread as a predictive engine ("similar past → predicted future"). Given the project's identity (DEC-009) as behavior-clustering analytics and **not** prediction, the role of historical retrieval must be fixed precisely.

## Decision

Historical pattern retrieval (and the outcome analysis built on it) is the **explanation layer** for current clusters. It exists to **explain and validate** "who is behaving alike now" by showing when comparable behavior occurred historically. Its outputs are **descriptive context, never forecasts or recommendations**.

## Rationale

- **Serves the main task** — retrieval makes clusters interpretable and credible, supporting DEC-009 rather than competing with it.
- **Honest framing** — because prices are non-stationary, historical analogs cannot be treated as reliable predictors; presenting them as context avoids implying prediction.
- **User value** — "this current behavior resembles past windows X, Y, Z, where Z typically saw …" aids understanding without claiming to know the future.
- **Bounded scope** — keeps the retrieval/outcome layer from drifting into signal generation.

## Consequences

- Outcome analysis is presented as **distributions of what historically followed**, explicitly labeled as context, not predictions.
- The UI must avoid language implying forecasts or trade signals from analogs.
- Retrieval runs **after** clustering in the pipeline (it explains the clusters), and outcome analysis runs after retrieval.

## Alternatives considered

- **Treat retrieval as a predictor** — contradicts the project's non-prediction stance and the non-stationarity of prices; rejected.
- **Drop historical retrieval entirely** — clusters would be less interpretable and harder to validate; rejected.
- **Make retrieval a standalone "find similar charts" product** — fragments the product identity and detaches it from the clustering purpose; rejected.

## Related decisions

- DEC-009 — the current behavior clustering that retrieval explains and validates.
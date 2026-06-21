# Architecture — Vietnamese Stock Behavior Clustering Dashboard

> Decision-support analytics, **not** a trading system. Current behavior clustering is the primary output; historical pattern retrieval is the explanation layer.

## 1. Components

| Component | Technology | Role |
|---|---|---|
| Frontend | **Next.js on Vercel** | Dashboard UI; reads precomputed snapshots via the API |
| Backend API | **FastAPI on AWS EC2** | Serves precomputed snapshots; no heavy compute at request time |
| Worker | **Python scheduler on AWS EC2** | Ingestion, features, windows, clustering, historical pattern retrieval, snapshot generation |
| Database | **Supabase Postgres** | Normalized tables, feature tables, windows, cluster/pattern outputs, snapshots, trading calendar |
| Object storage (optional) | **AWS S3** | Raw parquet, model artifacts, large distance matrices, saved vector artifacts |

## 2. Deliberate constraints (MVP)

**Avoided:** Lambda, ECS, EKS, Kafka, Airflow, Redshift, and other heavy enterprise infrastructure. The system is intentionally a small number of long-lived services on EC2 plus managed Postgres and a static-ish frontend. This keeps the MVP cheap, debuggable, and operable by a small team.

## 3. Deployment modes

The architecture supports two modes that share the same code and database.

### 3.1 One-EC2 staging mode

- A **single EC2** runs **both** the API and the worker as **separate systemd services**.
- FastAPI (optionally behind Nginx) and the Python scheduler coexist on one host.
- Used for staging / cost-controlled environments.

```
┌──────────────── EC2 (single host) ────────────────┐
│  systemd: fastapi-api.service   (FastAPI)          │
│  systemd: worker.service        (scheduler)        │
└───────────────────────────────────────────────────┘
        │                         │
        ▼                         ▼
   Supabase Postgres   ◄──────────┘
        ▲
        │ (optional) S3 for parquet / artifacts
```

### 3.2 Two-EC2 production-like mode

- **EC2-API:** runs **FastAPI + Nginx** (reverse proxy, TLS termination).
- **EC2-WORKER:** runs the **scheduler, ingestion, features, clustering, historical pattern retrieval, and snapshot generation**.
- Both connect to the same Supabase Postgres (and optional S3).

```
        ┌──────────── EC2-API ────────────┐      ┌────────── EC2-WORKER ──────────┐
        │  Nginx → FastAPI                 │      │  scheduler                     │
        │  (serves snapshots)              │      │  ingestion / features /        │
        │                                  │      │  windows / clustering /        │
        │                                  │      │  pattern retrieval / snapshots │
        └───────────────┬──────────────────┘      └───────────────┬────────────────┘
                        │                                          │
                        └───────────────┬──────────────────────────┘
                                        ▼
                              Supabase Postgres
                                        ▲
                                        │ (optional) S3 artifacts
```

## 4. Data plane vs serving plane

- **Serving plane (always on):** FastAPI reads the **latest valid snapshots** from Postgres and returns them. It performs no clustering or retrieval at request time, so it stays responsive and is unaffected by worker load.
- **Data plane (scheduled):** the worker produces snapshots on a schedule gated by the trading calendar (see `SCHEDULER_STRATEGY.md`). It writes outputs to Postgres (and large artifacts to S3).

The two planes are decoupled through the database: the worker is the sole writer of analytical outputs; the API is a reader.

## 5. Storage layout (logical)

- **Postgres (Supabase):** trading calendar; normalized daily + 5-minute OHLCV; feature tables; rolling behavior windows; clustering outputs; historical pattern outputs; lead-lag / outcome / watchlist; dashboard snapshots.
- **S3 (optional):** raw parquet dumps, serialized model artifacts, large distance/similarity matrices, and saved vector artifacts that are too large or too binary to keep in Postgres.

## 6. Two processing tracks

- **Daily official clustering flow** — authoritative, from daily OHLCV; produces clusters, historical patterns, and supporting outputs.
- **Hourly intraday flow** — from 5-minute OHLCV; updates the current-state snapshot only (hourly, not per-5-minutes). Never produces official clusters.

See `DATA_FLOW.md` for the full pipeline.

## 7. Why this shape

- **EC2 long-lived services** suit always-on scheduling and stateful Python ML libraries better than Lambda's short execution model.
- **Supabase Postgres** gives managed Postgres with minimal ops overhead.
- **Vercel** is a natural fit for Next.js and keeps the frontend deploy simple.
- **Optional S3** absorbs large binary artifacts without bloating Postgres.
- The **one-EC2 / two-EC2** split lets the same system run cheaply in staging and scale to a cleaner separation in production without code changes.

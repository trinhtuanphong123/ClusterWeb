# Feature Intake

Every implementation prompt enters the intake gate before code changes. A new
project spec also enters through this gate before it becomes product docs,
stories, or implementation work.

The human does not need to classify risk. The harness does.

## Intake Flow

```text
User prompt
    |
    v
Classify input type
    |
    v
Restate as work item
    |
    v
Find affected product docs and stories
    |
    v
Run risk checklist
    |
    v
Choose lane: tiny, normal, or high-risk
```


## Context Budget Rule

This project uses a lean harness mode to control coding-agent token usage.

Agents must not read the whole repository or the whole docs/ directory by default.

For each implementation request, the default context is:

AGENTS.md
docs/CONTEXT_INDEX.md
the assigned story packet in docs/stories/
the product docs explicitly listed in that story
the implementation files directly affected by the task

Agents should not read these files during normal implementation unless the story explicitly asks:

docs/HARNESS.md
all files in docs/templates/
all files in docs/decisions/
all product docs under docs/product/
all validation files under docs/test-matrix/

If more context appears necessary, the agent should ask which file to inspect next instead of scanning broadly.


## Input Types

Use the input type to decide where the work should land before choosing the risk
lane.

| Type | Use when | Typical artifact |
| --- | --- | --- |
| New spec | Turning a user-provided project spec into harness-ready docs | Product docs, candidate epics, decisions |
| Spec slice | Implementing selected behavior from an accepted spec | Story packet |
| Change request | Changing, fixing, or refining accepted behavior | Story packet or direct patch |
| New initiative | Adding a larger product area that needs multiple stories | Initiative notes plus story packets |
| Maintenance request | Changing technical, operational, or dependency behavior | Story packet, validation report, or decision |
| Harness improvement | Improving how humans and agents collaborate | Direct docs update or `scripts/bin/harness-cli backlog add` |

Do not create or extend a monolithic spec by default after intake. Use product
docs, stories, decisions, and initiative notes as the living surface.



## Project-Specific Work Types

Use these mappings for this stock clustering dashboard.

Work type	     Typical lane          	Product docs to consider	         Notes
Repository scaffold	    normal	    PROJECT_BRIEF.md, ARCHITECTURE.md	       Create folders and placeholders only. No business logic.
Supabase schema	        high-risk	     DATABASE_SCHEMA.md, DATA_FLOW.md	   Schema, migrations, uniqueness, indexes.
FastAPI backend endpoint	normal	   API_CONTRACT.md	                   High-risk if API contract changes.
Python worker skeleton	  normal	    DATA_FLOW.md, DEPLOYMENT.md	        High-risk if production scheduler behavior changes.
VNStock ingestion	      high-risk	      DATA_FLOW.md, DATABASE_SCHEMA.md	 External provider behavior, API key, rate limit, duplicate data.
Feature engineering	     normal	      METHODOLOGY.md, DATABASE_SCHEMA.md	High-risk if formulas or stored output contract change.
Clustering/modeling	    normal or high-risk 	METHODOLOGY.md, DATABASE_SCHEMA.md	High-risk if methodology or result schema changes.
Frontend dashboard page	 normal	      UI_SPEC.md, DESIGN_SYSTEM.md, API_CONTRACT.md	Tiny if mock-only placeholder.
AWS deployment	      high-risk	       DEPLOYMENT.md, ARCHITECTURE.md	   EC2, systemd, Nginx, env vars, deploy scripts.
Vercel deployment config	 normal or high-risk	 DEPLOYMENT.md, UI_SPEC.md	High-risk if secrets/env exposure changes.
Harness maintenance  	normal	          HARNESS.md, FEATURE_INTAKE.md, TEST_MATRIX.md    Only when changing agent workflow.




## Lanes

### Tiny

Use for low-risk docs, copy, names, or narrow edits.

Also use for initial project setup when the work is limited to installing
declared dependencies, wiring a server entrypoint, adding a health/smoke
endpoint, or opening a local development database connection without creating
domain schema, CRUD behavior, auth, authorization, provider integration, or
data migration. A health endpoint in a new benchmark or scaffolded project is
smoke proof, not a public contract escalation by itself.

Requirements:

- Record the intake row before implementation; tiny work skips story packet
  overhead, not durable task classification.
- Patch directly.
- Keep affected docs current.
- Run available quick checks.
- Update the harness only if friction was found.
### Normal
For this project, normal work should usually map to exactly one story packet and one subsystem: backend, frontend, pipeline, clustering, database, or deployment.

Use for story-sized behavior with bounded blast radius.

Requirements:

- Create or update one story file from `docs/templates/story.md`.
- Link relevant product docs.
- Add or update validation expectations.
- Implement the smallest vertical slice when implementation exists.
- Record or update proof status with `scripts/bin/harness-cli story add` and
  `scripts/bin/harness-cli story update`.

### High-Risk
For this project, database migration, deployment script, scheduler, external provider integration, secret handling, and API contract changes are high-risk by default.

Use when the work can affect security, data, scope, contracts, or multiple
roles/platforms.

Requirements:

- Create a story folder using `docs/templates/high-risk-story/`.
- Fill in `execplan.md`, `overview.md`, `design.md`, and `validation.md`.
- Ask for human confirmation before implementation if direction is ambiguous.
- Record a durable decision when behavior, architecture, authorization, data
  ownership, API shape, or validation requirements change meaningfully. Use a
  `docs/decisions/NNNN-*.md` file from `docs/templates/decision.md`, then add
  or refresh the durable row with `scripts/bin/harness-cli decision add`.
  Decision text in a trace is not a durable decision record.

## Risk Checklist

Mark one flag for each item that applies:

| Risk flag | Applies when the work touches |
| --- | --- |
| Auth | login, logout, sessions, JWT, password, refresh token |
| Authorization | roles, permissions, tenant or company scope |
| Data model | schema, migrations, uniqueness, deletion, retention |
| Audit/security | audit logs, privacy, sensitive data, access logs |
| External systems | email, payments, cloud services, provider SDKs, queues, webhooks |
| Public contracts | API shape, response envelope, client-visible behavior |
| Cross-platform | desktop/mobile/browser split, native shell behavior, deep links |
| Existing behavior | already implemented or test-covered behavior changes |
| Weak proof | unclear or missing tests around the affected area |
| Multi-domain | more than one product domain changes at once |

## Classification

```text
0-1 flags:
  tiny or normal, based on code impact

2-3 flags:
  normal with stronger validation

4+ flags:
  high-risk

Any hard gate:
  high-risk unless the human explicitly narrows scope
```

Hard gates:

- Auth.
- Authorization.
- Data loss or migration.
- Audit/security.
- External provider behavior.
- Removing or weakening validation requirements.


## Project-Specific Risk Overrides

The general risk checklist still applies. These rules override ambiguous cases in this project.

## Always high-risk

Classify as high-risk when the work touches:

- Supabase schema migrations for existing tables
- data deletion, retention, or backfill behavior
- production scheduler frequency
- VNStock API authentication, API key handling, or rate-limit behavior
- AWS EC2 deployment scripts
- systemd service files
- Nginx reverse proxy configuration
- environment variable handling involving secrets
- public API response shape used by the frontend
- cluster output table contracts
- switching deployment target, for example EC2 to Lambda/ECS
- replacing the core modeling method
- changing how dashboard reads production data

## Usually normal

Classify as normal when the work is bounded to one subsystem and follows existing contracts:

- add one FastAPI endpoint already defined in API_CONTRACT.md
- implement one dashboard page using existing API contract
- implement one feature computation module using existing schema
- add baseline clustering using existing output tables
- add mock worker job without production scheduling
- add local-only development script


## Usually tiny

Classify as tiny when the work is local and does not affect contracts:

- README wording
- placeholder UI component
- mock data object
- small utility function
- typo fix
- comment/doc clarification
- local smoke endpoint in a scaffolded backend

## Required decision record

Create or update a decision record in docs/decisions/ when the work changes:

- backend framework
- deployment target
- database ownership or storage boundary
- API contract
- scheduler strategy
- data retention strategy
- model/clustering methodology
- security or secret-handling policy
- agent workflow/harness policy



## Output

At the end of intake, the agent should be able to say:

```text
Lane: normal
Reason: touches authorization, API contract, and audit behavior.
Docs: permissions, account-settings, audit-log.
Story: docs/stories/epics/E02-access-control/US-014-manager-updates-role.md.
Validation: unit, integration, E2E.
```

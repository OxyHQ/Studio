# Roadmap

This roadmap deliberately starts narrow. Studio should earn the right to become Oxy's internal control plane by first solving the database workflow exceptionally well.

## Phase 0 — Foundation and decisions

**Goal:** establish the legal, product and technical boundary before importing upstream code.

Deliverables:

- project charter and product boundary
- naming decision
- Supabase Studio / pg-meta provenance audit
- third-party attribution strategy
- standalone monorepo/workspace bootstrap
- frontend + gateway skeleton
- Oxy Auth sign-in
- internal membership/role model
- environment model (`local`, `development`, `staging`, `production`)
- audit event schema
- adapter contracts
- CI, lint, tests and preview deployments

Exit criteria:

- an authorized internal user can sign in to an empty Studio shell
- an ordinary Oxy account cannot enter Studio
- no provider/database secret reaches the browser
- every privileged test action produces an audit event

## Phase 1 — PostgreSQL read-only MVP

**Goal:** make Studio immediately useful without taking production mutation risk.

Deliverables:

- connection registry using server-side secret references
- pg-meta integration behind the gateway
- Oxy RDS discovery/configuration
- database/schema/table navigation
- table rows with pagination, filtering and sorting
- columns, PK/FK relationships and constraints
- indexes
- enums/types
- functions/triggers/extensions
- schema diagram
- read-only SQL editor (`SELECT`, explainable safe statements)
- query history
- `EXPLAIN` / query plan viewer
- basic database/table size information
- sessions and locks read view
- deep links to database resources

Production policy:

- read-only by default
- SQL classifier blocks writes in the read-only execution path
- server-side statement timeout
- result row/size limits
- query cancellation

Exit criteria:

- Oxy developers no longer need a separate PostgreSQL GUI for normal inspection work
- Oxy's PostgreSQL 17 / PostGIS setup is fully inspectable
- database credentials remain server-side

## Phase 2 — Safe database mutations

**Goal:** support daily development/admin work while making production mistakes difficult.

Deliverables:

- row insert/update/delete
- general SQL execution for authorized roles
- DDL operations
- indexes, functions, triggers, roles and policies management
- transaction wrapper where appropriate
- mutation preview
- affected-row estimates where possible
- production write mode with explicit elevation
- step-up authentication for dangerous actions
- typed confirmation for destructive operations
- optional second-approver policy for catastrophic operations
- mutation audit diff/details
- saved/shared SQL snippets
- CSV import/export with limits and validation

Danger classes:

1. read
2. reversible write
3. schema mutation
4. destructive data operation
5. infrastructure/catastrophic operation

Exit criteria:

- routine non-production DB work can happen entirely in Studio
- every write is attributable to a human/service/agent and resource
- destructive production operations cannot happen accidentally through normal browsing

## Phase 3 — Drizzle + code-aware database workflows

**Goal:** connect the live database with the code that defines it.

Deliverables:

- GitHub repository linkage per Oxy service
- Drizzle schema/migration discovery
- migration history linked to commits
- declared-vs-live schema comparison
- drift detection
- missing/unexpected object warnings
- migration draft generation
- schema change preview
- link DB changes to PR/deployment
- migration safety checks

Exit criteria:

- a developer can answer “what does code think the schema is, what is actually in prod, and which deploy changed it?” from one place

## Phase 4 — Oxy service graph

**Goal:** move from “database client” to “Oxy engineering workspace”.

Deliverables:

- canonical service/project model
- service inventory: oxy-api, Mention, Alia, Homiio, Syra, Allo and future services
- per-service overview
- linked repo, database, deployment, URL and infrastructure
- health checks
- environment/version/commit display
- dependency relationships
- runbook links
- command palette across services/resources

Exit criteria:

- Studio becomes the fastest place to answer “what is this service, what is running, where is its DB/code/logs, and is it healthy?”

## Phase 5 — Deployments, GitHub and observability

**Goal:** connect operational symptoms to the code/deploy that caused them.

Deliverables:

- GitHub PR / Actions integration
- deployment timeline
- commit -> build -> deploy -> running task mapping
- ECS service/task state
- CloudWatch log discovery and scoped viewer
- RDS/ECS key metrics
- recent failures/errors
- deployment markers on metrics/incidents
- safe restart/redeploy actions
- incident timeline draft

Exit criteria:

- common deployment/debug workflows no longer require manually correlating GitHub, AWS and the DB

## Phase 6 — Infrastructure operations

**Goal:** expose high-value infrastructure actions while respecting Terraform ownership.

Deliverables:

- RDS snapshot inventory and controlled snapshot creation
- ECS scaling/restart operations
- Valkey health
- S3/storage metadata where useful
- SSM parameter metadata
- Cloudflare/DNS status
- drift warning when an imperative Studio action conflicts with IaC ownership
- explicit handoff to `oxy-infra` for durable changes

Rule:

Studio can perform operational actions; **Terraform remains authoritative for durable infrastructure configuration**.

## Phase 7 — AI / Alia engineering copilot

**Goal:** make Oxy operational context machine-usable without giving agents unrestricted production power.

Deliverables:

- natural-language SQL drafting
- explain query / query-plan analysis
- index suggestions
- migration drafts
- schema exploration
- log/deploy correlation
- incident investigation
- safe tool invocation via capability scopes
- human approval UX for write actions
- agent audit identity
- MCP endpoint/tool definitions for approved internal agents

Exit criteria:

- an agent can independently investigate most engineering questions and prepare fixes, while production mutations remain policy-controlled and attributable

## Phase 8 — Internal platform expansion

Possible modules after the core proves useful:

- storage/object browser
- background jobs and queues
- scheduled jobs/cron
- email delivery/SES diagnostics
- feature/config management
- domains/certificates
- internal user/support tools with strict privacy boundaries
- dependency/security status
- incident management
- cost/capacity views
- environment creation
- ephemeral database/preview environments

Each new module must justify why Studio is a better home than the provider's native UI.

## Phase 9 — Local/desktop experience (optional)

If the team needs a true local PostgreSQL client:

- local Studio gateway
- secure OS keychain storage
- localhost/LAN PostgreSQL connections
- optional Tauri desktop shell
- same database UI and policies, with local-vs-Oxy context clearly separated

This is not required for the internal web control plane MVP.

---

## Prioritization rule

For every proposed feature, ask:

1. Does it remove a recurring Oxy engineering pain?
2. Does cross-linking it with database/service/deploy context create more value than a provider UI already provides?
3. Can we implement it without weakening security boundaries?
4. Does it belong in internal Studio rather than public Oxy Console?

If the answer is weak, link out to the existing provider UI instead of rebuilding it.

# Oxy Studio

> **Working name.** `Studio` is provisional. The project is an internal engineering workspace for Oxy; see [`docs/NAMING.md`](docs/NAMING.md) for naming direction.

Oxy Studio is the **internal control plane for building, inspecting, debugging and operating the Oxy ecosystem**.

The first-class problem is PostgreSQL: Oxy needs a genuinely good interface for its databases without replacing PostgreSQL or moving the platform to Supabase. The longer-term goal is broader: one secure workspace where Oxy developers can understand and operate databases, services, deployments, logs, repositories, infrastructure and AI agents.

## Product boundary

Oxy already has a public-facing **Oxy Console** at `console.oxy.so` for developers integrating with Oxy: apps, OAuth credentials, webhooks, request logs and usage.

This project is intentionally different:

| Oxy Console | Oxy Studio |
| --- | --- |
| External + first-party API developers | Oxy employees and authorized internal developers |
| Applications, OAuth credentials, webhooks, API usage | Databases, infrastructure, services, deployments, logs, repos, operations |
| Product/API control plane | Internal engineering control plane |
| Safe for ordinary Oxy accounts | Privileged, explicit internal authorization |

Studio must not become a second implementation of Console.

## Principles

1. **Database-first, platform-second.** Build an excellent PostgreSQL experience first, but keep the architecture modular.
2. **PostgreSQL stays PostgreSQL.** Studio is an interface and control plane, not a new database platform.
3. **No privileged browser access.** Browsers never receive database passwords, AWS credentials or unrestricted provider tokens.
4. **Oxy Auth is the identity layer.** Internal authorization is explicit, scoped and auditable.
5. **Safe by default.** Production writes, destructive SQL and infrastructure mutations require stronger guardrails than reads.
6. **Provider-independent core.** PostgreSQL, GitHub, AWS, Cloudflare and future providers are adapters, not assumptions embedded throughout the UI.
7. **Humans stay in control of agents.** AI can inspect, explain and propose freely within its scope; dangerous mutations require human approval.
8. **Open-source provenance is explicit.** Where Apache-2.0 Supabase Studio / pg-meta code is reused, source and license attribution must remain traceable.

## Initial architecture

```text
Browser
  |
  | HTTPS + Oxy session
  v
Studio Web
  |
  v
Studio Gateway / Control Plane
  |-- authentication + internal authorization
  |-- audit log
  |-- policy / approvals
  |-- connection registry (secret references, never raw secrets in browser)
  |-- module adapters
  |
  +--> PostgreSQL / pg-meta ------> Oxy RDS PostgreSQL 17
  +--> Oxy APIs ------------------> oxy-api / Mention / Alia / ...
  +--> AWS adapter ---------------> ECS / RDS / CloudWatch / SSM / S3
  +--> GitHub adapter ------------> repos / PRs / Actions / issues
  +--> Cloudflare adapter --------> DNS / Pages (later)
  +--> AI / MCP policy layer -----> approved tools and scoped actions
```

Oxy production currently runs on AWS `us-west-2`, with PostgreSQL 17 on RDS, ECS Fargate services, Valkey, S3/SES and Cloudflare frontends. The privileged Studio gateway should therefore run where it can reach private Oxy infrastructure; the web UI can remain independently deployable.

## Modules

### 1. Databases — first milestone

- database and schema browser
- table data editor with filters and pagination
- SQL editor with saved queries and history
- relationships and schema visualization
- columns, constraints, indexes, enums and types
- functions, triggers and extensions
- roles and PostgreSQL policies
- query plans / `EXPLAIN`
- sessions, locks and basic performance views
- import/export
- migration history
- Drizzle schema awareness and production drift detection
- safe production mutations and complete audit history

### 2. Services & deployments

- Oxy service inventory and health
- environment/version currently deployed
- recent deploys and GitHub Actions state
- logs and error links
- restart/redeploy actions behind scoped permissions
- configuration metadata and secret references

### 3. Repositories

- OxyHQ repository inventory
- branches, pull requests and CI
- deployment relationship between commit and running service
- issues/runbooks linked to services and incidents

### 4. Infrastructure & observability

- RDS health/capacity/snapshots
- ECS tasks/services
- CloudWatch metrics/logs
- Valkey health
- S3/storage overview
- DNS/domain status
- environment and dependency map

### 5. AI & agents

- explain SQL and query plans
- propose indexes and schema changes
- generate migration drafts
- investigate incidents using logs + deploy history + schema context
- MCP/tool access with the same authorization model as humans
- read-only by default; explicit approval for write actions

## Repository plan

The intended standalone structure is documented in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md). We do **not** want to import the entire Supabase monorepo and inherit its platform-specific coupling. We will selectively reuse or adapt the excellent database UX and `pg-meta` concepts while progressively replacing Supabase-specific management, billing, auth and project assumptions with Oxy-native adapters.

## Roadmap

See [`docs/ROADMAP.md`](docs/ROADMAP.md).

The MVP is successful when an authorized Oxy developer can safely inspect Oxy PostgreSQL, edit non-production data, run SQL, understand schema/relationships and diagnose queries from one interface — without exposing production credentials to the browser.

## Security

This is privileged internal software. Read [`docs/SECURITY.md`](docs/SECURITY.md) before implementing provider access or mutation endpoints.

## Status

**Planning / bootstrap.** No production access should be added until the gateway, authorization model and audit trail exist.

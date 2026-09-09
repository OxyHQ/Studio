# Project Charter

## Mission

Build the internal engineering workspace Oxy developers use to understand and safely operate the Oxy ecosystem.

Studio starts with PostgreSQL because database inspection, querying, schema understanding and safe administration are immediate recurring needs. It expands only where bringing context together creates a meaningfully better Oxy engineering workflow than using isolated provider dashboards.

## Problem

Oxy engineering context is distributed across PostgreSQL clients, GitHub, AWS, logs, infrastructure code and application-specific tools. The result is unnecessary context switching and weak linkage between questions such as:

- What data/schema is actually in production?
- Which migration/commit created this?
- What version of Mention is currently running?
- Did errors begin after a deploy?
- Which database and infrastructure resources belong to this service?
- Is production drifting from the code/IaC that is supposed to define it?
- Can an agent investigate this safely without receiving broad credentials?

Studio should make these relationships explicit.

## Primary users

- Oxy developers
- Oxy operators/maintainers
- authorized internal support/operations roles where appropriate
- Oxy internal engineering agents acting under scoped permissions

Studio is **not** for ordinary Oxy accounts or third-party developers. Those workflows belong to Oxy Console and other public products.

## Product promise

From one service/resource context, an authorized developer should be able to move naturally between:

```text
service
  -> database/schema/data
  -> repository/PR/commit
  -> build/deployment
  -> running infrastructure
  -> logs/metrics
  -> audit/incident history
```

The product should answer “what is happening and why?” before it attempts to provide every possible mutation button.

## Scope

### Core

1. PostgreSQL exploration and administration
2. SQL/query analysis
3. schema + migration/code awareness
4. Oxy service graph
5. GitHub/deployment context
6. AWS operational context
7. logs/metrics/incident investigation
8. audited, policy-controlled operations
9. scoped AI/MCP engineering tools

### Possible later modules

- storage/object inspection
- background jobs/queues
- scheduled jobs/cron
- SES/email diagnostics
- config/feature management
- domains/certificates
- dependency/security views
- cost/capacity views
- preview environments/database branches
- highly scoped internal support tooling

A later module is accepted because it improves an Oxy workflow, not merely because its provider has an API.

## Non-goals

- replace Oxy Console
- replace PostgreSQL/RDS
- replace Terraform as source of truth for infrastructure
- clone the full AWS console
- clone all of GitHub
- become a customer-facing admin panel
- become a generic database SaaS during the initial project
- store arbitrary secrets in Studio
- give AI agents autonomous broad production access

## Product principles

### Context over duplication

Prefer linking and correlating provider information over rebuilding entire provider products.

### Read before write

Make inspection excellent first. Add mutations only after authorization, audit and safety boundaries exist.

### Production is a distinct mode

Production policy must differ materially from local/dev/staging. A colored badge alone is not a safety control.

### Code/IaC remain authoritative

Drizzle migrations own application schema intent. Terraform owns durable infrastructure intent. Studio shows, compares, diagnoses and performs controlled operations without creating hidden alternate sources of truth.

### Provider independence

Studio domain concepts should survive a future move from RDS to another PostgreSQL host, or from one observability/provider implementation to another.

### Open source without accidental lock-in

Reuse high-quality upstream open-source components when it reduces work, but maintain clear provenance and avoid inheriting unrelated hosted-platform architecture.

### Agents follow human policy

AI/MCP actions use the same capability, resource, environment, approval and audit system as human actions.

## Success measures

### Database MVP

- normal Oxy database inspection no longer requires a separate PostgreSQL GUI
- a developer can find table/schema/relationship/query-plan information quickly
- production credentials never reach the browser
- all production-sensitive operations are attributable and auditable

### Internal platform

- every first-party Oxy service has one canonical Studio page linking repo, DB, deployment and runtime context
- common incident/debug flows require materially less switching among GitHub/AWS/DB tools
- a developer can identify the running commit and relevant logs for a service from the same context
- live DB schema drift from declared migrations is visible

### Agent platform

- agents can investigate engineering problems through read-scoped tools
- proposed fixes/migrations are reviewable before execution
- no agent requires a generic production DB/AWS credential to do normal analysis

## Governance

Architecture-changing decisions should use short ADRs in `docs/adr/`.

An ADR is expected when changing:

- privileged trust boundaries
- authorization model
- database source-of-truth rules
- provider adapter contracts
- upstream Supabase reuse/fork strategy
- production mutation/approval policy
- major runtime/deployment architecture

Small implementation choices do not need ceremony.

## Initial strategic decision

The project is **database-first, platform-second**.

Do not delay the PostgreSQL MVP to build every future adapter. At the same time, do not take shortcuts that force database-specific privileged logic directly into the browser or make later modules impossible to secure consistently.

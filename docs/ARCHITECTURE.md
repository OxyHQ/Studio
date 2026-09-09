# Architecture

## Goal

Studio is an **internal engineering control plane**. The architecture must support a rich PostgreSQL experience first and then allow Oxy-specific operational modules to be added without coupling the frontend directly to AWS, GitHub, database credentials or other privileged systems.

## Core topology

```text
                         +----------------------+
                         |      Studio Web      |
                         | React / TypeScript   |
                         +----------+-----------+
                                    |
                                    | HTTPS
                                    v
                    +---------------+----------------+
                    |     Studio Gateway / BFF       |
                    |                                |
                    | Oxy Auth session validation    |
                    | internal RBAC / ABAC            |
                    | audit trail                    |
                    | action policy + approvals      |
                    | connection registry            |
                    | adapter orchestration          |
                    +----+-----------+-----------+---+
                         |           |           |
             +-----------+           |           +----------------+
             v                       v                            v
     +-------+-------+       +-------+-------+            +-------+-------+
     | Database      |       | AWS adapter   |            | GitHub adapter|
     | adapter       |       | ECS/RDS/etc.  |            | repos/CI/etc. |
     +-------+-------+       +---------------+            +---------------+
             |
             v
     +-------+-------+
     | pg-meta layer |
     +-------+-------+
             |
             v
     +-------+-----------------------+
     | PostgreSQL                    |
     | RDS / local / future providers|
     +-------------------------------+
```

## Why a gateway is mandatory

Studio is more privileged than an ordinary Oxy app. The browser must never be trusted with:

- PostgreSQL credentials
- unrestricted AWS credentials
- raw SSM/Secrets Manager values
- long-lived GitHub installation tokens
- service credentials with broad Oxy scopes

All privileged operations go through the gateway. The gateway resolves secret references server-side, enforces authorization, records audit events and applies environment-specific safety policy.

## Proposed repository layout

```text
apps/
  web/                    # Internal web application
  gateway/                # Privileged backend / BFF

packages/
  ui/                     # Oxy Studio design system and reusable UI
  core/                   # Shared domain types, routing, errors, policies
  authz/                  # Roles, capabilities, resource scopes, approvals
  audit/                  # Audit event model + helpers
  database/               # Database domain models and UI-independent logic
  adapter-sdk/            # Contract for Studio modules/adapters
  adapters/
    postgres/
    oxy/
    aws/
    github/
    cloudflare/           # later
  ai/                     # Agent policy, tool schemas, approval boundaries
  config/                 # Typed config

docs/
  ARCHITECTURE.md
  ROADMAP.md
  SECURITY.md
  NAMING.md
  adr/

third_party/
  README.md               # Provenance and license notes for reused code
```

The exact workspace tooling can change during bootstrap, but module boundaries should remain stable.

## Frontend

### Direction

Use a standalone React/TypeScript application rather than retaining the complete Supabase platform application.

Supabase Studio is currently transitioning across Next.js and TanStack-based paths and contains many Supabase-specific packages. We should reuse proven database UX/components where valuable, but avoid inheriting unrelated billing, organization, project-provisioning, Supabase Auth and hosted-platform assumptions.

### UI principles

- dense but understandable, optimized for desktop developer workflows
- keyboard-first SQL/editor interactions
- command palette spanning all Studio modules
- environment is always visible (`local`, `dev`, `staging`, `production`)
- dangerous production actions are visually unmistakable
- every resource has stable deep links
- shared entity model: project/service/database/repository/deployment/environment

## Gateway

The gateway is the central policy enforcement point.

Responsibilities:

1. validate Oxy identity/session
2. resolve internal membership and permissions
3. enforce resource/environment scopes
4. resolve provider credentials from server-side secret stores
5. expose normalized APIs to Studio Web
6. proxy/mediate pg-meta rather than exposing it publicly
7. generate immutable audit events for privileged reads and all mutations
8. implement approval/step-up flows for dangerous actions
9. issue short-lived scoped capabilities to agent jobs when needed
10. redact secrets and sensitive values from errors/logs

The gateway should be deployed in AWS where it can reach the private RDS instance over the existing VPC/security-group model.

## Adapter model

Studio should not become a collection of provider SDK calls scattered through UI routes.

Each integration implements a small capability-oriented adapter contract. Example:

```ts
interface StudioAdapter {
  id: string
  capabilities(): Promise<Capability[]>
  health(): Promise<AdapterHealth>
}

interface DatabaseAdapter extends StudioAdapter {
  listDatabases(): Promise<DatabaseSummary[]>
  inspectDatabase(ref: DatabaseRef): Promise<DatabaseMetadata>
  execute(request: SqlExecutionRequest): Promise<SqlExecutionResult>
}
```

Real interfaces will be narrower and split by capability, but the rule is important: **the UI depends on Studio domain contracts, not directly on AWS/Supabase/GitHub SDK shapes.**

## Database architecture

### Source of truth

Studio does not own Oxy application schemas. For Oxy services, schema ownership remains with each application's migration/schema system (currently Drizzle for the core API).

Studio provides:

- introspection of live PostgreSQL
- safe data browsing/editing
- SQL execution
- administration
- migration visibility
- comparison between declared schema and live schema

### pg-meta

Supabase `pg-meta` is the preferred starting point for PostgreSQL introspection/administration because it already models many operations Studio needs. It must sit behind the Studio gateway and must never be directly Internet-exposed.

Initial strategy:

1. depend on/reuse upstream pg-meta where practical
2. pin and record the upstream version/commit
3. wrap it in Oxy authorization/audit policy
4. fork only where Oxy-specific behavior actually requires divergence

This reduces maintenance compared with immediately maintaining a hard fork.

### Connection registry

Connections are resources, not raw URLs stored in the browser.

```text
Connection
- id
- display name
- provider (postgres/rds/supabase/neon/local/...)
- environment
- host metadata safe to display
- credential reference (server-side only)
- allowed capabilities
- ownership/team
- tags
```

For Oxy production, the credential reference should resolve inside AWS. Raw passwords must not be persisted in frontend state, local storage or client logs.

### Environments

At minimum:

- local
- development
- staging
- production

Policy is environment-aware. Production should default to read-only until a user explicitly enters a write-capable workflow.

## Database module capabilities

### Read path

- database/schema/table browser
- rows with pagination/filtering/sorting
- relationships
- schema graph
- columns/constraints/indexes
- functions/triggers/types/extensions
- roles/policies
- query plans
- sessions/locks
- table/database size
- migration history
- saved SQL

### Write path

- insert/update/delete rows
- DDL operations
- role/policy changes
- extension changes
- SQL execution

All writes produce audit events. Dangerous SQL should be classified before execution where possible.

### Drizzle integration

For Oxy repositories using Drizzle:

```text
Git repository schema/migrations
            |
            v
     Studio schema model
            |
            +------ compare ------> live PostgreSQL
                                     |
                                     v
                              drift report
```

Desired features:

- migration timeline linked to Git commits/deployments
- declared-vs-live schema diff
- unapplied/unknown migration warnings
- migration draft generation
- no automatic production migration by default

## Oxy module

Studio should model the Oxy ecosystem explicitly:

- services: oxy-api, Mention, Alia, Homiio, Syra, Allo, future services
- environments
- service URLs
- deployed version/commit
- health
- linked database
- linked repository
- linked infrastructure resources
- runbooks

This entity graph becomes the foundation for cross-tool navigation.

Example:

```text
Mention
├── repository
├── ECS service
├── deployment / commit
├── database
├── logs
├── health
└── incidents
```

## AWS adapter

Initial read capabilities:

- ECS cluster/service/task status
- RDS instance health and safe metadata
- CloudWatch metrics/log discovery
- snapshot inventory
- S3 bucket metadata where relevant
- SSM parameter names/metadata, not values by default

Later mutation capabilities:

- restart/redeploy
- snapshot creation
- controlled scaling

Infrastructure-as-code remains authoritative. Studio should not silently make configuration mutations that Terraform will later overwrite.

## GitHub adapter

Capabilities:

- repositories
- default branch/latest commit
- PRs
- Actions runs/jobs
- commit-to-deployment mapping
- issues/runbooks
- links to schema/migration changes

Write actions should initially remain in GitHub itself unless a Studio-native action clearly improves the workflow.

## Observability

Studio should normalize links and context before attempting to replace observability systems.

First:

- service health
- recent errors
- logs scoped to service/deployment/request
- key RDS/ECS metrics
- deployment markers

Later:

- saved dashboards
- cross-service tracing
- incident timelines
- anomaly detection

## AI and MCP

AI is a module governed by the same authorization model as human actions.

Agent capability classes:

1. **Observe** — inspect schemas, metrics, logs, code metadata
2. **Analyze** — explain SQL, correlate deploys/errors, propose fixes
3. **Draft** — prepare SQL, migrations, PR descriptions or runbook actions
4. **Execute safe** — narrowly scoped reversible actions where policy allows
5. **Dangerous mutation** — always requires explicit human approval; some actions may additionally require a second approver

Never grant an AI agent a generic production `DATABASE_URL` or broad AWS credential merely for convenience.

## Audit model

Every mutation and sensitive privileged read should answer:

- who/what acted?
- under which Oxy identity/service/agent?
- when?
- which environment/resource?
- what capability was used?
- what changed?
- was approval required and who approved?
- correlation/request id
- originating commit/deployment/issue if relevant

Audit data must not contain plaintext secrets.

## Deployment

Recommended initial deployment:

```text
Studio Web       -> Cloudflare Pages or equivalent static/web hosting
Studio Gateway   -> AWS ECS Fargate in Oxy production VPC
PostgreSQL       -> existing private RDS PostgreSQL 17
Audit storage    -> PostgreSQL initially, designed for append-only semantics
```

A future desktop/local client may connect to a local gateway for localhost databases, but production access should continue through the server-side control plane.

## Non-goals for MVP

- replacing Oxy Console
- replacing PostgreSQL/RDS
- replacing Terraform
- replacing GitHub
- becoming a generic public database SaaS
- building a new secrets manager
- direct browser-to-production database connections
- autonomous production operations by AI

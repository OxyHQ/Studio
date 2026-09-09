# Security Model

Studio is privileged internal infrastructure. Security is part of the product architecture, not a later hardening pass.

## Threat model

Assume:

- a browser session can be compromised
- an authenticated Oxy user may not be an authorized Oxy employee/developer
- an internal developer may have permission for one service/environment but not another
- a provider token/database credential is catastrophic if leaked
- SQL can be destructive even when issued by an authorized user
- AI-generated actions can be wrong or manipulated by untrusted context
- logs, query results and support data may contain personal/sensitive information

## Identity

Use Oxy Auth for authentication.

Authentication alone is insufficient. Studio requires **explicit internal membership** and capability grants.

Do not authorize access based solely on:

- email domain
- repository membership inferred in the browser
- a hidden URL
- a generic `isInternal` frontend flag

Internal access must be server-validated.

## Authorization

Prefer capabilities scoped by resource and environment rather than broad role checks spread throughout code.

Example capabilities:

```text
database.read
database.query.read
database.query.write
database.schema.write
database.destructive
service.read
service.restart
deployment.read
deployment.execute
logs.read
secrets.metadata.read
secrets.value.read
infrastructure.read
infrastructure.mutate
audit.read
agent.approve
```

Example principal:

```text
Principal
- user/service/agent id
- team memberships
- global role
- resource grants
- environment grants
- temporary elevated grants
```

The gateway is the final authorization authority. UI hiding is convenience, never enforcement.

## Suggested roles

Roles are presets; capabilities remain the real enforcement primitive.

### Viewer

- inspect services/databases/metrics
- read safe metadata
- no raw production SQL by default
- no mutations

### Developer

- Viewer capabilities
- development/staging SQL and data mutations
- logs
- deployment metadata

### Operator

- Developer capabilities
- approved production operational actions
- restart/redeploy
- production diagnostic queries

### Admin

- schema/admin operations
- permission management within policy
- sensitive production operations

### Owner

- break-glass / highest-risk capabilities
- security policy administration

Avoid making `admin` a bypass that silently disables policy.

## Environment policy

### Local / development

Fast workflow, standard auditing.

### Staging

Writes allowed for appropriate developers; destructive operations still require confirmation.

### Production

Production is a different security mode, not merely a badge.

Defaults:

- read-only browsing
- write capability must be explicit
- elevated sessions expire quickly
- step-up auth for dangerous operations
- statement timeout and resource limits
- destructive operations require typed confirmation
- catastrophic operations may require second approval
- all mutations audited

## Database credentials

Never send a production connection string/password to Studio Web.

The connection registry stores only a credential **reference**. The gateway resolves it server-side from an approved secret source.

Where possible:

- prefer short-lived credentials/tokens over static passwords
- separate read-only and write-capable DB principals
- separate environments
- set PostgreSQL `application_name` to identify Studio principal/session
- enforce connection/query timeouts
- terminate idle privileged sessions

## pg-meta

`pg-meta` is an internal implementation detail and is not an Internet security boundary.

Requirements:

- no publicly reachable pg-meta endpoint
- gateway authentication/authorization in front of every operation
- connection selection controlled server-side
- input/result limits
- audit at the gateway
- network access restricted to required DBs

## SQL execution safety

SQL classification is defense-in-depth, not a perfect parser-based guarantee.

Execution endpoints should carry explicit intent, for example:

```text
/query/read
/query/write
/query/destructive
```

Do not expose one universal SQL endpoint and rely exclusively on frontend warnings.

Controls:

- parse/classify statements server-side
- reject multiple statements where not required
- statement timeout
- row/result-byte limit
- cancellation
- transaction boundaries where useful
- explicit production write mode
- block known-dangerous statements without the required capability
- never interpolate identifiers/values unsafely in generated operations

`EXPLAIN ANALYZE` executes a query and must be treated according to the underlying statement's risk.

## Destructive operations

Examples:

- `DROP DATABASE`
- destructive `DROP TABLE`
- mass `DELETE`/`UPDATE` without safe predicates
- role/ownership changes
- destructive schema resets
- snapshot deletion
- infrastructure changes that remove capacity/data

Controls can include:

1. elevated capability
2. step-up WebAuthn/passkey auth
3. explicit resource/environment display
4. impact preview
5. typed resource confirmation
6. reason/ticket field
7. second approver for selected action classes
8. immutable audit event

## Secrets

Studio is not a secrets manager.

Default UX should display:

- secret/parameter name
- source/provider
- last updated metadata
- consuming services where available

Secret values remain hidden unless a narrowly defined workflow genuinely requires reveal permission. Every reveal is a sensitive audited read.

Never include secrets in:

- frontend telemetry
- URLs
- error messages
- audit payloads
- AI prompts/context
- screenshots/previews

## AWS

Use the narrowest possible IAM permissions.

Prefer workload identity/task roles for the Studio gateway over long-lived keys.

Separate read and mutation permissions where practical. Infrastructure mutation must respect Terraform ownership; Studio should not become an undocumented alternate control plane for durable IaC state.

## GitHub

Prefer a dedicated GitHub App / installation permissions model over personal access tokens.

Do not give Studio repo administration permission just to read CI state.

## AI / agents

Agents are principals, not trusted code.

Rules:

- every agent has an identity
- every tool call maps to a Studio capability
- agent scopes should normally be narrower than the human who invoked it
- production write tools require human approval
- tool output from code/logs/database content is untrusted input and can contain prompt injection
- secrets are excluded from model context
- agent actions are audited with model/tool/correlation metadata
- proposals and execution are separate actions

## Privacy

Studio can expose production data, so data minimization matters even internally.

- do not fetch table rows until requested
- provide column masking/redaction capability
- make sensitive schemas/tables separately permissionable where needed
- avoid logging SQL results
- audit access to highly sensitive datasets
- maintain clear boundaries for internal support tooling

## Audit log

Audit events should be append-only from the application's perspective.

Minimum event fields:

```text
id
timestamp
principal type/id
session/correlation id
source IP/device metadata where appropriate
action/capability
provider/module
resource id
environment
result (success/failure)
approval metadata
safe change summary
```

Do not store raw secrets or full sensitive result sets in audit events.

## Break-glass

A future emergency path should:

- require stronger auth
- be time-limited
- require a reason
- notify appropriate maintainers
- produce explicit high-severity audit events
- never become the normal workaround for missing permissions

## Public repository considerations

The repository may remain public, but runtime topology docs must avoid publishing:

- secrets
- credentials
- private hostnames/IPs that add unnecessary exposure
- access tokens
- production connection strings
- security-control bypass instructions

Architecture can be public; secret material cannot.

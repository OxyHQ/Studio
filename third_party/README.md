# Third-party provenance

Studio may selectively reuse or adapt open-source components from Supabase Studio and related projects, especially PostgreSQL administration/introspection work around `pg-meta`.

## Supabase

Upstream repository: `supabase/supabase`

License: Apache License 2.0.

Before copying any upstream source into this repository:

1. record the exact upstream repository, path and commit/tag
2. preserve applicable copyright, license and attribution notices
3. mark materially modified files as modified where required by the upstream license
4. include any upstream `NOTICE` obligations that apply to the copied work
5. avoid using Supabase trademarks/branding in a way that implies an official Supabase product
6. prefer depending on upstream packages over copying/forking when Oxy does not need divergence

## Initial strategy

Do **not** import the entire Supabase monorepo.

Evaluate components individually:

- database table/grid UX
- SQL editor and query tooling
- schema/relationship visualization
- PostgreSQL metadata types/helpers
- `pg-meta`
- reusable UI primitives where licensing/dependency boundaries make sense

Supabase-specific hosted-platform functionality such as billing, organizations, project provisioning and Supabase Auth management is outside Studio's intended core and should not be imported merely because it exists upstream.

When the first upstream code is brought into this repository, expand this file into a machine-readable provenance inventory with pinned source commits and affected paths.

# Naming

The final product name is **Oxy Studio**.

The repository remains `OxyHQ/Studio`, and documentation, UI, packages and deployment naming should use **Oxy Studio** consistently.

## Why Studio

The project is intended to become the internal place where Oxy engineers inspect, build, debug and operate the ecosystem. Although PostgreSQL is the first major module, the product is broader than a database client: databases, services, deploys, logs, repositories, infrastructure and agents all belong in the same engineering workspace.

`Studio` fits that model well because it describes an integrated visual workspace without limiting the product to SQL, operations or infrastructure.

## Distinction from Oxy Console

**Oxy Console** already means the developer-facing Oxy Cloud console for applications, OAuth credentials, webhooks, request logs and usage.

**Oxy Studio** is different:

- internal and privileged
- intended for Oxy employees and explicitly authorized developers/operators
- focused on databases, services, deployments, infrastructure, repositories, observability and engineering operations
- backed by stronger authorization, audit and production safety controls

The two products should remain clearly separate in language, routing and responsibilities.

## Decision

Selected: **Oxy Studio**

Repository: `OxyHQ/Studio`

Decision date: September 9, 2026.

Previous alternatives such as Oxy Forge, Oxy Workshop, Oxy Works, Oxy Ops and Oxy Command are no longer under consideration unless the product direction changes materially in the future.

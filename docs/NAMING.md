# Naming

`Oxy Studio` is a working name, not a final product decision.

The project is not merely a PostgreSQL client. It is intended to become the internal place where Oxy engineers inspect, build, debug and operate the ecosystem. The name should therefore still make sense when the product contains databases, services, deploys, logs, repositories, infrastructure and agents.

## Existing name to avoid overlapping with

**Oxy Console** already means the developer-facing Oxy Cloud console for applications, credentials, webhooks and usage. This internal tool should not also be called Console.

## Naming criteria

A good name should:

- feel internal/technical rather than consumer-facing
- work for both database and broader engineering operations
- be short and easy to say in conversation
- fit `Oxy <name>` naturally
- avoid implying it is only observability, only infrastructure or only SQL
- avoid confusing users with Oxy Console
- allow the repository/package namespace to remain sensible

## Candidates

### 1. Oxy Forge — recommended working direction

Why it works:

- a forge is where things are built and shaped
- fits developers, infrastructure and operations
- broad enough for databases, deploys and agents
- distinct from Oxy Console
- natural internal language: “check Forge”, “open it in Forge”, “Forge says prod is drifting”

Potential downside: `Forge` is used by many developer products, so trademark/domain/package availability should be checked before making it public-facing.

### 2. Oxy Studio

Why it works:

- excellent fit for an integrated visual workspace
- immediately understandable
- directly matches the Supabase Studio inspiration

Downside:

- generic
- suggests a UI/workbench more than an operational control plane
- could make the project sound like a Supabase derivative forever, even after it becomes much more Oxy-specific

### 3. Oxy Workshop

Friendly and accurate: a place where the team works on Oxy.

Downside: less crisp as a serious infrastructure tool.

### 4. Oxy Works

Broad and brandable; implies the place where Oxy work happens.

Downside: weaker immediate developer-tool meaning.

### 5. Oxy Ops

Clear for operations.

Downside: too narrow because development/database workflows are central.

### 6. Oxy Command

Strong “command center” feeling.

Downside: sounds more operational and action-oriented than exploratory/developer-oriented.

### 7. Oxy Lab

Good for experimentation.

Downside: implies non-production/experimental software, which is the opposite of a tool eventually trusted with production operations.

## Recommendation

Keep the GitHub repository as `OxyHQ/Studio` during bootstrap and use **Studio** as the neutral working name in code/docs.

Before the first polished internal release, decide whether to rename the product/repository. **Oxy Forge** is the strongest current candidate because it continues to fit the project after PostgreSQL stops being the only major module.

Do not spend engineering time on a rename until the first database MVP shape is visible; the real product vocabulary will make the naming decision easier.

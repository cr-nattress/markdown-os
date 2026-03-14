# Persistence Model

## Core Principle

The author never thinks about storage. They think about **memory.** The markup says what the application needs to remember, and the platform handles everything underneath. No database selection, no connection strings, no schema migrations, no capacity planning. The persistence layer is a **platform primitive**, not an application dependency.

## What the Author Sees

```markdown
Check whether we've already seen this order ID. If we have,
skip it — it's a duplicate.

Remember this order's ID, its fraud score, and when we
processed it. Keep this information for 90 days.
```

The author didn't choose a database, define a schema, or configure retention policies. They said "remember this for 90 days" and described why.

## Memory Profiles

When the LLM compiles the markup, it extracts a **memory profile** — a structured description of what the application needs to remember, including purpose, what data is stored, access patterns, uniqueness constraints, retention, and volume estimates.

The platform reads the memory profile and maps it to internal storage primitives. The author never sees this mapping.

## Storage Primitives

Five internal primitives cover the vast majority of application needs:

**Lookup** — Key-value storage. Fast reads by a known key. Use cases: deduplication, caching, configuration state. Maps from RECALL and CHECK intents.

**Log** — Append-only, time-ordered storage. Use cases: audit trails, event histories, processing records. Maps from REMEMBER intents describing events over time.

**Counter** — Numeric accumulators. Use cases: rate limiting, usage tracking, scoring aggregation. Maps from UPDATE intents describing incrementing/decrementing.

**Queue** — Ordered items awaiting processing. Use cases: retry buffers, batch collection, work distribution. Maps from REMEMBER + RECALL for deferred processing.

**Snapshot** — Point-in-time state capture. Use cases: "current state of X" patterns. Maps from UPDATE intents describing maintaining a current picture.

## Context Window Management

As applications accumulate state, the platform can't feed everything into the LLM context. The memory profile determines what to preload:

| Pattern | Context Strategy |
|---|---|
| **Lookup** | Platform performs the lookup and injects the result. LLM never sees the full table. |
| **Log** | Platform loads relevant slices (e.g., last 7 days for weekly summary). |
| **Counter** | Platform loads the current value. Tiny footprint. |
| **Queue** | Platform loads the next N items. LLM emits dequeue mutations. |
| **Snapshot** | Platform loads the current snapshot. One object. |

## Automatic Schema Evolution

No migrations needed. If the markup adds a new field, new records include it; old records don't — the LLM understands this naturally. If fields are removed, old records still have them (LLM ignores them). If retention changes, new writes get the new TTL; old data ages out naturally.

## Conversion Targets

When markup is converted to code, the persistence abstraction becomes an interface:

```typescript
interface MemoryClient {
  lookup(store: string, key: string): Promise<any | null>;
  append(store: string, data: Record<string, any>): Promise<void>;
  query(store: string, filter: QueryFilter): Promise<any[]>;
  delete(store: string, key: string): Promise<void>;
}
```

Three export modes: browser-only (localStorage), platform-connected (React frontend calls platform API), or self-contained (includes lightweight API + database).

## The Zero-Dependency Promise

~~Database~~ → Platform handles it. ~~ORM~~ → Intents replace queries. ~~Migration tooling~~ → Schemaless evolution. ~~Connection management~~ → Platform-managed. ~~Backup/scaling~~ → Platform-managed.

# Platform Architecture

## Overview

Markdown OS is three things:

1. **A markup authoring environment** — The customer describes what they need; the platform produces markup. The customer reviews, refines, and approves it.
2. **A runtime orchestrator** — When a request arrives, the platform loads markup, feeds it to the LLM with the request, manages the I/O envelope, handles persistence operations, and returns the result.
3. **A management plane** — Health, lifecycle, configuration, observability — all platform-level concerns that wrap around the markup-based application.

## Application Unit Anatomy

Each application has three parts:

**The Core** — The single-purpose logic. This is the "what it does." The core is opinionated: it makes decisions about *how* to do the thing based on best practices for that specific thing.

**The Contract** — Well-defined inputs and outputs. Every unit exposes a standard management plane: health/status, configuration, lifecycle (deploy/pause/teardown/snapshot), and observability (logs, metrics, traces shaped to the domain). The contract is the same *shape* across all units but the *content* is purpose-specific.

**The Persistence Layer** — What needs to be remembered and how. Not "use PostgreSQL with this table schema" but "maintain a deduplicated log of processed order IDs. Retention: 90 days." The platform maps this to actual storage; the markup declares the what and why.

## Compilation Pipeline

```
1. MARKUP (the application document)
   ↓
2. LANGUAGE REFERENCE (the intent catalog + execution semantics)
   ↓
3. MEMORY PROFILE (extracted state requirements)
   ↓
4. TOOL DEFINITIONS (platform primitives the LLM can invoke)
   ↓
5. ASSEMBLED SYSTEM PROMPT:
   ┌──────────────────────────────────────────┐
   │ Language Reference (how to interpret)     │
   │ Application Markup (what to do)           │
   │ Memory Profile (what state exists)        │
   │ Current State (pre-loaded relevant data)  │
   │ Tool Definitions (what you can invoke)    │
   │ Guardrails (what you must never do)       │
   └──────────────────────────────────────────┘
   ↓
6. INCOMING REQUEST (the user message)
   ↓
7. LLM RESPONSE:
   - Tool calls (side effects for platform to execute)
   - Text (application output, if any)
   ↓
8. PLATFORM EXECUTION:
   - Execute tool calls (storage, external APIs, delegation)
   - Package response for caller
   - Log metrics, traces, management signals
```

Steps 1-5 are "compile time" (cached and reused across requests). Steps 6-8 are "runtime" (per-request).

## Compilation Modes

**Just-in-Time (JIT)** — Every request, the LLM reads the full markup and processes the input. Simplest model. Higher latency, maximum flexibility. Good for low-throughput, high-value operations.

**Ahead-of-Time (AOT)** — The LLM reads the markup and produces an optimized artifact: a system prompt, a fine-tuned adapter, a structured pipeline, or even traditional code. Trades flexibility for performance.

**Hybrid** — The platform decides which compilation mode based on performance requirements. The customer never knows or cares — the markup is the same either way.

## Tool Calling as the Execution Model

The LLM executes the application by making tool calls — platform primitives registered as tools:

```json
{
  "tools": [
    {
      "name": "remember",
      "description": "Persist data to the application's memory",
      "parameters": {
        "memory_name": "string",
        "data": "object",
        "operation": "append | upsert | delete"
      }
    },
    {
      "name": "recall",
      "description": "Retrieve data from the application's memory",
      "parameters": {
        "memory_name": "string",
        "query": "object"
      }
    },
    {
      "name": "emit",
      "description": "Send output to an external system",
      "parameters": {
        "target": "string",
        "payload": "object",
        "method": "string"
      }
    },
    {
      "name": "alert",
      "description": "Signal a notable event",
      "parameters": {
        "severity": "info | warning | critical",
        "message": "string",
        "context": "object"
      }
    },
    {
      "name": "delegate",
      "description": "Pass work to another application unit",
      "parameters": {
        "target_app": "string",
        "input": "object"
      }
    }
  ]
}
```

The LLM never touches storage directly. It declares what should be persisted, and the platform executes those mutations. Same for external calls. The LLM is pure transformation logic; the platform handles all side effects.

This means the LLM can't go rogue. It can only request actions through the envelope. The platform validates, rate-limits, and audits every side effect.

## Multi-Application Composition

Applications compose through the DELEGATE intent. One application's EMIT feeds another's RECEIVE. The platform orchestrates this without either unit knowing about the other.

Shared state uses explicit declarations in markup. The platform creates read bridges — a report generator can RECALL from a monitor's log but can't REMEMBER or UPDATE into it. Ownership is clear.

## Management Plane

Every application unit, regardless of purpose, is managed through a standardized interface:

| Endpoint | Purpose |
|---|---|
| `GET /health` | Running/paused/error state |
| `GET /metrics` | Request rate, latency, error rate, intent distribution |
| `GET /config` | Current configuration values |
| `PUT /config` | Update configuration values |
| `POST /lifecycle/pause` | Pause processing |
| `POST /lifecycle/resume` | Resume processing |
| `POST /lifecycle/snapshot` | Capture current state |
| `POST /lifecycle/rollback` | Restore from snapshot |
| `GET /logs` | Structured intent-level logs |
| `GET /state` | Human-readable view of persisted data |

These are platform-generated from the markup. The author never defines them.

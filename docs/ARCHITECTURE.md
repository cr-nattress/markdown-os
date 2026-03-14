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

---

## I/O Envelope

Every application unit communicates through a standard envelope — the boundary between the application (markup + LLM) and the platform (infrastructure + management).

```
INBOUND:
┌─────────────────────────────────────────────┐
│  Platform Headers                           │
│    request_id, auth_context,                │
│    rate_limit_state, timestamp              │
│                                             │
│  Application Input                          │
│    payload (webhook, API call, event)       │
│                                             │
│  Markup Reference                           │
│    app_id, version, compiled_prompt         │
│                                             │
│  Persistence Context                        │
│    preloaded_state (relevant stored data)   │
└─────────────────────────────────────────────┘

        ↓ LLM "compiles" and executes ↓

OUTBOUND:
┌─────────────────────────────────────────────┐
│  Platform Headers                           │
│    request_id, processing_time_ms,          │
│    token_usage, trace_id, intent_chain      │
│                                             │
│  Application Output                         │
│    response (the result)                    │
│                                             │
│  Side Effects                               │
│    emit: [{ target, payload, method }]      │
│    delegate: [{ target_app, input }]        │
│                                             │
│  Persistence Mutations                      │
│    remember / update / forget declarations  │
│                                             │
│  Management Signals                         │
│    alerts, metrics                          │
└─────────────────────────────────────────────┘
```

The LLM declares what should happen. The platform executes it. Side effects are batched, validated, rate-limited, and audited by the platform.

---

## Provider Abstraction

Markdown OS uses AI capabilities that exist across at least Claude and OpenAI. If both providers support it, it's fair game as a language primitive. This guarantees provider independence.

### Common Capability Surface

**Tool / Function Calling** — Both providers support structured tool definitions. This is the entire execution model.

**Structured Output / JSON Mode** — Both produce validated JSON. The response envelope is structured JSON.

**System Prompts** — Both support system-level instructions. The compiled markup *becomes* the system prompt.

**Multi-turn Context** — Both support conversation history. Used for multi-step workflows.

**Vision / Multimodal Input** — Both accept images. Applications processing visual input work without special handling.

**Streaming** — Both support SSE. Real-time applications use this transparently.

### Adapter Interface

```
Provider Adapter
├── sendCompletion(systemPrompt, messages, tools) → response
├── parseToolCalls(response) → ToolCall[]
├── parseTextOutput(response) → string
├── supportsVision() → boolean
├── supportsStreaming() → boolean
└── estimateTokens(content) → number
```

| Aspect | Claude Adapter | OpenAI Adapter |
|---|---|---|
| Tool definition format | `tools[]` with `input_schema` | `tools[]` with `parameters` |
| Tool call response | `content[].type === "tool_use"` | `tool_calls[]` in message |
| System prompt | `system` parameter | `{"role": "system"}` message |
| Image input | `content[].type === "image"` base64 | `content[].type === "image_url"` |
| Streaming format | SSE `content_block_delta` | SSE `delta` |

### Provider Selection Strategy

The platform routes intelligently: cost optimization (simple requests to cheaper models), latency optimization (fastest responding provider), capability matching (vision-heavy to best vision model), and availability (failover between providers). The author can optionally express a preference but doesn't have to.

---

## The Converter

The converter transforms markup applications into deployable projects in traditional programming languages. The markup is the source of truth. The generated code is a projection — a different representation of the same application.

The converter is the escape hatch that eliminates vendor lock-in. It's also the proof that the markup is precise enough to be real software.

### Pipeline

```
Markup Document
    ↓
Intent Graph (resolved, structured)
    ↓
Target Language Adapter
    ↓
Complete Deployable Project
```

The intent graph is the key intermediary — the same representation used for platform execution, serialized for the converter.

### First Target: React

React is the first conversion target because it gives immediate visual proof of value (see the app running), the frontend IS the application for most early use cases, React's component model maps naturally to intents, and deployment is solved (Vercel, Netlify, static hosts).

#### Intent → React Mapping

| Intent | React Construct |
|---|---|
| RECEIVE | Props, API response handlers, event listeners |
| EMIT | fetch() calls, API integrations |
| TRANSFORM | Utility functions, useMemo |
| EXTRACT | Destructuring, selector functions |
| FILTER | Array.filter(), conditional rendering |
| BRANCH | Conditional rendering, ternary operators |
| ITERATE | Array.map() → component lists |
| REMEMBER | localStorage writes, state updates |
| RECALL | localStorage reads, state initialization |
| VALIDATE | Zod schemas, form validation |
| MASK | Utility functions (maskEmail, etc.) |
| DISPLAY | JSX components |
| CAPTURE | Form elements, controlled inputs |
| NAVIGATE | React Router, conditional view rendering |
| REACT | onClick, onChange, event handlers |
| REFRESH | setInterval, polling hooks |

#### Generated Project Structure

```
app-name/
├── src/
│   ├── app/           ← root layout, routing, providers
│   ├── features/      ← feature-based component organization
│   ├── shared/        ← memory, integrations, mask utilities
│   ├── domain/        ← domain logic as pure functions, types, validation
│   └── index.tsx
├── tests/              ← intent assertions as tests
├── package.json
├── tsconfig.json
├── tailwind.config.js
└── README.md           ← generated from markup purpose section
```

#### Quality Bar

Generated React apps must feel like a senior developer built them: idiomatic React (hooks, functional components, composition), proper TypeScript (real types, Zod schemas, strict mode), accessible by default (ARIA labels, keyboard nav, semantic HTML), responsive by default (Tailwind utilities), and error states handled (loading spinners, empty states, error boundaries).

### Future Targets

1. **React** (first) — visual proof, client-side apps
2. **React + platform API** — frontend with managed state
3. **Other frameworks** — Vue, Svelte, Angular
4. **Backend targets** — Node/Express, .NET, Java Spring
5. **Mobile** — React Native from the same markup

### Round-Tripping

The converter works in both directions:

```
Markup → Intent Graph → Code    (export / eject)
Code → Intent Graph → Markup    (import / onboard)
```

An existing application can be imported to the platform. Export back to code later, and the code might be *better* than the original — the intent graph enforces clean separation of concerns. The markup is a purification step.

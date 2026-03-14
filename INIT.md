# Markdown OS

## Applications as documents. LLMs as the runtime.

Markdown OS is a platform where applications are written as natural language markup documents, compiled and executed by LLMs, and managed through a standardized operational interface. The markup is the source of truth. The LLM is the compiler. Traditional code is an export target, not the authoring medium.

---

# Table of Contents

- [The Idea](#the-idea)
- [Core Tenets](#core-tenets)
- [Design Philosophy](#design-philosophy)
- [The Markup Language](#the-markup-language)
- [Intent Taxonomy](#intent-taxonomy)
- [Platform Architecture](#platform-architecture)
- [Persistence Model](#persistence-model)
- [I/O Envelope](#io-envelope)
- [Provider Abstraction](#provider-abstraction)
- [The Converter](#the-converter)
- [Examples](#examples)
- [Repository Structure](#repository-structure)

---

# The Idea

A customer describes what they need. The platform produces a markup document — a structured, natural language specification that *is* the application. The LLM reads the markup, understands its intent, and executes it. There is no build step. There is no deploy in the traditional sense. "Deploying" means making a markup document available to the LLM at invocation time. "Updating" means editing the document.

Every application is **single-purpose**. Not a company website that does everything. Not a generic API. A purpose-built application that does one thing perfectly. Persistence, integrations, error handling, and operational interfaces emerge from the purpose — they are not imposed by the platform.

```
Customer describes a need
        ↓
Platform generates MARKUP (the "application")
        ↓
Customer reviews/refines (it's readable — it's a document)
        ↓
LLM reads markup + incoming request → produces correct behavior
        ↓
Standard I/O envelope wraps the result
        ↓
Optionally: CONVERT markup to React, TypeScript, C#, etc.
```

---

# Core Tenets

These are the foundational axioms of the Markdown OS markup language. Every design decision, platform behavior, and converter output must be consistent with these principles. They are non-negotiable.

## 1. Applications Always Build

Any valid markup produces a working application. There is no state where the author has written "not enough" for the application to function. A single sentence is a valid application. A single paragraph is a valid application. A fifty-page specification is a valid application.

```markdown
Track my expenses.
```

This builds. It runs. The LLM interprets the intent, fills in every gap — what an expense looks like, how to store it, how to display it, how to input one — and produces a working expense tracker. It won't be precisely what the author wants, but it will be functional. Refinement comes through specificity, not through reaching some minimum threshold of completeness.

There is no syntax error. There is no missing semicolon. There is no "module not found." **The application always builds.**

This is the most important principle because it eliminates the single biggest barrier to software creation: the blank screen that produces nothing until you've written enough correct code. In Markdown OS, the first word you write already works.

## 2. Generic Markup Creates Generic Applications

The less specific the markup, the more the LLM decides. And the LLM's decisions are reasonable defaults — best practices, common patterns, sensible choices. A generic description produces a generic but functional application.

```markdown
# Expense Tracker
Track personal expenses with categories.
```

This produces an expense tracker with a form for entering expenses (amount, category, note), a list view showing recent expenses, common default categories (food, transport, entertainment, bills, etc.), browser-based storage, and a simple, clean UI.

Every one of those decisions was made by the LLM because the markup didn't specify otherwise. The application works. It's generic. And that's fine — it's the starting point.

## 3. Increasing Specificity Creates More Defined Applications

As the author adds detail, the application becomes more precise. Specificity replaces the LLM's defaults with the author's intent. The progression is continuous — there's no cliff between "generic" and "specific." Every sentence of additional detail sharpens the application.

```
"Track expenses"
  → LLM decides everything
  → Generic but functional

"Track expenses with these specific categories, this display
 format, this auto-categorization logic, this retention policy"
  → LLM fills only the remaining gaps
  → Precisely what the author described
```

Each addition makes the application more defined. Nothing that was working before breaks. The author is painting with increasing resolution — broad strokes first, fine detail when it matters.

## 4. The Language Never Dictates the Output Language

The markup language is fundamentally **target-agnostic.** Nothing in the markup implies, requires, or constrains what programming language, framework, or runtime the application eventually becomes. The markup describes *what* the application does, never *how* it's implemented.

This means the same markup can produce a React app, a C# service, a Python CLI, or a Java API. The markup never references programming concepts (classes, functions, imports, types). The markup never assumes a runtime environment (browser, server, container, edge). The markup is about the problem domain, not the solution technology.

The markup `"Show a table of recent expenses"` is equally valid whether the converter targets React (a `<table>` component), a CLI (a formatted terminal table), or a PDF (a typeset table). The intent is DISPLAY. The target determines the form.

## 5. Specificity Can Define the Target Language

While the markup itself never dictates the output, the author can *choose* to specify a target when they want one. Specificity works in every direction — including the technical direction.

```markdown
# Expense Tracker
Track personal expenses with categories.
Build this as a C# .NET 8 Web API with Entity Framework Core.
```

The author *can* get as specific as naming frameworks, versions, and infrastructure. But they never *have* to. A technical author who wants a .NET 8 API with specific NuGet packages can have that. A non-technical author who just wants "an app that tracks expenses" gets one without ever thinking about technology choices.

## 6. The LLM Fills Gaps and Makes Decisions

When the markup doesn't specify something, the LLM decides. This is not a bug or a limitation — it's the core execution model. The LLM's decisions are reasonable (based on best practices), consistent (similar gaps produce similar decisions), transparent (the platform can surface what decisions were made), and overridable (adding specificity replaces any LLM decision).

What the LLM decides when the markup is silent:

| Gap | LLM Decision |
|---|---|
| No UI described | Generate a clean, sensible default layout |
| No persistence described | Choose storage appropriate to the data patterns |
| No error handling described | Apply standard resilience (retry, log, continue) |
| No validation described | Infer required fields from context |
| No categories/options listed | Generate common defaults for the domain |
| No output format specified | Choose the most natural format for the target |
| No retention policy | Default to platform standards |
| No access control | Default to single-user, no auth |
| No integration details | Use the simplest available option (free APIs, no keys) |

The author can override any of these by adding a sentence. The LLM steps back wherever the author steps in.

### The Decision Transparency Principle

When the LLM fills a gap, the platform should be able to surface that decision:

```
ℹ️ Decisions made by the platform:
  • Persistence: Using browser localStorage (no retention policy specified)
  • Categories: Generated 8 default categories for expense tracking
  • Error handling: Standard retry (3 attempts) on external calls
  • UI layout: Single-page with form + list view

  Add specificity to any of these to override.
```

## 7. The Author Never Worries About Technical Infrastructure

The author's experience is **entirely in the problem domain.** They never encounter, configure, or troubleshoot: programming languages, language versions or runtimes, package managers or dependencies, framework configurations, database engines or connection strings, API keys or authentication flows, deployment pipelines, container orchestration, SSL certificates, environment variables, build tools, or version conflicts.

These are all platform concerns. The platform handles them invisibly. If the markup says "post a summary to Slack," the platform manages the Slack integration, credentials, webhooks, retry logic, and rate limiting. The author writes one sentence. The platform does the rest.

### The Infrastructure Boundary

```
Author's world           │  Platform's world
──────────────────────────┼───────────────────────────
"Track expenses"          │  React + localStorage
"Post to Slack"           │  Webhook + OAuth + retry
"Remember for 90 days"    │  Redis with TTL
"Show a chart"            │  Recharts component
"Score the fraud risk"    │  LLM inference call
"Check the CRM"           │  API call + caching
                          │
Natural language           │  Implementation details
Intent                     │  Technology
What                       │  How
```

The author lives entirely on the left side. The platform translates to the right side. The boundary is absolute — nothing from the right side leaks into the author's experience unless they specifically request it (Tenet 5).

### How the Tenets Interact

These seven principles form a coherent system:

1. **Always builds** means there's no minimum bar — start anywhere.
2. **Generic creates generic** means starting simple gives you something real.
3. **Specificity adds definition** means you refine toward exactly what you want.
4. **Language-agnostic** means the markup is about the problem, not the tech.
5. **Specificity can target** means technical authors aren't constrained.
6. **LLM fills gaps** means the author only specifies what they care about.
7. **No infrastructure** means the author stays in their domain.

Together: an author writes as much or as little as they want, in natural language, about what they want their application to do. It always works. It gets more precise as they add detail. They never touch infrastructure. And if they want to, they can specify technical targets — but they never have to.

This is the contract between the author and the platform.

---

# Design Philosophy

## The Core Insight

Every programming language is a lossy compression of intent. When a developer writes `for (var i = 0; i < items.length; i++)`, what they mean is "do this for each item." The syntax is an artifact of the machine needing exact instructions.

Markdown OS removes that constraint. The interpreter is an LLM, not a parser. The LLM doesn't need `foreach` — it needs to understand that you want iteration. The application is a document, not a program. The markup describes *intent, behavior, constraints, and domain knowledge.* The LLM is a universal interpreter that understands what the markup *means*.

## Applications as Documents

Traditional software is instructions for a dumb machine. Markdown OS applications are structured intent for an intelligent interpreter. The implications:

**No bugs in the traditional sense.** The application can't segfault, throw a null pointer exception, or have an off-by-one error. If the markup correctly describes the intent, the LLM executes it correctly. The failure mode shifts from "the code is wrong" to "the specification is ambiguous" — which is a much more tractable problem because ambiguity in natural language is something LLMs are specifically good at flagging.

**Automatic adaptation.** If Shopify changes their webhook payload format, a coded integration breaks. A markup-based application might just handle it, because the LLM understands what an order payload *is* even if the field names change. The markup says "extract the total" — it doesn't say `payload.order.total_price`. The LLM figures out where the total is.

**Auditable by non-engineers.** The customer can read their application's markup and understand exactly what it does. No code review required.

**Version control is document editing.** Diffing two versions of an application is diffing two documents. A customer can see "in v2 we added a rule that orders from repeat customers get a lower fraud score." That's readable without understanding any programming language.

## Post-Syntax Programming

The language has no syntax in the traditional sense. It has a **vocabulary of concepts** expressed in natural language. The construct "loop over these things" can be expressed as:

- "For each order in the batch, score it."
- "Process every incoming order."
- "Repeat the scoring step for all orders."
- "Take each order and run it through fraud analysis."
- "One by one, evaluate the orders."

All of these are the same construct. A traditional compiler would choke on all but one exact syntax. The LLM understands all of them identically.

This means the language is **syntax-free** (no wrong way to write it, only unclear ways), **naturally multilingual** (intents are universal; words are language-specific), **infinitely expressive** (new phrasings don't require language updates), **self-documenting** (the markup *is* the documentation), and **evolvable** (the language grows more powerful as LLMs improve).

## Intent as the Atomic Unit

In traditional languages, the atomic unit is a token: `if`, `for`, `return`, `=`. In Markdown OS, the atomic unit is an **intent**. An intent is a thing the application wants to accomplish, independent of the words used to express it.

The intent FILTER can be expressed as "Only process orders over $500," "Skip anything under the threshold," "Ignore low-value orders," "Exclude orders that don't meet the minimum," or "Weed out the small orders."

A human reading any of these understands the same thing. The LLM does too. **Intent is figuratively the keyword, and intent for the same thing can be written in many ways.**

## Specificity Is Optional

Because intents have default behaviors, the markup can be as specific or as vague as the author wants:

- **Vague:** "Retry if it fails." → Platform defaults (3 attempts, exponential backoff)
- **Moderate:** "Try up to 5 times with a short pause." → 5 attempts, linear backoff
- **Specific:** "Retry 3 times. Wait 1s, 4s, 16s. After third failure, log critical and move on." → Exact behavior specified

This is impossible in traditional languages. You can't write `retry if it fails` in C#. You have to specify everything. Markdown OS inverts this — **you specify only what matters, and intelligence fills in the rest.**

## Single Purpose

Every application does one thing. Not a company website. Not a generic API. A purpose-built application that exists because a specific problem needs solving. The persistence layer, API surface, validation rules, and operational interface all exist *because that purpose demands them*. Nothing more, nothing less.

If the customer needs a second thing, that's a second application. Applications compose (one's output is another's input) but each one stays single-purpose. This is the Unix philosophy taken to its AI-native conclusion.

## The Escape Hatch

The markup can be converted to real code: React, TypeScript, C#, Java, Python. This isn't a theoretical possibility — it's a first-class feature. The converter produces a complete, deployable project with tests, documentation, and containerization.

This eliminates vendor lock-in anxiety. The pitch isn't "trust us to run your application forever." It's "your application is a document. We run it as a service. But at any time, you can compile it to real code and take it anywhere."

Paradoxically, the exit door being open makes customers more likely to stay. The platform offers something the exported code doesn't: zero-infrastructure AI-native execution. They *can* leave, but why would they?

---

# The Markup Language

## Overview

The Markdown OS markup language is a natural language specification format for single-purpose applications. It is not a programming language in the traditional sense — it has no syntax, no keywords, and no grammar rules. It has an **intent taxonomy**: a set of semantic concepts that the language can express, each recognizable through a variety of natural language phrasings.

The language definition is a **reference document that the LLM compiler reads alongside the markup.** It tells the LLM: "When you encounter language that expresses iteration, here's how to treat it. When you encounter language that expresses a condition, here's the execution model."

## What the Markup Contains

**Identity** — What this application is, in one sentence. This constrains the LLM's behavior — anything outside this identity is out of scope.

**Domain Knowledge** — Facts, rules, and heuristics specific to this application's purpose. Best practices encoded as knowledge, not logic.

**Behavior Contracts** — What the application accepts, produces, and the transformation rules between them. Expressed in natural language with structured constraints.

**Integration Specs** — How to talk to external systems. Credential references (not credentials themselves), endpoint patterns, expected response shapes.

**Guardrails** — What the application must never do. Hard constraints the LLM respects.

**Persistence Schema** — What needs to be remembered and how. Declared as intent, not as database configuration.

## Loose Patterns, Not Strict Syntax

The language follows patterns but not strictly. C# has `foreach`, JavaScript has `for...of`, and this language has the concept of a loop — expressible in many ways that make sense with natural language.

The construct "loop over these things" is valid as: "For each order in the batch, score it." / "Process every incoming order." / "Repeat the scoring step for all orders." / "Take each order and run it through fraud analysis." / "One by one, evaluate the orders."

All are the same construct. The language doesn't require an exact keyword. The intent IS the keyword, and intent for the same thing can be written in many ways.

### Why Patterns Still Matter

Even without strict syntax, recognizable patterns serve three purposes:

1. **Disambiguation** — "Process the orders" (iteration) vs. "Process the order" (single action). Patterns help resolve ambiguity consistently.
2. **Composability** — The platform needs enough structure to understand "this section describes input handling" vs. "this section describes error behavior."
3. **Authoring assistance** — The platform needs conventions to generate readable markup. Patterns are a style guide, not a grammar.

## Document Structure

Applications follow a conventional (but not mandatory) structure using Markdown headings:

```markdown
# Application Name

## Purpose
Single-paragraph description of what this application does and why.

## When [trigger event]
What happens when the application receives input.

## [Domain-specific sections]
Processing logic, rules, transformations.

## [Display / Output sections] (if applicable)
What the user or downstream system sees.

## If something goes wrong
Error handling and resilience behavior.

## [Memory / Persistence sections]
What the application needs to remember.

## What this app does NOT do
Explicit boundaries and guardrails.
```

## The Language Evolves

Because intent is the fundamental unit and the LLM is the interpreter, the language evolves with the LLM. As models improve, the markup can become more expressive without changing any spec. A construct that needs explicit statement today ("retry up to 3 times with exponential backoff") might be implied tomorrow ("retry with standard backoff").

The language is also inherently multilingual. Intents are universal; words are language-specific. The markup can be written in English, French, Japanese, or Spanish. The constructs are the same — iteration, condition, sequence — but expressed in whatever language the author thinks in.

---

# Intent Taxonomy

Intents are the atomic units of the Markdown OS language. Every passage of markup resolves to one or more intents. The LLM compiler's job is to recognize meaning in natural language and map it to intents with parameters, then execute those intents through the platform's tool-calling interface.

## Intent Resolution

```
Markup text
    ↓ (LLM recognizes meaning)
Intent(s) identified
    ↓ (LLM fills in parameters)
Intent + parameters
    ↓ (Platform executes)
Side effects + output
```

A single sentence can express multiple intents. "Check the CRM for the customer. Repeat buyers with 5+ previous orders should be flagged as trusted." resolves to: `RECALL(CRM, customer) → BRANCH(order_count > 5) → UPDATE(customer, status=trusted)`.

## Data Flow Intents

**RECEIVE** — Accept input from an external source.
Default: Accept the payload as-is; validate only if VALIDATE follows.
Expressions: "when an order arrives," "accept the webhook," "on receiving a request," "when data comes in"

**EMIT** — Produce output to an external target.
Default: Fire-and-forget unless the markup specifies waiting for a response.
Expressions: "post to Slack," "send the summary," "notify the team," "output the result"

**TRANSFORM** — Reshape data from one form to another.
Default: Preserve all data unless the markup explicitly drops fields.
Expressions: "format as a Slack message," "convert to a summary," "reshape into," "build a report from"

**EXTRACT** — Pull specific pieces out of a larger structure.
Default: Return raw values; no formatting unless TRANSFORM follows.
Expressions: "grab the email from," "pull out the line items," "get the total," "isolate the relevant fields"

**COMBINE** — Merge multiple data sources into one.
Default: Include all fields from all sources; on conflict, later sources override.
Expressions: "merge the order with the CRM data," "combine these into one record," "enrich X with Y"

**FILTER** — Reduce a set based on criteria.
Default: Exclude items that don't match; preserve order.
Expressions: "only process orders over $500," "skip anything under the threshold," "ignore low-value," "keep only the"

**SORT** — Order a set by some dimension.
Default: Ascending unless the markup implies descending ("highest first," "most recent").
Expressions: "order by score," "newest first," "rank by priority," "sort from highest to lowest"

## Control Flow Intents

**BRANCH** — Do different things based on conditions.
Default: If no else/fallback is specified, do nothing when the condition isn't met.
Expressions: "if the score exceeds 70," "when the customer is new," "only when," "unless"

**ITERATE** — Repeat an action across a collection.
Default: Process in received order. If one item fails, continue to the next.
Expressions: "for each order," "process every," "repeat for all," "one by one," "go through the list and"

**SEQUENCE** — Do things in a specific order.
Default: Stop on failure unless RESILIENCE intents specify otherwise.
Expressions: "first, validate. then, enrich. finally, score." "after validation," "start by checking"

**PARALLEL** — Do multiple things simultaneously.
Default: Wait for all parallel operations to complete.
Expressions: "simultaneously look up the customer and validate," "at the same time," "in parallel"

**DELEGATE** — Hand work off to another application unit.
Default: Synchronous — wait for the delegated application to respond.
Expressions: "pass to the scoring unit," "send to the notification unit," "hand off to"

**WAIT** — Pause until a condition is met or time passes.
Default: Timeout after a platform-defined maximum.
Expressions: "wait for the response," "pause until confirmed," "hold for 5 minutes"

## State Intents

**REMEMBER** — Persist something for later.
Default: Upsert by default (overwrite if key exists). No expiration unless specified.
Expressions: "remember this order ID," "save the score," "record the timestamp," "keep track of," "log this event"

**RECALL** — Retrieve something previously persisted.
Default: Return null/empty if not found.
Expressions: "check whether we've seen this," "look up the customer," "retrieve the last score," "check our records"

**FORGET** — Remove persisted data.
Default: Soft delete (mark as removed) unless markup specifies permanent.
Expressions: "clear the cache," "remove the entry," "delete expired records," "purge old data"

**CHECK** — Query persisted state without modifying it.
Default: Return boolean (exists/doesn't exist) or count.
Expressions: "check if this order exists," "see whether we've processed," "count how many," "has this been seen before"

**UPDATE** — Modify existing persisted data.
Default: Fail silently if the record doesn't exist (unless markup says otherwise).
Expressions: "update the status to trusted," "increment the counter," "change the score to," "mark as processed"

## Resilience Intents

**RETRY** — Attempt something again on failure.
Default: 3 attempts, exponential backoff (1s, 2s, 4s).
Expressions: "retry if it fails," "try again a few times," "attempt up to 5 times"

**FALLBACK** — Use an alternative when primary fails.
Default: None — must be explicitly declared.
Expressions: "if the CRM is down, use a default profile," "fall back to," "as a backup"

**SKIP** — Continue past a failure without stopping.
Default: Log at warning level.
Expressions: "skip it and move on," "ignore the error and continue," "don't let one bad record stop everything"

**ALERT** — Signal that something noteworthy happened.
Default: Log at info level unless severity is specified.
Expressions: "flag this as unusual," "warn the team," "log a critical error"

**HALT** — Stop everything — something is critically wrong.
Default: Stop immediately; do not process remaining items.
Expressions: "stop everything," "abort," "this is critical — halt processing"

## Boundary Intents

**VALIDATE** — Confirm input meets expectations.
Default: Reject on the first validation failure.
Expressions: "validate that the payload contains," "make sure it has," "confirm the required fields"

**REJECT** — Refuse input that doesn't meet expectations.
Default: Return a descriptive error message.
Expressions: "reject the request," "refuse it," "send back an error"

**MASK** — Hide sensitive data in output.
Default: Show first 3 characters, mask the rest with asterisks.
Expressions: "mask the email," "hide the credit card number," "redact the SSN"

**LIMIT** — Cap the frequency or volume of something.
Default: Platform-defined rate limit.
Expressions: "no more than 100 requests per minute," "limit to 5 retries," "cap at"

**AUTHORIZE** — Verify the requester has permission.
Default: Reject unauthorized requests with a 401/403 equivalent.
Expressions: "only authorized users can," "require authentication," "check permissions"

## UI Intents (for React / frontend conversion targets)

**DISPLAY** — Render something visible.
Expressions: "show the details in a card," "display a table," "present the results," "render a chart"

**CAPTURE** — Accept user input.
Expressions: "let the user enter a threshold," "provide a dropdown," "accept a file upload"

**NAVIGATE** — Move between views.
Expressions: "when clicked, show details," "provide a way to go back," "the app has three sections"

**REACT** — Respond to user interaction.
Expressions: "when the user clicks rescore," "as the user types, filter," "on submit, save and clear"

**REFRESH** — Keep displayed data current.
Expressions: "update every 30 seconds," "show new orders as they arrive," "reload when the user returns"

## Intent Composition

Real markup rarely maps to a single intent. Sentences combine multiple intents:

| Markup | Intent Chain |
|---|---|
| "Check if we've seen this order. If so, skip it." | CHECK → BRANCH → SKIP |
| "Grab the email, look them up in the CRM, and flag repeat buyers as trusted." | EXTRACT → RECALL → BRANCH → UPDATE |
| "Post the summary to Slack. If Slack is down, retry 3 times, then queue it." | EMIT → RETRY → FALLBACK(queue) |
| "Show a table of today's orders, refreshing every minute." | RECALL → DISPLAY → REFRESH |

## Intent-Level Operations

Because intents are the execution unit, the platform provides:

- **Intent-level debugging** — trace shows `RECEIVE → VALIDATE → EXTRACT → RECALL(failed) → FALLBACK → TRANSFORM → EMIT`
- **Intent-level analytics** — "RETRY fires 400 times/day; FALLBACK at 12%"
- **Intent-level composition** — wire unit A's EMIT to unit B's RECEIVE
- **Intent-level testing** — "When duplicate arrives, SKIP should fire and no EMIT should occur"

---

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

---

# I/O Envelope

Every application unit communicates through a standard envelope — the boundary between the application (markup + LLM) and the platform (infrastructure + management).

## Envelope Structure

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

# Provider Abstraction

Markdown OS uses AI capabilities that exist across at least Claude and OpenAI. If both providers support it, it's fair game as a language primitive. This guarantees provider independence.

## Common Capability Surface

**Tool / Function Calling** — Both providers support structured tool definitions. This is the entire execution model.

**Structured Output / JSON Mode** — Both produce validated JSON. The response envelope is structured JSON.

**System Prompts** — Both support system-level instructions. The compiled markup *becomes* the system prompt.

**Multi-turn Context** — Both support conversation history. Used for multi-step workflows.

**Vision / Multimodal Input** — Both accept images. Applications processing visual input work without special handling.

**Streaming** — Both support SSE. Real-time applications use this transparently.

## Adapter Interface

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

## Provider Selection Strategy

The platform routes intelligently: cost optimization (simple requests to cheaper models), latency optimization (fastest responding provider), capability matching (vision-heavy to best vision model), and availability (failover between providers). The author can optionally express a preference but doesn't have to.

---

# The Converter

The converter transforms markup applications into deployable projects in traditional programming languages. The markup is the source of truth. The generated code is a projection — a different representation of the same application.

The converter is the escape hatch that eliminates vendor lock-in. It's also the proof that the markup is precise enough to be real software.

## Pipeline

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

## First Target: React

React is the first conversion target because it gives immediate visual proof of value (see the app running), the frontend IS the application for most early use cases, React's component model maps naturally to intents, and deployment is solved (Vercel, Netlify, static hosts).

### Intent → React Mapping

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

### Generated Project Structure

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

### Quality Bar

Generated React apps must feel like a senior developer built them: idiomatic React (hooks, functional components, composition), proper TypeScript (real types, Zod schemas, strict mode), accessible by default (ARIA labels, keyboard nav, semantic HTML), responsive by default (Tailwind utilities), and error states handled (loading spinners, empty states, error boundaries).

## Future Targets

1. **React** (first) — visual proof, client-side apps
2. **React + platform API** — frontend with managed state
3. **Other frameworks** — Vue, Svelte, Angular
4. **Backend targets** — Node/Express, .NET, Java Spring
5. **Mobile** — React Native from the same markup

## Round-Tripping

The converter works in both directions:

```
Markup → Intent Graph → Code    (export / eject)
Code → Intent Graph → Markup    (import / onboard)
```

An existing application can be imported to the platform. Export back to code later, and the code might be *better* than the original — the intent graph enforces clean separation of concerns. The markup is a purification step.

---

# Examples

## Example 1: Specificity Spectrum — Expense Tracker

Three versions of the same application demonstrating the "always builds" guarantee.

### Level 1: Minimal (2 lines)

```markdown
# Expense Tracker

Track personal expenses by amount and category.
```

This builds. The LLM decides everything: UI layout, categories, persistence, interactions.

### Level 2: Moderate

```markdown
# Expense Tracker

## Purpose

Track personal expenses with amount, category, date, and an
optional note. Show spending breakdown by category for the
current month.

## Adding an expense

Quick-entry form with amount (required), category from a
predefined list, and an optional short note. Date defaults
to today but can be changed.

## Viewing expenses

Show this month's expenses as a list, newest first. Include
a summary at the top with total spending and a breakdown by
category.

## Categories

Food, transport, entertainment, bills, health, savings, other.
The user can add custom categories.
```

More defined. The author shaped key behaviors. LLM still fills gaps (persistence strategy, error handling, mobile layout, etc.).

### Level 3: Detailed (with implementation target)

```markdown
# Expense Tracker

## Purpose

Track personal daily expenses. Designed for quick capture — the
user should be able to log an expense in under 5 seconds. Show
spending trends to build awareness, not to budget or restrict.

## Adding an expense

Always-visible form at the top of the screen. Three fields:

- Amount (required): numeric input, auto-focused on load. Accept
  decimals to two places. No currency symbol in the input — show
  it as a prefix label.
- Category (required): single-select from the list below. Most
  recently used category is pre-selected.
- Note (optional): single-line text, max 100 characters.
  Placeholder: "what was this for?"

On submit: save the expense, clear the amount and note, keep the
category selection, re-focus the amount field. The list below
should update immediately.

## Categories

Food & drink, transport, entertainment, bills & utilities, health,
clothing, gifts, education, other.

The user can add custom categories from a settings view. Custom
categories appear after the defaults. Limit to 20 total categories.

## The main view

Top section: summary card showing this month's total spending as a
large number, with a small comparison to last month ("12% more than
last month" or "8% less"). Below the number, show a horizontal
stacked bar representing the category breakdown.

Middle section: the quick-entry form (described above).

Bottom section: expense list for the current month, newest first.
Each row shows: amount (bold), category (color-coded chip), note
(if present, muted text), and relative time ("2 hours ago",
"yesterday"). Swipe left to delete on mobile. Click to edit on
desktop.

## Persistence

Remember all expenses with no expiration. Remember the user's
custom categories. Remember which category was used most recently
for pre-selection.

## What this app does NOT do

- No budgeting, goals, or spending limits
- No bank account or card integration
- No receipt scanning (just manual entry)
- No multi-currency support
- No data export (keep it simple)

## Implementation

Build this as a React application with TypeScript and Tailwind.
Use localStorage for persistence.
```

Precisely specified. The author controls most decisions. The `## Implementation` section pins the target language (Tenet 5). Without that section, the identical application logic would compile to any target.

## Example 2: Fraud Monitor — Backend Application

```markdown
# High Value Order Monitor

## Purpose

Monitor a Shopify store for orders exceeding a configurable dollar
threshold. Enrich each qualifying order with customer history from
the CRM, score it for fraud risk, and post a structured summary to
Slack. This is a fraud-scoring monitor, not an order management tool.

## When an order arrives

Validate that the payload contains an order ID, customer email,
line items, and a total amount. If any of these are missing, reject
the request and log a validation error.

Check whether we've already processed this order ID. If we have,
skip it silently — this is a duplicate.

Only process orders where the total exceeds the configured threshold,
which defaults to $500.

## Scoring

For the validated order, evaluate these fraud signals:

- The customer has never ordered before — this is a moderate signal.
- The shipping address doesn't match the billing address — moderate.
- There were multiple failed payment attempts — strong signal.
- The order was placed between 1 AM and 4 AM local time — weak signal.

Start with a base score of 10. Add 15 points for each weak signal,
20 for moderate, and 25 for strong. Cap the total at 100.

## After scoring

Record the order ID, the score, and the current timestamp in the
processed orders log. Keep this information for 90 days.

Format a Slack message that includes:
- The order number and total (but not payment details)
- The customer's name (mask the email — first 3 characters only)
- The fraud score with a color indicator: green below 40, yellow
  40-70, red above 70
- A list of which signals were triggered

Post this to the configured Slack channel.

If the score is above 90, also post to #security-alerts with an
urgent flag.

## If something goes wrong

If the Slack call fails, hold the message and retry up to 3 times
with increasing delays. After 3 failures, log a critical error.

If scoring fails for any reason, assign a score of 50 (uncertain)
and note in the Slack message that scoring was incomplete.

Never crash on a single bad order. Log it and continue.

## What this app does NOT do

- Does not modify orders in Shopify.
- Does not interact with customers directly.
- Does not make approval or rejection decisions.
- Does not store raw payment information.
```

This markup contains no implementation target. The same document runs on the platform via LLM, converts to a React dashboard, converts to a Node.js/Express webhook handler, or converts to a C# Azure Function. The intents are the same regardless of target.

## Example 3: Weather Wardrobe — Full Lifecycle (Markup + React Conversion)

### The Markup

```markdown
# Weather Wardrobe

## Purpose

Help someone decide what to wear today. Given their location, check
the current weather and today's forecast, then recommend clothing
that makes sense for the conditions — not just temperature, but
precipitation, wind, humidity, and how the day will change from
morning to evening.

This is not a weather app. This is a wardrobe advisor that happens
to use weather data.

## When the user opens the app

Ask for their location. If they've used the app before, remember
their last location and offer it as a default. Accept a city name
or zip code.

Once we have a location, fetch the current conditions and today's
hourly forecast.

## Understanding the weather

From the forecast, extract these key factors:

- **Temperature range**: the low and high for today, and the current temp.
- **Precipitation**: is rain, snow, or sleet expected? What probability? When?
- **Wind**: is it notably windy? Sustained speeds above 15mph matter.
- **Humidity**: is it muggy (above 70% with temp above 75°F) or dry?
- **UV index**: is sun protection needed (index above 5)?
- **Temperature swing**: does the day start cold and get warm, or vice versa?

These factors combine into a **comfort profile** — not just "it's
55°F" but "it's cool now, warming to mild by afternoon, with a
chance of rain after 3pm and moderate wind all day."

## Building the recommendation

Based on the comfort profile, recommend clothing across these
categories: base layer, mid layer, outer layer, bottom, footwear,
accessories.

Don't just list items. Explain *why* each recommendation makes
sense. "Light jacket — it's mild now but drops to 52°F by evening"
is better than just "light jacket."

## Handling tricky conditions

When the temperature swings more than 20°F through the day,
recommend layers and explicitly say "dress in layers."

When rain is possible but not certain (30-60%), recommend keeping
an umbrella handy but don't make rain gear the focus. Above 60%,
lead with rain protection.

When conditions are severe — extreme cold, heat advisory,
thunderstorms — lead with a safety note before any clothing advice.

## The display

Show the comfort profile visually — a human-readable summary.
Show clothing recommendations grouped by category with reasoning.
Show an hour-by-hour mini forecast for context.
Include a simple indicator for outfit complexity: "simple day,"
"plan ahead," or "stay flexible."

## Remembering preferences

Remember the user's location. If the user dismisses a recommendation
type consistently, deprioritize it — move it to an "also consider"
section. Remember the last 7 days of recommendations.

## What this app does NOT do

- No extended forecast beyond today.
- No shopping links or product recommendations.
- No social features.
- No outfit photos or style advice.
```

### The Intent Trace (Markup → React)

Every section of the markup resolved to concrete React constructs:

- "Ask for their location. Remember their last location" → CAPTURE + RECALL → `LocationInput` component + `localStorage.getItem("location")`
- "Fetch the current conditions" → RECEIVE → `fetchWeather()` calling Open-Meteo's free API (LLM chose the integration: free, no API key needed)
- "For each factor, evaluate these comfort signals" → ITERATE + BRANCH → `buildRecommendations()` walks through weather factors
- "If they dismiss a recommendation type, deprioritize it" → REMEMBER + FILTER → `dismissedCategories` state backed by localStorage, splitting recommendations into primary vs. "also consider"
- "Remember the last 7 days" → REMEMBER with retention → `memory.append()` with 7-day trim
- "Not a weather app. A wardrobe advisor." → BOUNDARY (identity) → UI leads with clothing recommendations; hourly strip is secondary

The markup file is ~80 lines of plain English. The working React app is ~450 lines of idiomatic code. A non-technical person can read the markdown, propose changes, and the converter regenerates the app.

The working React conversion is included in the repository at `examples/weather-wardrobe/weather-wardrobe.jsx`.

---

# Repository Structure

```
markdown-os/
├── README.md                           ← This document
├── docs/
│   ├── ARCHITECTURE.md                 ← Platform layers, compilation pipeline, tool-calling model
│   ├── PHILOSOPHY.md                   ← Post-syntax programming, intent-as-keyword, zero-infra
│   └── PROVIDER-ABSTRACTION.md         ← Claude/OpenAI common surface, adapter interface
├── spec/
│   ├── CORE-TENETS.md                  ← The 7 non-negotiable axioms (with examples and acid tests)
│   ├── LANGUAGE.md                     ← Language spec, document structure, pattern conventions
│   ├── INTENTS.md                      ← Full intent taxonomy (30+ intents across 6 categories)
│   ├── PERSISTENCE.md                  ← Memory profiles, storage primitives, context management
│   ├── IO-ENVELOPE.md                  ← Inbound/outbound envelope, management endpoints
│   └── CONVERTER.md                    ← Markup-to-code pipeline, React target, round-tripping
└── examples/
    ├── specificity-spectrum/           ← Same app at 3 detail levels (demonstrates "always builds")
    │   ├── README.md
    │   ├── level-1-minimal.md          ← 2 lines → working app
    │   ├── level-2-moderate.md         ← Moderate detail → more defined app
    │   └── level-3-detailed.md         ← Full spec with implementation target
    ├── fraud-monitor/
    │   └── fraud-monitor.md            ← Shopify fraud scoring (markup only, no target pinned)
    └── weather-wardrobe/
        ├── weather-wardrobe.md         ← Weather clothing advisor (markup source)
        └── weather-wardrobe.jsx        ← Converted React application (compiled output)
```

---

# Key Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Application format | Markdown / natural language | Readable by humans, interpretable by LLMs |
| Execution model | LLM as runtime via tool calling | Both Claude and OpenAI support this natively |
| Persistence | Platform-managed, intent-derived | Zero config for the author; storage shape emerges from purpose |
| First conversion target | React | Immediate visual proof of value; deployable to Vercel/Netlify |
| Language constructs | Intent-based, not keyword-based | Natural language is infinitely flexible; intents are the atoms |
| Provider strategy | Common capabilities across Claude + OpenAI | No vendor lock-in at the AI layer |
| Minimum viable markup | A purpose statement (one sentence) | The application always builds; complexity is never a prerequisite |
| Implementation agnosticism | Markup never implies a target language | Same markup → React, C#, Python, or platform-native execution |
| Gap-filling strategy | LLM makes opinionated best-practice decisions | Silence in the markup is a feature, not an error |

---

# Status

This project is in the design and specification phase. The documents in this repository capture the architecture, language design, intent taxonomy, and core tenets that define the platform.

**Source:** [github.com/cr-nattress/markdown-os](https://github.com/cr-nattress/markdown-os)

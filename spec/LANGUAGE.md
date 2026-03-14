# Markdown OS Language Specification

## Applications as documents. LLMs as the runtime.

This document defines the Markdown OS language — a system with two complementary layers of expression:

**Markup** — natural language documents that describe applications as structured intent. Readable by anyone. The authoring medium for non-technical users and the specification format for all applications.

**Promptdown** — a Turing-complete structured syntax where the `<-` operator is an LLM call instead of a CPU instruction. Every other construct (variables, loops, conditionals, functions) maps 1:1 to what you'd find in any traditional language. The executable layer for precise control flow, composition, and AI-native capabilities.

Both layers resolve to the same **intent taxonomy**. Markup expresses intents through natural language; Promptdown expresses them through explicit operators. The LLM is the compiler for both.

---

# Table of Contents

- [Design Philosophy](#design-philosophy)
- [The Markup Language](#the-markup-language)
- [Promptdown](#promptdown)
- [The Compiler](#the-compiler)
- [Testing](#testing)
- [Repositories](#repositories)
- [Intent Taxonomy](#intent-taxonomy)
- [Layer Mapping](#layer-mapping)

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

## Two Layers of the Same Language

Markdown OS provides two ways to express the same intents:

**Markup** is the high-level layer — natural language documents that describe applications. Any human can read and write them. The LLM fills gaps, makes decisions, and executes behavior. This is the authoring experience for most users.

**Promptdown** is the low-level layer — a structured syntax where `<-` invokes the LLM and everything else routes text into and out of that one operation. This is the precision tool for composable pipelines, reusable templates, and explicit control flow.

The layers are not alternatives — they're a spectrum. A single application can mix free-form markup sections with Promptdown blocks. The platform understands both because both resolve to the same intent taxonomy.

```
Markup (natural language)     Promptdown (structured syntax)
─────────────────────────     ──────────────────────────────
"Summarize the report"        $summary <- "Summarize: {{$report}}"
"For each order, score it"    FOR $order IN $orders:
                                $score <- "Score this: {{$order}}"
                              END
"If high risk, alert"         IF {{$score}} CONTAINS "high":
                                PRINT "ALERT: {{$order}}"
                              END
```

## Post-Syntax Programming

The markup layer has no syntax in the traditional sense. It has a **vocabulary of concepts** expressed in natural language. The construct "loop over these things" can be expressed as:

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

**Dependencies** — What external repositories, libraries, or shared knowledge this application draws from. Declared as references to git repos, not as installed packages.

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

## Uses
Repositories, shared knowledge, and reusable capabilities this app depends on.

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

# Promptdown

## The `<-` Operator

Promptdown is a Turing-complete language with one essential insight: **`<-` is an LLM call instead of a CPU instruction.** Just like `+` adds numbers, `<-` invokes an LLM. Everything else in the language exists to route text into and out of that one operation.

```promptdown
$result <- "Analyze this customer review for sentiment: {{$review}}"
```

The right side is a prompt. The left side captures the LLM's response. That's the entire execution model. Every Promptdown program is ultimately a composition of `<-` calls wired together with variables, control flow, and templates.

Turing completeness comes for free — any function computable in natural language is expressible in Promptdown. The LLM call handles what a CPU instruction set handles in traditional languages. The tradeoff is that "computation" is probabilistic, not deterministic.

## Language Primitives

| Construct | Syntax | Notes |
|---|---|---|
| Variable | `$name = "value"` | Assignment without LLM invocation |
| LLM call | `$result <- "prompt text"` | The core operator — invokes the LLM |
| Data type | Everything is text | Or a list of text |
| Interpolation | `{{$variable}}` | Embed variables inside prompts |
| Output | `PRINT $var` | Emit a value to stdout |
| Comment | `# text` | Ignored by the runtime |

### Variables

Variables hold text. They're set either by direct assignment or by capturing an LLM response:

```promptdown
# Direct assignment — no LLM call
$threshold = "500"
$company = "Acme Corp"

# LLM call — the response becomes the variable's value
$summary <- "Summarize this in one sentence: {{$document}}"

# Interpolation — variables expand inside prompts
$analysis <- "Given that the threshold is {{$threshold}}, evaluate: {{$summary}}"
```

### PRINT

`PRINT` is the output statement. It renders a variable's value to the application's output stream:

```promptdown
$greeting <- "Write a haiku about {{$topic}}"
PRINT $greeting
```

## Control Flow

### IF / ELSE / END

Conditional branching based on text matching. Because all values are text, conditions use natural language predicates:

```promptdown
$sentiment <- "What is the sentiment of: {{$review}}"

IF {{$sentiment}} CONTAINS "negative":
  $response <- "Write an empathetic reply to: {{$review}}"
  PRINT $response
ELSE:
  PRINT "No action needed."
END
```

### FOR / END

Iteration over a list:

```promptdown
$items = ["order-101", "order-102", "order-103"]

FOR $item IN $items:
  $score <- "Score this order for fraud risk: {{$item}}"
  PRINT $score
END
```

## Templates

Templates are prompt functions — reusable prompts with typed parameters. The same thing a function does for code, a template does for natural language reasoning.

```promptdown
TEMPLATE score_fraud($order, $history):
  $risk <- "Given this order: {{$order}} and customer history: {{$history}},
            score the fraud risk from 0-100. Return only the number."
  RETURN $risk
END

# Call a template
$fraud_score <- CALL score_fraud($current_order, $customer_history)
```

Templates enable:
- **Reuse** — define a prompt pattern once, call it many times
- **Composition** — templates call other templates
- **Testing** — assert on a template's output given known inputs
- **Versioning** — update the prompt in one place

## Pipelines

The most natural Promptdown programs look like Unix pipes — sequential `<-` calls where each step narrows and refines:

```promptdown
# Extract → Transform → Format
$raw <- "Extract all financial figures from: {{$document}}"
$clean <- "Normalize these figures to USD: {{$raw}}"
$final <- "Format as a markdown table with columns Amount, Currency, Context: {{$clean}}"
PRINT $final
```

Each step feeds the next. The pipeline idiom is Promptdown's equivalent of `cat file | grep pattern | sort | uniq` — small, composable transformations chained together.

## Skills

Skills are pre-wired prompt patterns that turn common AI tasks into single-operator calls. They call the LLM with carefully engineered system prompts under the hood.

```promptdown
# Built-in skills — each is a @skill(args) call
$summary    <- @summarize($document)
$translated <- @translate($text, "Spanish")
$mood       <- @sentiment($review)
$category   <- @classify($ticket, "bug, feature, question, billing")
$improved   <- @rewrite($draft, "more concise and professional")
$entities   <- @extract($paragraph, "names, dates, amounts")
$answer     <- @qa($question, $context)
$diff       <- @compare($version_a, $version_b)
```

### Built-in Skill Catalog

| Skill | Purpose | Parameters |
|---|---|---|
| `@summarize` | Condense text to key points | `(text)` or `(text, length)` |
| `@translate` | Convert between languages | `(text, target_language)` |
| `@sentiment` | Analyze emotional tone | `(text)` |
| `@classify` | Categorize into labels | `(text, categories)` |
| `@rewrite` | Transform style/tone | `(text, instruction)` |
| `@extract` | Pull structured data from text | `(text, fields)` |
| `@qa` | Answer questions given context | `(question, context)` |
| `@compare` | Identify differences | `(text_a, text_b)` |

Skills are composable — chain them like Unix pipes:

```promptdown
$raw <- @extract($email, "action items, deadlines, owners")
$translated <- @translate($raw, "Japanese")
$summary <- @summarize($translated)
PRINT $summary
```

### Custom Skills

Define your own skills as templates with the `SKILL` keyword:

```promptdown
SKILL tone_check($text):
  $tone <- "Analyze the professional tone of this text.
            Flag anything that could be perceived as passive-aggressive,
            dismissive, or overly casual in a business context: {{$text}}"
  RETURN $tone
END

$feedback <- @tone_check($draft_email)
```

## Tools

Tools are external integrations — capabilities that reach beyond the LLM into the outside world.

| Tool | Capability | Mechanism |
|---|---|---|
| `SEARCH` | Live web search | Fires a real web search via the API's web_search tool |
| `CODE` | Code interpretation | Routes through the LLM acting as an interpreter |
| `MATH` | Mathematical computation | Routes through the LLM acting as a solver |
| `FETCH` | HTTP requests | Retrieves content from URLs |

```promptdown
# Web search
$results <- SEARCH "latest FDA regulations on AI medical devices 2026"

# Code execution
$parsed <- CODE "parse this CSV and return the top 5 rows by revenue: {{$csv_data}}"

# Math
$projection <- MATH "compound interest on $50000 at 4.5% over 10 years"

# HTTP fetch
$api_data <- FETCH "https://api.example.com/orders/recent"
```

Tools compose naturally with skills and the `<-` operator:

```promptdown
$research <- SEARCH "competitor pricing for {{$product_category}}"
$analysis <- @compare($our_pricing, $research)
$recommendations <- "Given this competitive analysis: {{$analysis}},
                     recommend 3 pricing adjustments"
PRINT $recommendations
```

## CLI

CLIs are wrappers over functionality that is not provided by the language or the LLM. Any command-line tool installed on the platform becomes callable from Promptdown. CLIs bridge the gap between the LLM's natural language capabilities and the deterministic, specialized operations that traditional tools excel at.

```promptdown
# Run any CLI command
$result <- CLI "ffmpeg -i {{$video_path}} -ss 00:01:00 -t 10 -c copy clip.mp4"

# Use CLI output as input to an LLM call
$files <- CLI "ls -la /data/uploads"
$summary <- "Summarize what's in this directory listing: {{$files}}"

# Git operations
$log <- CLI "git log --oneline -20"
$analysis <- "What patterns do you see in these recent commits? {{$log}}"

# Data processing with traditional tools
$csv <- CLI "csvtool col 1,3,5 {{$data_file}}"
$insights <- @summarize($csv)
```

### Why CLIs Matter

The LLM is exceptional at reasoning, language, and transformation. But it can't transcode video, compress files, run database migrations, or execute `curl` against an authenticated endpoint. CLIs fill that gap — they provide **deterministic, specialized operations** as a first-class language construct.

| Use Case | CLI Command | Why Not LLM? |
|---|---|---|
| Image processing | `imagemagick convert ...` | LLM can't manipulate binary data |
| File compression | `tar -czf archive.tar.gz ...` | Deterministic byte-level operation |
| Database admin | `pg_dump`, `redis-cli` | Direct protocol access required |
| Network operations | `curl`, `wget`, `ssh` | Raw HTTP/network calls |
| Build tools | `npm run build`, `cargo build` | Compilation is deterministic |
| System inspection | `df -h`, `ps aux`, `top` | Real-time system state |

### CLI in Markup

In the markup layer, CLI usage is expressed as natural language intent:

```markdown
## Processing uploaded videos

When a video is uploaded, extract a 10-second preview clip
starting at the 1-minute mark. Use ffmpeg for the extraction.
Then generate a thumbnail from the first frame of the clip.
```

The platform resolves "use ffmpeg" to the appropriate CLI invocation. The author describes what they want; the platform wires the tool.

### CLI Safety

CLI commands run in a sandboxed environment. The platform enforces:
- **Allowlisting** — only approved CLIs are available (configured per application)
- **Resource limits** — CPU, memory, and time caps on CLI execution
- **No network egress by default** — CLI commands can't phone home unless explicitly permitted
- **Audit logging** — every CLI invocation is logged with command, arguments, exit code, and output

## Secrets

The language has a built-in secret store for managing credentials that CLIs, integrations, and AI capabilities require. Authors declare *what* they need access to; the platform manages *how* credentials are stored, rotated, and injected.

### The Problem

CLIs need API keys. Integrations need OAuth tokens. AI capabilities need provider credentials. Database connections need passwords. In traditional development, this means `.env` files, vault configurations, key management services, and environment variable wrangling. In Markdown OS, secrets are a platform primitive.

### Markup Layer

In markup, secrets are declared as access needs — never as actual values:

```markdown
## Integrations

This app posts summaries to Slack — use our team's Slack workspace.
It reads customer data from the Salesforce CRM.
Video processing requires an ffmpeg installation with codec licenses.

## Authentication

The Shopify webhook requires our store's API credentials.
Slack posting needs a bot token with chat:write scope.
```

The platform reads these declarations and provisions the necessary credentials from the secret store. The author never sees a token, key, or password.

### Promptdown Layer

In Promptdown, secrets are accessed with the `SECRET` construct:

```promptdown
# Retrieve a secret — the value is never logged or printed
$api_key <- SECRET "shopify_api_key"
$slack_token <- SECRET "slack_bot_token"
$db_url <- SECRET "postgres_connection_string"

# Secrets are injected into CLI commands securely
$result <- CLI "curl -H 'Authorization: Bearer {{SECRET \"api_token\"}}' https://api.example.com/data"

# Secrets work with tools
$data <- FETCH "https://api.shopify.com/orders.json" WITH SECRET "shopify_api_key"
```

### Secret Operations

| Operation | Syntax | Description |
|---|---|---|
| Read | `SECRET "key"` | Retrieve a secret value |
| Exists check | `HAS_SECRET "key"` | Returns true/false without exposing the value |
| Scoped access | `SECRET "stripe/live_key"` | Namespaced secrets (service/key) |
| Dynamic creation | `SET_SECRET "key" FROM $value` | Platform stores securely (admin only) |

### Secret Store Guarantees

- **Never in context** — secret values are never included in LLM prompts or context windows. They are injected at the platform execution layer, after the LLM has declared what to do.
- **Never in logs** — secret values are redacted from all logs, traces, and debug output. The log shows `SECRET "shopify_api_key"` not the actual key.
- **Never in markup** — hardcoding a secret in markup or Promptdown is a lint error. The platform rejects it.
- **Rotation-aware** — when a secret is rotated in the store, all applications using it pick up the new value immediately. No redeploy needed.
- **Scoped access** — each application can only access secrets it has been granted. An expense tracker can't read the fraud monitor's Shopify key.
- **Audit trail** — every secret access is logged: which application, which secret (by name, not value), when, and why (the intent chain that triggered it).

### Connecting Services

The secret store enables dynamic connections to external services. When the markup says "post to Slack" or "check the CRM," the platform:

1. Identifies the integration (Slack, Salesforce, etc.)
2. Looks up the required credentials in the secret store
3. Injects them at execution time
4. Handles token refresh, rotation, and expiry transparently

```promptdown
# The author writes this:
$orders <- FETCH "https://api.shopify.com/orders.json" WITH SECRET "shopify_api_key"

# The platform executes this (the author never sees it):
# 1. Reads shopify_api_key from encrypted secret store
# 2. Injects as Authorization header
# 3. Makes the HTTP call
# 4. Returns the response
# 5. Logs: FETCH shopify orders (200 OK, 340ms) — no key in log
```

### CLI + Secrets Together

CLIs commonly need credentials. The secret store integrates natively:

```promptdown
# Database operations with credentials from the store
$backup <- CLI "pg_dump {{SECRET \"db_connection_string\"}} > backup.sql"

# Authenticated API calls via curl
$response <- CLI "curl -s -H 'X-API-Key: {{SECRET \"service_api_key\"}}' https://api.internal/report"

# Cloud CLI tools
$deploy <- CLI "aws s3 cp report.pdf s3://bucket/ --region us-east-1"
# AWS credentials are injected from the secret store as environment variables
```

## PARALLEL

Wraps any set of assignments and runs them as concurrent operations. Independent branches execute simultaneously and merge back when all complete.

```promptdown
PARALLEL:
  $sentiment <- @sentiment($review)
  $entities <- @extract($review, "product, issue, urgency")
  $similar <- SEARCH "similar complaints about {{$product}}"
END

# All three variables are now available — resolved concurrently
$response <- "Given sentiment: {{$sentiment}}, entities: {{$entities}},
              and similar cases: {{$similar}}, draft a customer response"
PRINT $response
```

A 3-branch parallel block that would take ~9 seconds sequentially takes ~3 seconds with PARALLEL. The execution trace shows all branches resolving independently before variables merge back.

## AGENT

Defines a named sub-program with typed parameters. `RUN` spins up an isolated scope, executes the body, and `RETURN`s a value. Agents are the unit of delegation — self-contained programs within programs.

```promptdown
AGENT research_competitor($company):
  $overview <- SEARCH "{{$company}} product offerings 2026"
  $pricing <- SEARCH "{{$company}} pricing plans"
  $reviews <- SEARCH "{{$company}} customer reviews"
  $analysis <- "Synthesize a competitive brief from:
                Overview: {{$overview}}
                Pricing: {{$pricing}}
                Reviews: {{$reviews}}"
  RETURN $analysis
END

# Run the agent — isolated scope, returns a value
$competitor_brief <- RUN research_competitor("Acme Corp")
PRINT $competitor_brief
```

Agents enable:
- **Isolation** — each agent runs in its own scope; variables don't leak
- **Composition** — agents call other agents
- **Traceability** — the execution trace steps into the agent and shows its inner execution
- **Reuse** — define once, run with different parameters

## MEMORY

`REMEMBER` and `RECALL` give programs session-scoped persistence. Values survive across program segments within the same run.

```promptdown
# Persist a value
$user_preference <- "dark mode"
REMEMBER $user_preference AS "theme"

# ... later in the program or a subsequent segment ...

# Retrieve a persisted value
RECALL "theme" INTO $saved_theme
PRINT $saved_theme  # → "dark mode"
```

MEMORY maps directly to the State Intents in the intent taxonomy (REMEMBER, RECALL, FORGET, CHECK, UPDATE). It bridges the gap between Promptdown's explicit syntax and the platform's persistence layer.

```promptdown
# Check before processing (deduplication)
RECALL "processed_orders" INTO $seen
IF {{$seen}} CONTAINS "{{$order_id}}":
  PRINT "Duplicate — skipping"
ELSE:
  $score <- CALL score_fraud($order, $history)
  REMEMBER $order_id AS "processed_orders"
  PRINT $score
END
```

## Complete Example

A full Promptdown program demonstrating pipelines, skills, tools, parallel execution, and memory:

```promptdown
# Customer Support Triage Agent
# Reads an incoming ticket, researches context, and drafts a response

TEMPLATE classify_urgency($ticket):
  $urgency <- "Rate urgency 1-5 based on: customer tier, issue severity,
               time sensitivity. Ticket: {{$ticket}}. Return only the number."
  RETURN $urgency
END

AGENT handle_ticket($ticket_text):
  # Step 1: Parallel analysis
  PARALLEL:
    $category <- @classify($ticket_text, "billing, technical, account, feature-request")
    $sentiment <- @sentiment($ticket_text)
    $entities <- @extract($ticket_text, "customer_name, product, issue")
    $urgency <- CALL classify_urgency($ticket_text)
  END

  # Step 2: Check memory for repeat issues
  RECALL "recent_tickets" INTO $history

  # Step 3: Research if technical
  IF {{$category}} CONTAINS "technical":
    $docs <- SEARCH "{{$entities}} troubleshooting guide"
  ELSE:
    $docs = "N/A"
  END

  # Step 4: Draft response
  $response <- "Draft a support response given:
                Category: {{$category}}
                Sentiment: {{$sentiment}}
                Urgency: {{$urgency}}
                Extracted info: {{$entities}}
                Prior tickets from this customer: {{$history}}
                Relevant docs: {{$docs}}

                Be empathetic if sentiment is negative.
                Be concise if urgency is low."

  # Step 5: Remember for future context
  REMEMBER $ticket_text AS "recent_tickets"

  RETURN $response
END

# Entry point
$ticket = "My invoice is wrong again. This is the third time this month.
           I'm on your Enterprise plan and paying $50k/year —
           this level of service is unacceptable."

$reply <- RUN handle_ticket($ticket)
PRINT $reply
```

---

# The Compiler

## What the Compiler Is

The compiler is a markdown file. That's it. It is the language definition document — this specification — that describes how to interpret Markup and Promptdown constructs, what the intents mean, and how to execute them.

Compilation is passing the **code files** (the application's `.md` and `.pd` files) to an LLM **along with the compiler** (this language definition file). The LLM reads both, understands the application through the lens of the language spec, and produces the compiled output — whether that's direct execution, a system prompt, an optimized pipeline, or converted code.

```
Traditional compilation:
  source.c + gcc → binary

Markdown OS compilation:
  application.md + LANGUAGE.md + LLM → compiled application
```

The compiler is not a binary. It's not an installed tool. It's a document that instructs the LLM how to interpret another document. Any LLM that can read markdown can be the compilation runtime. The language definition *is* the compiler.

This means:
- **The compiler is portable** — copy the markdown file, and any LLM can compile your application
- **The compiler is readable** — a human can read the compiler and understand exactly how the language works
- **The compiler is versionable** — different versions of the language definition produce different compilation behavior
- **The compiler evolves with the language** — adding a new intent means adding a section to this document

## The Compiler Is Generative

Because the compiler is an LLM reading a language definition, it **understands what the program is trying to do** — not just its syntax, but its intent. This means it can do things no traditional compiler can:

- **Synthesize capabilities that don't exist yet** — if the program needs a skill that isn't defined, the compiler creates one
- **Recognize patterns and optimize** — if the program does the same thing 10 times linearly, the compiler introduces loops, recursion, or parallelism
- **Format and simplify** — verbose, messy, or redundant code is automatically reformatted into clean, efficient prompts
- **Choose the right abstraction** — the compiler decides whether something should be a skill, a template, an agent, or inline code

The compiler is not a passive translator. It is an **active capability creator** that produces the most efficient execution plan for the author's intent.

## Dynamic Skill Synthesis

When the compiler encounters an intent that no existing skill, template, or tool covers, it **creates one on the fly.** The synthesized skill is a first-class construct — it has a name, parameters, a prompt body, and can be reused.

### How It Works

```
Author writes:
  "For each customer complaint, identify the root product defect,
   suggest an engineering fix, and rate the fix complexity."

Compiler recognizes:
  1. No existing skill handles "identify root product defect"
  2. This is a repeating pattern (FOR EACH)
  3. The operation has clear inputs (complaint) and outputs (defect, fix, complexity)

Compiler synthesizes:
  SKILL @diagnose_defect($complaint):
    $defect <- "Identify the root product defect from: {{$complaint}}"
    $fix <- "Suggest an engineering fix for: {{$defect}}"
    $complexity <- @classify($fix, "trivial, moderate, complex, architectural")
    RETURN { defect: $defect, fix: $fix, complexity: $complexity }
  END

Compiler emits optimized code:
  FOR $complaint IN $complaints:
    $result <- @diagnose_defect($complaint)
    PRINT $result
  END
```

The author wrote 3 lines of natural language. The compiler produced a reusable skill with structured output, a classification sub-call, and a loop. The author never asked for any of this — the compiler recognized the need and built the right abstraction.

### When the Compiler Creates Capabilities

| Signal | What the Compiler Does |
|---|---|
| Repeated similar `<-` calls with varying inputs | Extracts a TEMPLATE or SKILL |
| Natural language describing a multi-step process | Synthesizes an AGENT with SEQUENCE |
| "For each X, do Y" in markup | Generates a SKILL + FOR loop |
| Independent operations on the same data | Wraps in PARALLEL |
| A pattern that matches an existing skill's shape | Routes to the existing skill instead of a raw `<-` |
| Recursive structure ("do this, then refine, then refine again") | Generates recursive TEMPLATE with termination condition |
| Error handling described generically | Synthesizes RETRY/FALLBACK/SKIP wrappers |

### Recursive Optimization

The compiler recognizes when a task benefits from iterative refinement and generates recursive structures:

```
Author writes:
  "Draft a press release. Review it for corporate tone issues.
   Fix any issues found. Repeat until clean."

Compiler synthesizes:
  TEMPLATE refine_press_release($draft, $max_iterations):
    $review <- @tone_check($draft)
    IF {{$review}} CONTAINS "no issues":
      RETURN $draft
    END
    IF $max_iterations <= 0:
      RETURN $draft
    END
    $improved <- @rewrite($draft, "fix these tone issues: {{$review}}")
    RETURN CALL refine_press_release($improved, $max_iterations - 1)
  END

  $first_draft <- "Write a press release about: {{$announcement}}"
  $final <- CALL refine_press_release($first_draft, 5)
  PRINT $final
```

The author said "repeat until clean." The compiler built bounded recursion with a termination condition, a refinement loop, and a maximum iteration guard. It chose recursion over a loop because each iteration's input depends on the previous iteration's output.

## Automatic Optimization

The compiler analyzes the entire program and optimizes it before execution. This isn't optional — every program is optimized as a compilation step.

### What the Compiler Optimizes

**Prompt consolidation** — Multiple sequential `<-` calls that could be a single, richer prompt are merged:

```
Before (author wrote):
  $name <- "Extract the customer name from: {{$email}}"
  $issue <- "Extract the main issue from: {{$email}}"
  $urgency <- "Rate the urgency of: {{$email}}"

After (compiler optimizes):
  $analysis <- "From this email, extract:
                1. Customer name
                2. Main issue
                3. Urgency rating (1-5)
                Format as structured output. Email: {{$email}}"
  # Single LLM call instead of three — same result, 1/3 the cost and latency
```

**Parallel detection** — Independent operations are automatically parallelized:

```
Before (author wrote sequentially):
  $sentiment <- @sentiment($review)
  $entities <- @extract($review, "product, issue")
  $category <- @classify($review, "bug, feature, complaint")

After (compiler wraps in PARALLEL):
  PARALLEL:
    $sentiment <- @sentiment($review)
    $entities <- @extract($review, "product, issue")
    $category <- @classify($review, "bug, feature, complaint")
  END
```

**Skill routing** — Raw `<-` calls that match an existing skill's semantics are routed through the skill instead:

```
Before (author wrote):
  $result <- "Summarize the following document in 3 bullet points: {{$doc}}"

After (compiler routes to skill):
  $result <- @summarize($doc, "3 bullet points")
  # Uses the skill's engineered system prompt — better results, consistent behavior
```

**Dead code elimination** — Variables that are computed but never used are dropped. LLM calls that have no downstream consumers are flagged as warnings.

**Memoization** — If the same `<-` call with the same input appears multiple times, the compiler caches the result:

```
Before:
  $score_a <- @sentiment($review)
  # ... other code ...
  $score_b <- @sentiment($review)  # same input as $score_a

After:
  $score_a <- @sentiment($review)
  $score_b = $score_a  # reuse — no duplicate LLM call
```

**Context windowing** — For programs that consume large inputs, the compiler splits data into chunks, processes in parallel, and aggregates:

```
Author writes:
  $analysis <- "Analyze this 200-page report for compliance issues: {{$report}}"

Compiler optimizes:
  $chunks <- SPLIT $report BY_PAGES 20
  PARALLEL:
    FOR $chunk IN $chunks:
      $partial <- "Analyze for compliance issues: {{$chunk}}"
    END
  END
  $analysis <- "Synthesize these partial analyses into a unified report: {{$partials}}"
```

## Automatic Formatting

The compiler reformats all code — author-written or synthesized — into clean, efficient form before execution. This applies to both Markup and Promptdown.

### What Formatting Does

**Prompt engineering** — Raw, casual prompts are reformatted into structured prompts with clear instructions, output format specifications, and constraint framing:

```
Before (author wrote):
  $result <- "look at this order and tell me if it seems fraudulent {{$order}}"

After (compiler reformats):
  $result <- "Evaluate this order for fraud indicators.

              Order data: {{$order}}

              Assess these signals:
              - New vs. returning customer
              - Address mismatch (shipping vs. billing)
              - Payment attempt patterns
              - Order timing anomalies

              Return: risk level (low/medium/high) and a one-sentence explanation."
```

The compiler understood the intent ("tell me if it seems fraudulent") and produced a structured prompt that will get a better, more consistent response from the LLM. The author's intent is preserved. The quality of the prompt is elevated.

**Code simplification** — Verbose or redundant Promptdown is simplified:

```
Before:
  $x <- "Extract name from: {{$data}}"
  $y = $x
  IF {{$y}} NOT EMPTY:
    $z = $y
    PRINT $z
  END

After:
  $name <- "Extract name from: {{$data}}"
  IF {{$name}} NOT EMPTY:
    PRINT $name
  END
```

**Intent deduplication** — When the same logical intent appears multiple times in different phrasings, the compiler normalizes:

```
Before (in markup):
  "Grab the customer's email address."
  ... (later in the same document) ...
  "Pull the email from the customer record."

After:
  Single EXTRACT intent, called once, result reused.
```

### Formatting Is Non-Destructive

The compiler never changes the author's **intent** — only the **form**. The optimization trace shows exactly what was changed and why:

```
Optimization trace:
  ✓ Consolidated 3 sequential extractions into 1 structured prompt (3x → 1x LLM calls)
  ✓ Parallelized 2 independent operations (estimated 2x speedup)
  ✓ Routed "summarize the document" to @summarize skill
  ✓ Reformatted 1 casual prompt into structured format
  ✓ Synthesized @diagnose_defect skill from repeated pattern
  ✓ Added recursion bound (max 5 iterations) to open-ended refinement loop
  ⚠ Dropped unused variable $temp_result (line 42)
```

The author can inspect every optimization, override any of them, or disable optimization for specific sections with `LITERAL:` blocks:

```promptdown
# This block is executed exactly as written — no compiler optimization
LITERAL:
  $result <- "look at this and tell me what you think: {{$data}}"
END
```

## Automatic Documentation

The compiler generates documentation for everything it touches — author-written code, synthesized capabilities, and optimization decisions. No code exists without explanation.

### Inline Comments

Every synthesized or optimized construct gets inline comments explaining what it does and why the compiler chose that structure:

```promptdown
# Synthesized by compiler: extracts defect info from complaints
# Created because lines 12-14 describe a repeating analysis pattern
# Uses @classify for complexity rating (routed from raw prompt)
SKILL @diagnose_defect($complaint):
  # Step 1: Identify the root cause from the complaint text
  $defect <- "Identify the root product defect from: {{$complaint}}"
  # Step 2: Generate an engineering fix based on the identified defect
  $fix <- "Suggest an engineering fix for: {{$defect}}"
  # Step 3: Rate fix complexity using classification skill
  $complexity <- @classify($fix, "trivial, moderate, complex, architectural")
  RETURN { defect: $defect, fix: $fix, complexity: $complexity }
END
```

### Companion Documentation

For complex programs, the compiler generates a companion markdown file that describes the program's structure, intent flow, synthesized capabilities, and optimization decisions:

```
app-name.pd          ← the program
app-name.docs.md     ← compiler-generated documentation
```

The companion file includes:
- **Program summary** — what the program does, derived from its intent graph
- **Capability catalog** — every skill, template, and agent (author-defined and synthesized)
- **Intent flow diagram** — the chain of intents from RECEIVE to EMIT
- **Optimization log** — what the compiler changed and why
- **Dependency map** — what repos, tools, CLIs, and secrets the program uses
- **Test coverage** — which constructs are tested and which need tests

### Documentation for Converted Code

When the converter produces traditional code (React, TypeScript, etc.), the compiler adds documentation to the generated code too:

- Every function gets a JSDoc/docstring explaining which intent it implements
- Every file gets a header comment linking back to the markup section that generated it
- The project README is generated from the markup's Purpose section
- Complex transformations get inline comments explaining the mapping from intent to code

Documentation is a compiler output, not an author burden.

## Capability Lifecycle

Dynamically synthesized capabilities aren't throwaway. The compiler manages their lifecycle:

1. **Synthesis** — the compiler creates the capability during compilation
2. **Testing** — synthesized capabilities are auto-tested before execution (the compiler generates test cases from the context that triggered synthesis)
3. **Caching** — a synthesized skill that works is cached and reused across requests
4. **Promotion** — if a synthesized skill is used frequently, the platform promotes it to a named, versioned skill that can be imported by other programs
5. **Drift detection** — synthesized skills are re-evaluated when the underlying model changes

```
Synthesis lifecycle:
  Compiler recognizes need → Synthesizes skill → Auto-generates tests →
  Tests pass → Skill cached → Skill used in execution →
  (Optional) Promoted to named skill → Available as dependency
```

### The Compiler as Collaborator

The compiler surfaces its decisions. After compilation, the author sees:

```
Compilation report:
  Capabilities synthesized:
    @diagnose_defect(complaint) — created from lines 12-14
    @refine_tone(draft, iterations) — created from line 23

  Optimizations applied:
    3 prompt consolidations (saved ~6 LLM calls)
    2 parallel wraps (estimated 4s faster)
    1 recursive structure with bound of 5

  Suggestions:
    "The @diagnose_defect skill could be extracted to a shared repo
     if other programs need similar defect analysis."
    "Consider adding a MOCK for @diagnose_defect to improve test coverage."
```

The author is always in control. The compiler proposes; the author disposes. But the default is smart — the compiler does the right thing unless told otherwise.

---

# Testing

## Nothing Ships Without Tests

Testing is not optional. It is not a phase. It is not something that happens after the application works. **Every template, skill, agent, and pipeline must have tests before it can be compiled, deployed, or published as a dependency.** The platform enforces this — untested constructs are rejected at the boundary.

This is a hard constraint because the execution model is probabilistic. Traditional code can be visually inspected — "this function adds two numbers, it's obviously correct." LLM-backed operations cannot. The only way to know that `$score <- "Score this order for fraud risk"` produces a useful result is to run it against known inputs and assert on the output. Testing compensates for the non-determinism that `<-` introduces.

```
Traditional language: code review can catch bugs visually
Markdown OS:          the LLM's behavior must be verified empirically

Therefore: no deploy without tests. No exceptions.
```

## Markup Layer

In the markup layer, tests are declared as expectations — natural language descriptions of what should happen given specific inputs.

```markdown
# Invoice Validator

## Purpose
Validate incoming invoices against accounting rules.

## Tests

When given a valid invoice with matching line items:
- The validator should approve it
- No alerts should fire

When given an invoice where line items don't sum to the total:
- The validator should reject it
- The rejection should mention the discrepancy amount

When given an invoice from a flagged vendor:
- The validator should approve it but flag it for review
- An alert should fire to the compliance channel

When given an empty payload:
- The validator should reject it with a clear error
- It should not crash or produce partial output
```

The platform reads these test sections and generates executable test cases. Each "When given..." block becomes a test. The LLM evaluates whether the application's output satisfies the natural language assertions. This is **semantic testing** — asserting on meaning, not exact string matches.

## Promptdown Layer

In Promptdown, tests use the `TEST` and `ASSERT` constructs. Tests are co-located with the code they verify — every `.pd` file with templates, skills, or agents should have corresponding tests.

### TEST / ASSERT / END

```promptdown
TEMPLATE score_fraud($order, $history):
  $risk <- "Given this order: {{$order}} and customer history: {{$history}},
            score the fraud risk from 0-100. Return only the number."
  RETURN $risk
END

# Tests for score_fraud
TEST "new customer with mismatched addresses scores high":
  $result <- CALL score_fraud(
    "order: $2000, shipping: 123 Main St, billing: 456 Oak Ave",
    "no previous orders"
  )
  ASSERT $result CONTAINS "7" OR "8" OR "9"
  ASSERT $result NOT CONTAINS "below" OR "low" OR "minimal"
END

TEST "repeat customer with matching addresses scores low":
  $result <- CALL score_fraud(
    "order: $50, shipping: 123 Main St, billing: 123 Main St",
    "47 previous orders, all fulfilled, no disputes"
  )
  ASSERT $result CONTAINS "1" OR "2" OR "3"
END

TEST "empty order input returns a score, does not crash":
  $result <- CALL score_fraud("", "unknown")
  ASSERT $result NOT EMPTY
END
```

### Assertion Types

Because `<-` produces text (not typed values), assertions operate on text semantics:

| Assertion | What It Checks |
|---|---|
| `ASSERT $var CONTAINS "text"` | Output includes the substring |
| `ASSERT $var NOT CONTAINS "text"` | Output excludes the substring |
| `ASSERT $var NOT EMPTY` | Output is non-empty |
| `ASSERT $var MATCHES /pattern/` | Output matches a regex |
| `ASSERT $var SATISFIES "criteria"` | LLM evaluates whether output meets natural language criteria |
| `ASSERT $var ONE_OF ["a", "b", "c"]` | Output is one of the listed values |

The `SATISFIES` assertion is the most powerful — it uses the LLM itself as the judge:

```promptdown
TEST "response is empathetic for angry customer":
  $response <- RUN handle_ticket("This is unacceptable! Fix it NOW!")
  ASSERT $response SATISFIES "tone is empathetic and acknowledges frustration"
  ASSERT $response SATISFIES "does not use defensive or dismissive language"
  ASSERT $response SATISFIES "includes a concrete next step"
END
```

### Testing Skills

Built-in skills have pre-defined test contracts. Custom skills must include tests:

```promptdown
SKILL tone_check($text):
  $tone <- "Analyze the professional tone of this text.
            Flag anything passive-aggressive, dismissive, or overly casual: {{$text}}"
  RETURN $tone
END

TEST "flags passive-aggressive language":
  $result <- @tone_check("Per my last email, as I already mentioned...")
  ASSERT $result CONTAINS "passive-aggressive"
END

TEST "approves neutral professional language":
  $result <- @tone_check("Thank you for your email. I'll review and respond by Friday.")
  ASSERT $result NOT CONTAINS "passive-aggressive"
  ASSERT $result NOT CONTAINS "dismissive"
END
```

### Testing Agents

Agent tests verify the full execution path — including internal state, tool calls, and the returned value:

```promptdown
AGENT research_competitor($company):
  $overview <- SEARCH "{{$company}} product offerings"
  $analysis <- "Synthesize a brief from: {{$overview}}"
  RETURN $analysis
END

TEST "produces a competitive brief with key sections":
  $brief <- RUN research_competitor("Stripe")
  ASSERT $brief SATISFIES "mentions Stripe's products or services"
  ASSERT $brief SATISFIES "is structured as a brief, not raw search results"
  ASSERT $brief NOT EMPTY
END

TEST "handles unknown company gracefully":
  $brief <- RUN research_competitor("Nonexistent Corp XYZ123")
  ASSERT $brief NOT EMPTY
  ASSERT $brief SATISFIES "acknowledges limited information available"
END
```

### Testing Pipelines

Pipelines are tested end-to-end. Given an input, assert on the final output:

```promptdown
TEST "full pipeline: extract, transform, format":
  $doc = "Revenue was $1.2M in Q1, $1.5M in Q2, and $900K in Q3."
  $raw <- @extract($doc, "quarterly revenue figures")
  $clean <- "Normalize to integers: {{$raw}}"
  $final <- "Format as markdown table: {{$clean}}"

  ASSERT $final CONTAINS "Q1"
  ASSERT $final CONTAINS "Q2"
  ASSERT $final CONTAINS "Q3"
  ASSERT $final MATCHES /\|.*\|/
END
```

### Testing with Dependencies

When a program uses repositories, tests can mock imports or run against the real dependency:

```promptdown
USE "github:acme/fraud-toolkit@v3" AS fraud
IMPORT score_order FROM fraud

# Test with real dependency
TEST "scoring integrates with fraud toolkit":
  $score <- CALL score_order("order: $5000, new customer, mismatched addresses")
  ASSERT $score NOT EMPTY
  ASSERT $score SATISFIES "indicates elevated risk"
END

# Test with mock (isolate from dependency)
MOCK score_order AS:
  RETURN "75 — high risk due to address mismatch"
END

TEST "pipeline handles high fraud score correctly":
  $score <- CALL score_order($test_order)
  ASSERT $score CONTAINS "75"
END
```

## Test Execution Model

Tests run against the same LLM that executes the application. This is important — the test environment matches the production environment exactly. There is no "works in test but fails in prod" gap.

### Non-Determinism Strategy

Because `<-` is probabilistic, the same test can produce different outputs across runs. The testing framework handles this:

1. **Semantic assertions over exact matches** — `SATISFIES "is empathetic"` succeeds regardless of specific wording
2. **Multiple runs** — the platform runs each test N times (configurable, default 3) and requires a pass rate threshold (configurable, default 100%)
3. **Structural assertions** — `MATCHES /pattern/` and `CONTAINS "text"` target the stable parts of output
4. **Snapshot testing** — capture a "golden" output and assert future runs are semantically similar (not identical)

### When Tests Run

| Event | What Runs |
|---|---|
| **Save / edit** | Tests for the changed template/skill/agent |
| **Before compile** | Full test suite — compilation is blocked on failure |
| **Before deploy** | Full test suite — deployment is blocked on failure |
| **Before publish** (as dependency) | Full test suite + exported interface tests |
| **On dependency update** | Tests for all consumers of the updated dependency |
| **Scheduled** | Full regression suite (catches model drift) |

### Model Drift Detection

LLM behavior changes over time as models are updated. Scheduled test runs detect **model drift** — cases where an application that passed tests last week now fails because the underlying model's behavior shifted. The platform alerts on drift and can pin to a specific model version if stability is required.

## The Testing Contract

Every publishable construct has a minimum testing requirement:

| Construct | Minimum Tests |
|---|---|
| TEMPLATE | At least one TEST per template |
| SKILL | At least one TEST covering the happy path and one for edge cases |
| AGENT | At least one TEST covering the full execution path |
| Pipeline (complete program) | At least one end-to-end TEST |
| Exported dependency | Tests for every EXPORT in the manifest |

**Untested constructs cannot be:**
- Compiled to a target language
- Deployed to the platform
- Published as a dependency (EXPORT without tests is rejected)
- Referenced by other applications

The platform enforces this at every boundary. There is no `--skip-tests` flag.

---

# Repositories

## Repos Are Packages

Traditional languages have package managers: npm, pip, cargo, NuGet. Markdown OS has git repositories. A repo is the universal unit of shared capability — it can contain knowledge, templates, skills, agents, data, code, or any combination. The platform consumes repos the way a build tool consumes dependencies.

```
Traditional                          Markdown OS
───────────────────────────────      ───────────────────────────────
npm install lodash                   USE "github:acme/scoring-rules"
import { merge } from 'lodash'      IMPORT score_order FROM scoring
pip install pandas                   USE "github:acme/data-utils@v2"
from pandas import DataFrame         IMPORT @clean_csv FROM data_utils
```

The key difference: in traditional languages, `import` gives you functions compiled to machine code. In Markdown OS, `USE` gives you **knowledge, templates, skills, agents, and context** — material the LLM can reason about, not just execute.

## What a Repo Provides

A single repository can expose multiple types of consumable content. The platform inspects the repo and builds a **manifest** of what's available:

| Content Type | What It Provides | How It's Consumed |
|---|---|---|
| **Markup documents** (`.md`) | Domain knowledge, rules, heuristics | Loaded into LLM context as reference material |
| **Promptdown files** (`.pd`) | Templates, skills, agents | Imported and called directly |
| **Data files** (`.csv`, `.json`, `.yaml`) | Structured datasets, configuration | Loaded as variables or queried on demand |
| **Code** (any language) | Implementation reference, APIs | LLM reads and reasons about the code |
| **Documentation** | API specs, guides, standards | Injected as domain knowledge for the LLM |

A repo doesn't need to be written *for* Markdown OS to be useful. Any GitHub repo with a README, docs, or source code becomes consumable knowledge. The LLM reads it the same way a developer would — it understands what the repo contains and how to use it.

## Markup Layer

In the markup layer, dependencies are declared as natural language references. The author describes what they need; the platform resolves where it comes from.

```markdown
# Invoice Validator

## Purpose
Validate incoming invoices against our company's accounting rules
and flag discrepancies.

## Uses
Use the accounting rules from our internal compliance repository.
Reference the Xero API documentation for invoice field mappings.
Use the shared fraud-scoring skills from the platform team's toolkit.

## When an invoice arrives
Validate it against the accounting rules. Check that the line items
match Xero's expected format. If anything looks unusual, run it
through the fraud scoring skills.
```

The author never specifies a git URL, a branch, or a version. They describe the dependency by purpose. The platform resolves it — matching "our internal compliance repository" to a configured repo, "Xero API documentation" to a public repo or fetched docs, and "the platform team's toolkit" to a shared internal repo.

For more precision, the author can be explicit:

```markdown
## Uses
Use acme/compliance-rules (the accounting standards from Q4 2025).
Use xero/xero-api-docs for invoice field definitions.
Use acme/shared-skills for fraud scoring templates.
```

## Promptdown Layer

In Promptdown, repositories are declared with `USE` and their contents are accessed with `IMPORT`.

### USE — Declare a Repository Dependency

```promptdown
# Full GitHub reference
USE "github:acme/scoring-rules" AS scoring

# Pinned to a tag (semver)
USE "github:acme/scoring-rules@v2.1.0" AS scoring

# Pinned to a branch
USE "github:acme/scoring-rules#production" AS scoring

# Pinned to a specific commit
USE "github:acme/scoring-rules@a1b2c3d" AS scoring

# Public repos work the same way
USE "github:shopify/shopify-api-node" AS shopify_docs

# Local path (for development)
USE "./shared/scoring-rules" AS scoring
```

`USE` tells the platform: fetch this repo, cache it, extract its manifest, and make its contents available under the given alias. Nothing is loaded into the LLM context yet — that happens on `IMPORT`.

### IMPORT — Bring Content Into Scope

```promptdown
USE "github:acme/fraud-toolkit@v3" AS fraud
USE "github:acme/shared-skills" AS lib
USE "github:acme/domain-knowledge" AS knowledge

# Import a template
IMPORT score_order FROM fraud

# Import a skill
IMPORT @clean_csv FROM lib

# Import an agent
IMPORT AGENT research_competitor FROM lib

# Import domain knowledge (loaded into LLM context)
IMPORT KNOWLEDGE "accounting-rules.md" FROM knowledge

# Import data
IMPORT DATA "thresholds.json" FROM fraud INTO $thresholds

# Import everything (entire manifest)
IMPORT * FROM lib
```

### Namespaced Access

Imported content is callable by name or via its namespace:

```promptdown
USE "github:acme/fraud-toolkit@v3" AS fraud
IMPORT score_order FROM fraud

# These are equivalent:
$score <- CALL score_order($order)
$score <- CALL fraud.score_order($order)

# Skills keep their @ prefix
IMPORT @sentiment FROM fraud
$mood <- @fraud.sentiment($review)
```

### Complete Example with Dependencies

```promptdown
USE "github:acme/fraud-toolkit@v3.1" AS fraud
USE "github:acme/crm-connectors" AS crm
USE "github:acme/compliance-rules#main" AS compliance

IMPORT score_order, classify_risk FROM fraud
IMPORT AGENT lookup_customer FROM crm
IMPORT KNOWLEDGE "pci-requirements.md" FROM compliance
IMPORT DATA "risk-thresholds.json" FROM fraud INTO $thresholds

AGENT process_order($order):
  # Look up customer using the CRM connector agent
  $customer <- RUN crm.lookup_customer($order)

  # Score using the fraud toolkit's template
  $score <- CALL score_order($order, $customer)

  # Classify using risk thresholds from the data file
  $risk <- CALL classify_risk($score, $thresholds)

  # The LLM has PCI compliance rules in context from the KNOWLEDGE import
  # so it can reason about compliance without explicit prompting
  $assessment <- "Given this order risk level: {{$risk}},
                  assess whether additional PCI verification is needed
                  for this transaction amount: {{$order}}"

  RETURN $assessment
END
```

## Repo Manifest

When the platform fetches a repository, it builds a manifest — a structured index of what the repo exposes. If the repo contains a `manifest.pd` file, the platform uses it. Otherwise, the platform auto-discovers content by scanning file types and structure.

### Explicit Manifest

A repo can declare its public surface with a `manifest.pd` file at its root:

```promptdown
# manifest.pd — Fraud Toolkit v3.1

EXPORT TEMPLATE score_order
EXPORT TEMPLATE classify_risk
EXPORT TEMPLATE explain_decision

EXPORT SKILL @fraud_sentiment
EXPORT SKILL @flag_anomaly

EXPORT AGENT deep_investigation

EXPORT KNOWLEDGE "docs/scoring-methodology.md"
EXPORT KNOWLEDGE "docs/risk-categories.md"

EXPORT DATA "data/thresholds.json"
EXPORT DATA "data/known-patterns.csv"
```

### Auto-Discovery

Without a manifest, the platform scans:

| Pattern | Discovered As |
|---|---|
| `*.pd` files with `TEMPLATE` definitions | Importable templates |
| `*.pd` files with `SKILL` definitions | Importable skills |
| `*.pd` files with `AGENT` definitions | Importable agents |
| `*.md` files in `docs/` or root | Knowledge (domain context) |
| `*.json`, `*.csv`, `*.yaml` in `data/` | Data files |
| `README.md` | Repo description and usage guide |
| Any source code files | Readable by the LLM as reference |

### Knowledge Loading

When `IMPORT KNOWLEDGE` pulls a markdown file into context, the platform doesn't dump the entire file into every LLM call. It uses the same context management strategy as the persistence layer:

- **Relevance filtering** — only sections relevant to the current request are loaded
- **Chunking** — large documents are split and indexed; the platform retrieves the right chunks
- **Summarization** — very large knowledge bases get a summary in context with detail available on demand
- **Caching** — extracted context is cached across requests for the same application

## Version Resolution

Repos are versioned the way git works — because they *are* git repos.

| Specifier | Resolves To | Use When |
|---|---|---|
| `@v2.1.0` | Exact tag | Production dependencies — pinned and reproducible |
| `@v2` | Latest `v2.x.x` tag | Accept patches within a major version |
| `#main` | Branch HEAD | Development — always latest |
| `#production` | Branch HEAD | Track a release branch |
| `@a1b2c3d` | Exact commit | Forensic reproducibility |
| *(none)* | Default branch HEAD | Quick iteration, prototyping |

### Lock File

The platform generates a `uses.lock` file that pins every dependency to an exact commit SHA — similar to `package-lock.json` or `go.sum`. This guarantees reproducible builds even when branches advance.

```
# uses.lock — auto-generated, do not edit
github:acme/fraud-toolkit@v3.1.0 → a1b2c3d4e5f6 (2026-03-10)
github:acme/crm-connectors#main → f6e5d4c3b2a1 (2026-03-14)
github:acme/compliance-rules#main → 1a2b3c4d5e6f (2026-03-12)
```

## Access Control

Private repos require credentials. The platform manages this — the author never handles tokens.

| Repo Type | Access Mechanism |
|---|---|
| Public GitHub | No auth required |
| Private GitHub (same org) | Platform's GitHub App installation |
| Private GitHub (cross-org) | Configured deploy keys or PATs in platform vault |
| Local path | Filesystem access (dev mode only) |

The author writes `USE "github:acme/private-repo"` and the platform handles authentication transparently. Credentials are never exposed in markup, Promptdown, or logs.

## Repos as Knowledge vs. Repos as Capability

There's an important distinction in how repos are consumed:

**Repos as knowledge** — the repo's content becomes domain context that the LLM reasons about. A compliance rules repo, an API documentation repo, or a style guide repo. The LLM *reads* the content and uses it to inform decisions.

```promptdown
USE "github:acme/compliance-rules" AS compliance
IMPORT KNOWLEDGE "pci-dss-v4.md" FROM compliance

# The LLM now has PCI rules in context — no explicit reference needed
$assessment <- "Is this transaction compliant? {{$transaction}}"
```

**Repos as capability** — the repo provides callable units (templates, skills, agents). A fraud toolkit, a data processing library, or a set of reusable agents. The program *calls* the content as functions.

```promptdown
USE "github:acme/fraud-toolkit@v3" AS fraud
IMPORT score_order FROM fraud

# Calling an imported template — explicit invocation
$score <- CALL score_order($order)
```

**Repos as both** — many repos serve dual purposes. A fraud toolkit might export callable templates *and* a methodology document that the LLM should reference when interpreting results.

```promptdown
USE "github:acme/fraud-toolkit@v3" AS fraud
IMPORT score_order FROM fraud
IMPORT KNOWLEDGE "docs/scoring-methodology.md" FROM fraud

$score <- CALL score_order($order)
# The LLM explains the score using the methodology doc as context
$explanation <- "Explain this fraud score: {{$score}}"
```

## Consuming Non-Markdown-OS Repos

Any git repo is a valid dependency — it doesn't need to be written for Markdown OS. This is the most powerful aspect of the system: the entire open-source ecosystem becomes a library.

```promptdown
# Use a public API's docs as knowledge
USE "github:stripe/stripe-api-docs" AS stripe
IMPORT KNOWLEDGE "docs/api/charges.md" FROM stripe

# Use a codebase as reference for the LLM to reason about
USE "github:expressjs/express" AS express_src
IMPORT KNOWLEDGE "README.md" FROM express_src

# Use someone's data repo
USE "github:datasets/covid-19" AS covid
IMPORT DATA "data/countries-aggregated.csv" FROM covid INTO $covid_data

$analysis <- "Analyze trends in: {{$covid_data}}"
```

The LLM reads the imported content the same way a developer would read documentation or source code. It doesn't need the repo to export a specific interface — it understands what the content *means*.

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

## Dependency Intents

**REQUIRE** — Declare a dependency on an external repository.
Default: Resolve to the default branch HEAD; cache locally.
Expressions: "use the scoring rules from," "reference the API docs from," "depend on the fraud toolkit"

**IMPORT** — Bring specific content from a dependency into scope.
Default: Load on first access; apply relevance filtering for knowledge imports.
Expressions: "import the scoring template," "use their sentiment skill," "load the compliance rules as context"

## Execution Intents

**EXEC** — Run an external CLI command.
Default: Sandboxed execution with resource limits. Fail on non-zero exit code.
Expressions: "use ffmpeg to extract a clip," "run the database migration," "compress the files," "execute the build"

**PROVISION** — Retrieve a secret or credential from the secret store.
Default: Scoped to the requesting application. Never exposed in logs or LLM context.
Expressions: "use our Shopify API key," "authenticate with Slack," "connect to the database," "get the API credentials"

## Compiler Intents

**SYNTHESIZE** — Create a new capability (skill, template, agent) dynamically.
Default: Auto-test before use; cache for reuse across requests.
Expressions: "the compiler creates a skill for," "dynamically generates a template," "synthesizes an agent to handle"

**OPTIMIZE** — Restructure code for efficiency without changing intent.
Default: Consolidate prompts, parallelize independent operations, route to skills, memoize duplicates.
Expressions: "the compiler consolidates," "parallelizes these operations," "routes to existing skill"

**DOCUMENT** — Generate documentation for code, constructs, and decisions.
Default: Add inline comments to all synthesized and optimized code. Generate companion markdown files for complex programs.
Expressions: "the compiler documents," "generates explanation for," "adds comments describing"

## Verification Intents

**TEST** — Define an expected behavior given specific inputs.
Default: Must pass before compilation, deployment, or publication.
Expressions: "when given a valid invoice, it should approve," "test that high-risk orders are flagged," "verify the output contains a score"

**ASSERT** — Declare a condition that must hold on an output.
Default: Fail the test immediately if the assertion is not met.
Expressions: "the result should contain," "the output must not be empty," "the response should be empathetic"

**MOCK** — Substitute a dependency or template with a controlled stand-in.
Default: Scoped to the test block; does not affect production execution.
Expressions: "mock the scoring template," "stub the CRM lookup," "simulate a Slack failure"

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
| "Use the fraud toolkit to score it, then check compliance rules." | REQUIRE → IMPORT → DELEGATE → RECALL(knowledge) → BRANCH |
| "Test that high-risk orders get flagged. Mock the CRM." | TEST → MOCK → ASSERT |

## Intent-Level Operations

Because intents are the execution unit, the platform provides:

- **Intent-level debugging** — trace shows `RECEIVE → VALIDATE → EXTRACT → RECALL(failed) → FALLBACK → TRANSFORM → EMIT`
- **Intent-level analytics** — "RETRY fires 400 times/day; FALLBACK at 12%"
- **Intent-level composition** — wire unit A's EMIT to unit B's RECEIVE
- **Intent-level testing** — "When duplicate arrives, SKIP should fire and no EMIT should occur"

---

# Layer Mapping

The Markup layer, Promptdown layer, and Intent Taxonomy all express the same concepts. This table shows the correspondence:

| Intent | Markup Expression | Promptdown Syntax |
|---|---|---|
| **REQUIRE** | "use the rules from our compliance repo" | `USE "github:org/repo@v2" AS alias` |
| **IMPORT** | "import the scoring template" | `IMPORT name FROM alias` |
| **TEST** | "when given a valid invoice, it should approve" | `TEST "description": ... END` |
| **ASSERT** | "the result should contain a score" | `ASSERT $var CONTAINS "text"` |
| **MOCK** | "mock the scoring template" | `MOCK name AS: RETURN "value" END` |
| **EXEC** | "use ffmpeg to extract a clip" | `$result <- CLI "command"` |
| **PROVISION** | "use our Shopify API key" | `SECRET "key_name"` |
| **SYNTHESIZE** | (compiler creates capabilities dynamically) | Automatic — compiler emits SKILL/TEMPLATE/AGENT |
| **OPTIMIZE** | (compiler restructures for efficiency) | Automatic — prompt consolidation, parallelization, memoization |
| **DOCUMENT** | (compiler generates documentation) | Automatic — inline comments, companion `.docs.md` files |
| **RECEIVE** | "when an order arrives" | `$input = ...` (entry point variable) |
| **EMIT** | "post to Slack" | `PRINT $var` or tool integration |
| **TRANSFORM** | "format as a summary" | `$result <- "transform: {{$input}}"` |
| **EXTRACT** | "pull out the line items" | `$fields <- @extract($data, "fields")` |
| **COMBINE** | "merge with CRM data" | Multiple `{{$vars}}` in one prompt |
| **FILTER** | "only orders over $500" | `IF {{$val}} CONTAINS "criteria": ... END` |
| **SORT** | "newest first" | `$sorted <- "Sort these by date: {{$list}}"` |
| **BRANCH** | "if the score exceeds 70" | `IF {{$score}} CONTAINS "high": ... ELSE: ... END` |
| **ITERATE** | "for each order" | `FOR $item IN $list: ... END` |
| **SEQUENCE** | "first validate, then enrich" | Sequential `<-` statements |
| **PARALLEL** | "simultaneously look up and validate" | `PARALLEL: ... END` |
| **DELEGATE** | "hand off to the scoring unit" | `$result <- RUN agent_name($args)` |
| **WAIT** | "pause until confirmed" | (platform-level; not yet in Promptdown) |
| **REMEMBER** | "remember this order ID" | `REMEMBER $var AS "key"` |
| **RECALL** | "look up the customer" | `RECALL "key" INTO $var` |
| **FORGET** | "clear the cache" | (platform-level; maps to MEMORY operations) |
| **CHECK** | "has this been seen before" | `RECALL "key" INTO $var` + `IF` |
| **UPDATE** | "mark as processed" | `REMEMBER $new_val AS "key"` (upsert) |
| **RETRY** | "retry if it fails" | (platform-level resilience) |
| **FALLBACK** | "if CRM is down, use default" | `IF`/`ELSE` with alternative path |
| **SKIP** | "skip and move on" | `IF` guard + continue |
| **ALERT** | "flag this as unusual" | `PRINT` with severity context |
| **HALT** | "stop everything" | (platform-level) |
| **VALIDATE** | "make sure it has required fields" | `$valid <- @classify($input, "valid, invalid")` |
| **MASK** | "hide the credit card number" | `$masked <- @rewrite($data, "mask PII")` |
| **DISPLAY** | "show a table" | `PRINT $formatted` (conversion target renders) |
| **CAPTURE** | "let the user enter a value" | (UI layer; not in Promptdown) |

## When to Use Each Layer

**Use Markup when:**
- A non-technical author is describing the application
- The specification is high-level and the LLM should fill gaps
- Readability and auditability are the priority
- The application is a document first

**Use Promptdown when:**
- You need explicit control flow and composition
- The pipeline has specific sequencing requirements
- You want reusable templates and testable units
- You're building AI-native pipelines with skills, tools, and agents
- Deterministic routing matters (even if individual `<-` calls are probabilistic)

**Mix both when:**
- The application has a natural language specification (Markup) with precision sections (Promptdown blocks embedded inline)
- A Markup document references Promptdown templates for complex logic
- The same intent needs both a human-readable description and an executable implementation

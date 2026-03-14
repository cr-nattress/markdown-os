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

# Markdown OS

## Applications as documents. LLMs as the runtime.

Markdown OS is a platform where applications are written as natural language markup documents, compiled and executed by LLMs, and managed through a standardized operational interface. The markup is the source of truth. The LLM is the compiler. Traditional code is an export target, not the authoring medium.

---

## The Idea

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

## The Language

The language has two complementary layers:

**Markup** — natural language documents that describe applications as structured intent. Readable by anyone. The authoring medium for non-technical users.

**Promptdown** — a Turing-complete structured syntax where `<-` invokes the LLM. Variables, loops, conditionals, templates, skills, tools, agents, and memory — all wired through that one operator. The executable layer for precise control flow and composition.

Both layers resolve to the same **intent taxonomy** — 35+ semantic intents across 8 categories (Data Flow, Dependency, Verification, Control Flow, State, Resilience, Boundary, UI). See [spec/LANGUAGE.md](spec/LANGUAGE.md) for the full specification.

## Key Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Application format | Markdown / natural language | Readable by humans, interpretable by LLMs |
| Execution model | LLM as runtime via tool calling | Both Claude and OpenAI support this natively |
| Structured syntax | Promptdown (`<-` operator) | Turing-complete with one essential primitive |
| Persistence | Platform-managed, intent-derived | Zero config for the author; storage shape emerges from purpose |
| Dependencies | Git repos as packages | Universal, versioned, works with any existing repo |
| Testing | Mandatory before compile/deploy | Non-deterministic execution demands empirical verification |
| First conversion target | React | Immediate visual proof of value; deployable to Vercel/Netlify |
| Language constructs | Intent-based, not keyword-based | Natural language is infinitely flexible; intents are the atoms |
| Provider strategy | Common capabilities across Claude + OpenAI | No vendor lock-in at the AI layer |
| Minimum viable markup | A purpose statement (one sentence) | The application always builds; complexity is never a prerequisite |
| Implementation agnosticism | Markup never implies a target language | Same markup → React, C#, Python, or platform-native execution |
| Gap-filling strategy | LLM makes opinionated best-practice decisions | Silence in the markup is a feature, not an error |

## Repository Structure

```
markdown-os/
├── README.md                           ← This document
├── docs/
│   ├── ARCHITECTURE.md                 ← Platform layers, compilation pipeline, tool-calling model
│   ├── PHILOSOPHY.md                   ← Post-syntax programming, intent-as-keyword, zero-infra
│   └── PROVIDER-ABSTRACTION.md         ← Claude/OpenAI common surface, adapter interface
├── spec/
│   ├── CORE-TENETS.md                  ← The 7 non-negotiable axioms (with examples and acid tests)
│   ├── LANGUAGE.md                     ← Full language spec: Markup, Promptdown, Intents, Testing, Repos
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
        └── weather-wardrobe.md         ← Weather clothing advisor (markup source)
```

## Status

This project is in the design and specification phase. The documents in this repository capture the architecture, language design, intent taxonomy, and core tenets that define the platform.

**Source:** [github.com/cr-nattress/markdown-os](https://github.com/cr-nattress/markdown-os)

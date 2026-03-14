# Specificity Spectrum — Expense Tracker

Three versions of the same application demonstrating the "always builds" guarantee (Core Tenet 1) and the specificity continuum (Core Tenets 2-3).

## The Files

- **[level-1-minimal.md](level-1-minimal.md)** — 2 lines. The LLM decides everything: UI layout, categories, persistence, interactions. It builds. It runs.
- **[level-2-moderate.md](level-2-moderate.md)** — Moderate detail. The author shapes key behaviors (form fields, categories, display). LLM still fills gaps (persistence strategy, error handling, mobile layout).
- **[level-3-detailed.md](level-3-detailed.md)** — Full specification with implementation target. The author controls most decisions. The `## Implementation` section pins the target to React + TypeScript + Tailwind.

## What This Demonstrates

Each level produces a working application. The difference is how much the author controls vs. how much the LLM decides.

```
Level 1: "Track expenses"
  → LLM decides everything → Generic but functional

Level 2: "Track expenses with these categories and this layout"
  → LLM fills remaining gaps → More defined

Level 3: "Track expenses with this exact form, this exact display,
          these exact categories, built as React with TypeScript"
  → LLM fills only minor details → Precisely specified
```

Without the `## Implementation` section, all three levels would compile to any target — React, C#, Python, CLI. The markup is about the problem, not the technology. Pinning a target is optional (Core Tenet 5).

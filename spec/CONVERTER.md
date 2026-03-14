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

# Skills

A `/skills` directory for agent skills I maintain.

## Install

```bash
npx skills@latest add aurorascharff/nextjs-app-architecture-skill
```

## Skills

### `nextjs-app-architecture`

Architecture patterns for Next.js 16+ App Router apps. Packages the patterns from [Component Architecture for React Server Components](https://aurorascharff.no/posts/component-architecture-for-react-server-components/) for AI coding agents.

The five principles from the post:

- Pages are synchronous compositors. They don't fetch, they compose.
- Async components fetch their own data. Co-locate the read with the JSX.
- Skeletons live next to their component. Same file, exported alongside it.
- Suspense boundaries go at the page level. The page designs the loading sequence.
- Client boundaries are leaf nodes. Push `'use client'` as deep as it can go.

Skill path: [`skills/nextjs-app-architecture`](skills/nextjs-app-architecture)

References load on demand:

- `skills/nextjs-app-architecture/references/feature-folders.md`
- `skills/nextjs-app-architecture/references/queries-actions.md`
- `skills/nextjs-app-architecture/references/components.md`
- `skills/nextjs-app-architecture/references/pages-suspense.md`
- `skills/nextjs-app-architecture/references/cache-components.md`
- `skills/nextjs-app-architecture/references/ux-patterns.md`

### `friction-log`

Documents agentic developer experience friction encountered during a development flow. Use when asked to log friction, document a pain point, or capture places where an agent gets stuck by unclear docs, framework behavior, tooling, sandboxing, or stale assumptions.

This collection includes the active `friction-log` skill only. The old passive `friction-report` skill is intentionally not included.

Skill path: [`skills/friction-log`](skills/friction-log)

## Companion

- [React View Transitions](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-view-transitions)

## Further reading

- [Server and Client Component Composition in Practice](https://aurorascharff.no/posts/server-client-component-composition-in-practice/)
- [Building Design Components with Action Props using Async React](https://aurorascharff.no/posts/building-design-components-with-action-props-using-async-react/)
- [Error Handling in Next.js with catchError](https://aurorascharff.no/posts/error-handling-in-nextjs-with-catch-error/)
- [Avoiding Server Component Waterfall Fetching with React 19 cache()](https://aurorascharff.no/posts/avoiding-server-component-waterfall-fetching-with-react-19-cache/)
- [next16-social-media](https://github.com/aurorascharff/next16-social-media) — demo app

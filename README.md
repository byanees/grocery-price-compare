# Grocery Price Compare

A grocery price comparison app, built end to end by an agentic pipeline driven by
[GitHub Spec Kit](https://github.com/github/spec-kit) and Claude Code. A human writes the
requirement; agents write the spec, plan, tasks, tests, and code, with human approval at
defined gates.

## How the pipeline works

```
requirement.md
  → /speckit-specify   spec.md: what and why, no tech
  → /speckit-clarify   resolve ambiguity                    Gate 1: spec ready
  → /speckit-plan      plan, data model, API contracts
  → /speckit-tasks     ordered tasks; [P] = parallel-safe
  → /speckit-analyze   cross-artifact consistency check     Gate 2: design consistent
  → /speckit-implement test-first, one user story at a time
  → verify             independent agent checks the spec    Gate 3: verified
```

The rules every agent follows are in
[`.specify/memory/constitution.md`](.specify/memory/constitution.md). Feature artifacts
are written to `specs/NNN-feature/`.

## Stack

- pnpm monorepo: `apps/web` and `apps/api`
- Frontend: Next.js (App Router), Tailwind CSS, shadcn/ui; mobile-first
- Backend: NestJS
- Database: free-tier, serverless-friendly (chosen during planning)
- Hosting: Vercel, deployed from this repository

## Status

The pipeline is set up and the constitution is ratified. The first feature spec has not
been written yet.

# Development Pipeline

Spec-driven project built with GitHub Spec Kit. Read `.specify/memory/constitution.md` before doing
any work. It overrides everything else, including this file.

## Pipeline

`/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-analyze`
→ `/speckit-implement` → independent verification. Gates and retry limits are defined in the
constitution. Feature artifacts live in `specs/NNN-feature/`.

## Stack at a glance

- pnpm monorepo: `apps/web` (Next.js App Router, Tailwind, shadcn/ui), `apps/api` (NestJS)
- TypeScript strict; database chosen per project (free tier, serverless-friendly)
- Deployed from GitHub to Vercel as two projects; the API runs as a single Vercel Function

## Rules agents most often break

- Build mobile-first at 390px, using shadcn/ui components and the shared design tokens only.
- No generic AI-template visuals or wording (constitution Principles VII and VIII).
- Write tests first and never weaken them; update the spec before changing behavior.
- Never push, merge, change Vercel settings, or provision services without human approval.

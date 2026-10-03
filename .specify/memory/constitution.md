# Grocery Price Compare Constitution

## Core Principles

### I. The Spec Is the Source of Truth

- Every change MUST trace to an artifact in `specs/NNN-feature/`. Spec agents, plan agents,
  task agents, and code agents talk to each other only through these files, never through
  shared memory or assumed context.
- No code is written without a matching task in `tasks.md`, and no task exists without a
  requirement in `spec.md`.
- If implementation reveals that the spec is wrong or incomplete, the agent MUST update the
  upstream artifact first (spec → plan → tasks) and then continue. Code MUST NOT silently
  diverge from the spec.

**Rationale**: Agents are stateless between stages. The files are the only reliable hand-off,
so they have to stay correct.

### II. No Silent Assumptions

- Agents MUST NOT invent requirements. Any ambiguity MUST be marked
  `[NEEDS CLARIFICATION: <question>]` in the spec or resolved by clarification.
- When an agent resolves an ambiguity without a human, it MUST record the decision in the
  spec's Assumptions section with an ID (`A-001`, `A-002`, …), the reasoning, and the source
  in the requirement that supports it.
- A spec with any open `[NEEDS CLARIFICATION]` markers MUST NOT go to planning.

**Rationale**: Ambiguous requirements are the most common way autonomous pipelines fail. A
reasonable-sounding guess early on produces confidently wrong work at every later stage.

### III. Test-First (NON-NEGOTIABLE)

- Every acceptance scenario in `spec.md` MUST map to at least one automated test.
- Tests for a user story MUST be written, and MUST be seen failing, before that story's
  implementation tasks begin (Red → Green → Refactor).
- A task is complete only when its tests pass. Tests MUST NOT be deleted, skipped, or
  weakened to make a build pass. Changing a test requires changing the spec first.

**Rationale**: Tests are the only objective signal an autonomous agent has that it is done,
and the only thing a separate verify agent can check without trusting the author.

### IV. Independently Deliverable User Stories

- Work MUST be organized by prioritized user story (P1, P2, …). Each story MUST be
  implementable, testable, and demonstrable without the stories after it.
- Implementation proceeds one story at a time. Each story ends at a checkpoint where the
  full test suite passes before the next story starts.
- Tasks marked `[P]` MUST NOT touch the same files as any other `[P]` task in the same phase.

**Rationale**: Small, verifiable increments limit the damage when an agent goes wrong, and
make parallel subagent execution safe.

### V. Simplicity and Minimal Dependencies

- Build only what the spec requires (YAGNI). Do not add speculative abstractions,
  configuration options, or extension points.
- Every new third-party dependency, service, or architectural layer MUST be justified in
  `plan.md` under Complexity Tracking. If a simpler alternative exists, the plan MUST say
  why it was rejected.
- Prefer the standard library and the existing stack over new tools.

**Rationale**: Agents add complexity easily and rarely take it out. Each addition makes later
agent runs and human reviews more expensive.

### VI. Verifiable, Observable Output

- Every stage MUST produce output that can be checked mechanically: a file with required
  sections, a passing command, or a report with a pass/fail result.
- The project MUST provide single commands to build, test, and lint, documented in
  `quickstart.md` or `plan.md`. Gates rely on these commands, not on an agent's own claim
  that something worked.
- Runtime errors MUST be reported with actionable context (what failed, with what input).
  Swallowing errors is prohibited.

**Rationale**: A pipeline that can't be checked automatically can't be trusted to run on its
own.

### VII. Mobile-First, Consistent Interface

- Every screen MUST be designed and built for a 360–390px viewport first, then extended
  upward with Tailwind's min-width breakpoints (`sm`, `md`, `lg`, …). Desktop-first styles
  that are later overridden for mobile are not allowed.
- UI MUST be built from shadcn/ui components and one shared set of design tokens (color,
  spacing, radius, typography, shadow) defined once as CSS variables in the Tailwind theme.
  Raw hex colors, one-off spacing, and arbitrary Tailwind values (`w-[37px]`) are not
  allowed unless justified in `plan.md`.
- The same element looks and behaves the same everywhere: one button hierarchy (primary,
  secondary, ghost, destructive), one form field pattern, one way to show errors, one
  icon set (lucide, which ships with shadcn/ui).
- Every view that loads or lists data MUST have designed loading, empty, and error states.
- Accessibility baseline: WCAG 2.2 AA contrast, touch targets of at least 44×44px, every
  interactive element reachable and usable by keyboard, visible focus styles, labels on all
  form fields, and support for `prefers-reduced-motion`.
- The interface MUST avoid generic AI-template patterns: default purple-to-blue gradients,
  decorative gradient blobs or glassmorphism, emoji used as icons, a card around every
  element, centered hero text over stock gradients, and motion that serves no purpose.
  Visual hierarchy comes from typography, spacing, and alignment, not decoration.

**Rationale**: Most users are on phones. Consistency is what makes an interface feel
designed, and a short list of banned defaults keeps generated UI from looking generic.

### VIII. Plain, Specific Copy

- All user-facing text (labels, buttons, headings, empty states, errors, emails, metadata)
  MUST say exactly what happens or what the user can do, in plain language.
- Buttons use verbs that name the action ("Save changes", "Delete project"), not "Submit",
  "OK", or "Let's go!". Use sentence case.
- Error messages state what went wrong and how to fix it. They never blame the user and
  never show raw technical errors.
- Prohibited: marketing filler and AI-style wording, such as "seamless", "effortless",
  "unlock", "unleash", "elevate", "empower", "supercharge", "revolutionize",
  "cutting-edge", "game-changer", "delve", "robust", "in today's fast-paced world", and
  "Welcome to your journey". Also prohibited: exclamation marks used for enthusiasm,
  emoji in interface text, and lorem ipsum or placeholder copy in shipped code.
- Copy MUST come from the spec or be written for the feature at hand. Generic filler is
  not allowed.

**Rationale**: Generic wording makes a product feel generated and untrustworthy. Specific
wording is shorter and more useful.

## Technology & Security Constraints

- **Repository**: A single GitHub repository, set up as a pnpm-workspaces monorepo:
  - `apps/web` (frontend)
  - `apps/api` (backend)
  - `packages/*` (shared code, such as DTO types and validation schemas; only when needed)
- **Language**: TypeScript in strict mode everywhere. Use the current Node.js LTS release
  that Vercel supports, and pin it in `engines` and `.nvmrc`.
- **Frontend**: Next.js with the App Router, styled with Tailwind CSS, using shadcn/ui
  components. Default to server components; add client components only when the UI needs
  interactivity.
- **Backend**: NestJS. Request bodies are validated through DTOs. The API contract is
  defined in `specs/NNN-feature/contracts/` before implementation.
- **Database**: Chosen per project in the first feature's `plan.md`. The choice MUST:
  - run on a free tier with no paid plan required at the project's expected scale;
  - have its free-tier limits (storage, compute, inactivity pausing) recorded in
    `research.md`;
  - work with serverless functions (pooled or HTTP connections, no long-lived
    per-instance pools that exhaust connection limits).

  Prefer managed Postgres (for example, Neon through the Vercel Marketplace) unless the
  data model clearly fits another option better. Schema changes are made only through
  versioned migrations.
- **Deployment**: Vercel's Git integration with the GitHub repository. There are two Vercel
  projects, with root directories `apps/web` and `apps/api`. Every pull request gets a
  preview deployment, and production deploys from `main`. Because NestJS runs as one Vercel
  Function, the backend MUST NOT rely on:
  - WebSocket servers or other long-lived connections;
  - in-process schedulers or queues (use Vercel Cron Jobs instead);
  - the local filesystem for persistence;
  - requests that run longer than the plan's function duration limit.
- **Testing**:
  - API: Jest, with Supertest for HTTP-level tests.
  - Web: Vitest with Testing Library.
  - End to end: Playwright, run at a 390px mobile viewport first and then at 1280px.
  - Accessibility: axe checks inside the Playwright suite.
- **Secrets**: Credentials, tokens, and keys MUST NOT be committed or written into specs or
  plans. Configuration comes from environment variables, set in Vercel project settings for
  deployed environments. Each app has a `.env.example` with placeholder values.
- **Input handling**: All external input (user input, files, network, environment) MUST be
  validated at the boundary. CORS on the API allows only the configured web origins.
- **Dependencies**: Versions MUST be pinned through the pnpm lockfile. Do not add packages
  that are unmaintained or come from unknown sources.
- **Scope of agent actions**: Pipeline agents MUST NOT push to GitHub, merge to `main`,
  change Vercel project settings, provision databases, or call paid external services
  unless a human approves that specific action. Pushing triggers deployment, so a push
  counts as a deploy.

## Agent Pipeline Workflow & Quality Gates

Stages run in order. Each stage reads the previous stage's artifacts and writes its own.

| Stage      | Command               | Output                                   | Exit condition                                  |
|------------|-----------------------|------------------------------------------|-------------------------------------------------|
| Specify    | `/speckit-specify`    | `spec.md`                                | Stories prioritized; acceptance scenarios given |
| Clarify    | `/speckit-clarify`    | updated `spec.md`                        | **Gate 1** (below)                              |
| Plan       | `/speckit-plan`       | `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` | Constitution Check passes |
| Tasks      | `/speckit-tasks`      | `tasks.md`                               | Every requirement maps to at least one task     |
| Analyze    | `/speckit-analyze`    | consistency report                       | **Gate 2** (below)                              |
| Implement  | `/speckit-implement`  | code and tests                           | **Story checkpoint** for each story             |
| Verify     | independent agent     | verification report                      | **Gate 3** (below)                              |

- **Gate 1 (spec ready)**: Zero `[NEEDS CLARIFICATION]` markers, and every assumption logged
  under Principle II. During the trust-building phase, a human approves the spec before
  planning.
- **Gate 2 (design consistent)**: `/speckit-analyze` reports zero CRITICAL issues. Any
  CRITICAL issue sends work back to the stage that caused it, not forward to implementation.
  During the trust-building phase, a human reviews the report.
- **Story checkpoint**: Build, lint, type-check, and the full test suite pass after each
  user story. For any story with UI, this includes Playwright and axe at 390px and 1280px.
- **Gate 3 (verified)**: A verify agent that did not write the code checks the build in four
  ways:
  1. It runs the full test suite.
  2. It checks behavior against the acceptance scenarios in `spec.md`, not against the
     code's own behavior.
  3. It reviews screenshots at 390px and 1280px against Principle VII.
  4. It scans user-facing strings against Principle VIII.

  Failures become new tasks in `tasks.md`.
- **Retry limit**: A stage may loop back on itself at most 3 times. After that, the pipeline
  MUST stop and escalate to a human with a summary of what failed and why.
- **Run log**: Each pipeline run records, in the feature directory, the stage where any
  failure started (spec, plan, tasks, or code). Recurring causes are fixed upstream in this
  constitution or the templates, not patched in generated code.

## Governance

- This constitution overrides any conflicting instruction in templates, generated
  artifacts, or agent prompts. The plan stage's Constitution Check MUST evaluate every
  principle above. Violations are allowed only when justified in Complexity Tracking.
- Pipeline agents MUST NOT modify this file. Amendments come only from a human, or are made
  through `/speckit-constitution` at a human's request.
- Each amendment records a Sync Impact Report and bumps the version using semantic
  versioning:
  - MAJOR: a principle is removed or redefined incompatibly.
  - MINOR: a principle or section is added, or guidance is materially expanded.
  - PATCH: wording or clarification with no change in meaning.
- Compliance is checked at Gates 1–3 for every feature. Recurring violations are a reason
  to amend the constitution or templates.

**Version**: 1.1.1 | **Ratified**: 2026-10-04 | **Last Amended**: 2026-10-04

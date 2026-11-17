---
name: spec-driven-development
description: Creates specs before coding. Use when starting a new project, feature, or significant change and no specification exists yet. Use when requirements are unclear, ambiguous, or only exist as a vague idea. Use when a single requirement spans several independently testable capabilities and needs decomposing into a capability map of modules before specifying.
---

# Spec-Driven Development

## Overview

Write a structured specification before writing any code. The spec is the shared source of truth between you and the human engineer — it defines what we're building, why, and how we'll know it's done. Code without a spec is guessing.

## When to Use

- Starting a new project or feature
- Requirements are ambiguous or incomplete
- The change touches multiple files or modules
- You're about to make an architectural decision
- The task would take more than 30 minutes to implement

**When NOT to use:** Single-line fixes, typo corrections, or changes where requirements are unambiguous and self-contained.

## The Gated Workflow

Spec-driven development has four phases, preceded by a scope check (Phase 0) that activates only when one request bundles several independently testable capabilities. Do not advance to the next phase until the current one is validated.

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 Human      Human    Human      Human
 reviews    reviews  reviews    reviews
```

### Phase 0: Scope Check

Most requests describe one capability. If this one does, skip this phase and go straight to Specify — Phase 0 exists for the exception, not the rule, and it puts no hierarchy on single-capability features.

**Detection.** Decompose before specifying when a single requirement bundles several independently testable capabilities:

- The requirement names distinct capabilities with their own consumers or data (e.g. identity, billing, notifications, reporting)
- Acceptance criteria cluster into groups that could ship and be verified separately
- One capability could be cut or replaced without rewriting the others' requirements

**Propose a capability map before writing any spec.** Small and reviewable — a module table plus a build order, not a project plan:

```markdown
# Capability Map: [Initiative Name]

| Module id | Responsibility | Depends on |
|---|---|---|
| identity | Accounts, sessions, SSO | — |
| billing | Plans, invoices, payments | identity |
| notifications | Email and webhook fan-out | identity |
| reporting | Usage dashboards | billing, notifications |

Build order: identity → billing, notifications → reporting
```

- **Stable module ids.** Kebab-case, chosen once, never renamed mid-initiative. Specs, plans, and downstream commands select work by these ids instead of guessing which spec is active.
- **Dependency direction, no cycles.** Arrows point one way. If two modules each need the other, they are one module.
- **Interfaces live at the boundary.** The map records that `billing` depends on `identity`; the contract between them belongs in the provider module's spec (see `api-and-interface-design` for designing it).

**The map is gated like every phase.** The human reviews module boundaries, dependency direction, and build order before any module spec is written. Getting the map wrong is expensive; reviewing ten lines is not.

**Then recurse per module.** Run Specify → Plan → Tasks → Implement for each module in dependency order. Each module gets its own spec, scoped to that module's objective, boundaries, and success criteria. Save the approved map to `spec/<initiative-slug>/capability-map.md` and each module's spec alongside it, named by module id (`spec/<initiative-slug>/spec-identity.md`, `spec/<initiative-slug>/spec-billing.md`) — the map, not filename guessing, is the index of what exists.

### Phase 1: Specify

Start with a high-level vision. Ask the human clarifying questions until requirements are concrete.

**Surface assumptions immediately.** Before writing any spec content, list what you're assuming:

```
ASSUMPTIONS I'M MAKING:
1. This is a web application (not native mobile)
2. Authentication uses session-based cookies (not JWT)
3. The database is PostgreSQL (based on existing Prisma schema)
4. We're targeting modern browsers only (no IE11)
→ Correct me now or I'll proceed with these.
```

Don't silently fill in ambiguous requirements. The spec's entire purpose is to surface misunderstandings *before* code gets written — assumptions are the most dangerous form of misunderstanding.

**Write a spec document covering these six core areas** (the template below also asks for Non-goals, Alternatives Considered and Risks)**:**

1. **Objective** — What are we building and why? Who is the user? What does success look like?

2. **Commands** — Full executable commands with flags, not just tool names.
   ```
   Build: npm run build
   Test: npm test -- --coverage
   Lint: npm run lint --fix
   Dev: npm run dev
   ```

3. **Project Structure** — Where source code lives, where tests go, where docs belong.
   ```
   src/           → Application source code
   src/components → React components
   src/lib        → Shared utilities
   tests/         → Unit and integration tests
   e2e/           → End-to-end tests
   docs/          → Documentation
   ```

4. **Code Style** — One real code snippet showing your style beats three paragraphs describing it. Include naming conventions, formatting rules, and examples of good output.

5. **Testing Strategy** — What framework, where tests live, coverage expectations, which test levels for which concerns.

6. **Boundaries** — Three-tier system:
   - **Always do:** Run tests before commits, follow naming conventions, validate inputs
   - **Ask first:** Database schema changes, adding dependencies, changing CI config
   - **Never do:** Commit secrets, edit vendor directories, remove failing tests without approval

**Spec template:**

```markdown
# Spec: [Project/Feature Name]

## Objective
[What we're building and why. User stories or acceptance criteria.]

## Tech Stack
[Framework, language, key dependencies with versions]

## Commands
[Build, test, lint, dev — full commands]

## Project Structure
[Directory layout with descriptions]

## Code Style
[Example snippet + key conventions]

## Testing Strategy
[Framework, test locations, coverage requirements, test levels]

## Boundaries
- Always: [...]
- Ask first: [...]
- Never: [...]

## Non-goals
[What this will deliberately not do]

## Alternatives Considered
[Each rejected option and the concrete reason it lost: complexity, cost, a conflict with an existing convention]

## Risks
[What could go wrong, and the mitigation for each]

## Success Criteria
[How we'll know this is done — specific, testable conditions]

## Open Questions
[Anything unresolved that needs human input]
```

**Reframe instructions as success criteria.** When receiving vague requirements, translate them into concrete conditions:

```
REQUIREMENT: "Make the dashboard faster"

REFRAMED SUCCESS CRITERIA:
- Dashboard LCP < 2.5s on 4G connection
- Initial data load completes in < 500ms
- No layout shift during load (CLS < 0.1)
→ Are these the right targets?
```

This lets you loop, retry, and problem-solve toward a clear goal rather than guessing what "faster" means.

### Goldfish review

Once the draft is saved, and before asking for approval, run the protocol in [../../references/goldfish-review.md](../../references/goldfish-review.md) with the `comprehension` and `zoom-out` lenses. Two fresh subagents read the saved spec and the repo, without this conversation, in parallel. Triage every finding, apply the fixes, and write the report to `spec/<feature-name>/goldfish.md`.

At the approval prompt, show what the reference's "What to show in chat" section lists: escalations first, then the comprehension paraphrase, then counts and the report path. The paraphrase is the fastest way for the human to see whether the spec says what they meant.

- This phase writes spec artifacts only (`spec/<feature-name>/`), never source files.
- If a new request contradicts the saved spec, name the contradiction and ask which one wins before editing any file.

### GitHub issue sync

When the argument is a GitHub issue URL or `#<number>`, the issue is both input and output.

1. **Read it first.** Run `gh issue view <n> --json title,body,labels` before asking clarifying questions, and treat the title and body as requirements input. Add `--repo <owner>/<name>` when the URL names a repo other than the current one.
2. **Update it after the goldfish review.** Once the triaged spec is saved, rewrite the issue. Do not ask first; passing the issue is the request. If the spec changes during approval, run this step again so the issue matches the approved spec.
3. **Title:** one line that names the change, not the symptom ("add goldfish review to /spec and /plan", not "spec improvements").
4. **Body:** the problem in two or three sentences, what will change, an acceptance checklist taken from the spec's Success Criteria, and the spec path. No "generated by" footer.
5. **Labels:** choose only from `gh label list`. Never create a label, and never remove one that is already set.
6. **Voice:** write the title and body through the `humanize` skill with its github voice (lowercase sentence starts, identifiers in their real casing).
7. **Post:** write the body to a temp file in the scratchpad, then run:

   ```bash
   gh issue edit <n> --title "<title>" --body-file <path> --add-label <label>
   ```

   The old body is replaced. GitHub keeps the edit history on the issue, so the original is recoverable there.
8. **On failure** (no auth, no permission, closed repo), report the `gh` error in one line and finish the spec normally.

### Phase 2: Plan

With the validated spec, generate a technical implementation plan:

1. Identify the major components and their dependencies
2. Determine the implementation order (what must be built first)
3. Note risks and mitigation strategies
4. Identify what can be built in parallel vs. what must be sequential
5. Define verification checkpoints between phases

> Follow `planning-and-task-breakdown` for the dependency-graph mapping and vertical-slicing mechanics behind these steps; it is the canonical source. The bullets above are a lightweight summary; if they ever diverge, `planning-and-task-breakdown` takes precedence.
>
> **Output convention:** Save the plan to `tasks/<feature-name>/plan.md` and record the task list in the task list target defined by `planning-and-task-breakdown` (default `tasks/<feature-name>/todo.md`; projects may designate an external tracker instead). Create `tasks/<feature-name>/` if it does not exist, reusing the same `<feature-name>` slug as the spec (`spec/<feature-name>/spec.md`). Downstream commands (`/build`, etc.) expect these defaults.

The plan should be reviewable: the human should be able to read it and say "yes, that's the right approach" or "no, change X."

### Phase 3: Tasks

Break the plan into discrete, implementable tasks:

- Each task should be completable in a single focused session
- Each task has explicit acceptance criteria
- Each task includes a verification step (test, build, manual check)
- Tasks are ordered by dependency, not by perceived importance
- No task should require changing more than ~5 files

> Follow `planning-and-task-breakdown` for the full task-sizing and dependency-ordering mechanics; it is the canonical source. The template below is a lightweight inline form; if they ever diverge, `planning-and-task-breakdown` takes precedence.

**Task template:**
```markdown
- [ ] Task: [Description]
  - Acceptance: [What must be true when done]
  - Verify: [How to confirm — test command, build, manual check]
  - Files: [Which files will be touched]
```

### Phase 4: Implement

Execute tasks one at a time following `skills/incremental-implementation/SKILL.md` (`incremental-implementation`) and `skills/test-driven-development/SKILL.md` (`test-driven-development`). Use `skills/context-engineering/SKILL.md` (`context-engineering`) to load the right spec sections and source files at each step rather than flooding the agent with the entire spec.

## Keeping the Spec Alive

The spec is a living document, not a one-time artifact:

- **Update when decisions change** — If you discover the data model needs to change, update the spec first, then implement.
- **Update when scope changes** — Features added or cut should be reflected in the spec.
- **Commit the spec** — The spec belongs in version control alongside the code.
- **Reference the spec in PRs** — Link back to the spec section that each PR implements.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is simple, I don't need a spec" | Simple tasks don't need *long* specs, but they still need acceptance criteria. A two-line spec is fine. |
| "I'll write the spec after I code it" | That's documentation, not specification. The spec's value is in forcing clarity *before* code. |
| "The spec will slow us down" | A 15-minute spec prevents hours of rework. Waterfall in 15 minutes beats debugging in 15 hours. |
| "Requirements will change anyway" | That's why the spec is a living document. An outdated spec is still better than no spec. |
| "I wrote it, so I know it's clear" | The author fills gaps from memory and cannot see what the page leaves out. That is what the goldfish review is for. |
| "The user knows what they want" | Even clear requests have implicit assumptions. The spec surfaces those assumptions. |
| "It's one big feature; splitting it is overhead" | If acceptance criteria cluster into independently testable groups, a monolithic spec forces every downstream task to reason over the whole contract. A ten-line capability map is the cheap alternative. |
| "I'll decompose during planning" | Planning slices tasks within a spec. By then the oversized artifact already exists — module boundaries and dependency direction must be decided before the spec is written, not after. |

## Red Flags

- Starting to write code without any written requirements
- Asking "should I just start building?" before clarifying what "done" means
- Implementing features not mentioned in any spec or task list
- Making architectural decisions without documenting them
- Skipping the spec because "it's obvious what to build"
- Asking for approval before `spec/<feature-name>/goldfish.md` exists
- Running the goldfish rubrics in the main session and calling it a goldfish review
- One spec whose requirements span several independently testable capabilities
- Module boundaries or build order decided implicitly during implementation because no capability map was approved up front

## Verification

Before proceeding to implementation, confirm:

- [ ] The spec covers all six core areas
- [ ] The human has reviewed and approved the spec
- [ ] Success criteria are specific and testable
- [ ] Boundaries (Always/Ask First/Never) are defined
- [ ] The spec is saved to a file in the repository
- [ ] The spec has Non-goals, Alternatives Considered and Risks
- [ ] If an issue was given: its title, body and labels were updated, and no label was created
- [ ] The goldfish report is written, and every finding has a decision (Fix, Contest, Escalate or Decline)
- [ ] If the request bundles several independently testable capabilities, a capability map (module ids, dependency direction, build order) was approved before any module spec was written
- [ ] Every module spec traces to a module id in the approved map

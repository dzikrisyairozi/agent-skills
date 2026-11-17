---
description: Conduct a five-axis code review — correctness, readability, architecture, security, performance
---

Invoke the agent-skills:code-review-and-quality skill.

## Scope

If an argument is a GitHub PR URL or `#<number>`, review that PR (`gh pr diff <n>`) and follow **PR mode** below. Otherwise review the current changes (staged or recent commits).

Run the review through the Fresh-Context Review section of the skill: the `code-reviewer` persona in a fresh subagent, given the diff plus `spec/<feature-name>/spec.md` and the feature's plan when they exist (derive `<feature-name>` from the branch name), with the drift check. Then triage its findings before reporting.

Review across all five axes:

1. **Correctness** — Does it match the spec? Edge cases handled? Tests adequate?
2. **Readability** — Clear names? Straightforward logic? Well-organized?
3. **Architecture** — Follows existing patterns? Clean boundaries? Right abstraction level?
4. **Security** — Input validated? Secrets safe? Auth checked? (Use agent-skills:security-and-hardening skill)
5. **Performance** — No N+1 queries? No unbounded ops? (Use agent-skills:performance-optimization skill)

For the Architecture axis, verify blast-radius claims with `grep`/`git grep` for the changed symbol rather than assuming them. serena's `find_referencing_symbols` under-counts references that cross the `apps/web` ↔ `packages/*` workspace boundary in this repo (verified 2026-08-07) — fine as a same-package supplement, not sufficient alone for a cross-package interface change.

Once this five-axis review is done, also run `/ponytail-review` on the same diff as a separate add-on pass — it's scoped exclusively to over-engineering/bloat findings (unneeded deps, speculative abstractions, hand-rolled stdlib) and is designed to complement this review, not replace it.

Categorize findings as Critical, Important, or Suggestion.
Output a structured review with specific file:line references and fix recommendations.

## PR mode

When the review target is a pull request, the review is not finished until **both** artifacts exist. Do not skip either, and do not ask for permission first — posting the review on a PR you were asked to review is the requested deliverable.

### 1. Post the review on the PR

Follow the Posting a PR Review section of the skill: one GitHub review with event `COMMENT`, a short summary body with the verdict and counts, and an inline comment on each finding that maps to a line in the diff, with a `suggestion` block when the fix is an exact replacement. All posted prose goes through the `humanize` skill's github voice, so it is lowercase with identifiers in their real casing. Do not ask before posting.

### 2. Save an actionable plan under `tasks/<feature>/review/`

Derive `<feature>` from the PR's branch slug — reuse the existing `tasks/<feature>/` directory when the branch was built from one (e.g. branch `feat/3-6-m0-dashboard-foundation` → `tasks/3-6-m0-dashboard-foundation/review/`). Create the directory if needed.

Write two files there, in the same shape `/plan` produces. These local files keep normal sentence case, and they include contested findings with their evidence:

- `review/plan.md` — the findings converted into remediation work: phases ordered by severity (Critical → Important → Suggestion), each task vertically sliced with acceptance criteria, verification steps, and the files it touches. Include a header linking back to the PR URL and noting which findings were accepted vs. deliberately declined.
- `review/todo.md` — the flat checkbox task list, one line per remediation task, mirroring `plan.md`'s ordering.

Task template (same as `agent-skills:planning-and-task-breakdown`):

```markdown
- [ ] Task: [Description]
  - Finding: [Critical|Important|Suggestion] — [file:line]
  - Acceptance: [What must be true when done]
  - Verify: [Test command, build, or manual check]
  - Files: [Which files will be touched]
```

This makes the review directly consumable by `/build` — pass `tasks/<feature>/review` as the tasks directory to work the findings off.

If there are zero findings above Suggestion, still write `review/plan.md` with an explicit "no remediation required" section rather than skipping the file — the absence of findings is itself a reviewable record.

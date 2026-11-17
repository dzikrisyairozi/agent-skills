# Goldfish review: tasks/5-adopt-egm/plan.md

Run on 2026-11-18, after T1 to T9 were built, so reviewers compared the plan with both `main` and the branch.

## Round 1

| Reviewer | Status |
|---|---|
| readiness | completed |
| codebase | completed |

Counts: 2 Blocker, 8 Major, 11 Nit (21 entries; some bundle points both reviewers raised). Decisions: 12 Fix, 5 Contest, 3 Escalate, 1 Decline.

## Findings

### 1. Plan misstates which files the artifact-path validator guards (Blocker)

- Lens: codebase
- Where: "Constraints found in the code"
- Problem: said the validator scans five files; `GUARDED_FILES` in `scripts/validate-artifact-paths.js` has seven, including `docs/getting-started.md` and `docs/adoption-guide.md`.
- Decision: Fix
- Edit: the constraint now lists all seven files.

### 2. Task 9 points at AGENTS.md sections that do not exist (Blocker)

- Lens: codebase (also readiness, Major)
- Where: Task 9
- Problem: AGENTS.md has no skills-by-phase list, persona list or Conventions section.
- Decision: Fix
- Edit: Task 9 now names the Lifecycle Mapping `REVIEW` line and the Orchestration paragraph, where the edits were made.

### 3. Issue sync timing is contradictory (Major)

- Lens: readiness
- Where: Task 4 and Checkpoint: /spec
- Problem: Task 4 said "approved-ready" (undefined), and the checkpoint put the sync after approval. Spec 4.7 puts it before.
- Decision: Fix
- Edit: Task 4 says "after the goldfish review"; checkpoint order is now goldfish, issue sync, approval, with a re-sync if the spec changes during approval. This matches the shipped `GitHub issue sync` step 2 in `skills/spec-driven-development/SKILL.md`.

### 4. Triage of code findings has no severity mapping (Major)

- Lens: readiness
- Where: Task 7, Triage
- Problem: the goldfish rules decline only Nits, but code review labels are Critical, Important, Suggestion. Local mode writes no review file for contested findings.
- Decision: Fix (in the shipped skill)
- Edit: `skills/code-review-and-quality/SKILL.md` Triage now maps Critical to Blocker, Important to Major, Suggestion to Nit, and lists contested findings under "Contested" in local mode output.

### 5. PR files listing pagination and missing patches (Major)

- Lens: readiness
- Where: Task 8, step 2
- Decision: Contest
- Evidence: the shipped step 2 in `skills/code-review-and-quality/SKILL.md` ("Posting a PR Review") runs `gh api .../files --paginate` and says a file with no `patch` has no commentable lines.

### 6. A 422 does not say which comment failed (Major)

- Lens: readiness
- Where: Task 8, step 7
- Decision: Fix (in the shipped skill)
- Edit: on a 422 the skill now moves every inline comment into the summary body and posts once more with no inline comments; on a second failure it reports and stops.

### 7. Success criteria 1 and 3 are never exercised (Major)

- Lens: readiness
- Where: Task 10 and Checkpoint: complete
- Problem: no task runs the `/spec` goldfish lenses, and nothing forces a reviewer failure to show `not run`.
- Decision: Escalate
- Why escalate: running the comprehension and zoom-out lenses on `spec/5-adopt-egm/spec.md` costs two more subagents, and the `not run` path can only be shown by making a reviewer fail. Both are the owner's call on cost.

### 8. humanize "Pick the target" rules may be dropped (Major)

- Lens: readiness
- Where: Task 2
- Decision: Contest
- Evidence: `skills/humanize/SKILL.md` has `### Pick the target` with both rules (touch only comments and strings in code; ask before amending a commit or pushing a PR body).

### 9. Skill discovery surfaces do not mention humanize (Major)

- Lens: codebase
- Where: Task 2 and Task 9
- Problem: `skills/using-agent-skills/SKILL.md`, `docs/opencode-setup.md` and `docs/getting-started.md` route to skills by name and have no `humanize` entry.
- Decision: Escalate
- Why escalate: the finding is right, but all three files are outside the File Boundary, and `/build` stops before touching those.

### 10. Other callers of the changed skill and persona are not mentioned (Major)

- Lens: codebase
- Where: Tasks 5 and 7
- Problem: `/build auto` step 4, `/ship`, `doubt-driven-development` and nested-subagent use pick up the changes silently.
- Decision: Fix
- Edit: added a row to "Risks and mitigations". `/build auto` running the plan goldfish step is intended; Spec Drift reports "no spec" when none is given; the nested case falls back to "fresh-context review not run".

### 11. AGENTS.md says to regenerate TOML from a generator that is not in the repo (Nit)

- Lens: codebase
- Decision: Fix
- Edit: AGENTS.md "Adding or Changing a Skill" now says to edit all three surfaces in one change, keep escapes and JSON out of command bodies, and diff the TOML files, because the validator does not compare bodies.

### 12. Plan understates what validate-commands.js checks (Nit)

- Lens: codebase
- Decision: Fix
- Edit: the constraint now says it checks presence on all three surfaces plus `description`, and not prompt bodies.

### 13. Task 1 heading-count check returns 5, not 4 (Nit)

- Lens: codebase (also readiness)
- Decision: Fix
- Edit: verification uses `grep -c "^### Lens:"`, which returns 4.

### 14. humanize Verification forbids the placeholder Process creates (Nit)

- Lens: codebase
- Decision: Contest
- Evidence: the shipped Verification in `skills/humanize/SKILL.md` reads "No placeholder (`[needs a fact: ...]` aside, which is deliberate)".

### 15a. README never names humanize (Nit)

- Lens: codebase
- Decision: Fix
- Edit: added one sentence to "Elephant and goldfish" naming `humanize`.

### 15b. README skill count is stale (Nit)

- Lens: readiness
- Decision: Contest
- Evidence: `README.md` already says "26 skill definitions" (T9).

### 16. /review checkpoint order and docs/agents.md command list (Nit)

- Lens: codebase and readiness
- Decision: Fix
- Edit: the checkpoint lists triage with the fresh-context review, matching `.claude/commands/review.md`. `docs/agents.md` now maps `/spec` and `/plan` to two `goldfish-reviewer` instances.

### 17. EGM not credited in THIRD_PARTY_NOTICES.md (Nit)

- Lens: codebase
- Decision: Escalate
- Why escalate: the file is outside the File Boundary. EGM is credited, with a link and its license, in `references/goldfish-review.md`, `README.md` and `CLAUDE.md`; its text was rewritten, not copied. Whether a notice entry is still wanted is the owner's call.

### 18a. Rule count, missing tools allowlist, commit count (Nit)

- Lens: readiness
- Points: "four rules" in Task 1 vs three in spec 4.3; persona has no `tools:` allowlist; commit count in the foundation checkpoint.
- Decision: Decline

### 18b. File Boundary omits the review files (Nit)

- Lens: readiness
- Decision: Fix
- Edit: `tasks/5-adopt-egm/review/plan.md` and `todo.md` added to the File Boundary.

### 18c. Labels not recorded before the re-sync; "File boundary" casing (Nit)

- Lens: readiness
- Decision: Contest
- Evidence: issue #5 labels were read as `["enhancement"]` before the re-sync (10 labels in the repo before and after), and the shipped rubric in `references/goldfish-review.md` says "File Boundary".

## Round 2

Ran because round 1 Fixes resolved two Blockers. Re-ran only the lens that raised them.

| Reviewer | Status |
|---|---|
| codebase | completed |

Counts: 0 Blocker, 2 Major, 6 Nit. Decisions: 5 Fix, 2 Contest, 1 Escalate. No Blocker, so no third round.

### 19. "Six core areas" sentences ignore the three new template sections (Major)

- Decision: Fix
- Edit: the three spec command files now add "Also fill in the template's Non-goals, Alternatives Considered and Risks sections." The skill's "six core areas" lead-in points at the three template sections.

### 20. New orchestration pattern not in references/orchestration-patterns.md (Major)

- Decision: Escalate
- Why escalate: right, but outside the File Boundary. `docs/agents.md` (inside the boundary) already got the `/spec`, `/plan` fan-out line in round 1.

### 21. Persona lists in other docs miss goldfish-reviewer (Nit)

- Where: `docs/getting-started.md`, `docs/antigravity-setup.md`, `docs/copilot-setup.md`
- Decision: Escalate (bundled with 9 and 20: all outside the File Boundary)

### 22. README count and /review on the agents line (Nit)

- Decision: Contest
- Evidence: `README.md` says "26 skill definitions" and "used by /spec, /plan, /review, /ship and /webperf".

### 23. CLAUDE.md references line omits goldfish review (Nit)

- Decision: Fix
- Edit: added "goldfish review" to the `references/` line.

### 24. AGENTS.md still claims validate catches drift between the three surfaces (Nit)

- Decision: Contest
- Evidence: `AGENTS.md` "Adding or Changing a Skill" now says the validator catches a missing surface or a drifted `description`, not a drifted prompt body (round 1, finding 11).

### 25. Plan constraints omit the persona lint warning (Nit)

- Decision: Fix
- Edit: added a constraint: refer to personas by path in SKILL.md files, because "`X` persona" is linted as a skill reference.

### 26. Which severity labels code-review output uses (Nit)

- Decision: Fix (in the shipped skill)
- Edit: Triage in `skills/code-review-and-quality/SKILL.md` maps the persona's Required to Important, and Optional and Nit to Suggestion.

## Escalations for the owner

1. **Discovery and docs outside the File Boundary** (9, 20, 21): add `humanize` to `skills/using-agent-skills/SKILL.md`, `docs/opencode-setup.md` and `docs/getting-started.md`; add the goldfish fan-out to `references/orchestration-patterns.md`; add `goldfish-reviewer` to the persona lists in `docs/getting-started.md`, `docs/antigravity-setup.md` and `docs/copilot-setup.md`. Seven files.
2. **Unexercised success criteria** (7): run the `/spec` lenses on `spec/5-adopt-egm/spec.md` (two subagents), and decide whether the `not run` path needs a live demonstration.
3. **THIRD_PARTY_NOTICES.md** (17): add an EGM entry or not.

## Escalations resolved (2026-11-18)

The owner approved all three. 1: done in `843ab44` (seven files added to the File Boundary). 2: the `/spec` lenses ran on the spec, report in `spec/5-adopt-egm/goldfish.md`; the `not run` path stays untested. 3: EGM entry added to `THIRD_PARTY_NOTICES.md`.

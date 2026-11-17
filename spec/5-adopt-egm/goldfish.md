# Goldfish review: spec/5-adopt-egm/spec.md

Run on 2026-11-18, after the feature was built, to exercise success criterion 1 on this spec.

| Reviewer | Status |
|---|---|
| comprehension | completed |
| zoom-out | completed |

Counts: 0 Blocker, 12 Major, 10 Nit (22). Decisions: 21 Fix, 1 Decline. No Blocker, so no second round.

## Paraphrase (comprehension)

What this proposes: `/spec`, `/plan` and `/review` get a review step run by two fresh-context subagents ("goldfish") that see only the document and the repo, each through a named lens: `comprehension` and `zoom-out` for specs, `readiness` and `codebase` for plans. The session that wrote the document triages every finding as fix, contest, escalate or decline, writes the results to `goldfish.md` next to the artifact, and then asks for approval. `/review` also runs its code review in a fresh subagent with a spec drift check and, on a PR, posts one inline GitHub review in lowercase. `/spec <issue>` rewrites the issue's title, body and labels, and a `humanize` skill with the writing rules moves into the repo.

How the existing system works, per this document: the commands are defined three times (`.claude/commands/*.md`, `commands/*.toml`, `.gemini/commands/*.toml`) and point at skills under `skills/`, which are copied to `.opencode/skills/`. The session that wrote a spec, plan or diff also judges it. `/review` has a local mode and a PR mode that posts one `gh pr comment` and writes `tasks/<feature>/review/{plan,todo}.md`. The writing rules live in a private skill reading `~/.claude/WRITING.md`. The reviewer added that the document never says what `/spec` and `/plan` do today beyond ending at an approval prompt.

The paraphrase matches the intent. The gap it noted is fixed by the new section 0.

## Findings

Comprehension gaps cannot be contested, so every comprehension finding is a Fix.

### 1. Contradiction stop has no design (Major)

- Lens: comprehension
- Decision: Fix
- Edit: section 3 now says it applies to `/spec` only, triggers on a request that conflicts with the saved spec's Objective, Non-goals or Boundaries, and is detected by the session comparing the two.

### 2. Triage rules use Blocker/Major/Nit, code review does not (Major)

- Lens: comprehension
- Decision: Fix
- Edit: 4.5 step 3 maps Critical to Blocker, Important or Required to Major, Suggestion, Optional or Nit to Nit, and says where contested findings go in each mode. The same mapping is in `skills/code-review-and-quality/SKILL.md`.

### 3. Local mode, PR mode and `/ponytail-review` are undefined (Major)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 describes both `/review` modes, how PR mode is triggered, and what `/ponytail-review` is, linking `.claude/commands/review.md`.

### 4. Feature slug derivation unstated (Major)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 gives both derivations: `<issue number>-<short title>` for an issue, and the branch minus its `<type>/` prefix for `/review`.

### 5. "Failed" and "timed out" undefined (Major)

- Lens: comprehension
- Decision: Fix
- Edit: `references/goldfish-review.md` Flow step 3 defines a failure: the call errors, the harness times it out, or it returns neither a finding nor "no findings".

### 6. How the `.opencode` copy happens (Major)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 and section 5 say the copy is manual (`cp -r`) and the validator only detects staleness.

### 7. How a lens reaches the persona; what "persona support" means (Major)

- Lens: comprehension
- Decision: Fix
- Edit: 4.1 says the spawn prompt template in `references/goldfish-review.md` carries lens, document, readable paths and rubric text. Section 0 defines persona support.

### 8. How a command "puts text through" humanize (Major)

- Lens: comprehension
- Decision: Fix
- Edit: 4.6 says the running session loads the skill and rewrites the text itself, under the rule that no fact or meaning changes.

### 9. Goldfish triage duplicates `doubt-driven-development` (Major)

- Lens: zoom-out
- Decision: Fix
- Edit: `references/goldfish-review.md` has a new section "How this relates to other review steps" that says when to use each and why the stop rule and fallback differ. Section 11 records building on RECONCILE as a rejected alternative.

### 10. No-subagent rule contradicts `/ship`'s in-thread fallback (Major)

- Lens: zoom-out
- Decision: Fix
- Edit: the same reference section explains the difference (`/ship` personas judge code on disk; a goldfish is only useful without the conversation) and links the parallel fan-out pattern in `references/orchestration-patterns.md`.

### 11. File Boundary duplicates each task's "Files likely touched" (Major)

- Lens: zoom-out
- Decision: Fix
- Edit: the planning skill template and spec 4.4 define the File Boundary as the union of the per-task lists, with Create, Modify or Delete added. The readiness rubric now reports any path in one list and not the other.

### 12. Items 2 and 3 have no stated problem (Major)

- Lens: zoom-out
- Decision: Fix
- Edit: section 1 gives the reason for each: issues are opened thin and `/spec` is when the requirements are known; one PR comment cannot anchor, suggest or resolve per finding.

### 13. `/build` boundary stop sits next to Rule 0.5 (Nit)

- Lens: zoom-out
- Decision: Fix
- Edit: the three build command files and spec 14.1 call the stop the hard stop behind Rule 0.5 (Scope Discipline) in `incremental-implementation`.

### 14. Spec Risks repeats the plan's risk table (Nit)

- Lens: zoom-out
- Decision: Fix
- Edit: 4.4 says spec Risks are design risks and the plan's table holds implementation risks.

### 15. 4.6 and 4.7 drop into implementation detail (Nit)

- Lens: zoom-out
- Decision: Decline
- Reason: the detail is already in the shipped skills, and the comprehension lens asked for more of it in the spec (findings 19 and 20), not less.

### 16. `/humanize` used before it is explained (Nit)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 explains the private skill before section 1 uses it.

### 17. "Six required sections" without a link (Nit)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 links `docs/skill-anatomy.md`.

### 18. EGM and conversation terms undefined (Nit)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 points the EGM terms to the EGM README and its review design doc. The "declined by the user" wording in sections 3 and 11 stays, because those are recorded decisions from issue #5's spec session.

### 19. Harness list missing (Nit)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 lists the harnesses from `README.md`.

### 20. `/build auto` not introduced (Nit)

- Lens: comprehension
- Decision: Fix
- Edit: section 0 defines it and links `.claude/commands/build.md`.

### 21. `{owner}/{repo}` source unstated (Nit)

- Lens: comprehension
- Decision: Fix
- Edit: 4.6 says they come from the PR URL or `gh repo view`.

### 22. How diff hunks become valid line numbers (Nit)

- Lens: comprehension
- Decision: Fix
- Edit: 4.6 gives the hunk-header rule and the `--paginate` call, and says `start_line` and `line` must fall in one hunk.

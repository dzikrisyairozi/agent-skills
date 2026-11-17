# Implementation plan: adopt the elephant-goldfish model

Spec: `spec/5-adopt-egm/spec.md`
Issue: https://github.com/dzikrisyairozi/agent-skills/issues/5
Branch: `feat/5-adopt-egm` (created by `/build` off `main`)

## Overview

Add a fresh-context goldfish review to `/spec`, `/plan` and `/review`, make `/spec <issue>` update the GitHub issue, make `/review <PR>` post an inline GitHub review in lowercase, and bring the `humanize` writing rules into the repo. Every change is markdown, TOML or a JSON version string. There is no runtime code, so each task is verified by `npm run validate`, `npm test`, and a grep for the text it adds.

## Decisions on the spec's open questions

| Question | Decision | Task |
|---|---|---|
| `/build` treats the file boundary as an allowlist | Yes. Stop and ask when a task needs a file outside it. Plans without the section skip the check. | T6 |
| Lowercase for issue text too | Yes. One github voice for issues and PR comments. | T2, T4 |
| Show the comprehension paraphrase in chat | Yes, at the `/spec` approval prompt. | T1, T3 |

## Architecture decisions

- **One reference file owns the goldfish protocol.** `references/goldfish-review.md` holds the four lens rubrics, the spawn prompt template, the flow, the triage rules and the report layout. Skills link to it and add only what is specific to them (which lenses, which report path). This follows the repo rule against duplicating content between skills.
- **One persona, four lenses.** `agents/goldfish-reviewer.md` takes the lens as an input. Harnesses with no persona support pass the rubric text from the reference file to a generic subagent.
- **Process text goes in skills, commands stay thin.** Each command gains one to three sentences that point at the skill section. The three command surfaces have identical bodies today (`commands/*.toml` and `.gemini/commands/*.toml` are byte-identical, and both wrap the `.claude/commands/*.md` body in `prompt = """ ... """`), so every command edit is applied three times by hand. The generator named in `scripts/validate-commands.js` (`scratchpad/gen-toml.mjs`) is not in the repo.
- **No JSON or backslashes in command files.** The TOML `"""` string treats `\n` and `\"` as escapes. The review JSON example lives in the skill, and commands refer to it.
- **Severity labels are not unified.** Goldfish document findings use Blocker, Major, Nit. Code review keeps Critical, Important, Suggestion as the command has today.
- **`.opencode/skills` is a plain copy.** Every task that edits a skill re-copies that skill directory, or `validate-opencode-sync.js` fails.

## Constraints found in the code

- `scripts/lib/skill-lint.js` requires these headings in every SKILL.md: `## Overview`, `## When to Use`, `## Common Rationalizations`, `## Red Flags`, `## Verification`. The description must contain "Use when".
- `scripts/validate-artifact-paths.js` scans seven files (`GUARDED_FILES`): `.claude/commands/{spec,plan,build}.md`, the spec and planning skills, `docs/getting-started.md` and `docs/adoption-guide.md`, for any path ending in `spec.md`, `plan.md` or `todo.md`, and fails on shapes other than `spec/<feature-name>/spec.md`, `tasks/<feature-name>/plan.md`, `tasks/<feature-name>/todo.md`. In those seven files, do not write `tasks/<feature-name>/review/plan.md`. `goldfish.md` is not matched and is safe.
- `scripts/lib/skill-lint.js` treats "`X` persona" in a SKILL.md as a skill reference and warns when no skill `X` exists. Refer to a persona by its path (`agents/code-reviewer.md`) instead.
- `scripts/validate-reference-links.js` resolves every `references/*.md` link in a SKILL.md relative to the skill's directory. Shared file: `../../references/goldfish-review.md`. Skill-local file: `references/writing-rules.md`.
- `scripts/validate-commands.js` checks that every command exists on all three surfaces and that the `description` field matches. Prompt bodies are not compared, so a drifted body passes. No description changes in this plan.
- `scripts/validate-versions.js` checks `plugin.json`, `.codex-plugin/plugin.json`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.agents/plugins/marketplace.json`. `package.json` also carries `0.1.0` and is bumped with them.

## File Boundary

Create:

- `references/goldfish-review.md`
- `agents/goldfish-reviewer.md`
- `skills/humanize/SKILL.md`
- `skills/humanize/references/writing-rules.md`
- `.opencode/skills/humanize/SKILL.md`
- `.opencode/skills/humanize/references/writing-rules.md`
- `tasks/5-adopt-egm/goldfish.md` (written by the T10 dogfood run)
- `tasks/5-adopt-egm/review/plan.md`, `tasks/5-adopt-egm/review/todo.md` (written by `/review` in T10)

Modify:

- `skills/spec-driven-development/SKILL.md` and `.opencode/skills/spec-driven-development/SKILL.md`
- `skills/planning-and-task-breakdown/SKILL.md` and `.opencode/skills/planning-and-task-breakdown/SKILL.md`
- `skills/code-review-and-quality/SKILL.md` and `.opencode/skills/code-review-and-quality/SKILL.md`
- `.claude/commands/spec.md`, `commands/spec.toml`, `.gemini/commands/spec.toml`
- `.claude/commands/plan.md`, `commands/plan.toml`, `.gemini/commands/plan.toml`
- `.claude/commands/build.md`, `commands/build.toml`, `.gemini/commands/build.toml`
- `.claude/commands/review.md`, `commands/review.toml`, `.gemini/commands/review.toml`
- `agents/code-reviewer.md`
- `docs/agents.md`, `README.md`, `AGENTS.md`, `CLAUDE.md`
- Added after goldfish escalations 9, 17, 20 and 21 (approved 2026-11-18): `skills/using-agent-skills/SKILL.md` and its `.opencode` copy, `docs/opencode-setup.md`, `docs/getting-started.md`, `docs/antigravity-setup.md`, `docs/copilot-setup.md`, `references/orchestration-patterns.md`, `THIRD_PARTY_NOTICES.md`
- `package.json`, `plugin.json`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.codex-plugin/plugin.json`, `.agents/plugins/marketplace.json`
- `tasks/5-adopt-egm/todo.md` (task status)

Delete: none.

## Task list

### Phase 1: foundation

#### Task 1: goldfish reference and persona

**Description:** Write the protocol every later task links to.

`references/goldfish-review.md`, sections in this order:

1. Credit line: adapted from google/elephant-goldfish-model (Apache-2.0), with the repo URL. Text is rewritten, not copied.
2. Roles: elephant (the session with context, writes and triages), goldfish (fresh subagent, read-only, sees the document and the repo), human (decides).
3. Lenses. One subsection per lens with its rubric:
   - `comprehension`: start with a paraphrase under two labels, "What this proposes" and "How the existing system works, per this document", two to five sentences each. A section that cannot be paraphrased is a Blocker. Then flag undefined project-specific terms, assumed setup steps, and explanations that say a thing happens without saying how. General engineering terms are not findings.
   - `zoom-out`: is the objective and its reason stated (if not, stop and report that alone); does it solve the root problem; what is built for a need nobody has yet; what does the repo already have that this reinvents (the reviewer must search the code and name the file); what is optimized before a measured problem; where does a high-level section drop into loop-level detail.
   - `readiness`: the zero-decision rule (the implementer chooses no schema, signature, dependency or error strategy the plan left open); each step names file, symbol and change; each task has a test or check; phases leave the repo passing; the File boundary exists and marks each path Create, Modify or Delete.
   - `codebase`: every file, function, command, flag and path the document says exists does exist; described current behavior matches the code (a contradiction is a Blocker); a path marked Create is not a finding; if paths are listed without Create, Modify or Delete, report that once as Major.
4. Spawn prompt template, exactly these slots and no others: `Lens`, `Document` (path), `Also readable` (paths the document cites, for `/plan` the spec), `Rubric` (the lens subsection). A line under the template states that the prompt must not include a summary of the conversation, the author's intent, or expected findings.
5. Finding format: severity (Blocker, Major, Nit), location (heading or line range), problem, why.
6. Flow: save the draft, spawn the lenses in parallel, retry a failed or empty reviewer once, then mark it `not run`; triage; write the report; apply fixes; one round by default, a second round only when a fix resolved a Blocker, further rounds only on request.
7. Triage: Fix, Contest, Escalate, Decline, with the four rules from spec 4.3.
8. No-subagent fallback: print "goldfish review not run: no subagent support", write the same in the report, continue to approval. Do not review in the main thread.
9. Report layout: the `goldfish.md` template from spec 4.3.
10. What to show in chat: the comprehension paraphrase when that lens ran, escalations first, counts by severity and decision, the report path.

`agents/goldfish-reviewer.md`: frontmatter `name: goldfish-reviewer` and a description in the style of the other personas. Body: read-only, no conversation history, knows software but not this project, takes a lens and a document path, reads the rubric from `references/goldfish-review.md`, returns findings in the finding format, does not narrate tool use, does not invoke other personas. End with a Composition section like `agents/code-reviewer.md` has: invoked by `/spec`, `/plan`.

`docs/agents.md`: add a row for `goldfish-reviewer` to the persona table (role: fresh-context document reviewer; use: spec and plan review in `/spec` and `/plan`).

**Acceptance criteria:**
- [ ] The reference file has all ten sections and four lens rubrics.
- [ ] The spawn template has four slots and none for conversation context.
- [ ] The persona file has valid frontmatter and is listed in `docs/agents.md`.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `grep -c "^### Lens:" references/goldfish-review.md` returns 4; `grep -in "conversation" references/goldfish-review.md` shows only the prohibition lines.

**Dependencies:** None

**Files:** `references/goldfish-review.md`, `agents/goldfish-reviewer.md`, `docs/agents.md`

**Scope:** S

#### Task 2: humanize skill

**Description:** Bring the writing rules into the repo and add the github voice.

`skills/humanize/references/writing-rules.md`: the content of `~/.claude/WRITING.md` with three edits that remove private-project wording: "including in marketing copy for this repo's sites" becomes "including in marketing copy"; "Site copy follows whatever the file already uses" becomes "Prose in a file follows whatever the file already uses"; the title becomes "Writing rules". Everything else is kept as written.

Append a section "Github voice" to that file with the four rules from spec 4.8, plus one example pair (a model-sounding review comment, then the lowercase rewrite with `README.md` and a code span left intact).

`skills/humanize/SKILL.md`: frontmatter `name: humanize`, description "Rewrites text so it reads like a person wrote it, using the rules in this skill's writing-rules reference. Use when writing or rewriting prose that people will read: issue text, PR comments, commit messages, docs. Use when the user asks to humanize or de-AI a text." Sections:

- `## Overview`: two sentences. The root problem is vague, inflated claims; the word list is secondary.
- `## When to Use`: pasted text, a file, a commit or PR body, the previous reply, and any text a command is about to post to GitHub.
- `## Process`: the six steps from the current `~/.claude/skills/humanize/SKILL.md` "Do the pass", with step 1 reading `references/writing-rules.md`, and step 6 reworded to "For non-English text, apply the structural rules and skip the English word list."
- `## Github voice`: when the target is an issue title, issue body or PR review comment, apply the Github voice section of the reference after the normal pass. Code and suggestion blocks are never changed.
- `## Report`: the three bullets from the current skill.
- `## Common Rationalizations`: table with three rows (stripping the word list is enough; adding a specific detail I am not sure of; lowercase means sloppy).
- `## Red Flags`: invented specifics, changed code blocks, changed identifiers' casing, a rewrite of text that was already clean.
- `## Verification`: checklist (facts and numbers unchanged, no code block touched, github text lowercase with identifiers intact, no placeholder left).

Copy `skills/humanize/` to `.opencode/skills/humanize/`.

**Acceptance criteria:**
- [ ] The skill passes lint and reads no path outside the repo.
- [ ] The reference file has a "Github voice" section with the four rules.
- [ ] `.opencode/skills/humanize/` matches `skills/humanize/`.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `grep -rn "\.claude/WRITING\|~/" skills/humanize/` returns nothing.

**Dependencies:** None

**Files:** `skills/humanize/SKILL.md`, `skills/humanize/references/writing-rules.md`, and their two `.opencode` copies

**Scope:** S

### Checkpoint: foundation

- [ ] `npm run validate` and `npm test` pass
- [ ] Two commits on `feat/5-adopt-egm`

### Phase 2: /spec

#### Task 3: goldfish step and template sections in /spec

**Description:** `skills/spec-driven-development/SKILL.md`:

- In the spec template code block, add `## Non-goals` after `## Boundaries`, then `## Alternatives Considered` ("Each rejected option and the concrete reason"), then `## Risks`, before `## Success Criteria`.
- After the "Reframe instructions as success criteria" block and before `### Phase 2: Plan`, add `### Goldfish review` with: run the protocol in `../../references/goldfish-review.md` with the `comprehension` and `zoom-out` lenses once the draft is saved; write the report to `spec/<feature-name>/goldfish.md`; show the paraphrase, escalations and counts; then ask for approval.
- In the same subsection, two one-line rules: this phase writes only spec artifacts, never source files; if a new request contradicts the saved spec, name the contradiction and ask before editing any file.
- Add a row to Common Rationalizations ("I wrote it, so I know it is clear": the author cannot see what the document leaves out), a Red Flag ("Asking for approval before the goldfish report exists"), and a Verification checkbox ("Goldfish report written, every finding has a decision").

Commands, all three surfaces: after the sentence that saves the spec, replace "then confirm with the user before proceeding" with "then run the Goldfish review step of the skill and ask for approval once `spec/<feature-name>/goldfish.md` is written".

Copy the skill to `.opencode/skills/spec-driven-development/`.

**Acceptance criteria:**
- [ ] The template has the three new sections.
- [ ] The skill names both lenses and the report path, and links the reference.
- [ ] The three spec command files carry the same new sentence.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `grep -c "goldfish" .claude/commands/spec.md commands/spec.toml .gemini/commands/spec.toml` gives the same count in each; `diff commands/spec.toml .gemini/commands/spec.toml` is empty.

**Dependencies:** T1

**Files:** `skills/spec-driven-development/SKILL.md` and its `.opencode` copy, the three spec command files

**Scope:** M

#### Task 4: issue sync in /spec

**Description:** In the same skill, add `### GitHub issue sync` after `### Goldfish review`, with the six rules of spec 4.7 as numbered steps and the two commands:

```
gh issue view <n> --json title,body,labels
gh issue edit <n> --title "<title>" --body-file <path> --add-label <label>
```

State: write the body to a temp file; add `--repo <owner>/<name>` when the URL names another repo; labels come only from `gh label list`; title and body go through the `humanize` skill with github voice; body contains the problem, what changes, an acceptance checklist from the success criteria, and the spec path; no footer; on `gh` failure report one line and continue.

Add a Verification checkbox: "If an issue was given: title, body and labels updated, no label created."

Commands, all three surfaces: add one paragraph at the end. "If the argument is a GitHub issue URL or `#<number>`, read the issue first as input, and after the goldfish review follow the GitHub issue sync step of the skill to update its title, description and labels. Do not ask before editing the issue."

Copy the skill to `.opencode`.

**Acceptance criteria:**
- [ ] The skill section covers all six rules and both commands.
- [ ] The three spec command files carry the same paragraph.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `diff commands/spec.toml .gemini/commands/spec.toml` is empty; `grep -n "gh issue edit" skills/spec-driven-development/SKILL.md` finds the command.

**Dependencies:** T2, T3

**Files:** same five files as T3

**Scope:** S

### Checkpoint: /spec

- [ ] `npm run validate` and `npm test` pass
- [ ] Read `.claude/commands/spec.md` top to bottom: the order is questions, grounding, draft, save, goldfish, issue sync, approval (re-sync if the spec changes during approval)

### Phase 3: /plan and /build

#### Task 5: file boundary, zero-decision rule and goldfish step in /plan

**Description:** `skills/planning-and-task-breakdown/SKILL.md`:

- In Step 4, after the task structure block, add a paragraph "Zero-decision rule": the implementer makes no design choice the plan left open. Include the pair: weak, "update the auth service"; ready, "add `expiresAt: Date` to `Session` in `src/auth/session.ts` and reject expired sessions in `requireSession()`".
- In the Plan Document Template code block, add `## File Boundary` after `## Architecture Decisions`, with three sub-lists Create, Modify, Delete and the line "Every path the tasks touch, repo-relative. A glob only for generated output."
- Add `### Step 6: Goldfish review` after Step 5: run the protocol in `../../references/goldfish-review.md` with the `readiness` and `codebase` lenses after the plan is saved; the reviewers may also read the spec; write `tasks/<feature-name>/goldfish.md`; then present the plan. One line: planning writes only plan artifacts, never source files.
- Verification: add "The plan has a File Boundary with Create, Modify or Delete on every path" and "Goldfish report written, every finding has a decision".
- Red Flags: add "A task step that names a goal but no file or symbol".

Commands, all three surfaces: change step 6 from "Present the plan for human review" to "Run the Goldfish review step of the skill, then present the plan for human review", and add to the save paragraph: "The plan includes a File Boundary section, and the goldfish report is saved next to it as `goldfish.md`."

Copy the skill to `.opencode`.

**Acceptance criteria:**
- [ ] The plan template has File Boundary and the skill has Step 6 with both lenses.
- [ ] The three plan command files carry the same changes.

**Verification:**
- [ ] `npm run validate` and `npm test` pass (artifact-path validator in particular).
- [ ] `diff commands/plan.toml .gemini/commands/plan.toml` is empty.

**Dependencies:** T1

**Files:** `skills/planning-and-task-breakdown/SKILL.md` and its `.opencode` copy, the three plan command files

**Scope:** M

#### Task 6: /build stops outside the file boundary

**Description:** In `.claude/commands/build.md`, add a section `## File boundary` after `## Branch safety`:

"If `tasks/<feature-name>/plan.md` has a File Boundary section, it is the list of files this build may create, modify or delete. Before editing a file that is not on it, stop and ask, in both modes. If the user approves, add the path to the boundary in the plan and continue. Task-status updates in the task list and files under `spec/` and `tasks/` are always allowed. A plan with no File Boundary section skips this check."

In the `/build auto` stop-and-ask list (step 7), add a bullet: "a task needs a file outside the plan's File Boundary".

Apply the same two edits to `commands/build.toml` and `.gemini/commands/build.toml`.

**Acceptance criteria:**
- [ ] The section and the bullet exist in all three build files.
- [ ] The text says plans without the section skip the check.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `diff commands/build.toml .gemini/commands/build.toml` is empty; `grep -c "File Boundary" .claude/commands/build.md` is at least 3.

**Dependencies:** T5

**Files:** the three build command files

**Scope:** S

### Checkpoint: /plan and /build

- [ ] `npm run validate` and `npm test` pass
- [ ] This plan's own File boundary section matches the heading name the skill now uses (rename to "File Boundary" if needed)

### Phase 4: /review

#### Task 7: fresh-context review, drift check and triage

**Description:** `skills/code-review-and-quality/SKILL.md`: add `## Fresh-Context Review` after `## Review Process`, before `## Multi-Model Review Pattern`:

- Why: the session that built the code reviews its own assumptions.
- How: spawn the `code-reviewer` persona as a subagent with the diff, the changed files, and the feature's spec and plan when they exist. No conversation summary. If the harness cannot spawn one, say "fresh-context review not run" in the output and review in the main session.
- `### Drift check`: only when a spec exists. The four categories from spec 4.5 as a table with one example each. Each drift finding states which side should change, spec or code, and the user chooses.
- `### Triage`: the main session applies the triage rules in `../../references/goldfish-review.md` to the subagent's findings. A contested finding is not posted and is kept in the local review file with its evidence.

Add a Red Flag ("Reviewing a diff in the same session that wrote it, with no fresh-context pass") and a Verification checkbox ("Fresh-context review ran, or the output says it did not").

`agents/code-reviewer.md`: add `### Spec Drift` to the Review Output Template after `### Nits` ("[category] [file:line] [what the spec says, what the code does, which should change]", or "none found", or "no spec"), and rule 7: "When given a spec, check the diff against it in the four drift categories."

`.claude/commands/review.md`, `commands/review.toml`, `.gemini/commands/review.toml`: insert before the five-axis list: "Run the review through the Fresh-Context Review section of the skill: the `code-reviewer` persona in a fresh subagent, given the diff plus `spec/<feature-name>/spec.md` and the feature's plan when they exist (derive `<feature-name>` from the branch name), with the drift check. Then triage its findings before reporting."

Copy the skill to `.opencode`.

**Acceptance criteria:**
- [ ] The skill has the section with Drift check and Triage subsections.
- [ ] The persona template has Spec Drift.
- [ ] The three review command files carry the same paragraph.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `diff commands/review.toml .gemini/commands/review.toml` is empty.

**Dependencies:** T1

**Files:** `skills/code-review-and-quality/SKILL.md` and its `.opencode` copy, `agents/code-reviewer.md`, the three review command files

**Scope:** M

#### Task 8: PR mode posts an inline review in lowercase

**Description:** `skills/code-review-and-quality/SKILL.md`: add `## Posting a PR Review` after `## Fresh-Context Review`:

1. Get owner, repo and number from the PR URL, or `gh repo view --json nameWithOwner` for `#<n>`.
2. Get the changed lines: `gh api repos/<owner>/<repo>/pulls/<n>/files`. From each file's `patch`, the hunk headers give the valid right-side line ranges.
3. Build the review JSON in the scratchpad: `event` is always `COMMENT`; `body` is the summary; `comments` has one entry per finding on a valid line (`path`, `line`, `side: "RIGHT"`, plus `start_line` for a range).
4. Include the JSON example from spec 4.6.
5. A `suggestion` block only when the fix is an exact replacement of the commented lines. Otherwise describe the change and show a short code block.
6. A finding on a line outside the diff goes in the summary body with its `file:line`.
7. Post: `gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input <file>`. On a 422, move the rejected comment into the body and post once more.
8. Summary body: verdict first, counts by severity, findings that could not be inline, the ponytail section.
9. Every body goes through the `humanize` skill with github voice before the JSON is written. Code and suggestion blocks are untouched.

Commands, all three surfaces: replace the whole "### 1. Post the review as a PR comment" subsection with "### 1. Post the review on the PR": two sentences pointing at the skill's Posting a PR Review section and stating the result (one review, event `COMMENT`, inline comments with suggested changes, lowercase via the `humanize` skill's github voice, no permission prompt). Remove the sentence "Prose in the PR comment is written normally". Leave "### 2. Save an actionable plan" as is, and add one sentence to it: local files keep normal sentence case and include contested findings with their evidence.

Copy the skill to `.opencode`.

**Acceptance criteria:**
- [ ] The skill section has the nine steps and the JSON example.
- [ ] No review command file mentions `gh pr comment`.
- [ ] No command file contains a backslash escape or JSON.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `grep -rn "gh pr comment" .claude/commands commands .gemini/commands` returns nothing; `diff commands/review.toml .gemini/commands/review.toml` is empty.

**Dependencies:** T2, T7

**Files:** same skill and three command files as T7

**Scope:** M

### Checkpoint: /review

- [ ] `npm run validate` and `npm test` pass
- [ ] Read `.claude/commands/review.md` top to bottom: fresh-context review and triage, five axes, ponytail, then PR mode steps 1 and 2

### Phase 5: docs, version, dogfood

#### Task 9: docs and version 0.2.0

**Description:**

- `CLAUDE.md`: add `humanize` to the **Review:** line of Skills by Phase, add `goldfish-reviewer` to the `agents/` line of Project Structure, and two lines at the end of Conventions (goldfish review in `/spec` and `/plan`; GitHub text goes through `humanize` with github voice).
- `AGENTS.md` has no skills-by-phase list, persona list or Conventions section. Append "(text posted to GitHub goes through `humanize` with its github voice)" to the `REVIEW` line of Lifecycle Mapping; append one sentence about the two `goldfish-reviewer` instances to the paragraph ending "Do not build a \"router\" persona."; and replace the TOML-generator pointer in Adding or Changing a Skill (no generator exists) with "edit all three surfaces in the same change".
- `README.md`: in the command table, update the `/spec`, `/plan` and `/review` rows to mention the goldfish review and, for `/review`, the inline PR review. Add one short paragraph "Elephant and goldfish" that says what the roles are and credits google/elephant-goldfish-model with a link. Update the `agents/*.md` line in the layout block to include `/spec` and `/plan`.
- Version `0.1.0` to `0.2.0` in `package.json`, `plugin.json`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.codex-plugin/plugin.json`, `.agents/plugins/marketplace.json`.

**Acceptance criteria:**
- [ ] The three docs name the new skill and persona.
- [ ] Six files read `0.2.0` and none reads `0.1.0`.

**Verification:**
- [ ] `npm run validate` and `npm test` pass.
- [ ] `grep -rn "0\.1\.0" --include=*.json . --exclude-dir=node_modules` returns nothing.

**Dependencies:** T1 to T8

**Files:** `CLAUDE.md`, `AGENTS.md`, `README.md`, six JSON files (nine files, all one-line or one-paragraph edits)

**Scope:** M

#### Task 10: dogfood

**Description:** Use the new commands on this feature.

1. Run the `/plan` goldfish step on `tasks/5-adopt-egm/plan.md` (lenses `readiness` and `codebase`). Write `tasks/5-adopt-egm/goldfish.md`. Fix what triage says to fix, in the plan or in the shipped files.
2. Run the issue sync step on issue #5 again. It is idempotent, so no throwaway issue is needed. Confirm the label set is unchanged and nothing was created.
3. **Ask first:** push the branch and open the PR. Then run `/review` on it and confirm one review with inline comments, lowercase prose, and `tasks/5-adopt-egm/review/` files.

**Acceptance criteria:**
- [ ] `tasks/5-adopt-egm/goldfish.md` lists both reviewers with a status and every finding has a decision.
- [ ] Issue #5 still has only labels that existed before.
- [ ] After approval to push: the PR shows one review of type comment with at least one inline comment.

**Verification:**
- [ ] `gh issue view 5 --json labels`
- [ ] `gh api repos/dzikrisyairozi/agent-skills/pulls/<n>/reviews --jq '.[].state'` shows `COMMENTED`

**Dependencies:** T9

**Files:** `tasks/5-adopt-egm/goldfish.md`, plus any fix inside the File boundary

**Scope:** S

### Checkpoint: complete

- [ ] All ten success criteria in spec section 13 are checked
- [ ] `npm run validate` and `npm test` pass
- [ ] `/quiz` comprehension gate passed (the `/build` hand-off rule)

## Parallel work

T1 and T2 are independent. After T1: T3, T5 and T7 touch different skills and different command files and can run in parallel. T4 follows T3, T6 follows T5, T8 follows T7, each in the same files.

## Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| TOML mirrors drift from the `.md` body, since only `description` is validated | Medium | Each command task ends with `diff` of the two TOML files and a matching grep count across all three |
| An escape sequence inside a TOML `"""` string breaks parsing on Gemini or Antigravity | Medium | No JSON or backslashes in command files; examples live in skills |
| A new path string trips `validate-artifact-paths.js` | Low | Guarded files only use `goldfish.md` and the three approved shapes; validator runs after every task |
| `.opencode/skills` copy forgotten | Low | Re-copy is a step in every skill task; validator fails otherwise |
| Skills grow long and cost context on every run | Medium | Protocol text is in one reference file, loaded only when the step runs |
| The installed plugin in `~/.claude` still serves 0.1.0 until reinstalled | Low | Noted for the user; reinstall after merge |
| Other callers pick up the changed skills: `/build auto` step 4 now runs the plan goldfish step; `/ship` and `doubt-driven-development` get the persona's Spec Drift section; `/review` inside a subagent cannot spawn one | Low | Intended for the first two. Spec Drift reports "no spec" when none is given. The nested case hits the skill's "fresh-context review not run" fallback |

## Open questions

None blocking. One follow-up outside this repo: point `~/.claude/CLAUDE.md` at the repo's writing rules after the plugin is reinstalled, and retire the private `humanize` skill.

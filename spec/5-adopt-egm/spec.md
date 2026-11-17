# Spec: adopt the elephant-goldfish model in /spec, /plan and /review

Issue: https://github.com/dzikrisyairozi/agent-skills/issues/5
Source model: https://github.com/google/elephant-goldfish-model (EGM, Apache-2.0), read at commit state of 2026-11-17.

## 1. Objective

`/spec`, `/plan` and `/review` are run daily, and today the same session that wrote a spec, plan or diff also judges it. That session already holds the missing context in memory, so it cannot see what the document fails to say. EGM's answer is two roles: an elephant (the session with full context) writes, and goldfish (fresh subagents that get only the document and the repo) review. The human decides on the findings.

This change adds three things:

1. A goldfish review step inside `/spec`, `/plan` and `/review`, ported natively (no dependency on the egm plugin).
2. `/spec <github issue>` rewrites the issue's title, description and labels once the spec is saved.
3. `/review <PR>` posts a real GitHub review with inline comments and suggested changes, written in lowercase, instead of one long comment.

Items 2 and 3 put text on GitHub, so the writing rules behind `/humanize` move into this repo as a skill.

User: the repo owner, on every harness this pack installs to.

Success looks like: a spec or plan that reaches the approval prompt has already been read by two reviewers who never saw the conversation, their findings are on disk with a recorded decision for each, and nothing posted to GitHub reads like model output.

## 2. Decisions already made

| Question | Answer |
|---|---|
| Use the egm plugin or port the ideas | Port natively. EGM writes `concepts_brief.md` and `docs/designs/`, which conflicts with `spec/<feature>/` and `tasks/<feature>/`. |
| When goldfish runs in `/spec` and `/plan` | Always, two reviewers each. No flag. |
| Where the writing rules live | In this repo, as `skills/humanize/`. |
| What `/review` posts on a PR | One GitHub review, event `COMMENT`, with a summary body and inline comments. |

## 3. What we take from EGM, and what we skip

Taken:

- Fresh-context reviewers that receive the document path and read access to the repo, and nothing from the conversation.
- Four of EGM's reviewer lenses (table in 4.1).
- "Paraphrase first": the comprehension reviewer restates what the document proposes before listing findings. A wrong paraphrase is a finding.
- The zero-decision rule for plans: the implementer makes no design choice the plan did not make.
- An explicit file boundary in the plan, each path marked Create, Modify or Delete.
- The triage categories from `egm-resolve`: fix, contest, escalate, decline, with the rule that a contest must cite evidence the reviewer could have read.
- A reviewer that fails or times out is recorded as `not run`. It is never counted as zero findings.
- Reports go to a file. Chat gets a count and a path.
- Spec template gains Non-goals, Alternatives considered, and Risks (from EGM's feature-spec standard).
- Drift check: `/review` compares the diff against the feature's spec and plan.
- Contradiction stop: if a new request contradicts the saved spec, stop and ask before editing any file.

Skipped, with the reason:

- `egm-brainstorm` and its 7-lens swarm: `idea-refine` and `interview-me` already cover pre-spec exploration.
- `inventive_catalyst` (TRIZ), `structural`, `style`, `design_critic` reviewers: this is the "full pipeline" option that was declined. `humanize` covers style.
- Running two instances (A and B) of each reviewer: doubles cost for recall we have not shown we need.
- `egm-bootstrap`: documents an existing codebase, unrelated to this workflow.
- A standalone drift-report command: folded into `/review`.
- The two-clean-rounds loop: replaced by a one-round default (4.3).

## 4. Design

### 4.1 Reviewers

One persona, `agents/goldfish-reviewer.md`, parameterized by a lens. The rubrics live in `references/goldfish-review.md`, so harnesses without persona support can hand the same text to a generic subagent.

| Lens | Runs in | Asks |
|---|---|---|
| `comprehension` | `/spec` | Can a reader who knows software but not this project restate what is proposed? Which project-specific terms are undefined? What setup is assumed? |
| `zoom-out` | `/spec` | Does this solve the root problem the simplest way? What is speculative? What already exists in the repo that this reinvents (must search the code)? |
| `readiness` | `/plan` | Could I build this now without making a design choice? Are steps specific (file, symbol, change)? Is there a test for each task? |
| `codebase` | `/plan` | Does every file, function, command and path the plan says exists actually exist, and does described current behavior match the code? |

Rules for every lens:

- Read-only. A goldfish does not edit files.
- Input is the document path, the lens name, and the rubric. The spawning prompt must not summarize the conversation or state what the author intended.
- For `/plan`, the goldfish may also read the spec, since the plan cites it.
- Output per finding: severity (Blocker, Major, Nit), location (heading or line), what is wrong, why. Plain text back to the caller.
- A path the plan marks Create is not a missing-reference finding.

### 4.2 Flow in `/spec` and `/plan`

```
draft saved to disk
  -> spawn 2 goldfish in parallel
  -> collect findings (retry a failed reviewer once, then mark it not run)
  -> elephant triages each finding
  -> write goldfish.md, apply the fixes to the document
  -> show the user: counts, escalations, path
  -> ask for approval
```

Report paths: `spec/<feature>/goldfish.md` and `tasks/<feature>/goldfish.md`.

If the harness cannot spawn a fresh-context subagent, the command says so in one line ("goldfish review not run: no subagent support"), records the same in `goldfish.md`, and continues to the approval prompt. It does not simulate the review in the main thread, because that is the self-review this change exists to replace.

### 4.3 Triage

The session that wrote the document gives each finding one decision:

- **Fix**: the finding is right. Edit the document.
- **Contest**: the finding is wrong, and a file, symbol or a section of the document proves it. Cite that.
- **Escalate**: the finding is wrong, but the only proof is something the reviewer could not read (a verbal decision, a team habit). The user decides. Escalations are listed first in the chat message.
- **Decline**: a Nit not worth the edit. Recorded, not fixed.

Rules: a comprehension gap cannot be contested (if a fresh reader could not follow it, context is missing). Severity is never changed; disagree by contesting. "This was intentional" is not evidence.

One round by default. A second round runs only when the first round's fixes resolved a Blocker, and it stops after that. More rounds happen only if the user asks.

`goldfish.md` layout:

```markdown
# Goldfish review: spec/5-adopt-egm/spec.md

| Reviewer | Status |
|---|---|
| comprehension | completed |
| zoom-out | not run (timed out twice) |

## Paraphrase (comprehension)
What this proposes: ...
How the existing system works, per this document: ...

## Findings

### 1. Plan references a validator that does not exist (Blocker)
- Lens: codebase
- Where: "Testing strategy"
- Problem: ...
- Decision: Fix
- Evidence or edit: replaced `validate-agents.js` with `validate-skills.js` (scripts/validate-skills.js exists)
```

### 4.4 Template changes

`spec-driven-development` spec template adds three sections: Non-goals, Alternatives considered (each rejected option with a concrete reason), Risks.

`planning-and-task-breakdown` adds:

- A "File boundary" section in `plan.md`: every repo-relative path, marked Create, Modify or Delete. A glob is allowed only for generated output.
- The zero-decision rule in the task-writing step, with one before/after example ("update the auth service" versus "add `expiresAt` to `Session` in `src/auth/session.ts`").

Both skills add one line to their process: these commands write only spec and plan artifacts, never source files.

### 4.5 `/review` changes

Order of work in every mode:

1. **Goldfish code review.** Spawn the existing `code-reviewer` persona as a fresh subagent with the diff, the changed files, and `spec/<feature>/spec.md` plus `tasks/<feature>/plan.md` when they exist. It gets no conversation summary. This matters most when the session running `/review` is the one that ran `/build`.
2. **Drift check**, same subagent, only when a spec exists. Four categories: a feature the spec describes that the diff lacks; a model or type missing specified fields; a hardcoded value or stub where the spec describes real logic; a different pattern or dependency than the spec chose. Each drift finding says which side should change, spec or code, and leaves the choice to the user.
3. **Triage** by the main session using 4.3. A contested finding is dropped from what gets posted and kept in the local file with its evidence.
4. `/ponytail-review` add-on, unchanged.
5. Output. Local mode prints the review as today. PR mode does 4.6 and still writes `tasks/<feature>/review/{plan,todo}.md` as today.

Feature slug for step 1 comes from the branch name, the same derivation PR mode already uses.

### 4.6 PR mode posting

Replace the single `gh pr comment` with one review:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/reviews --method POST --input <scratchpad>/review.json
```

```json
{
  "event": "COMMENT",
  "body": "looks good overall, 1 thing to fix before merge and 2 smaller ones.\n\n...",
  "comments": [
    {
      "path": "scripts/validate-commands.js",
      "line": 48,
      "side": "RIGHT",
      "body": "this regex misses CRLF frontmatter on windows, so `description` comes back null.\n\n```suggestion\n  const match = content.match(/^---[ \\t]*\\r?\\n([\\s\\S]*?)\\r?\\n---/);\n```"
    }
  ]
}
```

Rules:

- One inline comment per finding that maps to a line in the diff. Multi-line findings use `start_line` and `line`.
- A `suggestion` block is included only when the fix is an exact replacement of the commented lines. Otherwise the comment describes the change and shows a short code block.
- Before posting, check each `path` and `line` against `gh pr diff`. A finding on a line outside the diff goes into the summary body with its `file:line`. If the API still returns 422, move the rejected comment into the body and post once more.
- Summary body: verdict first, then counts by severity, then findings that could not be inline, then the ponytail section. No headings for a short review.
- Event is always `COMMENT`. GitHub rejects `APPROVE` and `REQUEST_CHANGES` on your own PR, and a comment review works everywhere.
- Every body goes through the `humanize` skill's github voice (4.8) before the JSON is written.

### 4.7 `/spec` with a GitHub issue

Trigger: an argument that is an issue URL or `#<n>`.

1. Start: `gh issue view <n> --json title,body,labels` (with `--repo owner/name` when the URL names another repo). The issue text is input to the spec.
2. After the spec is saved and goldfish triage is done: `gh issue edit <n> --title ... --body-file ... --add-label ...`. No confirmation prompt; the user asked for this by passing the issue.
3. Title: one line that names the change. Body: the problem in two or three sentences, what will change, an acceptance checklist taken from the spec's success criteria, and the spec path. No "generated by" footer.
4. Labels: chosen only from `gh label list`. Never create a label. Never remove one that is already set.
5. The old body is replaced. GitHub keeps the edit history on the issue, so the original is recoverable there.
6. If `gh` fails (no auth, no permission), report the error in one line and finish the spec normally.

Title and body go through `humanize` github voice.

### 4.8 `humanize` skill

New `skills/humanize/SKILL.md` with the six required sections, and `skills/humanize/references/writing-rules.md` holding the rules that are in `~/.claude/WRITING.md` today. The skill reads its own reference file, not a path in the user's home directory.

It adds one section the current skill lacks, **github voice**, for issue text and PR review comments:

- Lowercase sentence starts, including the first word of a comment. Identifiers, file paths, code, acronyms and proper nouns keep their real casing (`README.md`, `CI`, `GitHub`).
- Short and direct, second person, one point per comment. No severity banners in bold, no emoji, no "great work overall" opener, no sign-off.
- Severity is said in words when it matters ("this one blocks merge", "nit:").
- Code blocks and suggestion blocks are never rewritten.

Local artifacts (`goldfish.md`, `review/plan.md`) keep normal sentence case. Lowercase applies to text posted to GitHub.

## 5. Project structure

Create:

```
agents/goldfish-reviewer.md
references/goldfish-review.md                  # lenses, rubrics, triage rules, report layout
skills/humanize/SKILL.md
skills/humanize/references/writing-rules.md
.opencode/skills/humanize/**                   # copy, kept in sync by the validator
```

Modify:

```
skills/spec-driven-development/SKILL.md        # template sections, goldfish step, issue update, contradiction stop
skills/planning-and-task-breakdown/SKILL.md    # file boundary, zero-decision rule, goldfish step
skills/code-review-and-quality/SKILL.md        # fresh-context review, drift check, triage, PR posting
.opencode/skills/<the three above>/SKILL.md    # re-copied
.claude/commands/{spec,plan,review}.md
commands/{spec,plan,review}.toml
.gemini/commands/{spec,plan,review}.toml
agents/code-reviewer.md                        # drift check in the output template
docs/agents.md, README.md, AGENTS.md, CLAUDE.md  # list the new persona and skill
package.json, plugin.json, .claude-plugin/plugin.json, .claude-plugin/marketplace.json, .codex-plugin/plugin.json  # 0.1.0 -> 0.2.0
```

The process lives in the skills and the reference file. The command files stay short and point at them, so the three command surfaces carry little text that can drift.

## 6. Commands

```
npm run validate     # skill frontmatter and sections, command parity, versions, artifact paths, reference links, .opencode sync
npm test             # node --test on the validators
claude --plugin-dir .   # local smoke test of the commands
```

## 7. Code style

Skill and persona files follow `docs/skill-anatomy.md`. Headings in sentence case. Prose follows `skills/humanize/references/writing-rules.md`. EGM text is not copied verbatim; rubrics are rewritten for this repo, and `references/goldfish-review.md` credits EGM with a link and its Apache-2.0 license.

Example of the target register for a command step:

```markdown
After the plan is saved, run the goldfish review in `references/goldfish-review.md` with the
`readiness` and `codebase` lenses. Triage the findings, write `tasks/<feature-name>/goldfish.md`,
then ask for approval.
```

## 8. Testing strategy

This repo has no runtime code in the changed paths, so the checks are the validators plus dogfooding.

- `npm run validate` and `npm test` pass. The existing validators already cover the new skill (lint), the new reference links, command parity, version agreement and the `.opencode` copy. No new validator is planned.
- Dogfood `/plan` on this feature: the run must produce `tasks/5-adopt-egm/goldfish.md` with both reviewers listed and a decision on every finding.
- Dogfood `/review` on this feature's PR: one review appears on the PR with at least one inline comment, all prose lowercase, and `tasks/5-adopt-egm/review/{plan,todo}.md` exist.
- Dogfood `/spec` on a throwaway issue: title, body and label change, and no label is created.
- Isolation check: grep the spawn prompt templates in `references/goldfish-review.md` and confirm they contain no slot for conversation context.

## 9. Boundaries

- Always: spawn goldfish with the document path and rubric only; record every finding and its decision in `goldfish.md`; keep the three command surfaces in step; run `npm run validate` and `npm test` before committing; pick labels from the existing list.
- Ask first: adding a reviewer lens beyond the four; more than two review rounds; anything under `.claude-plugin/`, `.codex-plugin/`, `.agents/plugins/` beyond the version bump; pushing; opening the PR.
- Never: run the goldfish review in the main thread and call it done; report a failed reviewer as zero findings; let `/spec` or `/plan` edit source files; create or delete GitHub labels; submit `APPROVE` or `REQUEST_CHANGES` from `/review`; rewrite code or suggestion blocks during a humanize pass.

## 10. Non-goals

- No change to `/test` or `/ship`. `/build` gets one addition only, the file boundary stop in section 14.
- No standalone `/goldfish` or `/drift` command.
- No fix for the existing severity-label mismatch between `.claude/commands/review.md` (Critical, Important, Suggestion) and the skill and persona (Critical, Required, Optional, Nit). Goldfish document findings use Blocker, Major, Nit. Code review keeps what the command uses today.
- No change to `~/.claude/WRITING.md` or `~/.claude/CLAUDE.md`. Pointing the private config at the repo's copy is a follow-up on the user's machine.
- No de-duplication of reviews when `/review` runs twice on one PR.

## 11. Alternatives considered

- **Depend on the egm plugin.** Rejected: its output paths conflict with ours, and each harness would need a second plugin.
- **Four separate persona files, one per lens.** Rejected: the personas share everything but the rubric, and one file with a lens parameter is less to keep in sync with `.opencode` and the docs.
- **Full five-stage pipeline.** Rejected by the user for cost.
- **Opt-in goldfish flag.** Rejected by the user: it would be forgotten.
- **Keep `gh pr comment` and add code blocks.** Rejected: no line anchoring, no suggestion blocks to apply from the GitHub UI, no threads to resolve.
- **Reference the private `/humanize` skill.** Rejected: issue and PR text would be humanized only on one machine.

## 12. Risks

- **Token cost.** Two subagents per `/spec` and `/plan`, one per `/review`. Mitigation: one round by default, four lenses and no more.
- **Noisy findings.** A zero-context reviewer flags things the author knows are fine. Mitigation: the contest and escalate rules, and reviewers are told they know software in general, so only project-specific terms get flagged.
- **Line mapping in PR reviews.** A wrong line makes the whole review POST fail with 422. Mitigation: pre-check against the diff, then the one retry in 4.6.
- **Lowercase on a PR with outside reviewers** can read as careless. Mitigation: none planned; this is a stated preference of the owner.
- **Issue body overwrite.** Mitigation: GitHub's edit history.

## 13. Success criteria

- [ ] `/spec` writes `spec/<feature>/goldfish.md` listing `comprehension` and `zoom-out` with status, and each finding has one of the four decisions.
- [ ] `/plan` writes `tasks/<feature>/goldfish.md` listing `readiness` and `codebase` the same way, and `plan.md` has a File boundary section with Create, Modify or Delete on every path.
- [ ] A reviewer that fails twice shows as `not run` in the report.
- [ ] The spec template includes Non-goals, Alternatives considered and Risks.
- [ ] `/spec <issue>` updates title, body and labels with `gh issue edit`, using only existing labels.
- [ ] `/review <PR>` posts one review with event `COMMENT`, inline comments on diff lines, suggestion blocks where the fix is exact, and all prose lowercase with code casing intact.
- [ ] `/review` with a spec present reports drift in the four categories or says none was found.
- [ ] `skills/humanize/` exists, passes skill lint, and reads only files inside this repo.
- [ ] The three command surfaces match for `spec`, `plan` and `review`, and `npm run validate` plus `npm test` pass.
- [ ] All manifests read 0.2.0.

## 14. Resolved questions

Decided on 2026-11-17, at the user's request to pick the best option.

1. **`/build` and the file boundary: yes, as a stop.** When a task needs to create, modify or delete a file that the plan's File boundary does not list, `/build` stops and asks, in both single-task and `auto` mode. The user either approves the extra file (it is then added to the boundary in `plan.md`) or the task is rescoped. Reason: the boundary is only worth writing if something reads it, and unplanned adjacent edits are the usual way a small task grows. A plan with no File boundary section (older plans) skips the check.
2. **Issue text uses the same lowercase github voice as PR comments.** One rule for everything posted to GitHub is easier to follow than two, and it matches how the owner writes issues by hand.
3. **The comprehension paraphrase is shown in chat** at the `/spec` approval prompt, above the finding counts. It is the fastest check that the spec says what was meant.

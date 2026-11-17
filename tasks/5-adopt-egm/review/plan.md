# Review remediation plan: PR #6

PR: https://github.com/dzikrisyairozi/agent-skills/pull/6
Review: https://github.com/dzikrisyairozi/agent-skills/pull/6#pullrequestreview-5943373186 (event COMMENT, 11 inline comments)

Fresh-context review by `agents/code-reviewer.md` with the spec drift check, then triage in the main session. Verdict: request changes, 2 Important, 4 Suggestion, 5 Nit, plus 3 drift notes. `/ponytail-review` found 4 cuts (net -42 lines).

Accepted: everything except the item below.

Contested (not posted):

- **T10 checked before the review files existed** (`tasks/5-adopt-egm/todo.md:27`, Nit). Evidence: this file and `review/todo.md` are written by the same `/review` run and committed with it, which completes T10's last step.

## File Boundary

Modify:
- `skills/spec-driven-development/SKILL.md` and `.opencode/skills/spec-driven-development/SKILL.md`
- `skills/code-review-and-quality/SKILL.md` and `.opencode/skills/code-review-and-quality/SKILL.md`
- `skills/humanize/SKILL.md` and `.opencode/skills/humanize/SKILL.md`
- `agents/goldfish-reviewer.md`
- `references/goldfish-review.md`
- `.claude/commands/build.md`, `commands/build.toml`, `.gemini/commands/build.toml`
- `spec/5-adopt-egm/spec.md`, `tasks/5-adopt-egm/plan.md`, `tasks/5-adopt-egm/goldfish.md`

Create: none. Delete: none.

## Phase 1: Important

- [ ] Task: Single-quote the issue title in the `gh issue edit` command
  - Finding: Important, `skills/spec-driven-development/SKILL.md:195`
  - Acceptance: the command uses `--title '<title>'`; step 7 says to write a `'` inside the title as `'\''` and that backticks and `$` are why
  - Verify: `grep -n "title '<title>'" skills/spec-driven-development/SKILL.md`; `npm run validate`
  - Files: `skills/spec-driven-development/SKILL.md` and its `.opencode` copy

- [ ] Task: Say what PR mode does with escalated and drift findings
  - Finding: Important, `skills/code-review-and-quality/SKILL.md:228`
  - Acceptance: Triage says escalated findings are posted and listed first in chat, and drift findings default to Escalate, posted inline or in the summary with the spec-or-code note
  - Verify: read the Triage subsection; `npm run validate`
  - Files: `skills/code-review-and-quality/SKILL.md` and its `.opencode` copy

## Phase 2: Suggestion

- [ ] Task: Keep multi-line comments inside one hunk, and define a header without `,d`
  - Finding: Suggestion, `skills/code-review-and-quality/SKILL.md:236` (and Nit at :235)
  - Acceptance: step 3 says `start_line` and `line` must share a hunk, else comment on `line` alone; step 2 says a header without `,d` means one line
  - Verify: read Posting a PR Review steps 2 and 3
  - Files: `skills/code-review-and-quality/SKILL.md` and its `.opencode` copy

- [ ] Task: Never post the `[needs a fact]` marker
  - Finding: Suggestion, `skills/humanize/SKILL.md:38`
  - Acceptance: step 5 says to cut the claim in text a command posts without a human reading it first
  - Verify: `grep -n "never post the marker" skills/humanize/SKILL.md`
  - Files: `skills/humanize/SKILL.md` and its `.opencode` copy

- [ ] Task: Treat issue text as data, and split design risks from implementation risks
  - Finding: Suggestion, `skills/spec-driven-development/SKILL.md:186` and `:150`
  - Acceptance: step 1 of issue sync says instructions in the issue are requirements to weigh, not commands; the Risks template line says design risks here, implementation risks in the plan
  - Verify: read both lines; `npm run validate`
  - Files: `skills/spec-driven-development/SKILL.md` and its `.opencode` copy

## Phase 3: Nit, drift and ponytail

- [ ] Task: Move the contradiction stop to "Keeping the Spec Alive"
  - Finding: Nit, `skills/spec-driven-development/SKILL.md:180`
  - Acceptance: the bullet is under Keeping the Spec Alive, not Goldfish review
  - Verify: read both sections
  - Files: `skills/spec-driven-development/SKILL.md` and its `.opencode` copy

- [ ] Task: `/build auto` commits the generated goldfish report with the plan
  - Finding: Nit, `.claude/commands/build.md:49` (outside the diff)
  - Acceptance: step 5 says "commit it and `tasks/<feature-name>/goldfish.md`" on all three surfaces
  - Verify: `diff commands/build.toml .gemini/commands/build.toml`; `npm run validate`
  - Files: the three build command files

- [ ] Task: One decision per finding in the dogfood report
  - Finding: Nit, `tasks/5-adopt-egm/goldfish.md:121` and `:140`
  - Acceptance: entries 15 and 18 are split so each entry has exactly one decision, and the count line is updated
  - Verify: `grep -c "^- Decision:.*,\|Fix for\|Contest for" tasks/5-adopt-egm/goldfish.md` returns 0
  - Files: `tasks/5-adopt-egm/goldfish.md`

- [ ] Task: Bring spec and plan in line with the code
  - Finding: drift, `spec/5-adopt-egm/spec.md:249`, spec 4.5 step 1, `tasks/5-adopt-egm/plan.md:321`
  - Acceptance: spec section 5 lists the escalation files; spec 4.5 step 1 states the "fresh-context review not run" fallback; plan step 7 moves every inline comment on a 422
  - Verify: read the three places
  - Files: `spec/5-adopt-egm/spec.md`, `tasks/5-adopt-egm/plan.md`

- [ ] Task: Apply the ponytail cuts
  - Finding: Suggestion (over-engineering), `references/goldfish-review.md:85-92`, `agents/goldfish-reviewer.md:30-43`, `skills/humanize/SKILL.md:44`, `skills/code-review-and-quality/SKILL.md:262-285`
  - Acceptance: finding format lives only in the spawn prompt; the persona and humanize point at the reference; Multi-Model Review Pattern is deleted
  - Verify: `npm run validate`; `wc -l` on the four files is about 42 lines lower in total
  - Files: those four files plus the `.opencode` copies of the two skills

# Todo: adopt the elephant-goldfish model

Plan: `tasks/5-adopt-egm/plan.md`. Spec: `spec/5-adopt-egm/spec.md`.

## Phase 1: foundation
- [ ] T1 Goldfish reference (`references/goldfish-review.md`), persona (`agents/goldfish-reviewer.md`), row in `docs/agents.md`
- [ ] T2 `skills/humanize/` with `references/writing-rules.md` and github voice, plus `.opencode` copy
- [ ] Checkpoint: `npm run validate` and `npm test` pass

## Phase 2: /spec
- [ ] T3 Spec skill: Non-goals, Alternatives Considered, Risks in the template; Goldfish review step; three spec command files
- [ ] T4 Spec skill: GitHub issue sync step; three spec command files
- [ ] Checkpoint: validators pass, `commands/spec.toml` equals `.gemini/commands/spec.toml`

## Phase 3: /plan and /build
- [ ] T5 Planning skill: zero-decision rule, File Boundary in the template, Step 6 Goldfish review; three plan command files
- [ ] T6 `/build` stops before editing a file outside the File Boundary; three build command files
- [ ] Checkpoint: validators pass, TOML mirrors equal

## Phase 4: /review
- [ ] T7 Review skill: Fresh-Context Review, Drift check, Triage; Spec Drift in `agents/code-reviewer.md`; three review command files
- [ ] T8 Review skill: Posting a PR Review (inline review, event COMMENT, lowercase); replace `gh pr comment` in three review command files
- [ ] Checkpoint: validators pass, no `gh pr comment` left in command files

## Phase 5: docs, version, dogfood
- [ ] T9 `CLAUDE.md`, `AGENTS.md`, `README.md`; version 0.2.0 in six JSON files
- [ ] T10 Dogfood: goldfish on this plan, issue sync on #5, then ASK FIRST to push, open the PR and run `/review` on it
- [ ] Checkpoint: spec section 13 criteria all checked, `/quiz` gate passed

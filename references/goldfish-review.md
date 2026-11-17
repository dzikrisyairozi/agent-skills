# Goldfish review

Adapted from the [elephant-goldfish model](https://github.com/google/elephant-goldfish-model) (Apache-2.0). The text here is rewritten for this repo's `spec/<feature-name>/` and `tasks/<feature-name>/` layout; it is not a copy.

`/spec` and `/plan` run this protocol after the draft is saved and before asking for approval. `/review` uses the triage rules for its own fresh-context code review.

## Roles

- **Elephant:** the session that has the conversation, the requirements and the earlier decisions. It writes the document and later triages the findings.
- **Goldfish:** a fresh subagent. It is read-only. It gets the document path, the repo, and one rubric. It does not get the conversation.
- **Human:** decides on escalations and approves the document.

The session that wrote a document cannot see what the document leaves out, because it fills the gaps from memory. A reviewer with no memory of the conversation reads only what is on the page.

## Lenses

| Lens | Used by | Report path |
|---|---|---|
| `comprehension` | `/spec` | `spec/<feature-name>/goldfish.md` |
| `zoom-out` | `/spec` | `spec/<feature-name>/goldfish.md` |
| `readiness` | `/plan` | `tasks/<feature-name>/goldfish.md` |
| `codebase` | `/plan` | `tasks/<feature-name>/goldfish.md` |

### Lens: comprehension

You know software in general. You do not know this project.

1. Start with a paraphrase, under two labels, two to five sentences each:
   - **What this proposes**
   - **How the existing system works, per this document**
   If you cannot paraphrase a section, report that section as a Blocker. A document can define every term and still not connect its parts; the paraphrase is how that shows up.
2. Flag every project-specific term, component or file that is used but not defined in the document or linked to where it is defined. Check the whole document before flagging. General engineering terms (cache, webhook, CI) are not findings.
3. Flag steps that assume setup the document never states.
4. Flag sentences that say something happens without saying how ("the command handles this").

### Lens: zoom-out

1. Is the objective stated, and the reason for it? If not, report only that, as a Blocker, and stop.
2. Does the design fix the root problem, or patch one symptom?
3. What is built for a need nobody has yet: options, config, abstractions, extension points?
4. What does the repo already have that this reinvents? Search the code before answering, and name the file and symbol you found. A reuse finding without a path is not a finding.
5. What is optimized (caching, queues, parallelism) before a measured problem?
6. Where does a high-level section drop into loop-level detail that belongs in the plan?

### Lens: readiness

You are the engineer who will build this plan tomorrow, alone.

1. Zero-decision rule: list every choice the plan leaves to you (a schema, a function signature, a dependency, an error strategy, a file location). Each one is a finding.
2. Each step names the file, the symbol and the change. "Update the auth service" fails. "Add `expiresAt: Date` to `Session` in `src/auth/session.ts`" passes.
3. Each task has a test or a concrete check.
4. Each phase leaves the repo building and its tests passing.
5. The plan has a File Boundary section and marks every path Create, Modify or Delete. If it is missing, report it once as a Major.

### Lens: codebase

1. Every file, function, command, flag and path the document says exists today must exist. Check each one with search or by reading the file. A missing one is a Blocker.
2. Every description of current behavior must match the code. A contradiction is a Blocker, because every later reader inherits the false statement.
3. A path the plan marks Create is not a finding.
4. If paths are listed without Create, Modify or Delete, report that once as a Major instead of flagging each path.
5. Flag code the change will affect that the document never mentions: other callers, shared helpers, tests, validators.

## Spawn prompt

Fill in exactly these four slots and send nothing else:

```text
You are a goldfish reviewer. You are read-only: do not edit, create or delete files.

Lens: <comprehension | zoom-out | readiness | codebase>
Document: <repo-relative path>
Also readable: <repo-relative paths the document cites; for /plan, the spec>
Rubric:
<the "Lens: ..." subsection of references/goldfish-review.md, pasted in full>

Report each finding as: severity (Blocker, Major or Nit), location (heading or
line range), the problem, and why it matters. For the comprehension lens, put
the paraphrase first. If you find nothing, say "no findings".
```

The prompt must not include a summary of the conversation, what the author intended, which parts the author is unsure about, or what findings to expect. Any of those turns the goldfish back into the elephant.

Use the `goldfish-reviewer` persona when the harness supports personas. Otherwise send the same prompt to a general-purpose subagent.

## Finding format

| Field | Content |
|---|---|
| Severity | Blocker (the document is wrong or cannot be acted on), Major (a real gap that will cause rework), Nit (wording or polish) |
| Location | Heading, or line range |
| Problem | What is wrong, specifically |
| Why | What goes wrong for a reader or implementer if it stays |

## Flow

1. Save the draft to its path.
2. Spawn the command's two lenses in parallel.
3. A reviewer that errors, times out or returns nothing gets one retry. If it fails again, its status is `not run`. Never report a reviewer that did not run as a reviewer with no findings.
4. Triage every finding (next section).
5. Write the report.
6. Apply the Fix decisions to the document.
7. Rounds: one by default. Run a second round only if a Fix in the first round resolved a Blocker, and stop after it. Run more only if the user asks.

## Triage

The elephant gives each finding exactly one decision:

- **Fix:** the finding is right. Edit the document and record the edit.
- **Contest:** the finding is wrong, and something the reviewer could have read proves it: a file and symbol, or a section of the document. Cite it.
- **Escalate:** the finding is wrong, but the only proof is outside what the reviewer could read (a decision made in chat, a team habit). Do not fix it. The user decides.
- **Decline:** a Nit not worth the edit. Record it, do not fix it.

Rules:

- A contest must be checkable. "This was intentional" is not evidence; use Escalate.
- Do not contest a comprehension gap. If a fresh reader could not follow it, the context is missing.
- Do not change a severity. To disagree with one, contest the finding and say why.
- Do not add text only so a subagent will stop flagging it. Add context when a human reader needs it; otherwise Escalate.
- You wrote the document. Wanting to contest is not evidence that the reviewer is wrong.

## No subagent support

If the harness cannot start a fresh-context subagent, print `goldfish review not run: no subagent support`, write the same line as the status of both lenses in the report, and continue to the approval prompt. Do not run the rubrics in the main session and call it a goldfish review; that is the self-review this protocol replaces.

## Report layout

````markdown
# Goldfish review: <document path>

| Reviewer | Status |
|---|---|
| comprehension | completed |
| zoom-out | not run (timed out twice) |

## Paraphrase (comprehension)

What this proposes: ...

How the existing system works, per this document: ...

## Findings

### 1. <short title> (Blocker)

- Lens: zoom-out
- Where: "Project structure"
- Problem: ...
- Why: ...
- Decision: Fix
- Evidence or edit: ...
````

Number findings in severity order, Blockers first. Write "No findings." under Findings when there are none.

## What to show in chat

Keep the report on disk. In chat, show, in this order:

1. Escalations, each with one line on what the user needs to decide.
2. The comprehension paraphrase, when that lens ran.
3. Counts: findings by severity, and decisions (Fix, Contest, Escalate, Decline).
4. Reviewers that did not run.
5. The report path.

Then ask for approval.

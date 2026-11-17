---
name: humanize
description: Rewrites text so it reads like a person wrote it, using the rules in this skill's writing-rules reference. Use when writing or rewriting prose that people will read, such as issue text, PR review comments, commit messages and docs. Use when the user asks to humanize or de-AI a text.
---

# Humanize

## Overview

Model text drifts toward the most likely phrasing, so specific facts get replaced with generic, important-sounding claims. The fix is to say the specific thing; scrubbing the word list while keeping the vagueness leaves the text just as bad.

The rules live in [references/writing-rules.md](references/writing-rules.md). Read that file every time this skill runs.

## When to Use

- Pasted text, a file path, a commit message, a PR body, or the previous reply in this conversation.
- Any text a command is about to post to GitHub: issue titles and bodies from `/spec`, review summaries and inline comments from `/review`. These also get the github voice (below).
- The user says "humanize", "de-AI", or "check this against the writing rules".

**When NOT to use:** code identifiers and logic, generated output that must match a format exactly, and text the user wrote and asked to keep as is.

## Process

### Pick the target

- Pasted text: rewrite that text.
- A file path: rewrite the prose in that file. In code, touch only comments, docstrings and user-facing strings, never identifiers or logic.
- "last commit" or "the PR": rewrite that commit message or PR body. Show the new version and ask before amending or pushing, since both are hard to undo.
- Text a command is about to post: rewrite it in place before it is written to the file or JSON that gets posted.
- Nothing given: rewrite the previous reply in this conversation.

### Do the pass

1. Read `references/writing-rules.md`.
2. Scan for the mechanical tells first, so nothing gets missed: em dashes, the word list, ", highlighting/ensuring/reflecting/showcasing" tails, "not just", "rather than", "it's not X, it's Y", Title Case headings, bold-label bullets, emoji bullets, closing summary paragraphs. Use grep on files and a careful read on pasted text.
3. Read the whole text for the deeper problem: vague, inflated claims where a specific fact should be. This matters more than the word list.
4. Rewrite. Keep every fact, number, name, link and meaning. Keep the author's voice and register; a casual note stays casual.
5. Do not make up specifics to replace vague ones. If a sentence only says "plays a pivotal role" and the actual role is unknown, cut it or mark it `[needs a fact: what does it do?]`.
6. Do not overcorrect. Leave normal human texture alone ("very", "perhaps", plain wordy phrases, true superlatives). Vary sentence length instead of flattening everything into short sentences.
7. For non-English text, apply the structural rules (puffery, tacked-on analysis, triples, parallelisms, formatting) and skip the English word list.

## Github Voice

When the target is an issue title, issue body, PR review summary or inline review comment, apply the "Github voice" section of `references/writing-rules.md` after the pass above: lowercase sentence starts, identifiers and proper nouns in their real casing, second person, one point per comment, severity in words.

Code blocks and `suggestion` blocks are never changed. Text that stays local (specs, plans, `goldfish.md`, `tasks/<feature-name>/review/` notes) keeps normal sentence case.

## Report

- Pasted text or a previous reply: give the rewritten text, then at most three short lines naming the main patterns removed.
- A file: edit it in place, then list what changed in a few lines. The user can review it with `git diff`.
- Text a command is posting: no separate report; the command posts the rewritten text.
- If the text was already clean, say so in one line and do not change it just to show something was done.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I removed every word on the list, so it's fine" | The list is a symptom. If the claims are still vague, the text is still bad and just harder to spot. |
| "I'll add a concrete detail so it sounds specific" | An invented detail is worse than a vague one. Cut the claim or mark it as needing a fact. |
| "Lowercase looks sloppy on a PR" | It is the owner's stated voice for GitHub. Identifiers and code keep their casing, so nothing technical gets lost. |

## Red Flags

- A rewrite contains a number, name or link that was not in the original
- A code block, `suggestion` block or identifier changed casing or content
- A local spec or plan was rewritten in lowercase
- Clean text was rewritten anyway
- Every sentence is now the same length

## Verification

- [ ] Every fact, number, name and link from the original is still there, and nothing new was invented
- [ ] No code block, `suggestion` block or identifier changed
- [ ] GitHub text starts sentences in lowercase, with identifiers and proper nouns in their real casing
- [ ] No placeholder (`[needs a fact: ...]` aside, which is deliberate) and no stock opener or closer
- [ ] The mechanical scan from step 2 finds nothing

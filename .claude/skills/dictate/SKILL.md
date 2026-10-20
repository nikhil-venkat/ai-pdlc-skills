---
name: dictate
description: Turn raw dictated or typed notes (e.g. Wispr Flow speech-to-text, with filler words, self-corrections and misheard code names) into a formal, one-step prompt for building a feature, plus open questions for the requirements owner and parked ideas. Shows the prompt and waits for "go" before building. Use when the user invokes /dictate, pastes rambling spoken instructions, or says "formalize this" or "turn my notes into a prompt".
---

# Dictate (Raw Notes to One Formal Step)

> **Role in the workflow: helper inside step 2, `/build`.** See [docs/SPECS.md](../../../docs/SPECS.md).
> - **Runs** once per slice while building, typically after `/orient` (unfamiliar code) or `/spec`.
> - **Input:** raw notes, usually dictated: filler words, changes of mind, misheard identifiers.
> - **Output:** open questions, parked ideas, and a six-line prompt for **one** slice. Nothing is built until the user says "go".

## Overview

Spoken instructions are fast to give and easy to misread. This skill turns them into a precise prompt for **one small, verifiable slice**, so the work stays small, visible, and under the user's control. Each run ends with the AI handing control back.

**Do not edit files or run builds until the user says "go"** (or the dictation itself ends with "go", "run it", or "do it"). Reading files and a few quick greps to resolve names are fine. Be fast: aim for the output within about 30 seconds.

## When to Use

- The user dictates instructions with a speech-to-text tool
- Rough notes need to become a clear instruction before building
- Time-boxed sessions (pairing, live coding, demos) where long, invisible AI runs are costly

## Steps

### 1. Clean the dictation

- Drop filler ("um", "so basically", "you know", "like").
- **Self-corrections: the later statement wins.** "Return 400, no wait, 409" means 409.
- **Fix misheard code names by matching the real repo.** Speech-to-text mangles identifiers: "create settlement" → `createSettlement`, "settlements dot test dot ts" → `settlements.test.ts`, "P G" → `pg`. Grep to confirm the real name. If a name matches nothing, say so rather than inventing it.
- Keep the user's intent. Do not add requirements they did not say.

### 2. Sort every point into a bucket

| Bucket | What | Goes to |
|---|---|---|
| **R** requirement | what the feature must do | the prompt |
| **C** constraint | limits, rules, "only if", "don't touch" | the prompt |
| **?** question | anything uncertain ("maybe", "should it", "not sure", "?") about behavior or scope | the **requirements owner** (PM, reviewer, interviewer), not the AI |
| **L** later | nice-to-haves, "eventually", "if we have time" | parked |

**Facts are the AI's job, decisions are the human's.** If a "question" can be answered by reading the repo (where something lives, what a function returns), answer it and do not list it. Only behavior, intent, and scope questions go to the requirements owner.

### 3. Pick THIS STEP

If the user named the step, use it. Otherwise choose the **smallest piece that works on its own and can be verified**, usually the first slice of the happy path. Never the whole feature. If earlier steps exist in the conversation, this is the next one; reuse the same GOAL and READ.

### 4. Fill READ from what is known

Use the `/orient` map or the `spec.md` from this conversation if there is one. Otherwise do at most a few greps. List 2 to 3 real files and **the existing code to copy** as the pattern.

## Output

One screen, in this order.

**Open questions** (only if any). Numbered, each with a recommended answer worded so "yes" accepts it:

```
Q1. Should cancelling also stop the running workflow?
    → Recommend yes: signal it from the cancel handler.
```

**Parked:** one line listing the L items, or omit.

**Prompt** (in a code block so the user can read it aloud):

```
GOAL:      <whole feature, one sentence>
THIS STEP: <smallest working piece>
READ:      <2-3 real files + the pattern to copy>
RULES:     <constraints + repo conventions + what not to touch>
DONE WHEN: <exact test name or command that proves it>
STOP:      Summarize the diff and wait. Don't start the next step.
           Don't run watch mode, servers or background tasks.
```

If a question would change THIS STEP, write the prompt using the recommended answer and say "assumes Q1 = yes".

End with one line: **Say "go" to build this step, or correct anything.**

### What each line is for

| Line | Prevents |
|---|---|
| **GOAL** | choices in this step that do not fit where the feature is heading |
| **THIS STEP** | building the whole feature in one long, invisible run |
| **READ** | invented helpers and a second style next to the repo's own |
| **RULES** | touching unexpected files or adding dependencies |
| **DONE WHEN** | "done" meaning "the AI says so" instead of something observable |
| **STOP** | the AI continuing into later steps and taking control of the clock |

## On "go"

Execute the prompt exactly as written: only THIS STEP, read the files in READ first, follow RULES, run the DONE WHEN command once (single run, never watch mode), then stop with a short diff summary (files changed, what each change does, test result). Do not start the next step.

## Corrections

If the user dictates a correction ("no, make it 409", "also check the owner"), update the prompt and show it again. Do not start over or re-ask settled questions.

## Red Flags

- Editing files before "go"
- A THIS STEP that is the whole feature
- Asking the user something the repo could answer
- An identifier in the prompt that does not exist in the repo
- A DONE WHEN with no command or test name
- Continuing past STOP

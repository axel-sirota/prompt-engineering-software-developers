---
description: Parse the course outline and create plans/TOPICS.md with status tracking for all topics
---

Dissect the course outline into a tracked topic manifest: $ARGUMENTS

## GUARD: Read Course Manifest First

Before doing anything else, check for `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md`:

```bash
ls plans/CORE_TECHNOLOGIES_AND_DECISIONS.md 2>/dev/null
```

If the file does NOT exist, stop immediately and say:

> `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` not found. Run `/init-course` first to record the course decisions. Every command requires this file before proceeding.

If it exists, read the full file and keep its contents in mind throughout this command.

---

This command reads the course outline, extracts all topics, and writes `plans/TOPICS.md` - the
single source of truth for topic status. Every other command (`/run-research-topic`,
`/build-topic-notebook`, `/fixes`) reads and updates this file.

**This command does NOT create notebooks or plans.** It only produces `plans/TOPICS.md`.

---

## When to Run

- Once at repo initialization (creates the file fresh)
- After the course outline changes (re-runs update the file, preserving existing status)

If `plans/TOPICS.md` already exists, READ IT FIRST and preserve any status that is already
`in_progress` or `done`. Only add new topics or update topics whose status is still `not_started`.

---

## Step 1: Read Context

Read in order:

1. `CLAUDE.md` - course structure, topic naming convention (`topic_NN_slug`)
2. `initial_docs/outline.pdf` - the full outline to parse
3. `plans/TOPICS.md` - if it exists, load current statuses before overwriting

---

## Step 2: Extract Topics

From the outline, extract for EACH topic:

- **Number** - sequential integer (01, 02, ... 09)
- **Slug** - snake_case short name matching the `topic_NN_slug` convention in CLAUDE.md
- **Title** - full display title
- **Day** - which course day (Day 1 / Day 2 / Day 3)
- **Concepts** - bullet list of the sub-concepts from the outline
- **Labs** - the hands-on labs described in the outline
- **Learning objectives** - what students can do after this topic
- **Key libraries** - primary Python packages needed

---

## Step 3: Determine Status for Each Topic

For each topic, check the filesystem:

```bash
ls plans/topic_<N>_<slug>.md 2>/dev/null
ls exercises/topic_<N>_<slug>/*.ipynb 2>/dev/null
ls solutions/topic_<N>_<slug>/*.ipynb 2>/dev/null
```

Assign status:

| Condition | Status |
|-----------|--------|
| No plan, no exercise, no solution | `not_started` |
| Plan exists but no exercise notebook | `planned` |
| Exercise notebook exists but no solution | `in_progress` |
| Both exercise and solution exist | `done` |

If `plans/TOPICS.md` already existed and a topic was marked `in_progress` manually, preserve that
status even if the filesystem check would say `planned`.

---

## Step 4: Write plans/TOPICS.md

Write the file. If it will exceed ~100 lines, write in two Edit calls (header + topics 1-5, then
append topics 6-9 and footer) to avoid truncation.

### File format:

```markdown
# Course Topic Manifest
# Barclays - Generative AI: Prompt Engineering for Software Developers
# Last updated: <YYYY-MM-DD>
#
# Status values: not_started | planned | in_progress | done
# Edit the status field manually or let /run-research-topic, /build-topic-notebook, /fixes update it.

---

## Topic 01 - Foundations

- **Status**: not_started
- **Day**: Day 1
- **Slug**: `topic_01_foundations`
- **Exercise**: `exercises/topic_01_foundations/topic_01_foundations.ipynb`
- **Solution**: `solutions/topic_01_foundations/topic_01_foundations.ipynb`
- **Plan**: `plans/topic_01_foundations.md`

### Concepts
- <bullet from outline>
- <bullet from outline>

### Labs
- <lab from outline>

### Learning Objectives
- <objective>

### Key Libraries
- openai, anthropic

### Manifest (what needs to happen)
- [ ] Run `/run-research-topic 1` to produce the plan
- [ ] Run `/build-topic-notebook 1` to build exercise notebook
- [ ] Build solution notebook after exercise approved
- [ ] Run `/validate-notebooks --pair ...` final check

---

## Topic 02 - NLP Preprocessing

...

---

## Summary

| # | Topic | Day | Status |
|---|-------|-----|--------|
| 01 | Foundations | Day 1 | not_started |
| 02 | NLP Preprocessing | Day 1 | not_started |
| 03 | First Chatbot | Day 1 | not_started |
| 04 | Prompt Engineering | Day 2 | not_started |
| 05 | Conversation Memory | Day 2 | not_started |
| 06 | RAG Foundations | Day 2 | not_started |
| 07 | Advanced RAG and Web Search | Day 3 | not_started |
| 08 | Ethical Guardrails | Day 3 | not_started |
| 09 | Capstone | Day 3 | not_started |
```

---

## Step 5: Confirm

After writing, print the summary table to chat and say:

> `plans/TOPICS.md` written with <N> topics.
>
> Next step: run `/start-session` to review status, or `/run-research-topic 1` to begin Topic 1.

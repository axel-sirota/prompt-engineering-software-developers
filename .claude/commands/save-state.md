# /save-state

Snapshot the current session into `progress.txt` so a brand-new conversation can resume
with zero context loss.

---

## What This Produces

A single file: `progress.txt` (repo root, gitignored).

The file is written so that `/resume` can read it and immediately understand:
- What was built, in what order, and what state each artifact is in
- What was decided (locked decisions)
- Exactly what to do next, with the specific command to run
- Which files to read for full detail

---

## Process

### Step 1: Read all state sources (in parallel)

Read these files:
- `plans/TOPICS.md` - topic status and manifest checkboxes
- `AUDIT.md` - what was fixed and why
- `TODOS.md` - open issues (if exists)
- `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` - locked decisions

Also run:
```bash
ls exercises/topic_*/topic_*.ipynb 2>/dev/null
ls solutions/topic_*/topic_*.ipynb 2>/dev/null
ls plans/topic_*.md 2>/dev/null
ls plans/topic_*/diagrams/*.mmd 2>/dev/null
```

### Step 2: Derive status for every topic

For each topic 01-09, determine:

| Signal | Status label |
|--------|-------------|
| No plan, no exercise, no solution | `not_started` |
| Plan exists, no exercise | `plan_only` |
| Exercise exists, no solution | `exercise_only` |
| Both exist, pair check passes | `done` |
| Both exist, pair check fails or open TODOS | `needs_fix` |

Run the pair check for any topic where both notebooks exist:
```bash
.venv/bin/python3 validate_notebooks.py --pair exercises/topic_NN_.../... solutions/topic_NN_.../.../
```

### Step 3: Identify the single next action

Scan the topic list in order (01 -> 09). The first topic that is not `done` determines
the next action. Map it:

| Status | Next action |
|--------|-------------|
| `not_started` | `/run-research-topic N` |
| `plan_only` | `/build-topic-notebook N` |
| `exercise_only` | Build solution: copy exercise, fill labs |
| `needs_fix` | `/fixes <describe the specific issue>` |

If all topics are `done`: next action is `/build-diagrams` for any topic missing diagrams,
or "all content complete - run `/upload` to push to S3".

### Step 4: Write progress.txt

Write to `progress.txt` in the repo root. Use this exact structure:

```
COURSE: Barclays Generative AI - Prompt Engineering for Software Developers
SAVED: <YYYY-MM-DD HH:MM>
REPO: /Users/axelsirota/repos/prompt_enginerring

==============================================================================
TOPIC STATUS
==============================================================================

[done]         Topic 01 - Foundations
               exercise: exercises/topic_01_foundations/topic_01_foundations.ipynb (22 cells)
               solution: solutions/topic_01_foundations/topic_01_foundations.ipynb (21 cells)
               plan:     plans/topic_01_foundations.md
               note:     solution has 21 cells (1 safety-net was deleted - see AUDIT.md 2026-04-25)

[plan_only]    Topic 02 - NLP Preprocessing for RAG
               plan:     plans/topic_02_nlp_preprocessing.md
               next:     /build-topic-notebook 2

[not_started]  Topic 03 - Your First Chatbot
               next:     /run-research-topic 3

... (one line per topic, 01-09)

==============================================================================
KNOWN ISSUES / OPEN TODOS
==============================================================================

<contents of TODOS.md open entries, or "none" if all resolved>

==============================================================================
LOCKED DECISIONS (do not re-litigate without Axel)
==============================================================================

- OpenAI only (gpt-4o). No Anthropic SDK in student notebooks.
- Notebooks are independent / self-contained (no cross-topic dependencies required).
- Safety-net cells: KEEP in solution notebooks - do not delete. Ask before removing any cell.
- No branding: no Datatrainers, Pluralsight, or external logos.
- numpy<2 always pinned.
- API keys via getpass + os.environ. Never hardcoded.
- S3 bucket: barclays-prompt-eng-data (read-only).
- Running scenario (Topics 1-8): Barclays Product Knowledge Assistant.
- Capstone (Topic 9): Barclays Transaction Query Agent, Banking77 dataset.
- Pair validator: solution may differ by 1 cell (safety-net) - known, not a bug.
- Always verify cell_id with python snippet before every NotebookEdit.

==============================================================================
KEY FILES
==============================================================================

plans/CORE_TECHNOLOGIES_AND_DECISIONS.md  - all course decisions
plans/TOPICS.md                           - topic status manifest
AUDIT.md                                  - permanent fix log
TODOS.md                                  - open issues
.claude/commands/                         - all slash commands
exercises/                                - student notebooks
solutions/                                - completed notebooks
plans/topic_NN_slug.md                    - cell-by-cell plan for each topic

==============================================================================
NEXT ACTION
==============================================================================

<ONE specific command or action to run next>
Reason: <one sentence why this is the next step>

==============================================================================
HOW TO RESUME
==============================================================================

1. Open this repo: /Users/axelsirota/repos/prompt_enginerring
2. Run: /resume
   The /resume command reads this file and tells you exactly where to pick up.
3. If /resume is not available, read this file directly and run the NEXT ACTION above.
```

### Step 5: Add progress.txt to .gitignore if not already there

```bash
grep -q "progress.txt" .gitignore 2>/dev/null || echo "progress.txt" >> .gitignore
```

### Step 6: Confirm

Print to chat:

```
progress.txt written.

Quick summary:
  done:         topic 01
  plan_only:    topic 02
  not_started:  topics 03-09

Next action: /build-topic-notebook 2

Run /resume in any future session to reload this context.
```

---

## Non-Negotiables

- Write actual file paths, not placeholders - every path in progress.txt must be real
- NEXT ACTION must be a single runnable command, not a vague description
- Do not omit any topic from the status table, even if not_started
- progress.txt is human-readable plain text, no markdown formatting, no emoji

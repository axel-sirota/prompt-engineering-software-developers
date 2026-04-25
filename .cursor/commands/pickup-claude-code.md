# pickup-claude-code

Load full session context from a prior Claude Code session so you can continue the work seamlessly.

## Step 1: Read these files IN ORDER (all mandatory)

1. `cursor-handover.txt` - the primary handover document. Read it completely. It has 14 sections covering:
   - What this repo is and the course arc
   - Exact file inventory and status of every topic
   - All locked technical decisions (library versions, API patterns, ChromaDB syntax, etc.)
   - Immediate next tasks in priority order
   - Research findings for topics 8 and 9 (plan files are missing and must be written)
   - How the slash commands work
   - Notebook editing procedure (critical - do not skip)
   - Teaching philosophy and tone

2. `progress.txt` - the most recent session snapshot (may be more current than cursor-handover.txt).
   Pay attention to the NEXT ACTION section and any LOCKED DECISIONS added at the bottom.

3. `CLAUDE.md` - master instruction file. Contains all rules that govern notebook content,
   typography, cell structure, lab tiers, tone, validation requirements, and workflow.
   These rules override anything else. Violating them requires a redo.

4. `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` - locked course decisions.
   Read before touching any notebook or plan file.

5. `plans/TOPICS.md` - current status of all 9 topics (not_started / planned / in_progress / done).

6. `AUDIT.md` - history of all fixes. Shows decisions that were made and must not be reversed.

7. `TODOS.md` - open issues (check if anything is still open).

## Step 2: Read the relevant plan file for the topic you will work on

Plan files live at `plans/topic_NN_slug.md` (gitignored, but present on disk).
These are the cell-by-cell blueprints for building notebooks.

For Topic 4 (most likely next): `plans/topic_04_prompt_engineering.md`
For Topic 5: `plans/topic_05_conversation_memory.md`
For Topic 6: `plans/topic_06_rag_foundations.md`
For Topic 7: `plans/topic_07_advanced_rag_web_search.md`
For Topic 8: MISSING - all research is in cursor-handover.txt Section 6. Write the plan first.
For Topic 9: MISSING - all research is in cursor-handover.txt Section 7. Write the plan first.

## Step 3: Check current notebook state before editing

Always run this before touching any notebook:

```bash
.venv/bin/python3 -c "
import json
with open('PATH_TO_NOTEBOOK') as f:
    nb = json.load(f)
for i, c in enumerate(nb['cells']):
    print(i, c.get('id','?'), c['cell_type'], repr(''.join(c['source'])[:60]))
"
```

This gives you the current cell_id for every cell. You MUST pass cell_id in every
notebook edit after the first cell. Never assume a cell is at a position - verify first.

## Step 4: Read the existing slash commands in .claude/commands/

These markdown files define the behaviors Claude Code used. Even if Cursor does not run
them as slash commands, reading them tells you exactly what process was used:

- `.claude/commands/build-topic-notebook.md` - how notebooks are built (5 cells at a time)
- `.claude/commands/build-diagrams.md` - how Mermaid diagrams are built and inlined
- `.claude/commands/run-research-topic.md` - the 5-cycle research process
- `.claude/commands/validate-notebooks.md` - how validation works

## Step 5: Orient yourself and confirm next action

After reading all the above, print a summary like:

```
ORIENTED. Current state:
  Topics done: 01, 02, 03
  Topics with plans but no notebooks: 04 (in progress), 05, 06, 07
  Plans missing (need writing): 08, 09
  
Next action: [state what you will do and why]
```

Then proceed with the work. Do not start editing notebooks until you have read
cursor-handover.txt, CLAUDE.md, and the relevant plan file completely.

## Critical rules to keep in mind at all times

- NEVER add more than 5 notebook cells without validating
- ALWAYS verify cell_id before every notebook edit
- NEVER delete a cell - ask the user first, every time
- NEVER hardcode API keys
- ALWAYS pin numpy<2 in every install cell
- NO em dashes, NO en dashes in any notebook cell content
- Build exercise FIRST, get approval, THEN build solution
- Keep safety-net cells in solution notebooks - never delete them
- Diagrams must be INLINED as ```mermaid fenced blocks in markdown cells (not linked files)
- First-person voice throughout: "I'm going to show you..." not "In this section we will..."
- Run .venv/bin/python3 validate_notebooks.py after every 5-cell batch

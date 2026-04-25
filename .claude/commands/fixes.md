---
description: Log a problem, research it, fix it in the affected notebooks, and record the change in AUDIT.md
---

Fix a problem: $ARGUMENTS

## GUARD: Read Course Manifest First

Before doing anything else, check for `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md`:

```bash
ls plans/CORE_TECHNOLOGIES_AND_DECISIONS.md 2>/dev/null
```

If the file does NOT exist, stop immediately and say:

> `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` not found. Run `/init-course` first to record the course decisions. Every command requires this file before proceeding.

If it exists, read the full file and keep its contents in mind throughout this command.

---

This command handles any notebook issue - wrong code, bad cell order, incorrect API usage,
pedagogy problems, stale library versions, broken imports, or anything else. It follows a
strict log-first, research, fix, audit trail.

---

## Command Arguments

Describe the problem as clearly as possible. Examples:

- `/fixes topic_04 - the few-shot prompt demo uses the wrong API syntax for openai v1`
- `/fixes topic_06 - ChromaDB collection.query() call is broken, API changed in v0.6`
- `/fixes topic_03 - lab 2 safety-net cell is missing`
- `/fixes topic_07 - cell order is wrong, the retrieval demo comes before the embedding demo`

---

## Phase 1: Log the Problem (DO THIS BEFORE ANY OTHER ACTION)

### 1a. Append to TODOS.md

Read `TODOS.md` first if it exists. Then append (do not overwrite):

```markdown
## [OPEN] <short title> - <YYYY-MM-DD>

**Topic**: topic_NN_slug
**Reported**: <today's date>
**Description**: <full description from the command argument>
**Status**: open
**Fix applied**: (pending)
```

If `TODOS.md` does not exist, create it with a header first:

```markdown
# TODOS and Known Issues
# Barclays - Generative AI: Prompt Engineering for Software Developers
#
# Entries are added by /fixes and resolved entries are moved to AUDIT.md.

---
```

### 1b. Update plans/TOPICS.md

Find the affected topic entry in `plans/TOPICS.md` (read it first). Change its status to
`in_progress` if it is currently `done` or `planned`. Add a note under its entry:

```markdown
### Open Issues
- [ ] <short description of the fix needed> (added <YYYY-MM-DD>)
```

---

## Phase 2: Research the Fix (1-2 Cycles Only)

This is NOT a full 5-cycle research pass. Run exactly the cycles needed - usually 1, max 2.

**For ALL web research: invoke the `/research` skill with a specific, targeted question.**
Do not attempt raw web lookups yourself.

### Cycle 1 - Diagnose and Propose Fix

- Read the affected notebook(s): exercise and solution for the topic
- Confirm the exact cell(s) involved - read the notebook JSON and list each cell's `id` and
  first line of source to locate the problem cell precisely
- State the diagnosis: what is wrong and why
- Propose the specific fix: exact code change, cell reorder, or content edit

**If the issue is a library API change**: invoke `/research` with:
"What is the current API for [function/class] in [library] version [X]? What changed from [old version]?"

**If the issue is a cell order problem**: do NOT research - diagnose from reading the notebook
directly. Skip to Phase 3.

**If the issue is a pedagogy problem** (lab too hard, hint leaks answer, demo missing):
invoke `/research` with the specific question about best practices.

### Cycle 2 (only if Cycle 1 raised new unknowns)

- Run a second targeted `/research` call to resolve any remaining uncertainty
- If Cycle 1 gave a clear answer, skip Cycle 2 entirely

---

## Phase 3: Apply the Fix

### CRITICAL: Verify Cell IDs Before EVERY Edit (no exceptions)

Before EVERY NotebookEdit call (replace, insert, OR delete), run this command to print the
current cell order and confirm the target cell_id is at the expected position:

```bash
.venv/bin/python3 -c "
import json
with open('exercises/topic_NN_slug/topic_NN_slug.ipynb') as f:
    nb = json.load(f)
for i, c in enumerate(nb['cells']):
    print(i, c.get('id','?'), c['cell_type'], repr(''.join(c['source'])[:60]))
"
```

Read the output. Confirm the cell_id you are about to edit is:
1. At the position you expect (the right cell number)
2. Contains content matching what you intend to modify

Only THEN make the edit. This step is MANDATORY - skip it and you risk deleting or overwriting
the wrong cell. Run it before EVERY individual NotebookEdit, not just once at the start.

### Fix Rules

- **NEVER delete a cell. Ever.** Only replace cell content. If a cell needs to be removed,
  stop and ask Axel explicitly. Deleting without approval is forbidden regardless of context.
- **One cell at a time**: edit one cell, then re-verify the cell order before the next edit
- **Cell order fixes**: if cells need reordering, read the full cell list, plan the new order,
  then use NotebookEdit with the correct `cell_id` to insert cells in the right position
- **Apply to BOTH notebooks**: fix the exercise notebook first, then apply the same fix to
  the solution notebook. Never fix only one.
- **Run validation after each notebook**:
  ```bash
  python validate_notebooks.py exercises/topic_NN_slug/topic_NN_slug.ipynb --type exercise
  python validate_notebooks.py solutions/topic_NN_slug/topic_NN_slug.ipynb --type solution
  ```
- **If cell count changes** (added or removed cells), run the pair check:
  ```bash
  python validate_notebooks.py --pair exercises/.../nb.ipynb solutions/.../nb.ipynb
  ```

---

## Phase 4: Update the Audit Trail

### 4a. Mark TODOS.md entry as resolved

Read `TODOS.md`, find the entry for this fix, and update it:

```markdown
## [RESOLVED] <short title> - <YYYY-MM-DD>

**Topic**: topic_NN_slug
**Reported**: <original date>
**Resolved**: <today's date>
**Description**: <original description>
**Status**: resolved
**Fix applied**: <one paragraph describing exactly what was changed and why>
```

### 4b. Append to AUDIT.md

Read `AUDIT.md` first if it exists. Then append:

```markdown
## <YYYY-MM-DD> - <short title>

**Topic**: topic_NN_slug
**Exercise notebook**: exercises/topic_NN_slug/topic_NN_slug.ipynb
**Solution notebook**: solutions/topic_NN_slug/topic_NN_slug.ipynb

**Problem**: <what was wrong>

**Root cause**: <why it was wrong>

**Fix**: <what was changed, cell IDs affected, code before and after if applicable>

**Validation**: exercise passed / solution passed / pair check passed
```

If `AUDIT.md` does not exist, create it with a header:

```markdown
# Change and Fix Audit Log
# Barclays - Generative AI: Prompt Engineering for Software Developers
#
# Every fix applied by /fixes is recorded here permanently.
# TODOS.md tracks open issues; AUDIT.md tracks resolved ones.

---
```

### 4c. Update plans/TOPICS.md

Read `plans/TOPICS.md`, find the affected topic entry, and:

- Move the `[ ]` issue under "Open Issues" to `[x]` with the resolution date
- If no open issues remain and both notebooks exist, set status back to `done`

---

## Phase 5: Confirm

End your turn with a summary:

> Fix applied to `topic_NN_slug`.
>
> - TODOS.md: entry marked resolved
> - AUDIT.md: change recorded
> - plans/TOPICS.md: status updated
> - Validation: exercise [pass/fail], solution [pass/fail], pair check [pass/fail/skipped]
>
> If validation failed, describe what still needs to be done.

---

## Non-Negotiables

1. NEVER delete a cell without explicit approval from Axel - ask first, every time
2. ALWAYS log to TODOS.md BEFORE touching any notebook
3. ALWAYS read the notebook and verify cell IDs BEFORE calling NotebookEdit
4. ALWAYS fix both exercise AND solution notebooks
5. ALWAYS run validation after each notebook fix
6. ALWAYS update AUDIT.md - this log is permanent and never deleted
7. Research via `/research` skill only - no raw web lookups
8. Maximum 2 research cycles - this is a fix, not a full research pass

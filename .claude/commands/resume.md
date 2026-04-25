# /resume

Load the saved session state from `progress.txt` and orient the current conversation
so work can continue immediately. Ask at most one clarifying question.

---

## Process

### Step 1: Check for progress.txt

```bash
ls progress.txt 2>/dev/null
```

If not found:

> `progress.txt` not found. Run `/save-state` first to snapshot session state, or
> run `/start-session` to load context from scratch.

Stop here if not found.

### Step 2: Read progress.txt

Read the full file. Extract:
- All topic statuses from the TOPIC STATUS section
- All open issues from KNOWN ISSUES
- All locked decisions from LOCKED DECISIONS
- The NEXT ACTION line

### Step 3: Read supporting files to verify state is current

The progress.txt snapshot may be hours or days old. Verify it is still accurate:

```bash
ls exercises/topic_*/topic_*.ipynb 2>/dev/null
ls solutions/topic_*/topic_*.ipynb 2>/dev/null
```

Cross-check: does the filesystem match what progress.txt says?
- If a notebook exists that progress.txt says is missing -> update your understanding
- If progress.txt says done but the notebook is missing -> flag it

Do NOT re-run full validation. Trust progress.txt unless filesystem contradicts it.

### Step 4: Print orientation summary

Print this exact format to chat (no preamble, no "let me tell you what I found"):

```
=== RESUMED SESSION ===

Saved: <date from progress.txt>

DONE
  Topic 01 - Foundations (exercise + solution)

IN PROGRESS / NEXT
  Topic 02 - NLP Preprocessing  [plan exists, no notebooks yet]

NOT STARTED
  Topics 03-09

OPEN ISSUES
  <list from TODOS or "none">

NEXT ACTION
  /build-topic-notebook 2
  Reason: plan exists at plans/topic_02_nlp_preprocessing.md, no exercise built yet.

KEY DECISIONS (active)
  - OpenAI only (gpt-4o)
  - Safety-net cells: keep in solution notebooks, ask before deleting
  - Verify cell_id with python snippet before every NotebookEdit
  - Notebooks are self-contained / independent
  - numpy<2 always pinned
  - No branding

=== READY ===
```

### Step 5: Ask at most one clarifying question

After printing the summary, ask ONE question only if the intent is genuinely ambiguous:

- If NEXT ACTION is obvious from progress.txt -> do not ask, just confirm ready
- If multiple topics are in_progress simultaneously -> ask "Which topic do you want to
  continue with first?"
- If progress.txt NEXT ACTION conflicts with filesystem state -> describe the conflict
  and ask which is correct

Never ask more than one question. Never ask for information already in progress.txt or
the filesystem.

---

## Non-Negotiables

- Read progress.txt first, verify against filesystem second
- Print the orientation summary before asking anything
- One question max - if you have to ask more than one, you have not read the files carefully enough
- Do not start building or fixing anything unless the user says "go" or "continue"

---
description: Begin a work session - load context, review progress, identify next steps
---

Start a new work session for this project.

## GUARD: Read Course Manifest First

Before doing anything else, check for `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md`:

```bash
ls plans/CORE_TECHNOLOGIES_AND_DECISIONS.md 2>/dev/null
```

If the file does NOT exist, stop immediately and say:

> `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` not found. Run `/init-course` first to record the course decisions. Every command requires this file before proceeding.

If it exists, read the full file and keep its contents in mind throughout this command.

---

## Step 1: Load All Context

Read these files in order:

1. `CLAUDE.md` - full project context, rules, and workflow
2. `initial_docs/outline.pdf` - course outline (9 topics)
3. `initial_docs/technical_setup.md` - Barclays infra spec

## Step 2: Load TOPICS.md (Primary Source of Truth)

Read `plans/TOPICS.md` if it exists - this is the authoritative status tracker.

If it does NOT exist, say:
> `plans/TOPICS.md` not found. Run `/dissect-topics` first to initialize the topic manifest.

If it does exist, read the Summary table and the Open Issues sections for each topic.

Also check for open items in TODOS.md:
```bash
grep -c "\[OPEN\]" TODOS.md 2>/dev/null || echo "0"
```

## Step 3: Cross-Check Filesystem

Verify TOPICS.md matches reality:

```bash
find exercises/ solutions/ -name "*.ipynb" -exec wc -c {} \; 2>/dev/null
```

Flag any notebook that:
- Exists in the filesystem but is not marked done/in_progress in TOPICS.md
- Is marked done but the file is suspiciously small (under 10KB)
- Has an open issue in TOPICS.md

## Step 4: Report Status

Report the Summary table from TOPICS.md, plus:
- Any open TODOS.md items (list them)
- Any filesystem mismatches found in Step 3

## Step 5: Recommend Next Step

State clearly:
- Next topic to work on (first not_started or in_progress)
- Whether to run `/run-research-topic N` (no plan), `/build-topic-notebook N` (plan exists), or `/fixes <description>` (open issues)
- If TOPICS.md is missing: recommend `/dissect-topics` first

## Step 6: Ask

Ask the user: "Ready to proceed with [recommended next step]? Or is there something else you want to work on?"

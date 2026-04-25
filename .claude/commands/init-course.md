---
description: Initialize plans/CORE_TECHNOLOGIES_AND_DECISIONS.md by asking questions one at a time about the course
---

Initialize the course manifest: $ARGUMENTS

This command creates `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` - the single source of truth
that every other command reads before doing any work. Run this once at the start of a new cohort
or when course details change.

---

## Guard

If `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` already exists, read it first and show the user
the current contents, then ask: "Do you want to update specific sections, or start fresh?"
If updating: only re-ask questions for sections the user wants to change.
If starting fresh: proceed with the full interview below.

---

## Process: One Question at a Time

Ask each question individually. Wait for the answer before asking the next one.
Do NOT dump all questions at once.
After each answer, confirm you understood it before moving on.

Keep a running draft in memory as answers come in.
After the final question, show the complete draft and ask for approval before writing the file.

---

## Questions (ask in this order)

**Block 1 - The Students**

Q1: "Who are the students? What's their job at Barclays - backend devs, full-stack, architects, data engineers, or a mix?"

Q2: "How comfortable are they with Python? And have they used any LLM APIs before this course?"

Q3: "Any pain points or misconceptions you've seen in past similar cohorts that I should design around?"

**Block 2 - The Course Arc**

Q4: "The capstone is a Customer Service Assistant. What specific Barclays scenario does it handle? (e.g. account opening, credit cards, mortgages, or a mix?)"

Q5: "Is there one core document set or dataset we carry through all 9 topics as the running example? If so, what is it - a Barclays product PDF, FAQ docs, terms and conditions, something else?"

Q6: "Do students build on their notebooks across days (Topic 3 code feeds Topic 5), or does each day start fresh?"

**Block 3 - The Environment**

Q7: "Do students have open internet access from inside SageMaker? This affects how we teach the web search topic."

Q8: "Are the OpenAI and Anthropic API keys provided by Barclays/Datacouch, or do students bring their own?"

Q9: "Any model restrictions? For example, only GPT-4o, no o4-mini, specific Claude models only, or free choice?"

Q10: "What Python packages are pre-installed in the SageMaker image? Or do we pip install everything fresh in each notebook?"

**Block 4 - The Delivery**

Q11: "Is this instructor-led live coding (you code on screen, students follow along), or students work independently while you circulate?"

Q12: "Roughly how long is each topic slot? Are all 9 topics equal time, or do some get more?"

Q13: "Will diagrams be shown on a projector during class, or only visible in student notebooks on their own screen?"

**Block 5 - Tone and Constraints**

Q14: "Any Barclays-specific compliance or brand constraints I should know? For example, topics to avoid, terminology to use or avoid, examples that would be inappropriate?"

Q15: "Anything else about how you want to teach this that isn't captured above - a strong opinion about pacing, lab style, what makes a good session vs a bad one?"

---

## After All Answers: Write the File

Show the complete draft in chat and ask for approval.

Once approved, write `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` with this structure:

```markdown
# Course Manifest: Generative AI - Prompt Engineering for Software Developers
# Barclays cohort
# Last updated: <YYYY-MM-DD>
#
# This file is read by every command before any work begins.
# Update it by running /init-course.

---

## The Students

- **Role at Barclays**: <answer>
- **Python level**: <answer>
- **LLM experience**: <answer>
- **Known pain points / misconceptions**: <answer>

## The Course Arc

- **Capstone scenario**: <answer>
- **Core running example / dataset**: <answer>
- **Continuity across days**: <answer - do notebooks carry over or restart?>

## Environment

- **SageMaker internet access**: <yes / no / restricted>
- **API keys**: <provided by Barclays / students bring own>
- **Model restrictions**: <list or "none">
- **Pre-installed packages**: <list or "pip install everything">
- **Instance**: ml.t3.medium, no GPU, us-east-2, Python 3.11

## Delivery

- **Teaching style**: <live coding / independent work / mixed>
- **Time per topic**: <e.g. "~60 min each" or specific per-topic times>
- **Diagrams**: <projector / student screens only>

## Constraints and Tone

- **Compliance / brand constraints**: <answer>
- **Teaching philosophy notes**: <answer>

## Key Decisions (locked)

- Environment: SageMaker Studio JupyterLab on ml.t3.medium
- S3 datasets bucket: barclays-prompt-eng-data (read-only)
- Default OpenAI model: gpt-4o
- Default Anthropic model: claude-sonnet-4-6
- Credentials: getpass for API keys, sagemaker.Session() + get_execution_role() for AWS
- numpy: always pin numpy<2
- No em dashes, no en dashes, no bare --- separators in notebook cells
- Emojis and first-person voice are encouraged
```

---

## Confirm

After writing, say:

> `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` written.
>
> Every command will now read this file before starting work. Run `/start-session` to see your current progress, or `/run-research-topic 1` to begin Topic 1.

# Generative AI: Prompt Engineering for Software Developers - Repository Guide

## Project Overview

This repository contains all student and instructor materials for the **Barclays Generative AI: Prompt Engineering for Software Developers** course - a 3-day intensive hands-on training for software developers.

**Program Details:**
- 25 students per cohort
- 3-day intensive hands-on course
- Environment: Ubuntu VM (jumphost) SSH'd into AWS SageMaker Studio JupyterLab
- Instance type: ml.t3.medium (no GPU needed - this course is API calls only)
- Region: us-east-2 (Ohio)
- S3 datasets bucket: barclays-prompt-eng-data (read-only)

**Course Arc:**
Students build a production-ready Barclays Customer Service Assistant over 3 days, progressing from LLM API basics through RAG, web search, guardrails, and production best practices.

## Repository Structure

```
prompt_enginerring/
├── CLAUDE.md                          # This file
├── exercises/                         # Student notebooks (distributed before each session)
│   ├── topic_01_foundations/
│   │   └── topic_01_foundations.ipynb
│   ├── topic_02_nlp_preprocessing/
│   │   └── topic_02_nlp_preprocessing.ipynb
│   ├── topic_03_first_chatbot/
│   │   └── topic_03_first_chatbot.ipynb
│   ├── topic_04_prompt_engineering/
│   │   └── topic_04_prompt_engineering.ipynb
│   ├── topic_05_conversation_memory/
│   │   └── topic_05_conversation_memory.ipynb
│   ├── topic_06_rag_foundations/
│   │   └── topic_06_rag_foundations.ipynb
│   ├── topic_07_advanced_rag_web_search/
│   │   └── topic_07_advanced_rag_web_search.ipynb
│   ├── topic_08_ethical_guardrails/
│   │   └── topic_08_ethical_guardrails.ipynb
│   └── topic_09_capstone/
│       └── topic_09_capstone.ipynb
├── solutions/                         # Complete solutions (shared after each session)
│   └── (mirrors exercises/ structure)
├── plans/                             # Research and cell-by-cell plans (gitignored)
│   └── topic_N_slug.md
├── initial_docs/                      # Course reference documents
│   ├── outline.pdf                    # Original course outline PDF
│   └── technical_setup.md            # Barclays infrastructure spec
├── validate_notebooks.py              # Notebook validation script
├── requirements.txt                   # Python deps for this repo tooling
└── .venv/                             # Local virtualenv (gitignored)
```

## Course Topics

The course covers 9 topics across 3 days, each building toward the capstone:

| # | Topic | Day | Key Libs |
|---|-------|-----|----------|
| 1 | Foundations - LLMs, APIs, Tokens | Day 1 | openai, anthropic |
| 2 | NLP Preprocessing for RAG | Day 1 | pymupdf, beautifulsoup4 |
| 3 | Your First Chatbot | Day 1 | openai, anthropic |
| 4 | Prompt Engineering | Day 2 | openai, anthropic |
| 5 | Conversation Memory | Day 2 | openai, anthropic |
| 6 | RAG Foundations | Day 2 | chromadb, openai |
| 7 | Advanced RAG and Web Search | Day 3 | chromadb, openai |
| 8 | Ethical Guardrails | Day 3 | openai, anthropic |
| 9 | Capstone: Customer Service Assistant | Day 3 | all of the above |

## Notebook Naming Convention

- Format: `topic_NN_slug_name.ipynb`
- Examples:
  - `topic_01_foundations.ipynb`
  - `topic_04_prompt_engineering.ipynb`
  - `topic_09_capstone.ipynb`

## Environment

**All notebooks run inside SageMaker Studio JupyterLab** accessed from a student Ubuntu VM.
- No GPU. Instance is ml.t3.medium.
- SageMaker session and execution role are available via `sagemaker.Session()` + `get_execution_role()`.
- API keys (OpenAI, Anthropic) are passed via `getpass` or environment variables - NOT hardcoded.
- S3 bucket for datasets: `barclays-prompt-eng-data`
- Python version: 3.11 (SageMaker Distribution default image)
- All pip installs use `!pip install -q` inside notebook cells.
- Pin `numpy<2` in every install cell.

**Credential pattern (SageMaker):**
```python
import sagemaker
from sagemaker import get_execution_role
import getpass, os

sess = sagemaker.Session()
role = get_execution_role()

# For external API keys only (OpenAI, Anthropic)
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
os.environ["ANTHROPIC_API_KEY"] = getpass.getpass("Anthropic API Key: ")
```

## Notebook Structure and Teaching Philosophy

### 1. Storytelling - Start With the Problem, Not the Solution
- I open every topic by showing the PROBLEM first, not the answer
- Students see a real broken or naive attempt before they see the right way
- Example flow: "Here's a customer message. Watch what happens if we just dump it raw into the API... (bad output). Now let's fix that."
- This creates the motivation for every concept we introduce

### 2. Build Up to Each Concept in Four Beats
Every concept in a notebook follows this exact arc:

**Beat 1 - Problem Introduction (Markdown + broken/naive code)**
Show what goes wrong without this concept. Make students feel the pain before the cure.

**Beat 2 - Diagram (Markdown with Mermaid placeholder)**
A visual of how the concept works. During notebook building I insert a placeholder comment. The `/build-diagrams` command fills these in one by one afterward.
Placeholder format:
```
<!-- DIAGRAM: describe what this diagram should show -->
```

**Beat 3 - Full Demo (Code cell, complete and runnable)**
A working solution demonstrated by the instructor. Heavily commented. Students watch first, then do.

**Beat 4 - Lab**
Students implement it themselves. See lab structure rules below.

### 3. Lab Structure - Three Tiers Per Topic
Each topic has labs at three tiers. The tiers are not labeled by difficulty in the notebook - students discover the challenge:

**Tier 1 - Guided labs (most labs, ~2 per topic)**
Medium difficulty. `variable = None  # YOUR CODE` placeholders with numbered step comments above. Verification code at the end so students know if they got it right. Achievable in 15-20 minutes.

**Tier 2 - Hard lab (one per day, not one per topic)**
One lab per day is harder: multi-step, requires combining what was learned earlier in the session. Still uses `None  # YOUR CODE` placeholders but the steps are less spelled out. Students need to think, not just fill in blanks. Aim for 25-35 minutes.

**Tier 3 - Open-ended lab (one per day, at end of last topic of that day)**
No `None` placeholders. No step-by-step hints. Just a clear problem statement and a docstring/comment block describing inputs and expected outputs. Students figure out the approach themselves. These are the "go home and finish it" labs. No verification code - the proof is whether it works end-to-end.

### 4. Heavy Documentation
- Code cells: every non-obvious line has a comment explaining the WHY
- Markdown cells: detailed explanations with inline ```python``` examples
- Demo cells are fully commented as live-coding reference
- Lab starter code has enough structure to orient students without revealing the answer

### 5. Tone - First Person, Friendly, Encouraging
- I write in first person: "I'm going to show you...", "Let's build...", "Here's what I ran into..."
- Conversational and warm - students may be new to AI and need to feel capable
- No jargon unless I explain it in the same cell
- Emojis are welcome in markdown cell headers and section titles

### 6. Public/Provided Data Only
- HuggingFace datasets, public URLs, or barclays-prompt-eng-data S3 bucket
- No private data hardcoded
- Document dataset sources clearly

### 7. Building Toward the Capstone
- Every topic explicitly names which component of the Customer Service Assistant it builds
- Variables and helpers from earlier topics are reused - students feel the accumulation
- Topic 6 RAG output feeds Topic 7. Topic 3 chatbot becomes Topic 5 memory chatbot.

## Notebook Cell Structure Per Topic

### Header Section (Cells 0-4)
1. **Cell 0 (Markdown)**: Topic title with emoji, learning objectives, what component of the assistant we are building today
2. **Cell 1 (Markdown)**: "Section 0: Environment Setup"
3. **Cell 2 (Code)**: `!pip install -q ...` with pinned versions + `numpy<2`
4. **Cell 3 (Code)**: All imports + SageMaker session + API keys via getpass
5. **Cell 4 (Markdown)**: "What are we building today?" - names the Customer Service Assistant component, shows a before/after teaser

### Per Concept (repeat 2-4 times per notebook, each concept = 4-6 cells)
1. **Markdown**: Concept header + problem introduction (what breaks without this concept)
2. **Code**: Naive/broken demo showing the problem
3. **Markdown**: Diagram placeholder + explanation of how the concept solves it
   - Insert `<!-- DIAGRAM: ... -->` here for `/build-diagrams` to fill later
4. **Code**: Full working demo (complete, heavily commented, runnable)
5. **Markdown**: Lab instructions
6. **Code**: Lab starter (Tier 1 guided, or Tier 2 hard, or Tier 3 open-ended - one Tier 2 per day, one Tier 3 per day at the end of the last topic)
7. **Code** (conditional): Safety-net cell if lab output feeds a later cell

### Closing Section
1. **Markdown**: Key takeaways, what I built, connection to next topic, homework extensions

## Exercise vs Solution Notebooks

### Exercise Notebook
- Distributed to students before the session
- Lab cells use `variable = None  # YOUR CODE` pattern
- `# YOUR CODE` comment MUST NOT reveal the answer
- Safety-net cells follow any lab whose output feeds a later cell
- Verification code included so students know if they got it right

### Solution Notebook
- Shared with students after the session
- Built by copying the exercise notebook, then filling in the `None` placeholders
- Solution cells include: full implementation + explanatory comments + common mistakes
- **Safety-net cells are KEPT** - do not delete them. They maintain cell parity with the
  exercise. In the solution they never fire, but removing them breaks the pair validator
  by shifting all downstream cell positions. Ask before deleting any cell.

## CRITICAL Rules (Violating Any = Redo)

1. **Comments must be single-line only**: Every `#` comment in a code cell must fit on one line. Never wrap a comment across multiple lines. If a comment is too long, shorten it. The NotebookEdit tool has caused char-by-char rendering bugs when multi-line comment blocks were used - single-line comments avoid this entirely.
2. **No AI typography in notebook cells or plan files**:
   - NO em dashes (the long dash: —). Use a plain hyphen or rewrite the sentence.
   - NO en dashes (–). Use a plain hyphen.
   - NO horizontal rule separators (---) inside notebook markdown cells. Use a blank line or a header instead.
   - NO Unicode multiplication sign (×). Write "x" or "times".
   - NO bullet unicode symbols like •, ✓, ✗ in code cell output strings. Use plain ASCII: *, [x], [ ].
   - Emojis ARE allowed and encouraged in markdown cell headers and section titles.
   - First person voice IS the correct tone. Write "I'm going to show you..." not "In this section we will...".
3. **5 cells at a time**: Never add more than 5 cells without stopping and asking for approval.
4. **cell_id always**: After the first cell, ALWAYS pass `cell_id` to NotebookEdit to avoid reordering.
   - **MANDATORY before ANY NotebookEdit**: run this python snippet first to verify the target cell_id and its position:
     ```bash
     .venv/bin/python3 -c "
     import json
     with open('PATH_TO_NOTEBOOK') as f:
         nb = json.load(f)
     for i, c in enumerate(nb['cells']):
         print(i, c.get('id','?'), c['cell_type'], repr(''.join(c['source'])[:60]))
     "
     ```
   - Never assume a cell_id is at the right position - always verify the order first.
5. **NEVER delete a cell**: Only replace content or insert new cells. Deleting a cell requires explicit approval from Axel every time - ask first, no exceptions. This applies to exercise notebooks, solution notebooks, and any future notebook regardless of context.
6. **Exercise first**: Build the full exercise notebook and get approval BEFORE starting the solution.
7. **No answer hints in `# YOUR CODE`**: The comment after `# YOUR CODE` must not describe the solution.
8. **Safety-net cells required**: Any lab output used by a later cell needs a gated fallback cell in the exercise notebook.
9. **numpy<2 always**: Pin numpy<2 in every install cell.
10. **API keys via getpass**: Never hardcode API keys. Always use `getpass.getpass()` or `os.environ`.
11. **Validate after every 5 cells**: Run `python validate_notebooks.py <path> --type exercise` after each batch.
12. **Plans folder gitignored**: Never commit plan files (except TOPICS.md, TODOS.md, AUDIT.md).
13. **AI-tells scan before finalizing any notebook**: Run the AI-tells checker on every completed notebook before marking it done. See `/validate-notebooks` for the command.
14. **After any NotebookEdit, run the source normalization check**: Run `.venv/bin/python3 -c "import json; nb=json.load(open('PATH')); [print('BAD', i, c.get('id')) for i,c in enumerate(nb['cells']) if c['cell_type']=='code' and len(c['source'])>5 and sum(1 for s in c['source'][:20] if len(s)<=2)>=15]"` to confirm no cells were stored char-by-char. If any are found, re-run the normalization script immediately before committing.

## Workflow (Standard for Every Topic)

1. Run `/run-research-topic N` to produce `plans/topic_N_slug.md`
2. Review and approve the plan
3. Run `/build-topic-notebook N sagemaker` to build notebooks incrementally
4. Validate after each 5-cell batch
5. Approve full exercise notebook
6. Build solution notebook (copy + fill)
7. Final pair validation

## Commands Available

- `/dissect-topics` - Parse the outline and initialize plans/TOPICS.md with status tracking
- `/run-research-topic N` - Research a topic and produce a cell-by-cell plan
- `/build-topic-notebook N` - Build exercise + solution notebooks from the plan (5 cells at a time)
- `/build-diagrams N` - Build Mermaid `.mmd` files into `plans/topic_N/diagrams/` one by one, with approval between each
- `/validate-notebooks` - Run validation checks including AI-tells scan
- `/fixes <description>` - Log a problem, research it, fix notebooks, record in AUDIT.md
- `/start-session` - Load context, read TOPICS.md, and identify next step

## Validation Script

Run `python validate_notebooks.py` to check notebooks. See `initial_docs/` for full docs.

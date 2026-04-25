# Course Manifest: Generative AI - Prompt Engineering for Software Developers
# Barclays cohort
# Last updated: 2026-04-25
#
# This file is read by every command before any work begins.
# Update it by running /init-course.

---

## The Students

- **Role at Barclays**: Mixed - data engineers and full-stack developers wanting to use
  prompt engineering in their apps, agents, and daily work
- **Python level**: Intermediate - comfortable with syntax, classes, and logic; may have
  called an LLM API once or can adapt quickly
- **LLM experience**: Minimal to none; treat first API call as near-new but do not
  over-explain Python basics
- **Known pain points / misconceptions**: First cohort - no prior data. Design for maximum
  "aha moments" grounded in scenarios they would actually build at Barclays

## The Course Arc

- **Running problem (Topics 1-8)**: Barclays Product Knowledge Assistant - a chatbot that
  answers customer service agent questions about Barclays products, built up topic by topic
  from a raw API call to a full RAG + memory + guardrails assistant using public Barclays
  product PDFs as the corpus
- **Capstone (Topic 9)**: Barclays Transaction Query Agent - a single-pass tool-augmented
  agent students build independently; classifies transaction-related customer intent,
  calls manually coded tools (policy lookup via RAG, mock transaction lookup, human
  escalation), returns a grounded compliant response; uses Banking77 HuggingFace dataset
  for intent examples
- **Core dataset**: Public Barclays PDFs (Barclaycard T&C, personal loan FAQ, credit card
  product summaries) pre-loaded to barclays-prompt-eng-data S3 bucket by instructor
  before class; Banking77 from HuggingFace for capstone
- **Continuity across days**: Notebooks are independent and self-contained. Each rebuilds
  what it needs. If a prior topic artifact exists (e.g. ChromaDB collection), later topics
  can optionally load it - if not, they rebuild from scratch. No topic fails because a
  previous one was skipped.

## Environment

- **SageMaker internet access**: Yes - open internet access
- **API keys**: Provided by instructor at start of class; students set via os.environ in
  first cell using getpass
- **Model restrictions**: OpenAI only - default model gpt-4o; no Anthropic SDK in
  student notebooks
- **Pre-installed packages**: Standard SageMaker Distribution image - pip install
  everything explicitly in each notebook; always pin numpy<2; no torch needed (API calls
  only, no local model inference)
- **Instance**: ml.t3.medium, no GPU, us-east-2, Python 3.11

## Delivery

- **Teaching style**: Instructor-led live coding on screen via Zoom; students follow
  along then do labs independently
- **Time per topic**: Flexible - 3 days total, minimum 3 hours reserved for capstone;
  instructor expands or compresses topics based on room pace; notebooks must be
  self-contained so topics can be skipped without breaking later ones
- **Diagrams**: Rendered in JupyterLab on student and instructor screens via Zoom screen
  share; Mermaid diagrams render natively in JupyterLab markdown cells

## Constraints and Tone

- **Compliance / brand constraints**: No branding of any kind - no Datatrainers, no
  Pluralsight, no external company logos or names; mentioning Barclays in scenario
  context is appropriate since the course is for Barclays developers
- **Teaching philosophy notes**: Every lab must have a stretch version for fast finishers
  (varied difficulty to entertain the whole room); maximize "aha moments" per topic by
  grounding every concept in the running Barclays scenario; first-person instructor voice
  throughout; emojis welcome in headers; no AI typography tells

## Key Decisions (locked)

- Environment: SageMaker Studio JupyterLab on ml.t3.medium
- S3 datasets bucket: barclays-prompt-eng-data (read-only, instructor pre-loads before class)
- Default model: gpt-4o (OpenAI only - no Anthropic in student notebooks)
- Credentials: getpass + os.environ for API keys, sagemaker.Session() + get_execution_role() for AWS
- numpy: always pin numpy<2
- No torch unless explicitly needed (this course is API-only)
- No em dashes, no en dashes, no bare --- separators in notebook cells
- Emojis and first-person voice are encouraged
- Notebook independence: every notebook is self-contained; optional loading of prior
  artifacts where beneficial but never required
- Capstone dataset: Banking77 (HuggingFace PolyAI/banking77)
- No branding: no external company names or logos in any notebook cell
- Safety-net cells in solution notebooks: KEEP them, do not delete. The solution notebook
  should maintain cell parity with the exercise - even if a safety-net never fires (because
  the solution cell above it is complete), keeping it preserves structural alignment. Only
  the None placeholder lines are replaced; the safety-net cell stays in place. Ask before
  deleting any cell from a solution notebook.

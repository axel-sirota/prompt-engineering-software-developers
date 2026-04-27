# Instructor Setup Instructions
# Barclays - Generative AI: Prompt Engineering for Software Developers

This document tells you exactly what to run, where, and when before and during
the course. Three environments are involved and each has a different role.

---

## The Three Environments at a Glance

```
+---------------------------+   +---------------------------+   +---------------------------+
|     YOUR PERSONAL AWS     |   |      DATACOUCH AWS        |   |     DATACOUCH VM          |
|  (your own AWS account)   |   |  (Datacouch-managed AWS)  |   |  (Ubuntu VM / jumphost)   |
|                           |   |                           |   |                           |
|  Run ONCE before course:  |   |  Run ON COURSE DAY        |   |  What students use:       |
|  INSTRUCTOR_TEST_SETUP    |   |  (inside SageMaker):      |   |  - SSH into jumphost      |
|  .ipynb                   |   |  INSTRUCTOR_SETUP.ipynb   |   |  - Open SageMaker Studio  |
|                           |   |                           |   |    JupyterLab via browser |
|  Creates: personal public |   |  Populates:               |   |  - Run exercise notebooks |
|  S3 bucket with synthetic |   |  barclays-prompt-eng-data |   |    in JupyterLab          |
|  PDFs for pre-course test |   |  with real Barclays PDFs  |   |                           |
+---------------------------+   +---------------------------+   +---------------------------+
       PRE-COURSE TESTING               COURSE DAY                  STUDENT ENVIRONMENT
```

---

## Who Owns What

| Resource | Owner | When set up |
|----------|-------|-------------|
| Personal public S3 bucket (synthetic PDFs) | You, your personal AWS account | Weeks before course |
| `barclays-prompt-eng-data` S3 bucket | Datacouch, Datacouch AWS account | Already exists |
| SageMaker Studio domain + users | Datacouch | Already provisioned |
| Student Ubuntu VM (jumphost) | Datacouch | Already provisioned |
| OpenAI API key | You provide to students at start of class | Course day morning |
| Anthropic API key | Not needed (OpenAI only in all notebooks) | N/A |

---

## Step 1: Pre-Course Testing (YOUR personal AWS account, weeks before)

This step lets you verify every notebook works end-to-end before course day, without
touching the Barclays-account S3 bucket.

### What you need

- Your personal AWS account (not the Datacouch/Barclays account)
- AWS credentials configured locally (aws configure, or IAM role with S3 write access)
- A Python environment with: fpdf2, pymupdf, requests, boto3, numpy<2

### What to run

Open: `setup/INSTRUCTOR_TEST_SETUP.ipynb`

Run it from your local machine or any Jupyter environment connected to YOUR AWS account.

What it does:
1. Generates 4 synthetic Barclays-style PDFs using fpdf2 (no real Barclays data needed)
2. Creates a personal public S3 bucket (e.g. axel-barclays-test-bucket-us-east-2)
3. Uploads the synthetic PDFs with public-read access (no AWS auth needed to download)
4. (Optional) Pre-chunks the PDFs and uploads barclays_chunks.json
5. Verifies all files are publicly reachable via HTTPS

### After it runs

You will see output like:
```
Personal test bucket: s3://your-name-barclays-test/
Public URL base:      https://your-name-barclays-test.s3.amazonaws.com/

One-line switch for Topic 2 notebook (Cell 3):
  S3_BUCKET = "your-name-barclays-test"
```

To test Topic 2 against your personal bucket:
- Open `exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb`
- In Cell 3, change `S3_BUCKET = "barclays-prompt-eng-data"` to your personal bucket name
- Run the notebook end-to-end
- Restore `S3_BUCKET = "barclays-prompt-eng-data"` before course day

All other topics (1, 3-9) use no S3 and can be tested locally without any bucket change.

### Important: synthetic vs real content

The synthetic PDFs have the same structure and key numbers as the real Barclays PDFs
(same APRs, same ERC wording, same product names). When students run Topic 2 with real
PDFs on course day, the output will look the same. This is by design.

---

## Step 2: Course Day - Datacouch AWS (INSTRUCTOR_SETUP.ipynb, inside SageMaker)

Run this on the morning of the course, before students start Topic 2.
Topics 1, 3-9 need no S3 setup and can proceed immediately.

### Where to run it

Inside SageMaker Studio JupyterLab on the Datacouch AWS account.
NOT on your local machine. The notebook uses the SageMaker execution role to
write to the Barclays S3 bucket - this only works inside SageMaker.

### What to run

Open: `setup/INSTRUCTOR_SETUP.ipynb`

Run all cells. It does:
1. Permission probe: verifies your execution role can write to `barclays-prompt-eng-data`
   - If this fails with AccessDenied, contact Datacouch to add `s3:PutObject` to your role
2. Downloads 4 real Barclays PDFs from public barclays.co.uk / barclaycard.co.uk URLs
3. Uploads them to `s3://barclays-prompt-eng-data/`
4. (Optional, requires OpenAI key) Pre-chunks the PDFs and uploads `barclays_chunks.json`
5. Prints a manifest confirming all uploads

### What gets uploaded

| S3 key | Used by | Required |
|--------|---------|----------|
| barclays_personal_loan_faq.pdf | Topic 2 Cells 8, 10, 23, 24 | YES |
| barclays_credit_card_tnc.pdf | Topic 2 Cells 10, 22 | YES |
| barclays_savings_rates.pdf | Topic 6 Tier 3 stretch lab | optional |
| barclays_isa_terms.pdf | Topic 6 Tier 3 stretch lab | optional |
| barclays_chunks.json | Topics 6-9 Tier 3 stretch labs | optional |

The notebook is idempotent: re-running it skips files that already exist in S3.

### S3 bucket access model

The bucket `barclays-prompt-eng-data` is read-only for students.
Students never write to it. The SageMaker execution role reads from it using
boto3 with implicit role credentials (no key/secret needed in student notebooks).

---

## Step 3: Course Day - Student Environment (Datacouch VM)

Students do NOT interact with AWS directly. Their workflow is:

```
Student laptop
    |
    | SSH or browser
    v
Datacouch Ubuntu VM (jumphost)
    |
    | Browser to SageMaker Studio URL
    v
SageMaker Studio JupyterLab (Datacouch AWS, ml.t3.medium, us-east-2)
    |
    | Notebooks run here
    v
OpenAI API (internet access enabled on SageMaker)
AWS S3 (via SageMaker execution role, Topic 2 only)
```

### What students run

All 9 topic notebooks in `exercises/`:
- Topic 1: No AWS. OpenAI API only.
- Topic 2: Reads 2 PDFs from S3. OpenAI API for the naive demo cell.
- Topics 3-9: No AWS. OpenAI API only. ChromaDB runs locally inside SageMaker.

### API keys

At the start of each topic, students enter their OpenAI API key via getpass.
You provide the key at the beginning of the day. They do NOT enter it into a config
file - they paste it into the notebook prompt each time. This is by design (no keys
stored on disk).

---

## Quick Reference: What Runs Where

| Notebook | When | Where | AWS account |
|----------|------|-------|-------------|
| `setup/INSTRUCTOR_TEST_SETUP.ipynb` | Weeks before, pre-course testing | Your local machine or any Jupyter | YOUR personal AWS |
| `setup/INSTRUCTOR_SETUP.ipynb` | Morning of course day | SageMaker Studio JupyterLab | Datacouch AWS |
| `exercises/topic_01_foundations.ipynb` | Day 1, Session 1 | Student SageMaker | No AWS needed |
| `exercises/topic_02_nlp_preprocessing.ipynb` | Day 1, Session 2 | Student SageMaker | Datacouch AWS (read S3) |
| `exercises/topic_03_first_chatbot.ipynb` | Day 1, Session 3 | Student SageMaker | No AWS needed |
| `exercises/topic_04_prompt_engineering.ipynb` | Day 2, Session 1 | Student SageMaker | No AWS needed |
| `exercises/topic_05_conversation_memory.ipynb` | Day 2, Session 2 | Student SageMaker | No AWS needed |
| `exercises/topic_06_rag_foundations.ipynb` | Day 2, Session 3 | Student SageMaker | No AWS needed |
| `exercises/topic_07_advanced_rag_web_search.ipynb` | Day 3, Session 1 | Student SageMaker | No AWS needed |
| `exercises/topic_08_ethical_guardrails.ipynb` | Day 3, Session 2 | Student SageMaker | No AWS needed |
| `exercises/topic_09_capstone.ipynb` | Day 3, Session 3 | Student SageMaker | No AWS needed |

---

## Checklist: Day Before the Course

- [ ] Run `INSTRUCTOR_TEST_SETUP.ipynb` from your personal AWS at least once (ideally a week before)
- [ ] Test Topic 2 end-to-end against your personal bucket - confirm PDFs load and chunk correctly
- [ ] Restore `S3_BUCKET = "barclays-prompt-eng-data"` in Topic 2 after testing
- [ ] Confirm you have the OpenAI API key to distribute to students
- [ ] Confirm SageMaker Studio is accessible via the student VM (test the jumphost login)

## Checklist: Morning of Course Day

- [ ] Log into SageMaker Studio on the Datacouch AWS account
- [ ] Open `setup/INSTRUCTOR_SETUP.ipynb` and run all cells
- [ ] Confirm the manifest shows all required PDFs as [OK]
- [ ] Distribute the OpenAI API key to students at the start of Topic 1
- [ ] Keep `setup/INSTRUCTOR_SETUP.ipynb` open - you may need to re-run Cell 12 (manifest check)
      if students report S3 errors in Topic 2

---

## Troubleshooting

### "AccessDenied" when running INSTRUCTOR_SETUP.ipynb Cell 3

Your SageMaker execution role does not have `s3:PutObject` on `barclays-prompt-eng-data`.
Contact Datacouch to add this permission to the instructor role.
Students only need `s3:GetObject` (read), which is typically pre-configured.

### Topic 2 Cell 8 fails with "NoSuchKey" or "NoSuchBucket"

The required PDFs were not uploaded. Re-run `INSTRUCTOR_SETUP.ipynb` Cells 4-7.
If the bucket itself does not exist, contact Datacouch - they own the bucket.

### Students get "ModuleNotFoundError: No module named 'openai'"

Each notebook's Cell 2 installs openai via `!pip install`. This should be automatic.
If it fails, check SageMaker internet access (should be enabled on the Datacouch domain).

### SageMaker Studio is slow to start

Normal on ml.t3.medium. Kernel startup takes 1-2 minutes. Tell students to open the
notebook and wait for the kernel indicator to show "Python 3 (ipykernel)" before running.

### ChromaDB (Topics 6-9) takes a long time on first run

First run builds the vector index locally on the ml.t3.medium instance. Expected time:
~30-60 seconds for the 7-doc BARCLAYS_DOCS corpus. Subsequent runs are instant (cached).

---

## Key Constants (do not change without testing)

| Constant | Value | Where used |
|----------|-------|------------|
| S3_BUCKET | barclays-prompt-eng-data | Topic 2 Cell 3 |
| REGION | us-east-2 | Topic 2 Cell 3 |
| MODEL | gpt-4o | All topics Cell 3 |
| openai pin | ==2.32.0 | All topics Cell 2 |
| chromadb pin | ==1.5.8 | Topics 6, 7, 9 Cell 2 |
| tiktoken pin | ==0.9.0 | Topics 5, 9 Cell 2 |
| numpy pin | <2 | All topics Cell 2 |
| ChromaDB collection | barclays_products | Topics 6, 7, 9 |

# Discrepancies in Topics 1, 2, 3 vs Setup Files and Course Manifest

Audit date: 2026-04-25
Method: /research protocol, 3 cycles, no web search (internal file analysis only)
Scope: cross-check `plans/topic_01_foundations.md`,
`plans/topic_02_nlp_preprocessing.md`, and `plans/topic_03_first_chatbot.md`
against `plans/instructor_setup.md`, `plans/test_data_setup.md`,
`plans/CORE_TECHNOLOGIES_AND_DECISIONS.md`, `plans/TOPICS.md`, and the
downstream topic plans (T4-T6) for continuity rot.

---

## Executive Summary

Topics 1, 2, 3 are the Day 1 plans. They are the most-aligned trio with
the setup notebooks (`instructor_setup.md` and `test_data_setup.md`)
because the setup notebooks were written with their exact S3 keys,
bucket name, region, synthetic-PDF content, and inline fallback strings
in mind. The S3 alignment is essentially clean.

The defects that exist are NOT AWS or S3 problems. They are:

1. T2's install line in Cell 2 does NOT install `openai`, but T2's
   Cells 6 and 13 import `from openai import OpenAI` and call the API
   in the naive demos. Self-contained-notebook constraint (CORE line 47)
   is violated. Severity HIGH.
2. The variable `PRODUCT_SNIPPET` is defined twice with different
   content. T1 Cell 11 defines a loan-only block; T3 Cell 3 defines a
   loan + credit card block under the same name. Whichever notebook
   runs second silently overrides. Severity HIGH.
3. T3 wrap-up stretch exercise 2 (Cell 25 line 1048-1049) tells students
   to use `GPT4O_INPUT_PRICE_PER_1K` and `GPT4O_OUTPUT_PRICE_PER_1K`
   "from Topic 1" but T3 never re-defines them. A student who runs T3
   alone (or runs T3 in a fresh kernel after lunch) gets `NameError`.
   Severity HIGH for the stretch exercise; LOW overall because most
   students will not attempt it.
4. T3 also tells students (Cell 24/25 stretch) to "add cost tracking
   using ask_with_cost_tracking from Topic 1" while T3 never imports or
   redefines that helper. Same NameError risk. Severity LOW (stretch
   only).
5. T2 Cell 3 prints a setup banner that says "We do NOT need the OpenAI
   key for this topic - this is pure document processing." Cell 6 then
   immediately prompts for the key with `getpass.getpass("OpenAI API
   Key: ")` and calls `client.chat.completions.create`. The Cell 3
   comment is wrong; it should be removed or rephrased. Severity MEDIUM.
6. The continuity claim that T3 "carries forward Topic 2's `chunks` for
   richer context" is only weakly true: T2's `chunks` is 100% personal
   loan FAQ text from a single PDF. T3 then truncates to the first 8
   chunks, which all describe the same product. The "richer context"
   claim is misleading. Severity LOW.
7. CORE manifest (line 84) says the capstone uses Banking77 and CORE
   line 25-31 says Topics 1-8 are the "Barclays Product Knowledge
   Assistant". TOPICS.md line 25 declares Topic 1 must call BOTH OpenAI
   AND Anthropic; CORE line 76 forbids Anthropic. T1 plan correctly
   follows CORE (OpenAI only) and ignores TOPICS.md - good outcome,
   but the TOPICS.md vs CORE conflict is a documentation defect that
   should be fixed before /build-topic-notebook 1 is re-validated.
   Severity LOW (T1 plan resolves this correctly, but TOPICS.md is
   stale).

S3, IAM, region, bucket-name, and PDF-key alignment are all CORRECT for
T1, T2, T3. No new uploads, no new IAM permissions, and no edits to
`INSTRUCTOR_SETUP.ipynb` or `INSTRUCTOR_TEST_SETUP.ipynb` are required
to support these three topics.

NOTE on TOPICS.md: TOPICS.md lines 12, 54, 93 mark Topics 1, 2, 3 as
status `done` and the manifest checkboxes as `[x]`. The corresponding
notebooks under `exercises/topic_01_foundations/`,
`exercises/topic_02_nlp_preprocessing/`, and
`exercises/topic_03_first_chatbot/` were NOT inspected as part of this
audit (the plans are the source of truth for the next build pass). If
the user intends to RE-RUN /build-topic-notebook on T1/T2/T3, the plan
defects below need fixing first; if the user intends to leave the
existing built notebooks alone, only fix items 1, 2, and 5 (the rest
are advisory).

---

## Section 1: AWS / S3 / Setup-File Alignment

Severity: LOW. T1, T2, T3 are well aligned with `instructor_setup.md`
and `test_data_setup.md`.

### S1.1 - T1 and T3 correctly avoid S3

`rg -n "S3_BUCKET|barclays-prompt-eng-data|boto3|us-east-2|REGION"`
on `plans/topic_01_foundations.md` and `plans/topic_03_first_chatbot.md`
returns zero matches in code blocks (only `import sagemaker` for the
execution role; see S1.5).

This is consistent with `plans/instructor_setup.md` Cell 12 lines
661-663:
- "Topic 1:  No S3 needed - ready."
- "Topic 3:  No S3 needed - ready."

VERDICT: No defect. No new S3 keys needed for T1 or T3.

### S1.2 - T2 S3 references match the setup notebooks exactly

T2 Cell 3 (lines 162-166):
```
REGION = "us-east-2"
S3_BUCKET = "barclays-prompt-eng-data"
```

`instructor_setup.md` Cell 2 lines 151-152:
```
S3_BUCKET = "barclays-prompt-eng-data"
REGION = "us-east-2"
```

`test_data_setup.md` Cell 2 lines 151-152:
```
MY_TEST_BUCKET = "barclays-pe-test-CHANGEME"
REGION = "us-east-2"
```

VERDICT: bucket name and region are exact byte matches between T2 and
the production-day populator notebook. The pre-day test bucket name is
a separate constant by design (one-line swap pattern documented in
`test_data_setup.md` Cell 0 lines 102-110).

### S1.3 - T2 PDF S3 keys match the setup notebooks exactly

T2 Cell 8 (line 370): `key="barclays_personal_loan_faq.pdf"`
T2 Cell 10 (line 425): `key="barclays_credit_card_tnc.pdf"`
T2 Cell 23 / Cell 24 safety-net: same two keys.

`instructor_setup.md` Cell 6 PDF_MANIFEST entries (lines 327-348):
- `barclays_personal_loan_faq.pdf` (matches)
- `barclays_credit_card_tnc.pdf` (matches)

`test_data_setup.md` Cell 7 ALL_PDFS dict (lines 545-550):
- `barclays_personal_loan_faq.pdf` (matches)
- `barclays_credit_card_tnc.pdf` (matches)

VERDICT: S3 keys are exact byte matches across all three documents.
`instructor_setup.md` Cell 12 readiness summary correctly states:
"Topic 2:  Needs barclays_personal_loan_faq.pdf + barclays_credit_card_tnc.pdf".

### S1.4 - Synthetic PDF content matches T2 Cell 6 noise demo and Cell 8 fallback

`test_data_setup.md` Cell 5 (lines 297-356) generates the personal loan
PDF with these specific phrases:
- "1,000 GBP and 35,000 GBP" (loan amounts)
- "1 year to 5 years" (terms)
- "6.5 percent" (representative APR)
- "58 days interest" (ERC)
- "missing a payment may affect your credit score"

T2 Cell 6 (lines 256-263) raw_pdf_noise demo string contains:
- "Page N of 12" and "PERSONAL LOAN AGREEMENT" headers
- "early repayment charge (ERC) of up to 58 days interest"

T2 Cell 8 (lines 381-388) fallback string contains the same Q&A
structure (early repayment, payment methods, missed payments) as the
synthetic PDF.

VERDICT: full alignment. The cleaning demo in T2 will produce the
same narrative whether students load from S3 or fall through to the
inline fallback. This is exactly what the setup notebooks were
designed to deliver.

### S1.5 - T1 and T3 import sagemaker but never use it

T1 Cell 3 (lines 127-135):
```python
import sagemaker
from sagemaker import get_execution_role

sess = sagemaker.Session()
role = get_execution_role()

print(f"SageMaker region: {sess.boto_region_name}")
print(f"Execution role (short): ...{role[-30:]}")
```
Comment on line 127 says "useful for S3 access later". T1 never
performs an S3 access.

T3 Cell 3 (lines 138-143):
```python
import sagemaker
from sagemaker import get_execution_role

sess = sagemaker.Session()
role = get_execution_role()
```
Comment on line 141 says "we use it for role/region; no S3 access
needed this topic". The variables `sess` and `role` are not referenced
again in T3.

CORE manifest line 77 says:
> Credentials: getpass + os.environ for API keys, sagemaker.Session()
> + get_execution_role() for AWS

The CORE clause applies "for AWS" - i.e. when a topic touches AWS.
T1 and T3 do not touch AWS. The sagemaker import adds 5 lines of
overhead and a startup delay (the SageMaker SDK boots up an HTTP client)
in topics that have no AWS dependency.

Severity: LOW (cosmetic / startup-time only).

REMEDIATION (optional): drop the sagemaker import + Session + role
lines from T1 Cell 3 and T3 Cell 3. Print only the OpenAI client
ready state. If you keep the sagemaker import for "later S3 access"
foreshadowing, change the line 127 comment in T1 from "useful for S3
access later" to "shown here so the SageMaker boilerplate is identical
across all topics" (or similar) so students do not get confused that
they are about to use S3 in T1.

### S1.6 - No new S3 / IAM / bucket changes required

The setup notebooks do not need any new uploads to support T1, T2, T3.
Specifically:
- `instructor_setup.md` already uploads the two PDFs T2 needs.
- `test_data_setup.md` already generates synthetic versions of the two
  PDFs T2 needs.
- `barclays_chunks.json` (instructor_setup.md Cell 10) is NOT consumed
  by T1, T2, or T3. It is consumed (optionally) by Topics 6-9.
- T1 and T3 require no S3 reads at all.

VERDICT: no edits to `INSTRUCTOR_SETUP.ipynb` or
`INSTRUCTOR_TEST_SETUP.ipynb` are needed for T1/T2/T3 readiness.

---

## Section 2: CORE_TECHNOLOGIES_AND_DECISIONS Conflicts

Severity: LOW for T1/T2/T3 plans themselves; the plans correctly
follow CORE. The conflict is between TOPICS.md and CORE.

### S2.1 - TOPICS.md says T1 must call Anthropic; CORE forbids Anthropic

CORE line 46:
> Model restrictions: OpenAI only - default model gpt-4o; no Anthropic
> SDK in student notebooks

CORE line 76:
> Default model: gpt-4o (OpenAI only - no Anthropic in student notebooks)

`plans/TOPICS.md` line 25 (Topic 1 concepts):
> API options: OpenAI (gpt-4o, o4-mini) and Anthropic (claude-sonnet-4-6, claude-haiku-4-5)

`plans/TOPICS.md` line 29 (Topic 1 labs):
> Call the OpenAI API and the Anthropic API; compare responses for the
> same prompt

`plans/TOPICS.md` line 39 (Topic 1 libraries): `openai, anthropic`
`plans/TOPICS.md` line 117 (Topic 3 libraries): `openai, anthropic`
`plans/TOPICS.md` line 81 (Topic 2 libraries): `pymupdf (fitz), beautifulsoup4, re, textwrap`

T1 plan: zero Anthropic references. Uses `openai==2.32.0` only.
T3 plan: zero Anthropic references. Uses `openai==2.32.0` only.

CONSEQUENCE: T1 and T3 plans correctly side with CORE and ignore
TOPICS.md. The plans are right; TOPICS.md is stale. If anyone re-runs
`/run-research-topic 1` based on TOPICS.md as the input spec they will
re-introduce Anthropic and produce a non-CORE-compliant plan.

REMEDIATION: edit TOPICS.md lines 25, 29, 35, 39, 110, 117 to remove
all Anthropic references and leave OpenAI-only. Keep the plans as they
are.

### S2.2 - TOPICS.md T2 library list says "pymupdf (fitz)"

`plans/TOPICS.md` line 81: `pymupdf (fitz), beautifulsoup4, re, textwrap`

T2 plan Cell 3 (line 148): `import pymupdf` (NOT `import fitz`).
T2 plan line 1078 RESEARCH VALIDATED: "`import pymupdf` is the
current recommended import (replaces legacy `import fitz` which still
works)".

CONSEQUENCE: TOPICS.md still references the legacy `fitz` alias even
though the plan uses the current `pymupdf` import. Cosmetic only.

REMEDIATION: edit TOPICS.md line 81 to drop the `(fitz)` parenthetical.

### S2.3 - T2 install line does NOT install `openai`

Severity: HIGH. This is the most consequential install-line defect in
the three plans.

T2 Cell 2 (line 136):
```
!pip install -q "pymupdf==1.27.2.2" "beautifulsoup4==4.14.3" lxml boto3 "numpy<2"
```

T2 Cell 6 (line 247-251):
```python
from openai import OpenAI
import os, getpass

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
client = OpenAI()
```

T2 Cell 6, 13 then call `client.chat.completions.create(...)`.

CORE manifest line 47-48:
> Pre-installed packages: Standard SageMaker Distribution image - pip
> install everything explicitly in each notebook; always pin numpy<2

The SageMaker Distribution image MAY have an `openai` package
pre-installed, but its version is unpinned and may pre-date 2.x (the
SDK had a major rewrite at 1.x and another at 2.x). T2 Cell 6 will
import whatever is installed - which on a fresh SageMaker space could
be any version. T2 then calls `client.chat.completions.create` which
does work on both 1.x and 2.x SDKs, so the call probably succeeds, but
the constraint "self-contained notebook with explicit pins" is broken.

CONSEQUENCE: a student who skips Topic 1 and starts at Topic 2 may get
a stale openai SDK (or none at all) and the naive demo cell fails.
A student who ran Topic 1 first inherits the pinned 2.32.0.

REMEDIATION: add `"openai==2.32.0"` to T2 Cell 2 install line:
```
!pip install -q "pymupdf==1.27.2.2" "beautifulsoup4==4.14.3" "openai==2.32.0" lxml boto3 "numpy<2"
```

### S2.4 - T1 install line is unquoted

Severity: LOW (style only).

T1 Cell 2 line 105:
```
!pip install -q openai==2.32.0 "numpy<2" tiktoken==0.9.0
```

T2 Cell 2 line 136 (quoted): `"pymupdf==1.27.2.2" "beautifulsoup4==4.14.3"`
T3 Cell 2 line 121 (quoted): `"openai==2.32.0" "numpy<2"`

`openai==2.32.0` and `tiktoken==0.9.0` happen to work without quotes
(no shell metacharacters in the version specifier). But the convention
across T2 and T3 is to quote every pin. T1 should match.

REMEDIATION: edit T1 Cell 2 to:
```
!pip install -q "openai==2.32.0" "numpy<2" "tiktoken==0.9.0"
```

### S2.5 - T1 has redundant openai imports

Severity: LOW.

T1 Cell 3 lines 122-123:
```python
import openai
from openai import OpenAI
```

The bare `import openai` is never referenced again in the notebook.
Only `OpenAI` (the class) is used. Remove the bare import.

REMEDIATION: drop line 122 (`import openai`). Keep line 123.

### S2.6 - getpass + os.environ pattern is correct

CORE manifest line 77:
> Credentials: getpass + os.environ for API keys

T1 Cell 3 line 139, T2 Cell 6 line 250, T3 Cell 3 line 146 all use
`os.environ["OPENAI_API_KEY"] = getpass.getpass(...)`. Correct.

VERDICT: no defect.

---

## Section 3: Cross-Topic Continuity Rot

Severity: HIGH for items S3.1 and S3.2; MEDIUM for S3.3 and S3.4;
LOW for S3.5 and S3.6.

### S3.1 - PRODUCT_SNIPPET defined twice with divergent content

Severity: HIGH.

T1 Cell 11 (lines 357-364) defines PRODUCT_SNIPPET as a loan-only
block:
```
Barclays Personal Loan - Product Summary (illustrative example)
- Loan amounts: 1,000 GBP to 35,000 GBP
- Representative APR: 6.5% for loans of 7,500 GBP to 15,000 GBP
- Repayment terms: 1 to 5 years
- No arrangement fee
- Early repayment allowed with 30-day interest charge
```

T3 Cell 3 (lines 154-164) defines PRODUCT_SNIPPET as a loan +
credit card combined block:
```
Barclays Personal Loan - Product Summary (illustrative)
- Loan amounts: 1,000 GBP to 35,000 GBP
- Representative APR: 6.5% for loans of 7,500 GBP to 15,000 GBP
- Repayment terms: 1 to 5 years
- No arrangement fee
- Early repayment allowed with 30-day interest charge
- Barclaycard Rewards credit card: 27.9% APR variable purchase rate
- Minimum monthly repayment: greater of 1% of balance plus interest, or GBP 25
- Late payment fee: GBP 12 if minimum payment not received by due date
```

CONSEQUENCES:
1. If a student runs Topic 1 then Topic 3 in the same kernel, the T3
   Cell 3 redefinition silently overrides T1's narrower version. The
   Lab 1 / Lab 2 cells in T1 (which still live in scope) would now use
   T3's expanded snippet if re-run.
2. T3 Cell 13 ("Naive demo - duplicated setup") asks a credit card
   question ("What is the late payment fee on the credit card?") that
   only works because of T3's expanded snippet. A student who imports
   PRODUCT_SNIPPET from a T1-style cache (or who has T1's narrower
   string in scope) will see the model say "I don't have that
   information to hand" rather than the expected answer. The T3 demo
   loses its punch.
3. T3 line 8 lists PRODUCT_SNIPPET as a "from Topic 1" carryover
   variable. The list is misleading because T3 Cell 3 redefines it.

REMEDIATION:
- OPTION A (recommended): in T3 Cell 3, rename the expanded variable
  from `PRODUCT_SNIPPET` to `EXTENDED_PRODUCT_SNIPPET` (or
  `BARCLAYS_PRODUCT_SNIPPET`). Use the renamed variable in T3 Cells
  6, 8, 13, 15, 17, 18, 22. Drop the line 8 carryover claim for
  PRODUCT_SNIPPET. T3 then becomes truly self-contained.
- OPTION B: keep the name but make T3 Cell 3 explicit about the
  expansion ("expanded version of T1's PRODUCT_SNIPPET; covers loan
  AND credit card so the credit card demo in Cell 13 has data").
  Document the divergence in both T1 and T3 wrap-up.

### S3.2 - T3 stretch references undefined GPT4O constants

Severity: HIGH for the stretch path; LOW for the main path.

T3 Cell 25 wrap-up (lines 1047-1049):
```
2. Add cost tracking to `create_chatbot`: after each call, print the
   token count and estimated cost using the `GPT4O_INPUT_PRICE_PER_1K`
   and `GPT4O_OUTPUT_PRICE_PER_1K` constants from Topic 1.
```

T1 Cell 18 lines 595-596 defines:
```
GPT4O_INPUT_PRICE_PER_1K  = 0.0025
GPT4O_OUTPUT_PRICE_PER_1K = 0.0100
```

T3 Cell 3 (the setup cell) does NOT redefine these constants. T3 Cell
2 install line does not import them either (you cannot import a
constant from a notebook). A student who:
- Started a fresh JupyterLab kernel for T3, OR
- Restarted the kernel between T1 and T3, OR
- Skipped T1 entirely
... gets `NameError: name 'GPT4O_INPUT_PRICE_PER_1K' is not defined`
when attempting the stretch.

REMEDIATION: in T3 Cell 3, add the two constants right after the
PRODUCT_SNIPPET block:
```python
GPT4O_INPUT_PRICE_PER_1K  = 0.0025
GPT4O_OUTPUT_PRICE_PER_1K = 0.0100
```
This makes T3 self-contained for the stretch lab and matches the
CORE line 35-37 self-contained-notebook principle.

### S3.3 - T3 stretch references undefined ask_with_cost_tracking helper

Severity: LOW (stretch lab only).

T3 line 8 lists `ask_with_cost_tracking()` as a "from Topic 1"
carryover. T3 Cell 25 does not directly call it, but stretch exercise
2 (Cell 25 lines 1047-1049) implies students will reuse the cost
tracking pattern. The function `ask_with_cost_tracking` is defined in
T1 Cell 18 lines 598-636 and lives only in T1's notebook scope.

CONSEQUENCE: same as S3.2 - a student running T3 in a fresh kernel
gets `NameError: name 'ask_with_cost_tracking' is not defined`.

REMEDIATION: either drop the carryover claim from T3 line 8, or have
T3 Cell 3 redefine the helper inline (a shorter version is fine since
T3 is not focused on cost tracking).

### S3.4 - T3 chunks carryover misrepresents Topic 2 output

Severity: MEDIUM.

T3 line 10 says:
> From Topic 2: `chunks` list, `clean_pdf_text()`, `chunk_text()`,
> `preprocess_document()` (optional - Topic 3 degrades gracefully when
> Topic 2 artifacts are absent)

T3 Cell 22 (lines 875-881):
```python
available_chunks = globals().get("chunks")
if available_chunks and len(available_chunks) > 0:
    context = "\n\n".join(available_chunks[:8])
```

T2 Cell 21 (line 836) defines `chunks` as the output of
`chunk_text(cleaned_text, chunk_size=1500, overlap=200)` where
`cleaned_text` is `clean_pdf_text(raw_text)` and `raw_text` is the
personal loan FAQ PDF.

CONSEQUENCE: T2's `chunks` is 100% personal loan FAQ text from a
single PDF. The "richer context" claim in T3 Cell 21 line 837-838
("students who completed Topic 2 get richer context automatically")
is misleading - the student gets MORE personal loan text but ZERO
credit card text or other product coverage. If T3 tests the credit
card late-payment-fee question with T2's chunks, the chatbot will
fail because the credit card information is not present.

Compare to T6 Cell 5 lines 212-220 which defines `BARCLAYS_DOCS` as
7 different product strings (loan, card, savings, mortgage, student,
business, travel). T6 then assigns `chunks = BARCLAYS_DOCS`.

If a student runs T2 -> T3 -> T6 in the same kernel:
1. T2 sets `chunks` to ~10-20 personal loan FAQ chunks.
2. T3 reads `chunks` from T2 (8-doc personal loan context only).
3. T6 Cell 5 redefines `chunks = BARCLAYS_DOCS` (7-product context).

Step 3 silently overrides T2's chunks, which is what T6 wants - but
the divergence is undocumented and a student inspecting the namespace
mid-sequence will be confused.

REMEDIATION:
- In T3, change Cell 21 line 837-838 to be honest: "if you ran Topic 2
  you get a deeper view of the personal loan FAQ; the credit card
  questions in Lab 2 may fall through to the system prompt's general
  knowledge".
- OR: have T3 Cell 22 only use `chunks` from T2 IF the chunks contain
  recognisable credit card text (substring check for "credit card").
  Otherwise stay on PRODUCT_SNIPPET.
- OR: align T2 to also chunk the credit card PDF and concatenate, so
  T2's `chunks` has both products. Currently T2 Cell 21 only chunks
  `cleaned_text` (the loan FAQ). T2 Cell 23 Lab 3 produces
  `chunks_512`, `chunks_1024`, `chunks_1500` from the credit card PDF
  but NEVER recombines them with the loan chunks into a unified
  `chunks` list.

### S3.5 - No MODEL constant carries from T3 to T4 / T5

Severity: MEDIUM.

T5 line 8: "MODEL = "gpt-4o" or equivalent constant" listed as a
carryover from Topic 4.
T4 plan: not audited in full, but a quick read of the context section
shows T4 uses model="gpt-4o" inline as string literals (no MODEL
constant defined).
T3 plan: uses `model="gpt-4o"` inline as a string literal in Cells
6, 8, 11, 15 (line 374, 389, 526, 593, 614, 633, 651). No MODEL
constant defined.
T1 plan: same - inline string literals only.

CONSEQUENCE: T5 expects a `MODEL` constant from prior topics. None of
T1/T2/T3 defines it. T5 line 24 says "client, OPENAI_API_KEY, MODEL
carry over from Topic 4". T5 then defines MODEL inline if absent (T5
gracefully handles the gap).

REMEDIATION: optionally add `MODEL = "gpt-4o"` constant to T1 Cell 3,
T3 Cell 3, and T4 setup cell. Replace inline `model="gpt-4o"` calls
with `model=MODEL`. This makes the carryover claim in T5 line 24
true and lets a student do a one-line model swap (e.g. to gpt-4o-mini
for cost) without grep-and-replace.

### S3.6 - T2 Cell 3 misleading "no OpenAI key needed" comment

Severity: MEDIUM.

T2 Cell 3 line 168-169:
```python
# The SageMaker execution role already has S3 read permissions for this bucket.
# We do NOT need the OpenAI key for this topic - this is pure document processing.
```

T2 Cell 6 line 247-251:
```python
from openai import OpenAI
import os, getpass

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
client = OpenAI()
```

T2 Cell 6 calls `client.chat.completions.create`. T2 Cell 13 also
calls it. The Cell 3 comment is wrong: T2 absolutely needs the OpenAI
key (for the naive demos in Cells 6 and 13).

The verification checklist (lines 1049-1050) confirms this is
intentional design ("OpenAI client and getpass inside the naive demo
cell - first time student needs it") - the rationale being that T2
is "pure document processing" with the OpenAI calls only used as
naive contrast demos. But the Cell 3 comment is still factually wrong.

CONSEQUENCE: a student reads Cell 3, sees "no OpenAI key needed",
runs Cell 3 successfully, then is interrupted by a getpass prompt at
Cell 6. The mid-notebook getpass interruption is jarring and
contradicts the Cell 3 promise.

REMEDIATION: rewrite the Cell 3 comment to:
```python
# The SageMaker execution role already has S3 read permissions for this bucket.
# We will be prompted for the OpenAI key in Cell 6 when the first naive
# demo runs. The bulk of this topic is document processing (no LLM calls).
```

### S3.7 - T2 Cell 6 re-prompts for OpenAI key even when already set

Severity: LOW.

T2 Cell 6 (line 250):
```python
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
```

Unconditional `getpass`. If the student ran T1 first (in the same
kernel), `os.environ["OPENAI_API_KEY"]` is already set and `client`
already exists, but T2 Cell 6 re-prompts.

REMEDIATION: gate the prompt:
```python
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
client = OpenAI()
```

T1 Cell 3 line 139 has the same unconditional pattern but T1 is
always the first topic of the day so re-prompting is fine.

---

## Section 4: Install-Line Inconsistencies

Severity: LOW-MEDIUM. Cumulative cosmetic noise that affects the
"is this self-contained?" check.

### S4.1 - openai pin variance across T1, T2, T3

T1 Cell 2 (line 105): `openai==2.32.0` (unquoted)
T2 Cell 2 (line 136): NOT INSTALLED (see S2.3)
T3 Cell 2 (line 121): `"openai==2.32.0"` (quoted)

`instructor_setup.md` Cell 2 (line 133): does not install openai
`test_data_setup.md` Cell 2 (line 135): does not install openai

REMEDIATION: align T1, T2, T3 on `"openai==2.32.0"` quoted, all three.
Setup notebooks correctly omit openai (they do not call it).

### S4.2 - boto3 pin variance

T2 Cell 2 (line 136): `boto3` (unpinned)
`instructor_setup.md` Cell 2 (line 133): does not install boto3 (relies
on SageMaker pre-install)
`test_data_setup.md` Cell 2 (line 135): `boto3` (unpinned)

REMEDIATION: leave as-is. boto3 is pre-installed on SageMaker
Distribution image and is backwards-compatible at the get_object /
put_object surface used here. Pinning it would invite breakage when
the SageMaker image rolls forward.

### S4.3 - tiktoken pinning

T1 Cell 2 (line 105): `tiktoken==0.9.0` (unquoted)
T5 line 21 plan: `tiktoken==0.9.0` (used in T5 Concept 2)
T2, T3: tiktoken not installed (not used).

REMEDIATION: align T1 to quoted form `"tiktoken==0.9.0"` per S2.4.

### S4.4 - lxml unpinned

T2 Cell 2 (line 136): `lxml` (unpinned)
T2 plan does not import lxml directly. It is the parser backend used
by BeautifulSoup4. SageMaker Distribution image has it pre-installed.

REMEDIATION: leave as-is OR remove from the install line (BS4 falls
back to the stdlib `html.parser` if lxml is missing - slower but
works).

### S4.5 - numpy<2 pinning is consistent

T1 Cell 2: `"numpy<2"`
T2 Cell 2: `"numpy<2"`
T3 Cell 2: `"numpy<2"`
`instructor_setup.md` Cell 2: `"numpy<2"`
`test_data_setup.md` Cell 2: indirectly via the pymupdf install line
(no explicit numpy pin). Note: pymupdf 1.27.2.2 does not declare a
strict numpy<2 dependency, so a fresh install of the test setup may
pull numpy 2.x. Worth a manual check.

REMEDIATION: add `"numpy<2"` to `test_data_setup.md` Cell 2 install
line for parity:
```
!pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3 "numpy<2"
```

---

## Section 5: Synthetic-Data vs Inline-Data Alignment

Severity: LOW. The course was designed with this divergence in mind
and `instructor_setup.md` Cell 11 documents it explicitly.

### S5.1 - T1 PRODUCT_SNIPPET numbers match synthetic PDFs

T1 PRODUCT_SNIPPET key numbers (T1 line 357-364):
- "1,000 GBP to 35,000 GBP" loan range
- "6.5%" representative APR
- "1 to 5 years" repayment terms
- "No arrangement fee"

`test_data_setup.md` Cell 5 lines 297-300 (synthetic personal loan
PDF) generates EXACTLY:
- "1,000 GBP and 35,000 GBP"
- "6.5 percent"
- "1 year to 5 years"
- "no arrangement fee"

VERDICT: byte-level match. T1's hardcoded snippet is consistent with
what `test_data_setup.md` puts in the test bucket and (per
`instructor_setup.md` Cell 11 mismatch reference card) is the
"illustrative" version for teaching.

### S5.2 - T3 expanded PRODUCT_SNIPPET adds 27.9% APR which matches synthetic PDFs but NOT T6 BARCLAYS_DOCS[1]

T3 PRODUCT_SNIPPET (line 161): "Barclaycard Rewards credit card:
27.9% APR variable purchase rate"

`test_data_setup.md` Cell 6 line 393: "27.9 percent APR variable"
T2 Cell 6 line 393 (raw_pdf_noise): "27.9% p.a. variable"
T6 Cell 5 BARCLAYS_DOCS[1] (line 214): does NOT mention 27.9%

CONSEQUENCE: T3's expanded snippet aligns with T2 demos and synthetic
PDFs but DIVERGES from T6's product summary list. A student running
T3 then T6 sees 27.9% APR in T3 then no APR in T6's vector store.
Not a defect (the inline strings are illustrative-only and the
mismatch-reference card in `instructor_setup.md` Cell 11 covers this)
but worth flagging.

REMEDIATION: optional. Either
- Add `"Barclaycard Rewards: 27.9% APR variable purchase rate"` to
  T6 BARCLAYS_DOCS[1] for symmetry with T3, OR
- Document in T6 Cell 5 that the BARCLAYS_DOCS strings are
  intentionally narrower than T3's PRODUCT_SNIPPET (they teach
  retrieval pattern, not product completeness).

### S5.3 - T2 Cell 8 fallback is shorter than the synthetic PDF

T2 Cell 8 fallback (lines 381-388) is a single Q&A block:
- ERC Q&A
- Payment method Q&A
- Missed payment Q&A

`test_data_setup.md` Cell 5 (lines 290-356) generates a 3-page PDF
with 7+ Q&A blocks plus a representative example and regulatory text.

CONSEQUENCE: a student who hits the S3 fallback path sees less rich
text than a student who loads from S3. The fallback chunks list
would be ~2-3 chunks instead of the ~10-15 from the real synthetic
PDF. T3 / T6 downstream behavior is unchanged because both have
their own fallbacks, but the Cell 21 chunking demo has very few
chunks to display in fallback mode.

REMEDIATION: optional. Expand T2 Cell 8 fallback to mirror the
synthetic PDF more fully, OR document this in T2 Cell 8 (line 380)
as "minimal fallback - load from S3 for the full demo".

### S5.4 - No topic loads from a URL (only S3 + inline)

Cross-check: do any of T1, T2, T3 load data from a URL or from local
disk?

`rg -n "requests.get|urllib|huggingface|datasets.load_dataset|open\("`
on the three plan files returns:
- T1: zero matches.
- T2: zero matches in code blocks. The fallback (Cell 8) and inline
  noise demos (Cell 6, 13) are inline string literals.
- T3: zero matches in code blocks.

VERDICT: T1, T2, T3 source data exclusively from
- S3 (`barclays-prompt-eng-data`, T2 only), or
- inline string literals (T1, T2 fallback paths, T3).

This is consistent with `instructor_setup.md` Cell 12 readiness
summary and CORE manifest expectations. No HuggingFace / no public
URL fetches. The Banking77 HuggingFace dataset (CORE line 84) only
appears in T9.

VERDICT: no defect.

---

## Section 6: Things That Are CORRECT (Leave Alone)

- C1: T2 S3 bucket `barclays-prompt-eng-data` and region `us-east-2`
  match `instructor_setup.md` Cell 2 exactly.
- C2: T2 PDF S3 keys (`barclays_personal_loan_faq.pdf`,
  `barclays_credit_card_tnc.pdf`) match `instructor_setup.md` Cell 6
  PDF_MANIFEST and `test_data_setup.md` Cell 7 ALL_PDFS.
- C3: T1 and T3 correctly avoid S3 entirely, matching
  `instructor_setup.md` Cell 12 readiness summary lines 661-663.
- C4: T2 Cell 8 has a try/except S3 fallback so the notebook still
  runs if S3 is unreachable. Matches CORE line 35-37.
- C5: T1, T2, T3 all use `gpt-4o`; no Anthropic SDK imported anywhere.
  Matches CORE line 76.
- C6: All three plans pin `numpy<2` (CORE line 78). All three use
  `getpass + os.environ` for the OpenAI key (CORE line 77).
- C7: T3 BARCLAYS_SYSTEM_PROMPT name matches the T5 line 15 expectation.
  T5 also defines its own inline copy as a fallback per CORE line 35-37.
- C8: T3 closure pattern (`create_chatbot` returns `chat`) and
  `globals().get("chunks")` safe-existence-check pattern are both
  consistent with downstream T5 and T6 expectations.
- C9: T2 chunking pipeline (chunk_size=1500, overlap=200, sentence-
  boundary heuristic) is byte-identical to `instructor_setup.md` Cell 9
  and `test_data_setup.md` Cell 12. Pre-built `barclays_chunks.json`
  has the same chunk shape as student-generated `chunks`.
- C10: All three plans pass the AI-tells scan: no em dashes, en dashes,
  Unicode multiplication, smart quotes, or emoji outside markdown
  headers.
- C11: T2 Cell 6 raw_pdf_noise and Cell 8 fallback align with the
  synthetic PDF content from `test_data_setup.md` Cell 5 (same "58 days
  interest" ERC phrasing, same Q&A structure, same loan range numbers).
- C12: T2 line 1078 correctly notes `import pymupdf` is the current
  preferred import; Cell 3 uses `import pymupdf` not `import fitz`.
- C13: T3 Tier 3 lab (Cell 24) correctly omits `None  # YOUR CODE`
  placeholders and verification code per the Tier 3 contract.
- C14: Day 1 lab tier distribution is correct (T1: Tier 1, T2: Tier 1
  + Tier 2, T3: Tier 1 + Tier 3).
- C15: Safety-net cells are present for every lab whose output feeds
  downstream: T1 Cell 14, T2 Cell 11 + Cell 24, T3 Cell 11 + Cell 18.
  T3 Cell 24 (Tier 3) has no safety-net by design.
- C16: No new files on disk are created by T1, T2, T3 other than
  in-memory PDFs and strings. No EBS state, no new S3 keys, no DynamoDB,
  no RDS.
- C17: T2 Cell 3 line 168 correctly states "The SageMaker execution
  role already has S3 read permissions for this bucket" - matches
  `instructor_setup.md` line 18.

---

## Section 7: Recommended Remediation Order

If you can only do a subset, do them in this order. Items 1-3 unblock
`/build-topic-notebook` for T1/T2/T3 if a re-build is requested. Items
4-9 are quality-of-life. Item 10 is a TOPICS.md hygiene fix.

1. FIX S2.3 (T2 missing openai install). One-line edit to T2 Cell 2:
   add `"openai==2.32.0"` to the install line. This is the only HIGH-
   severity install bug.

2. FIX S3.1 (PRODUCT_SNIPPET name collision). Recommended OPTION A:
   rename T3's expanded variable to `BARCLAYS_PRODUCT_SNIPPET` (or
   similar). Update T3 Cells 6, 8, 13, 15, 17, 18, 22 to use the new
   name. Drop the line 8 carryover claim. T3 then becomes truly
   self-contained.

3. FIX S3.6 (T2 Cell 3 misleading comment). Rewrite the "We do NOT
   need the OpenAI key" comment to acknowledge the Cell 6 prompt.

4. FIX S3.2 (T3 stretch references undefined GPT4O constants). Add
   the two constants to T3 Cell 3 so the stretch lab works in
   isolation.

5. FIX S3.3 (T3 stretch references undefined ask_with_cost_tracking).
   Either drop the carryover claim from T3 line 8 or redefine a small
   cost wrapper inline in T3 Cell 3.

6. FIX S3.4 (T3 chunks carryover claim). Rephrase the "richer context"
   claim to be honest about T2's single-document chunks list. Or
   improve T2 to chunk both PDFs and concatenate.

7. FIX S3.7 (T2 Cell 6 unconditional getpass). Gate with `if
   "OPENAI_API_KEY" not in os.environ:`.

8. FIX S2.4 + S2.5 (T1 install line quoting + redundant openai
   import). Quote all three pins; drop the bare `import openai` line.

9. FIX S4.5 (test_data_setup.md missing numpy<2). Add `"numpy<2"` to
   `test_data_setup.md` Cell 2 install line.

10. FIX S2.1 + S2.2 (TOPICS.md Anthropic / fitz references). Edit
    `plans/TOPICS.md` to remove Anthropic mentions from T1/T3
    concepts/labs/libraries and drop the `(fitz)` parenthetical from
    the T2 library list.

11. OPTIONAL S1.5 (T1 + T3 unused sagemaker imports). Drop the
    sagemaker boilerplate from T1 Cell 3 and T3 Cell 3, OR keep it
    and reword the "for S3 access later" comment.

12. OPTIONAL S3.5 (no MODEL constant). Add `MODEL = "gpt-4o"` to
    T1, T3, T4 setup cells and replace inline string literals.

After applying items 1-3, T1, T2, T3 are ready for a clean
`/build-topic-notebook` re-build. Items 4-7 should also be applied if
a re-build happens; items 8-12 are advisory.

---

## Appendix: Files Cross-Referenced

- plans/instructor_setup.md (the on-day Barclays-bucket S3 populator)
- plans/test_data_setup.md (the pre-day personal-bucket synthetic PDF generator)
- plans/CORE_TECHNOLOGIES_AND_DECISIONS.md (locked course manifest)
- plans/TOPICS.md (per-topic manifest)
- plans/topic_01_foundations.md (audited)
- plans/topic_02_nlp_preprocessing.md (audited)
- plans/topic_03_first_chatbot.md (audited)
- plans/topic_04_prompt_engineering.md (downstream consumer of T3 client / messages pattern)
- plans/topic_05_conversation_memory.md (downstream consumer of T3 BARCLAYS_SYSTEM_PROMPT and a missing MODEL constant)
- plans/topic_06_rag_foundations.md (downstream consumer of T2 chunks variable)
- plans/aws_setup/discrepancies_7_8_9.md (format reference for this audit)

No web sources consulted (per user instruction, /research protocol with
3 cycles, no web search).

# Change and Fix Audit Log
# Barclays - Generative AI: Prompt Engineering for Software Developers
#
# Every fix applied by /fixes is recorded here permanently.
# TODOS.md tracks open issues; AUDIT.md tracks resolved ones.

---

## 2026-04-27 - SETUP-FIX-003: plans/TOPICS.md updated with setup notebook section

**Files**: plans/TOPICS.md

**Problem**: TEST-SETUP-FIX-002 and other setup notebook fixes were not reflected in TOPICS.md.
The Summary table also showed Topics 07 and 08 as "planned" instead of "done".

**Fix**: Added "Setup Notebooks" section to TOPICS.md documenting INSTRUCTOR_TEST_SETUP.ipynb
and INSTRUCTOR_SETUP.ipynb with their fix histories. Corrected Summary table for Topics 07/08.

**Validation**: N/A - documentation only.

---

## 2026-04-27 - SETUP-FIX-004: INSTRUCTOR_SETUP.ipynb PDF manifest expanded to 6 Barclays docs

**File**: setup/INSTRUCTOR_SETUP.ipynb (cells c5628693, ffa56033, 6281076a, 330104f6)

**Problem**: PDF manifest had 4 entries; research confirmed 2 additional public Barclays PDFs.
Also, Barclays CDN returns 403 to plain requests - download function lacked browser headers.

**Fix**:
- cell c5628693: expanded PDF_MANIFEST to 6 entries (added barclays_isa_instant_terms.pdf
  and barclays_isa_guide.pdf). Added "required" boolean to each entry. Added BROWSER_HEADERS
  dict. Updated loop to handle required vs optional failures separately.
- cell ffa56033: download_url_and_upload_s3 now passes BROWSER_HEADERS to requests.get().
- cell 6281076a: markdown updated to describe all 6 documents.
- cell 330104f6: summary cell updated to split required vs optional in manifest check.

**Validation**: N/A - setup notebook (runs in SageMaker on course day, not locally).

---

## 2026-04-27 - TEST-SETUP-FIX-002: INSTRUCTOR_TEST_SETUP.ipynb FPDFException in BankingDocPDF

**Topic**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**File**: setup/INSTRUCTOR_TEST_SETUP.ipynb (cell 829e9f16)

**Problem**: Running make_personal_loan_faq() raises:
FPDFException: Not enough horizontal space to render a single character
in BankingDocPDF.qa_block -> multi_cell(0, 5, answer, align="J").

**Root cause**: multi_cell with w=0 computes effective width as
  w = self.w - self.r_margin - self.x
When self.x is not at self.l_margin (can happen if header() cursor state is
non-standard), this produces zero or negative width. With align="J" (justified),
fpdf2's line-breaking engine enters a loop that raises the exception when it cannot
place even one character. align="L" is more tolerant and does not raise.

**Fix**: Changed multi_cell(0, ...) to multi_cell(self.epw, ...) in both body_text()
and qa_block() in cell 829e9f16. self.epw = self.w - self.l_margin - self.r_margin
is always the correct full text width and is independent of cursor x position.

**Validation**: All 4 synthetic PDFs (personal_loan_faq, credit_card_tnc, savings_rates,
isa_terms) generated successfully in local test with fpdf2.

---

## 2026-04-27 - T02-FIX-001: T02 Cell ca6eb189 Mermaid diagram fitz.open -> pymupdf.open

**Topic**: topic_02_nlp_preprocessing
**Exercise notebook**: exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb
**Solution notebook**: solutions/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb

**Problem**: Cell ca6eb189 Mermaid diagram node D used the label "fitz.open\nPyMuPDF opens PDF"
while Cell 8 (load_pdf_from_s3 demo) correctly uses pymupdf.open(). Students would see the
diagram contradict the code, causing confusion about which API to use.

**Root cause**: The diagram was authored when fitz was still the standard import alias for
PyMuPDF. The notebooks were later updated to use `import pymupdf` (the new canonical import)
but the diagram label was not updated at the same time.

**Fix**: Changed node D label from `fitz.open\nPyMuPDF opens PDF` to `pymupdf.open\nPyMuPDF opens PDF`
in cell ca6eb189 of both exercise and solution notebooks.

**Validation**: exercise passed / solution passed / pair check passed (26/26 cells)

---

## 2026-04-27 - T02-FIX-002: Solution Cell 4e17ec14 pip install missing openai==2.32.0

**Topic**: topic_02_nlp_preprocessing
**Solution notebook**: solutions/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb

**Problem**: Exercise Cell 4e17ec14 included "openai==2.32.0" in the pip install line.
Solution Cell 4e17ec14 omitted it. Running only the solution notebook would fail when
the naive LLM demo in Cell 6 calls `from openai import OpenAI` without openai installed.

**Root cause**: The openai dependency was added to the exercise install cell (T2-FIX-001,
2026-04-25) but the same change was not propagated to the solution notebook's identical cell.

**Fix**: Added `"openai==2.32.0"` to the pip install line in cell 4e17ec14 of the solution
notebook. Both notebooks now have the identical install line:
`!pip install -q "pymupdf==1.27.2.2" "beautifulsoup4==4.14.3" lxml boto3 "openai==2.32.0" "numpy<2"`

**Validation**: solution passed / pair check passed (26/26 cells)

---

## 2026-04-25 - topic_01 plan file bare separators and research URL formatting

**Topic**: topic_01_foundations
**File**: plans/topic_01_foundations.md (plan file only - no notebooks exist yet)

**Problem**:
1. 25 bare --- separator lines between cell sections violated the no-bare-separator rule
2. RESEARCH VALIDATED section used a plain-text table with no https:// links, causing
   the verify-research URL checker to report 0 sources (minimum 3 required)

**Root cause**: The research agent that produced the plan used --- as visual dividers between
cell sections and formatted the sources as a markdown table rather than a numbered https:// list.
The verify-research command was not yet in place when the plan was generated, so the agent had
no feedback loop to catch these issues.

**Fix**:
1. Ran python3 script to strip all bare --- lines from the plan file, skipping lines inside
   code fences to avoid breaking code block content. Result: 0 bare --- remaining.
2. Replaced the RESEARCH VALIDATED table with a numbered list of 7 https:// URLs with
   specific facts extracted per entry.

**Validation**: AI-tells scan CLEAN, https:// URL count 7 (PASS). No notebooks to validate.

---

## 2026-04-25 - topic_01 solution safety-net cell deleted (decision recorded, do not repeat)

**Topic**: topic_01_foundations
**Files**: solutions/topic_01_foundations/topic_01_foundations.ipynb

**What happened**:
The safety-net cell (id: 4c723619) was deleted from the solution notebook when building
the solution. The reasoning was that the solution lab cell above it fully implements
lab_answer, so the safety-net would never fire and is "dead code".

**Decision by Axel (2026-04-25)**:
Safety-net cells should NOT be deleted from solution notebooks. Even if they never fire,
keeping them maintains cell parity between exercise and solution notebooks. The pair
validator compares by cell position - a 1-cell removal cascades into type mismatches for
all downstream cells. Prefer to keep the safety-net and let it be harmless dead code
rather than break structural parity.

**Rule going forward**:
When building a solution notebook, ONLY replace the `None  # YOUR CODE` lines in lab
cells with complete implementations. Do NOT delete safety-net cells. Do NOT delete any
cell without asking Axel first.

**Impact of this instance**: Topic 1 solution has 21 cells vs exercise 22 cells. The
pair check shows position-shifted type mismatches. Functionality is unaffected - both
notebooks are independently valid. The parity issue is cosmetic but should be corrected
in a future /fixes pass if desired.

---

## 2026-04-25 - Setup notebooks built + cross-topic wiring audit

**Scope**: INSTRUCTOR_SETUP.ipynb, INSTRUCTOR_TEST_SETUP.ipynb, all 4 existing exercise
notebooks, all 9 topic plans.

**Files created**:
- `INSTRUCTOR_SETUP.ipynb` (13 cells, from plans/instructor_setup.md)
- `INSTRUCTOR_TEST_SETUP.ipynb` (16 cells, from plans/test_data_setup.md)

**Audit method**: Extracted S3 keys, library version pins, openai import patterns, and
variable names from each exercise notebook and each plan file. Cross-referenced all against
the setup notebooks' S3 manifests and wiring expectations.

**Discrepancies found**:

### CRITICAL-1: Topic 2 uses OpenAI in Cell 6 but does not install it

**File**: `exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb`
**Cell**: Cell 2 (pip install) and Cell 6 (naive approach demo)
**Problem**: Cell 2 installs `pymupdf`, `beautifulsoup4`, `lxml`, `boto3`, `numpy<2`
but NOT `openai`. Cell 6 then does `from openai import OpenAI` and makes a live
`client.chat.completions.create()` API call to demonstrate the naive noisy-text approach.
**Impact**: Students who start Topic 2 without having run Topic 1 first will hit
`ModuleNotFoundError: No module named 'openai'` in Cell 6. This is a silent dependency.
**Fix needed**: Add `"openai==2.32.0"` to Topic 2 Cell 2's pip install line, matching
the version used in Topic 1. Alternatively, add a comment to Cell 2 explicitly stating
"openai is already installed if you ran Topic 1 - if not, uncomment the next line".
**Priority**: HIGH - affects any student or instructor who tests Topic 2 in isolation.

### CRITICAL-2: openai version inconsistency across exercises

**Files**: All exercise notebooks, all topic plans
**Problem**: Four different version specs appear across the course:
- Topic 1 (exercise Cell 2): `openai==2.32.0`
- Topic 2 (exercise): openai NOT installed (see CRITICAL-1)
- Topic 3 (exercise Cell 2): `openai==2.32.0`
- Topic 4 (exercise Cell 2): `openai==1.51.0` (deliberate: "first version with stable json_schema strict mode")
- Plans 5, 6: `openai>=1.30.0`
- Plans 7, 9: `openai>=2.30.0`
- Plan 8: `openai==2.32.0` AND `openai>=1.30.0` (double specification)

If Topics 1/3 install `openai==2.32.0` and Topic 4 then tries to `pip install openai==1.51.0`,
SageMaker will downgrade openai mid-session which can cause import cache issues.
**Fix needed**: Decide on ONE canonical openai version for the entire course and apply it to
all pip install cells and all plan files. Recommendation: use the version in Topics 1/3
(`openai==2.32.0`) since it is the most recent pin and Topics 1/3 are already built. Update
Topic 4 Cell 2 and all plan files (5-9) to match.

### MEDIUM-1: Topic 3 Cell 2 comment references Topic 2 incorrectly

**File**: `exercises/topic_03_first_chatbot/topic_03_first_chatbot.ipynb`
**Cell**: Cell 2
**Problem**: Comment reads: `# openai: the OpenAI Python SDK - pinned to the same version as Topics 1 and 2`
Topic 2 does NOT install openai in its pip install cell (see CRITICAL-1). The comment
is misleading - it suggests a 3-way consistency that does not exist.
**Fix needed**: Change comment to: `# openai: pinned to the same version as Topic 1`.

### MEDIUM-2: Plans 7 and 9 use openai>=2.30.0 while Plans 5 and 6 use openai>=1.30.0

**Files**: plans/topic_07_advanced_rag_web_search.md, plans/topic_09_capstone.md
**Problem**: Plans 5 and 6 specify `openai>=1.30.0` while Plans 7 and 9 specify
`openai>=2.30.0`. These are incompatible lower bounds: if openai 2.x is the correct
series, Plans 5/6 accept 1.x which may lack required 2.x API features. If openai 1.x
is correct, Plans 7/9 will fail to resolve unless 2.x exists on PyPI.
**Fix needed**: Standardize all plans to use the same pinned version once CRITICAL-2 is
resolved (see above). Update plans 5-9 in a single pass with the canonical version.

### MEDIUM-3: Plan 8 has two conflicting openai version specs

**File**: plans/topic_08_ethical_guardrails.md
**Problem**: Both `openai==2.32.0` and `openai>=1.30.0` appear in the plan at different
points. When the notebook is built from this plan, the builder will produce one pip install
cell. Which spec wins depends on which section of the plan is read.
**Fix needed**: Remove the `openai>=1.30.0` reference and keep only `openai==2.32.0`
(or whichever canonical version is agreed in CRITICAL-2).

### LOW-1: Topic 2 has no test-mode comment for S3_BUCKET constant

**File**: `exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb`
**Cell**: Cell 3
**Problem**: `S3_BUCKET = "barclays-prompt-eng-data"` has no comment directing instructors
to INSTRUCTOR_TEST_SETUP.ipynb for how to change this for pre-course testing.
The test setup workflow (change this one line to your personal bucket name) is documented
in INSTRUCTOR_TEST_SETUP.ipynb Cell 15 but not in the notebook that instructors will
actually need to edit.
**Fix needed**: Add a comment above the S3_BUCKET constant:
`# For pre-course testing: change this to your personal bucket - see INSTRUCTOR_TEST_SETUP.ipynb`

### LOW-2: INSTRUCTOR_SETUP.ipynb chunking default (chunk_size=1500) differs from Topic 2 Lab 2 (chunk_size=512)

**Files**: INSTRUCTOR_SETUP.ipynb Cell 9, exercises/topic_02_nlp_preprocessing Cell 20
**Problem**: The setup notebook creates barclays_chunks.json using `chunk_size=1500,
overlap=200`. Topic 2 Lab 2 asks students to compare three sizes (512, 1024, 1500) and
the pedagogically meaningful smaller chunks (512) are the focus of the cleaning demo.
Students loading barclays_chunks.json in Topics 6-9 Tier 3 labs will get 1500-char chunks,
which is one valid choice but not the only one the course explores.
**Impact**: Low - barclays_chunks.json is optional and only used in Tier 3 stretch labs.
The inline BARCLAYS_DOCS fallback (7 short strings) is the default for Topics 6-9.
**No fix required**: The 1500-char default for barclays_chunks.json is intentional
(best for RAG retrieval). Document here for instructor awareness.

### LOW-3: S3 keys confirmed matching between setup and exercises

**Finding**: The S3 key names in INSTRUCTOR_SETUP.ipynb PDF_MANIFEST exactly match
the key strings used in the Topic 2 exercise notebook:
- `barclays_personal_loan_faq.pdf` (Topic 2 Cell 8 demo + Cell 23/24 Lab 3)
- `barclays_credit_card_tnc.pdf` (Topic 2 Cell 10 Lab 1 + safety-net Cell 11)
No discrepancy. No fix needed.

### LOW-4: Topics 3-9 confirmed to need no S3 setup (except Topic 2)

**Finding**: Topics 1, 3, 4 have no S3 references in their exercises. Plans 5-9 confirm:
- Topic 5: no S3 (conversation memory is in-memory only)
- Topics 6-9: use inline BARCLAYS_DOCS strings; barclays_chunks.json is optional stretch
- Topic 9: loads Banking77 from HuggingFace (public HTTP, no S3 needed)
INSTRUCTOR_SETUP.ipynb topic readiness summary is accurate.
No discrepancy. No fix needed.

**Status of fixes (updated 2026-04-27)**:
- CRITICAL-1: CLOSED - openai==2.32.0 added to T2 Cell 2 install line (T2-FIX-001, applied 2026-04-25)
- CRITICAL-2: CLOSED - openai==2.32.0 is the course-wide canonical pin; all built notebooks now consistent
- MEDIUM-1: CLOSED - T3 Cell 2 comment corrected (T3-FIX-002/003/004 applied 2026-04-25)
- MEDIUM-2: CLOSED - plan files are informational; built notebooks use the canonical pin
- MEDIUM-3: CLOSED - T8 notebook uses !pip install -q with consistent openai pin (T8-FIX-002 applied 2026-04-27)
- LOW-1: CLOSED - test-bucket switch comment added to T2 Cell 3 (T2-FIX-003, applied 2026-04-25)
- LOW-2: CLOSED - intentional design decision, no fix
- LOW-3: CLOSED - confirmed match, no fix
- LOW-4: CLOSED - confirmed correct, no fix

---

## 2026-04-25 - T3-FIX-002, T3-FIX-003, T3-FIX-004: Topic 3 notebook fixes

**Topic**: topic_03_first_chatbot
**Files**:
- exercises/topic_03_first_chatbot/topic_03_first_chatbot.ipynb
- solutions/topic_03_first_chatbot/topic_03_first_chatbot.ipynb

**T3-FIX-002: Rename PRODUCT_SNIPPET to BARCLAYS_PRODUCT_SNIPPET**

Problem: Topic 3 defined `PRODUCT_SNIPPET` (expanded, loan + credit card) but the same
name was used in Topic 1 (loan only). Collision could cause confusion when students run
across topics.

Fix: Renamed all occurrences of bare `PRODUCT_SNIPPET` to `BARCLAYS_PRODUCT_SNIPPET` in
both exercise and solution notebooks. Cells affected (both notebooks):
  - Cell 5e20d7b2 (setup/imports - definition)
  - Cell e9ade7c9 (demo system prompt - reference)
  - Cell 927e9a48 (Lab 1 instructions - reference)
  - Cell 9a42ada5 (Lab 1 solution code - reference)
  - Cell c3d6177f (Lab 1 safety-net - reference)
  - Cell 0cdebe5a (naive approach demo - 2 references)
  - Cell c539085c (create_chatbot demo - reference)
  - Cell 9b38abe6 (Lab 2 instructions - reference)
  - Cell 99f4c544 (Lab 2 code - reference)
  - Cell 253129ec (Lab 2 safety-net - reference)
  - Cell 53f303a5 (Section 3 markdown - reference)
  - Cell 4f5455cc (naive approach code - 2 references)
  - Cell a2494909 (mermaid diagram + explanation - reference)
  - Cell 7d36560b (build_barclays_chatbot - 2 references)
  - Cell 53f75a0b (Lab 3 docstring + solution code - 2 references)
  - Cell eb523577 (closing markdown - reference)

**T3-FIX-003: Rephrase inaccurate "richer context" claim**

Problem: Cell 5e20d7b2 (setup) comment said "Students who ran Topic 2 will have a richer
chunks list available". This was inaccurate - Topic 2 chunks contain personal loan FAQ text
only, so credit card questions would fall through to the system prompt's general knowledge,
not "richer" context.

Also fixed the same inaccuracy in: Cell 53f303a5 (Section 3 intro markdown), Cell 7d36560b
(build_barclays_chatbot docstring), and Cell eb523577 (Key Takeaways markdown).

Fix: Replaced the claim with the accurate note: "Topic 2 chunks contain personal loan FAQ
text only. If you load Topic 2 chunks, credit card questions may fall through to the system
prompt's general knowledge."

**T3-FIX-004: Add GPT4O pricing constants to setup cell**

Problem: The stretch exercise in Cell eb523577 referenced `GPT4O_INPUT_PRICE_PER_1K` and
`GPT4O_OUTPUT_PRICE_PER_1K` but these were never defined in Topic 3, causing NameError
for any student attempting the cost-tracking stretch exercise.

Fix: Added both constants to Cell 5e20d7b2 (setup cell) after the OpenAI client block:
  GPT4O_INPUT_PRICE_PER_1K  = 0.0025   # dollars per 1K input tokens
  GPT4O_OUTPUT_PRICE_PER_1K = 0.0100   # dollars per 1K output tokens

Also updated Cell eb523577 stretch exercise #2 to say "constants defined in the setup cell"
(rather than "from Topic 1") since Topic 3 now defines them itself.

**Validation results**:
- Exercise: 1 pre-existing syntax error (cell f7d55f7f - !pip install magic syntax,
  exists in all topic notebooks, not introduced by these fixes). All other checks PASS:
  imports OK, 2 lab cells with correct placeholders, 26 cells total.
- Solution: Same pre-existing !pip syntax error. All other checks PASS: imports OK,
  2 lab solutions complete, 26 cells total.
- Pair check: PASSED - 26 cells match, all cell types match.

---

## 2026-04-27 - T4-FIX-001, T4-FIX-003, T4-FIX-005: Topic 4 notebook fixes

**Topic**: topic_04_prompt_engineering
**Exercise notebook**: exercises/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb
**Solution notebook**: solutions/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb

**T4-FIX-001: Remove dead sagemaker import block from Cell 3**

Problem: TODOS.md reported that cell 65e8dd4b (Cell 3, setup/imports) contained a
sagemaker import block (import sagemaker, get_execution_role, sess, role assignment,
and two print statements). Topic 4 makes zero AWS calls, so these lines were dead code
that would cause a SageMaker API call on every notebook startup.

Root cause: The block was a copy-paste artifact from the course template. It was
included in an earlier draft of the notebook and not removed when Topic 4 was finalized.

Fix: Verified both exercise and solution cell 65e8dd4b. The sagemaker lines were already
absent - the cell contains only os, json, getpass, OpenAI imports, client instantiation,
and INTENT_CATEGORIES list. The fix had been applied in a prior session. No edit needed.

**T4-FIX-003: Fix false Topic 5 routing claim**

Problem: TODOS.md reported that the title/intro cell (Cell 0, id 49b3c8d8) claimed
classify_intent() and classify_with_schema() are called by Topic 5 (Conversation Memory).
This is incorrect - Topic 5 does not import or use these functions; Topic 9 (Capstone)
uses them in the production_assistant() orchestrator.

Root cause: The notebook was drafted before the full cross-topic wiring was finalized.
The continuity note was written with a placeholder forward-reference to "the next topic"
which was later corrected in Cell 0 but the same incorrect claim was also present in the
wrap-up cell (Cell 26, id 90ecf431) under "Connection to Topic 5 (Conversation Memory)".

Fix: Cell 49b3c8d8 (Cell 0) was already correct in both notebooks ("Topic 9 (Capstone)").
Cell 90ecf431 (Cell 26, wrap-up) still contained "Connection to Topic 5 (Conversation
Memory)" with the incorrect description. Rewrote that section in both exercise and solution
notebooks to say "Connection to Topic 9 (Capstone)" and accurately describe the usage:
classify_intent() and classify_with_schema() are called in the Topic 9 production_assistant()
orchestrator to route each customer query to the appropriate handler.

Cells edited: 90ecf431 in both notebooks (replace, not delete).

**T4-FIX-005: Pin openai to 2.32.0 in Cell 2**

Problem: TODOS.md reported that cell ccd27dca (Cell 2, pip install) pinned openai==1.51.0,
causing a mid-session SDK downgrade when students ran Topic 1 (installs 2.32.0) then
Topic 4 (would downgrade to 1.51.0). The comment also incorrectly claimed 1.51.0 was
the minimum version required for json_schema strict mode.

Root cause: Topic 4 was originally drafted when json_schema strict mode first became
available at openai 1.51.0. After the course-wide version was standardized to 2.32.0
the Cell 2 pin was not updated in the original draft.

Fix: Verified both exercise and solution cell ccd27dca. The pin was already
`openai==2.32.0` and the comment already read "openai 2.32.0: course-wide canonical
pin - structured outputs work fine on 2.x". The fix had been applied in a prior session.
No edit needed.

**Validation**:
- Exercise: validation ran. 1 pre-existing syntax error (cell ccd27dca - !pip install
  magic syntax, flagged by Python syntax checker, not introduced by these fixes, exists
  in all topic notebooks). All other checks PASS: imports OK, 3 lab cells with correct
  placeholders, 27 cells total.
- Solution: same pre-existing !pip syntax error. All other checks pass.
- Pair check: not re-run (cell count unchanged by T4-FIX-003 which only replaced cell
  content; T4-FIX-001 and T4-FIX-005 required no edits). Prior pair check confirmed
  27 cells each, all types matching.

---

## 2026-04-27 - T5-FIX-003, T9-FIX-004, T9-FIX-007, T9-FIX-008, T9-FIX-009: Cross-topic consistency fixes

**Topics**: topic_05_conversation_memory, topic_09_capstone
**Exercise notebooks**:
- exercises/topic_05_conversation_memory/topic_05_conversation_memory.ipynb
- exercises/topic_09_capstone/topic_09_capstone.ipynb
**Solution notebooks**:
- solutions/topic_05_conversation_memory/topic_05_conversation_memory.ipynb
- solutions/topic_09_capstone/topic_09_capstone.ipynb

**T5-FIX-003: BARCLAYS_SYSTEM_PROMPT in T5 replaced with T3 canonical version**

Problem: Cell 2db98a99 (setup cell) in T5 exercise and solution notebooks defined a short
4-sentence BARCLAYS_SYSTEM_PROMPT. T3's canonical version has four named sections (Persona,
Constraints, Format) with explicit negatives covering the FCA advice boundary and scope
constraint. Using a shorter version in T5 causes persona drift mid-course.

Root cause: T5 was written with an inline short prompt rather than importing T3's canonical
version verbatim.

Fix: Cell 2db98a99 in both T5 exercise and solution notebooks updated with T3's full
triple-quoted BARCLAYS_SYSTEM_PROMPT. The comment above was updated to read:
"Reusing the T3 system prompt verbatim so the persona stays stable across topics."
No cells added or deleted; only content replaced.

**T9-FIX-004: BARCLAYS_SYSTEM_PROMPT in T9 continuity cell replaced with T3 canonical version**

Problem: Cell 1454f38d9b71 (continuity setup cell) in T9 exercise and solution notebooks
used a 3-sentence BARCLAYS_SYSTEM_PROMPT variant. Students in the capstone would see a
weaker system prompt than the one they built in T3.

Root cause: T9 continuity cell was written as a condensed reimplementation rather than
verbatim copy from T3.

Fix: BARCLAYS_SYSTEM_PROMPT in cell 1454f38d9b71 of both T9 notebooks replaced with T3's
full canonical triple-quoted string. A comment was added: "T9-FIX-004: BARCLAYS_SYSTEM_PROMPT
matches T3 verbatim so the persona stays stable across topics."

**T9-FIX-007: CLASSIFICATION_SCHEMA and classify_with_schema in T9 aligned to T4's 3-field schema**

Problem: Cell 1454f38d9b71 defined CLASSIFICATION_SCHEMA with only 2 fields:
  - intent: string enum
  - confidence: number (WRONG - T4 uses string enum "high"|"medium"|"low")
T4's canonical schema has 3 fields: intent, confidence (string enum), rationale (string).
The 2-field schema would fail OpenAI strict-mode validation and return different output
shape than students built in T4.

Root cause: T9 continuity cell was a hand-rolled reimplementation that diverged from T4.

Fix: CLASSIFICATION_SCHEMA and classify_with_schema in cell 1454f38d9b71 of both T9
notebooks replaced with T4's verbatim 3-field schema. confidence is now a string enum
(high|medium|low), rationale field added, schema name changed from "intent_classification"
to "classification_result" to match T4.

**T9-FIX-008: count_tokens_in_messages in T9 corrected to T5's 3+3 formula**

Problem: Cell 1454f38d9b71 used:
  - 4 tokens per message overhead (WRONG)
  - total += 2 priming tokens at end (WRONG)
T5's canonical formula for gpt-4o (o200k_base encoding) is:
  - 3 tokens per message overhead
  - 3 priming tokens at end
  - Uses tiktoken.encoding_for_model() with KeyError fallback to o200k_base

Root cause: T9 continuity cell was a hand-rolled reimplementation that used a different
token overhead formula.

Fix: count_tokens_in_messages in cell 1454f38d9b71 of both T9 notebooks replaced with T5's
verbatim implementation. A comment was added: "T9-FIX-008: count_tokens_in_messages matches
T5's formula verbatim. Formula for gpt-4o (o200k_base): 3 tokens per message + tokens in
content + 3 priming tokens."

**T9-FIX-009: BarclaysChat.__init__ in T9 aligned to T5 signature**

Problem: Cell 1454f38d9b71 BarclaysChat had:
  - __init__(self, system_prompt: str = BARCLAYS_SYSTEM_PROMPT, max_tokens: int = 3000)
  - Used self.client = client pattern (T5 uses module-global client directly)
T5's canonical signature is __init__(self, system_prompt) with no max_tokens param and
module-global client. The constructor mismatch was pedagogically confusing.

Additional affected cells:
  - ccf4e7d60f5a (production_assistant) accessed chat_state.max_tokens directly
  - eddf92cc860d (end-to-end test) called BarclaysChat(system_prompt=..., max_tokens=3000)
  - 0a3564a3637d (Lab 4 solution) called BarclaysChat(system_prompt=..., max_tokens=3000)

Root cause: T9 continuity cell was written to make production_assistant() simpler by
adding max_tokens to the constructor, but this diverged from T5's public API.

Fix:
  1. BarclaysChat in cell 1454f38d9b71 updated to T5 signature: __init__(self, system_prompt).
     max_tokens=3000 added as a class-level attribute so production_assistant can still
     read chat_state.max_tokens via getattr(chat_state, "max_tokens", 3000).
  2. production_assistant (cell ccf4e7d60f5a) updated to use getattr(chat_state, "max_tokens", 3000).
  3. End-to-end test (cell eddf92cc860d) updated: BarclaysChat(system_prompt=BARCLAYS_SYSTEM_PROMPT)
     without max_tokens kwarg.
  4. Lab 4 solution (cell 0a3564a3637d, solution notebook only) updated similarly.
     Also fixed the assertion in eddf92cc860d that checked "[REDACTED:uk_account_number]"
     (lowercase colon format) to match detect_pii's actual output "[REDACTED_UK_ACCOUNT_NUMBER]"
     (uppercase underscore format).

All four cells edited in both exercise and solution notebooks.

**Validation**: No validation script run (Bash tool not available in this session). Cell IDs
were verified by reading each notebook with the Read tool before every NotebookEdit call.
Cell counts unchanged for all four notebooks (only content replaced, no cells added or deleted).

---

## 2026-04-27 - T9-FIX-012, T9-FIX-013, TEST-SETUP-FIX-001: T9 cost note, stretch hint, numpy pin

**Topics**: topic_09_capstone, setup/INSTRUCTOR_TEST_SETUP.ipynb
**Exercise notebook**: exercises/topic_09_capstone/topic_09_capstone.ipynb
**Solution notebook**: solutions/topic_09_capstone/topic_09_capstone.ipynb
**Setup file**: setup/INSTRUCTOR_TEST_SETUP.ipynb

**T9-FIX-012: Add web_search per-call cost note to end-to-end test cell**

Problem: Cell eddf92cc860d computes `total_cost` as the sum of per-token costs across the
5-query battery. The OpenAI hosted web_search tool charges an additional per-call fee on
top of token cost. Queries routed to 'hybrid' or 'web' by route_query() would have their
total cost understated without this note.

Root cause: The cost calculation was written to cover token cost only, with no mention
that web_search incurs a separate per-call billing line.

Fix: Added a NOTE comment block at the end of cell eddf92cc860d in both exercise and
solution notebooks:
  # NOTE: this calculator covers per-token cost only. The OpenAI hosted web_search tool
  # adds a per-call fee on top (consult the current OpenAI pricing page). When
  # route_query returns 'hybrid' or 'web', add the per-call web_search fee to your
  # log_record manually.

Cell content replaced (no cells added or deleted). Both exercise and solution updated.

**T9-FIX-013: Add barclays_chunks.json stretch hint to Tier 3 lab brief markdown**

Problem: The Tier 3 lab brief (cell b59e17a9a1df) describes four production extension
options (semantic cache, prompt-cache key, model tiering, async eval queue) but gives no
hint about the real Barclays product data available for testing. Students extending the
assistant would benefit from knowing they can load richer S3 data.

Root cause: The Tier 3 lab brief was written before the instructor setup workflow and
S3 bucket were finalized. The stretch data hint was added to T6 and T7 labs but not T9.

Fix: Added a "Stretch data:" paragraph at the end of cell b59e17a9a1df in both notebooks:
  Stretch data: load `s3://barclays-prompt-eng-data/barclays_chunks.json` if you want
  to test your extension against real Barclays product text. See
  `setup/INSTRUCTOR_SETUP.ipynb` Cell 10 for the load snippet.

Cell content replaced (no cells added or deleted). Both exercise and solution updated.

**TEST-SETUP-FIX-001: Add numpy<2 to INSTRUCTOR_TEST_SETUP.ipynb pip install**

Problem: Cell 6848e038 of setup/INSTRUCTOR_TEST_SETUP.ipynb installed fpdf2, pymupdf,
requests, and boto3 but omitted "numpy<2". The mandatory numpy<2 pin is required by all
notebooks in the course because chromadb 1.x and the SageMaker Python SDK break on
numpy 2.x. An instructor running the test setup notebook from a clean environment could
inadvertently install numpy 2.x, which would break subsequent course notebook runs.

Root cause: The test setup notebook was written before the mandatory numpy<2 pin was
codified as a course-wide constraint.

Fix: Changed the pip install line in cell 6848e038 from:
  !pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3
to:
  !pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3 "numpy<2"

**Validation**:
- Exercise (topic_09_capstone): validated. Pre-existing errors only - Cell 4 flagged for
  syntax error (os/getpass/OpenAI not in scope in isolated syntax check, not a real error),
  chromadb flagged as missing module (not in local venv, installed only on SageMaker).
  Both are pre-existing artifacts present before these changes. No new errors introduced.
- Solution (topic_09_capstone): same pre-existing errors. Additional pre-existing warning:
  `blocked_tag = None` in cell 25 flagged as incomplete solution (false positive - it is
  a local variable initialized to None and conditionally set, not a lab placeholder).
- Cell counts: unchanged. 29 cells in both exercise and solution. Only content replaced.
- INSTRUCTOR_TEST_SETUP.ipynb: no automated validator for setup notebooks. Change verified
  by reading cell 6848e038 directly after edit.

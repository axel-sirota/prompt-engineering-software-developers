# Change and Fix Audit Log
# Barclays - Generative AI: Prompt Engineering for Software Developers
#
# Every fix applied by /fixes is recorded here permanently.
# TODOS.md tracks open issues; AUDIT.md tracks resolved ones.

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

**Status of fixes**:
- CRITICAL-1: OPEN - needs fix in Topic 2 Cell 2
- CRITICAL-2: OPEN - needs decision and fix across all topics/plans
- MEDIUM-1: OPEN - needs comment fix in Topic 3 Cell 2
- MEDIUM-2: OPEN - needs plan file updates (Plans 7, 9) after CRITICAL-2 is decided
- MEDIUM-3: OPEN - needs plan file cleanup (Plan 8) after CRITICAL-2 is decided
- LOW-1: OPEN - needs comment added to Topic 2 Cell 3
- LOW-2: CLOSED - intentional design decision, no fix
- LOW-3: CLOSED - confirmed match, no fix
- LOW-4: CLOSED - confirmed correct, no fix

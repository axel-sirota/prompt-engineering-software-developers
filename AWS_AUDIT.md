# AWS_AUDIT.md - Cross-Topic Wiring Audit and Surgical Remediation Plan

**Course**: Barclays - Generative AI: Prompt Engineering for Software Developers
**Audit date**: 2026-04-25
**Scope**: All 9 topic plans (`plans/topic_01...md` through `plans/topic_09...md`)
plus the two setup notebook plans (`plans/instructor_setup.md`,
`plans/test_data_setup.md`) plus the locked course manifest
(`plans/CORE_TECHNOLOGIES_AND_DECISIONS.md`) and the per-topic manifest
(`plans/TOPICS.md`).

**Method**: Consolidation of three per-day discrepancy audits stored at
`plans/aws_setup/discrepancies_1_2_3.md`,
`plans/aws_setup/discrepancies_4_5_6.md`, and
`plans/aws_setup/discrepancies_7_8_9.md`. Each was produced via the
`/research` protocol, 3 cycles, no web search, internal file analysis only.
This file does NOT introduce new findings; it integrates and re-orders the
existing findings into a single surgical action plan.

**Source-of-truth choice**: Plan files (`plans/topic_NN_*.md`), not the built
`.ipynb` notebooks. This is consistent with the build process: every
`/build-topic-notebook N` run reads the plan file. If the plan is wrong,
the next build is wrong. Where the built notebook diverges from the plan
(only T5 install line; see S5-FIX-001), the file lists the plan-side fix
and notes whether the built notebook also needs updating.

**No fixes applied by this document**. This is a specification for surgical
edits to be applied later (manually, via `/fixes`, or via re-runs of
`/build-topic-notebook` after the plan files are corrected).

---

## 1. Executive Summary

### 1.1 The good news (AWS / S3 / setup-notebook integration)

The two setup notebooks under `setup/` (`INSTRUCTOR_SETUP.ipynb` and
`INSTRUCTOR_TEST_SETUP.ipynb`) are correctly aligned with the topic plans.

- Bucket name `barclays-prompt-eng-data` matches between Topic 2 and
  `plans/instructor_setup.md`. Byte-for-byte match.
- Region `us-east-2` matches across Topic 2, `plans/instructor_setup.md`,
  and `plans/test_data_setup.md`. Byte-for-byte match.
- The two required S3 keys (`barclays_personal_loan_faq.pdf`,
  `barclays_credit_card_tnc.pdf`) match between Topic 2 Cell 8 / Cell 10 /
  Cell 23 / Cell 24 and the setup notebooks' PDF_MANIFEST / ALL_PDFS dicts.
- Topics 1, 3, 4, 5, 6, 7, 8, 9 correctly avoid S3 entirely. This matches
  `plans/instructor_setup.md` Cell 12 topic-readiness summary.
- The synthetic PDF generators in `plans/test_data_setup.md` produce text
  that matches Topic 2 Cell 6's raw_pdf_noise demo and Cell 8 fallback
  string (same "58 days interest" ERC, same loan amounts, same APR).

**No new uploads, no new IAM permissions, no edits to the setup notebooks
under `setup/` are required to support any of T1-T9.**

### 1.2 The bad news (cross-topic wiring)

Roughly 10 HIGH-severity defects, 6 MEDIUM, 13 LOW were identified
across the 9 topic plans. The 5-category roll-up below counts each
*defect cluster* once at its dominant severity; the literal per-fix
recount in Section 4 is somewhat higher because some defects are
addressed by multiple fix entries (e.g. T9 carryover errors split into
T9-FIX-005, T9-FIX-006, T9-FIX-007, T9-FIX-008, T9-FIX-009 individually).

| Category | HIGH | MEDIUM | LOW | Total |
|----------|------|--------|-----|-------|
| Install line bugs (missing pkg, shell-redirect, version drift) | 2 | 1 | 6 | 9 |
| Variable name collisions across topics | 3 | 2 | 0 | 5 |
| Continuity claims that do not match the actual functions | 1 | 3 | 4 | 8 |
| Content drift (BARCLAYS_DOCS, BARCLAYS_SYSTEM_PROMPT) | 3 | 0 | 2 | 5 |
| Manifest conflicts (CORE vs TOPICS.md vs T9 plan) | 1 | 0 | 1 | 2 |

**Notebook re-verification roll-up (2026-04-25, all 9 notebooks now built)**:

| Status (vs built notebook) | Count |
|----------------------------|-------|
| ALREADY FIXED IN NOTEBOOK  | 6     |
| STILL NEEDS FIX IN NOTEBOOK| 34    |
| NOT APPLICABLE (plan-side / setup-only / metadata) | 13 |
| NEEDS USER DECISION        | 1 (S2-FIX-001) |

Notable resolutions since the original audit:
- T1-FIX-001, T1-FIX-002, T2-FIX-001, T4-FIX-006, T5-FIX-001 are all
  ALREADY FIXED in the built notebooks (install line quoting, redundant
  `import openai` removal, missing openai package in T2 install line, T4
  title emoji, T5 install line quoting).
- T2-FIX-004 was a verification-only entry; confirmed correct in T2 Cell 6.
- The S2-FIX-001 capstone scope conflict was de facto resolved by Option B
  during the build (T9 was built as the integration capstone, not the
  Banking77 transaction agent), but CORE_TECHNOLOGIES_AND_DECISIONS.md
  was not updated to reflect that decision.
- All HIGH-severity defects that remain in the built notebooks are in
  T6 (BARCLAYS_DOCS ERC), T7 (BARCLAYS_DOCS drift), and T9 (collection
  name, BARCLAYS_DOCS drift, detect_pii / should_escalate signatures).

### 1.3 The single blocker

**T9 capstone scope conflict** (S2-FIX-001, originally S2.1 in
`discrepancies_7_8_9.md`). CORE manifest says T9 is the "Barclays
Transaction Query Agent" using Banking77; the T9 plan delivers a
"Production Customer Service Assistant" that integrates T1-T8.
TOPICS.md agrees with the T9 plan; CORE does not. This needs a user
decision (Option A / B / C) before `/build-topic-notebook 9` can run.

### 1.4 The single byte fix that unblocks the most downstream value

**T6 BARCLAYS_DOCS[0] ERC value** (S6-FIX-002, originally S3.1 in
`discrepancies_4_5_6.md`). T6 line 213 says "30-day interest"; every
other source (Topic 2, the synthetic PDFs in test_data_setup.md, T7
demos) says "58 days interest". One-token edit. Fixes the ERC narrative
across T2 -> T6 -> T7 -> T9.

---

## 2. AWS / S3 / Setup-Notebook Integration Status

### 2.1 What lives in each S3 bucket

```
s3://barclays-prompt-eng-data/                      (instructor populates ON COURSE DAY)
  barclays_personal_loan_faq.pdf                    (REQUIRED by T2 Cell 8, 10, 23, 24)
  barclays_credit_card_tnc.pdf                      (REQUIRED by T2 Cell 10 safety-net, 22)
  barclays_savings_rates.pdf                        (OPTIONAL, T6 Tier 3 stretch)
  barclays_isa_terms.pdf                            (OPTIONAL, T6 Tier 3 stretch)
  barclays_chunks.json                              (OPTIONAL, Topics 6-9 Tier 3 stretch)

s3://<MY_TEST_BUCKET>/                              (instructor populates BEFORE COURSE, personal AWS account)
  same 4 PDFs (synthetic, fpdf2-generated, content matched to T6 BARCLAYS_DOCS)
  barclays_chunks.json                              (re-chunked from synthetic PDFs)
```

### 2.2 Per-topic AWS dependencies

| Topic | Reads S3? | Writes S3? | Imports boto3? | Imports sagemaker? | Verdict |
|-------|-----------|------------|----------------|--------------------|---------|
| T1 | No | No | No | YES (dead) | T1-FIX-003: drop dead import |
| T2 | YES | No | YES | YES | Correct - this is the only S3 topic |
| T3 | No | No | No | YES (dead) | T3-FIX-001: drop dead import |
| T4 | No | No | No | YES (dead) | T4-FIX-001: drop dead import |
| T5 | No | No | No | No | Correct - exemplar pattern |
| T6 | No | No | No | YES (dead) | T6-FIX-001: drop dead import |
| T7 | No | No | No | No | Correct |
| T8 | No | No | No | No | Correct |
| T9 | No | No | No | No | Correct |
| INSTRUCTOR_SETUP | YES | YES | YES | YES | Correct |
| INSTRUCTOR_TEST_SETUP | No | YES | YES | No (uses local creds) | Correct |

The "dead sagemaker import" pattern (T1, T3, T4, T6) is harmless in
SageMaker Studio but raises `botocore.exceptions.NoRegionError` or similar
in any non-SageMaker environment (local Jupyter, Codespace, CI). This
contradicts the "self-contained notebook" principle in CORE line 35-37.

### 2.3 Confirmed: setup notebooks under `setup/` need ZERO changes for T1-T9

| Setup notebook | Reason no change needed |
|----------------|-------------------------|
| `setup/INSTRUCTOR_SETUP.ipynb` | All required keys (T2 PDFs) and optional keys (chunks.json) are already in PDF_MANIFEST. T6 Tier 3 stretch (S5.1) optionally wants 4 more PDFs (mortgage, student, business, travel) but this is a quality-of-life improvement, not a required fix. |
| `setup/INSTRUCTOR_TEST_SETUP.ipynb` | Synthetic PDFs match Topic 2 Cell 6 / Cell 8 / T6 BARCLAYS_DOCS values byte-for-byte (after the T6-FIX-002 ERC fix). Adding `"numpy<2"` to the install line (TEST-SETUP-FIX-001) is the only LOW-severity hardening item. |

### 2.4 What is NOT in any plan and does not need to be

No DynamoDB. No RDS. No additional EBS state beyond `./chroma_db`
(local persistent ChromaDB introduced by T6 and reused by T7, T9).
No new IAM policies beyond what the instructor role already has
(`s3:PutObject` + `s3:DeleteObject` for the bucket, per
`plans/instructor_setup.md` Cell 3 probe). No HuggingFace token
required (Banking77 is `PolyAI/banking77`, public).

---

## 3. Cross-Cutting Wiring Matrix

This is the master reference for every variable, function, constant, and
external resource that crosses topic boundaries. Each row is a contract
between topics; the Status column says whether the contract is honoured.

### 3.1 Variables and constants

| Name | Defined in | Re-defined / consumed by | Status | Fix ref |
|------|------------|--------------------------|--------|---------|
| `OPENAI_API_KEY` (env) | T1 Cell 3, T2 Cell 6, T3 Cell 3 | T4-T9 all topics | OK (always re-prompted) | T2-FIX-005 (gate the prompt) |
| `client` (OpenAI) | T1 Cell 3, T2 Cell 6, T3 Cell 3 | T4-T9 | OK | none |
| `MODEL = "gpt-4o"` | T5 Cell 3 ONLY | T5, T8 (claims), T9 (claims) | DRIFT - T8 line 7 and T9 line 41 falsely claim MODEL is a T1/T3 carryover | T5-FIX-002, T8-FIX-001, T9-FIX-006 |
| `S3_BUCKET = "barclays-prompt-eng-data"` | T2 Cell 3 | T2 only | OK (matches setup notebooks) | none |
| `REGION = "us-east-2"` | T2 Cell 3 | T2 only | OK (matches setup notebooks) | none |
| `PRODUCT_SNIPPET` | T1 Cell 11 (loan only), T3 Cell 3 (loan + credit card) | T1, T3 | COLLISION - same name, different content | T3-FIX-002 |
| `BARCLAYS_SYSTEM_PROMPT` | T3 Cell 3 (full ~120 word), T5 Cell 5 (50 word redef), T9 Cell 5 (third version) | T3, T5, T9 | TRIPLE DEFINITION | T5-FIX-003, T9-FIX-007 |
| `chunks` | T2 Cell 21 (loan FAQ only), T6 Cell 5 (chunks = BARCLAYS_DOCS, 7 docs) | T3 Cell 22 reads, T6/T7/T9 redefine | DRIFT - T2's chunks list is loan-only but T3 claims "richer context" | T2-FIX-006 OR T3-FIX-003 |
| `BARCLAYS_DOCS` | T6 Cell 5 (7 docs, canonical), T7 Cell 5 (6 docs, different products), T9 Cell 5 (5 docs, third set) | T6, T7, T9 | TRIPLE DEFINITION with content drift | T7-FIX-001, T9-FIX-002 |
| `GPT4O_INPUT_PRICE_PER_1K`, `GPT4O_OUTPUT_PRICE_PER_1K` | T1 Cell 18 | T3 Cell 25 stretch claims them | NOT DEFINED in T3 | T3-FIX-004 |
| `ask_with_cost_tracking()` | T1 Cell 18 | T3 line 8 carryover claim | NOT IMPORTED in T3 | T3-FIX-005 |
| `INTENT_CATEGORIES` | T4 (5 categories) | T9 line 381 | OK (exact match) | none |
| `CLASSIFICATION_SCHEMA` | T4 (3 fields: intent, confidence enum, rationale) | T9 Cell 5 reimpl (2 fields, confidence is numeric) | DRIFT | T9-FIX-008 |
| `classify_intent(text)` | T4 (returns string) | T5 line 9 falsely claims it returns a JSON-schema dict | DOC DRIFT | T5-FIX-005 |
| `classify_with_schema(text)` | T4 (returns 3-field dict) | T9 reimpl returns 2-field dict | SIGNATURE DRIFT | T9-FIX-008 |
| `count_tokens_in_messages(messages)` | T5 (3 tokens/msg + 3 priming, gpt-4o cookbook formula) | T9 reimpl (4+2, off by 1 in both dims) | SIGNATURE DRIFT | T9-FIX-009 |
| `BarclaysChat` class | T5 (`__init__(system_prompt)`, attrs: history, system_prompt) | T8 line 9 falsely claims `self.client` attribute, T9 reimpl adds max_tokens param | SIGNATURE DRIFT | T8-FIX-002, T9-FIX-010 |
| `create_chatbot()` | T3 (`(system_prompt, context="", model="gpt-4o")`) | T8 line 7 omits the `context` param | DOC DRIFT | T8-FIX-001 |
| `embed_texts(texts)` | T6 | T7, T9 by name | OK | none |
| `add_to_store`, `retrieve`, `rag_answer` | T6 | T9 lines 51-56 | OK | none |
| `web_search_barclays`, `extract_citations`, `route_query`, `hybrid_answer`, `vector_confidence` | T7 | T9 line 59, 530+ | OK | none |
| `detect_pii(text)` | T8 (returns `GuardrailResult` dataclass) | T9 reimpl returns `list[dict]` | SIGNATURE DRIFT | T9-FIX-011 |
| `should_escalate(query, history)` | T8 (returns `GuardrailResult`) | T9 reimpl `(text)` returning `(bool, str)` tuple | SIGNATURE DRIFT | T9-FIX-012 |
| `GuardrailResult` dataclass | T8 Cell 7 | T9 (does not import / does not define) | NOT REUSED | T9-FIX-011, T9-FIX-012 |

### 3.2 Local files / persistent state

| Path | Created by | Read by | Status |
|------|------------|---------|--------|
| `./chroma_db/` (SQLite + Parquet) | T6 Cell 22 (`PersistentClient`) | T7 Cell 5, T9 Cell 5 (claims) | T9 uses different collection name (S3.2 in 7_8_9 audit) | T9-FIX-001 |
| ChromaDB collection `barclays_products` | T6 Cell 22 | T7 Cell 5, T9 (should) | T9 uses `barclays_capstone` instead | T9-FIX-001 |
| ChromaDB collection `barclays_capstone` | T9 Cell 5 (current) | nothing else | redundant; should not exist after fix | T9-FIX-001 |
| Spurious file `=1.30.0` | (would be created by T5 plan line 127 if rebuilt verbatim) | nothing | shell-redirect bug | T5-FIX-001 |

### 3.3 External APIs and datasets

| Resource | Used by | Notes | Fix ref |
|----------|---------|-------|---------|
| OpenAI Chat Completions API | T1, T2 (naive demo), T3, T4, T5, T6, T7, T8, T9 | Standard | none |
| OpenAI Embeddings API (`text-embedding-3-small`) | T6, T7, T9, INSTRUCTOR_SETUP (chunking only) | Standard | none |
| OpenAI Responses API + web_search tool | T7, T9 | Per-call fee NOT in T9 cost calculator | T9-FIX-012 |
| HuggingFace `PolyAI/banking77` dataset | T9 (per CORE) but T9 plan does NOT use it | Resolved by S2-FIX-001 (capstone scope) | S2-FIX-001 |
| barclays.co.uk via web_search allowed_domains | T7, T9 | OK | none |
| AWS S3 via boto3 | T2, INSTRUCTOR_SETUP, INSTRUCTOR_TEST_SETUP | OK | none |
| AWS STS via boto3 | INSTRUCTOR_TEST_SETUP only | OK (account-ID safety check) | none |

### 3.4 Library version pins (the install-line matrix)

This is the most cosmetically inconsistent surface in the course.

| Plan / file | openai pin | chromadb | other notable |
|-------------|------------|----------|---------------|
| T1 Cell 2 | `openai==2.32.0` (UNQUOTED) | n/a | `tiktoken==0.9.0` UNQUOTED |
| T2 Cell 2 | NOT INSTALLED (HIGH BUG) | n/a | `pymupdf`, `bs4`, `lxml`, `boto3` |
| T3 Cell 2 | `"openai==2.32.0"` (quoted) | n/a | |
| T4 Cell 2 | `"openai==1.51.0"` (1.x exact) | n/a | comment claims this version is required for json_schema strict |
| T5 Cell 2 (plan) | `openai>=1.30.0` UNQUOTED (HIGH BUG, shell redirect) | n/a | `tiktoken==0.9.0` UNQUOTED |
| T5 Cell 2 (built notebook) | `"openai>=1.30.0"` quoted | n/a | manually fixed at build time, plan was not synced |
| T6 Cell 2 | `"openai>=1.30.0"` | `"chromadb==1.5.8"` (first in line, correct) | |
| T7 Cell 2 | `"openai>=2.30.0"` (2.x lower bound) | `"chromadb==1.5.8"` | |
| T8 Cell 2 | `"openai==2.32.0"` (2.x exact) - and uses `%pip install --quiet` instead of `!pip install -q` | NOT INSTALLED (confirmed unused) | |
| T9 Cell 2 | `"openai>=2.30.0"` | `"chromadb==1.5.8"` | adds `tenacity`, `tiktoken` |
| INSTRUCTOR_SETUP Cell 2 | not installed (correct, no LLM calls) | n/a | `requests`, `pymupdf`, `numpy<2` |
| INSTRUCTOR_TEST_SETUP Cell 2 | not installed (correct) | n/a | `fpdf2`, `pymupdf`, `requests`, `boto3` - missing `numpy<2` |

Three problems jump out:
1. **HIGH**: T2 imports `from openai import OpenAI` in Cell 6 but never installs it.
2. **HIGH**: T5 plan install line has `openai>=1.30.0` UNQUOTED. In bash/zsh, the `>=` is parsed as a stdout redirect to a file literally named `=1.30.0`. The built notebook has the quotes; the plan does not.
3. **MEDIUM**: T4 pins `openai==1.51.0` (1.x); T7-T9 pin `openai>=2.30.0` (2.x). A student running T4 then T7 in the same kernel gets a silent SDK upgrade between topics.

---

## 4. Per-Topic Surgical Fix List

Each fix has: **ID** (referenced from Section 3 and Section 5), **plan
file path**, **plan file line**, **current text** (verbatim), **replacement
text** (verbatim), **severity**, and **source-of-truth justification**
(why we believe the fix is correct).

Severity levels: **HIGH** = blocks `/build-topic-notebook` or breaks runtime.
**MEDIUM** = silent contract break, surfaces as mysterious bug. **LOW** =
cosmetic / documentation only.

### 4.1 Topic 1 - Foundations

#### T1-FIX-001: Quote install line specs

- **Plan file**: `plans/topic_01_foundations.md`
- **Line**: 105
- **Current**:
  ```
  !pip install -q openai==2.32.0 "numpy<2" tiktoken==0.9.0
  ```
- **Replacement**:
  ```
  !pip install -q "openai==2.32.0" "numpy<2" "tiktoken==0.9.0"
  ```
- **Severity**: LOW
- **Justification**: Course convention (T2, T3 already quote every spec).
  No runtime difference today (no shell metacharacters in `==` specs).
  Source: discrepancies_1_2_3.md S2.4.
- **NOTEBOOK STATUS (2026-04-25)**: ALREADY FIXED IN NOTEBOOK
  Evidence: T1 Cell 2 reads `!pip install -q "openai==2.32.0" "numpy<2" "tiktoken==0.9.0"` (all specs quoted).

#### T1-FIX-002: Drop redundant `import openai`

- **Plan file**: `plans/topic_01_foundations.md`
- **Lines**: 122-123
- **Current**:
  ```python
  import openai
  from openai import OpenAI
  ```
- **Replacement**:
  ```python
  from openai import OpenAI
  ```
- **Severity**: LOW
- **Justification**: bare `import openai` is never used. Only `OpenAI`
  class is referenced. Source: discrepancies_1_2_3.md S2.5.
- **NOTEBOOK STATUS (2026-04-25)**: ALREADY FIXED IN NOTEBOOK
  Evidence: T1 Cell 3 has only `from openai import OpenAI` plus `import tiktoken`; no bare `import openai`.

#### T1-FIX-003: Drop dead sagemaker import OR fix the misleading comment

- **Plan file**: `plans/topic_01_foundations.md`
- **Lines**: 127-135
- **Current**:
  ```python
  import sagemaker
  from sagemaker import get_execution_role

  sess = sagemaker.Session()
  role = get_execution_role()

  print(f"SageMaker region: {sess.boto_region_name}")
  print(f"Execution role (short): ...{role[-30:]}")
  ```
  with comment on line 127: `# useful for S3 access later`
- **Replacement (Option A, recommended)**: delete ONLY the sagemaker /
  session / role / print block (lines 127-135 in the current plan; lines
  shift down by 2 if T1-FIX-002 is applied first because that fix removes
  one line above). Keep `import openai` / `from openai import OpenAI` /
  `import tiktoken` (lines 122-125) intact - those are required by Cell 3
  and later cells.
- **Replacement (Option B)**: keep the sagemaker block but change the
  line 127 comment to:
  ```python
  # SageMaker boilerplate kept here so all topics share an identical setup cell.
  # T1 itself does not call any AWS API; S3 reads start in Topic 2.
  ```
- **Severity**: LOW (cosmetic)
- **Justification**: T1 never calls boto3 / S3. The "for S3 access later"
  comment is misleading. Source: discrepancies_1_2_3.md S1.5.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T1 Cell 3 still contains `import sagemaker` / `from sagemaker import get_execution_role` / `sess = sagemaker.Session()` / `role = get_execution_role()` plus the misleading comment "useful for S3 access later".

#### T1-FIX-004 (OPTIONAL): Add MODEL constant

- **Plan file**: `plans/topic_01_foundations.md`
- **Line**: in Cell 3, after the OpenAI client setup (around line 140)
- **Add**:
  ```python
  MODEL = "gpt-4o"
  ```
- **Severity**: LOW (optional, for cross-topic continuity)
- **Justification**: T5 line 24 and T8 line 7 and T9 line 41 all claim
  `MODEL` is a carryover from earlier topics. No earlier topic defines it.
  Adding to T1 (and T3, T4) makes the claim true and lets students do
  one-line model swaps. Source: discrepancies_1_2_3.md S3.5.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T1 Cell 3 does not define `MODEL = "gpt-4o"`; later cells use the inline literal. T6 Cell 3 defines `MODEL = "gpt-4o"` with a comment "Same model constant we have used since Topic 1" but this is only true in T5 and T6.

### 4.2 Topic 2 - NLP Preprocessing

#### T2-FIX-001 (HIGH): Add openai to install line

- **Plan file**: `plans/topic_02_nlp_preprocessing.md`
- **Line**: 136
- **Current**:
  ```
  !pip install -q "pymupdf==1.27.2.2" "beautifulsoup4==4.14.3" lxml boto3 "numpy<2"
  ```
- **Replacement**:
  ```
  !pip install -q "pymupdf==1.27.2.2" "beautifulsoup4==4.14.3" "openai==2.32.0" lxml boto3 "numpy<2"
  ```
- **Severity**: HIGH
- **Justification**: Cell 6 line 247 does `from openai import OpenAI` and
  Cell 6 / Cell 13 call `client.chat.completions.create`. A student who
  starts at T2 (skipping T1) gets either an unpinned openai or none at
  all. CORE line 47 mandates self-contained notebooks. Source:
  discrepancies_1_2_3.md S2.3.
- **NOTEBOOK STATUS (2026-04-25)**: ALREADY FIXED IN NOTEBOOK
  Evidence: T2 Cell 2 reads `!pip install -q "pymupdf==1.27.2.2" "beautifulsoup4==4.14.3" lxml boto3 "openai==2.32.0" "numpy<2"` - openai is included.

#### T2-FIX-002 (MEDIUM): Rewrite Cell 3 misleading "no OpenAI key" comment

- **Plan file**: `plans/topic_02_nlp_preprocessing.md`
- **Lines**: 168-169
- **Current**:
  ```python
  # The SageMaker execution role already has S3 read permissions for this bucket.
  # We do NOT need the OpenAI key for this topic - this is pure document processing.
  ```
- **Replacement**:
  ```python
  # The SageMaker execution role already has S3 read permissions for this bucket.
  # We will be prompted for the OpenAI key in Cell 6 when the first naive
  # demo runs. The bulk of this topic is document processing (no LLM calls).
  ```
- **Severity**: MEDIUM
- **Justification**: Cell 6 prompts for the key via getpass. The Cell 3
  comment contradicts that. Source: discrepancies_1_2_3.md S3.6.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T2 Cell 3 still contains the misleading comment: "We do NOT need the OpenAI key for this topic - this is pure document processing." while Cell 6 prompts for the key.

#### T2-FIX-003 (LOW): Add test-bucket switch comment near S3_BUCKET

- **Plan file**: `plans/topic_02_nlp_preprocessing.md`
- **Line**: 165 (just above `S3_BUCKET = "barclays-prompt-eng-data"`)
- **Add**:
  ```python
  # For PRE-COURSE TESTING ONLY: change to your personal test bucket.
  # See setup/INSTRUCTOR_TEST_SETUP.ipynb for the test workflow.
  # Restore "barclays-prompt-eng-data" before the course.
  ```
- **Severity**: LOW
- **Justification**: The test-mode workflow exists in
  `setup/INSTRUCTOR_TEST_SETUP.ipynb` Cell 15 but the T2 plan / built
  notebook has no pointer to it. Source: pre-course test workflow in
  `plans/test_data_setup.md` (Cell 15) cross-referenced against
  `plans/topic_02_nlp_preprocessing.md` Cell 3 (no test-bucket
  switch comment present today). Originated outside the three
  per-day discrepancy reports as a workflow-integration item.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T2 Cell 3 has no test-bucket switch comment above `S3_BUCKET = "barclays-prompt-eng-data"`.

#### T2-FIX-004 (HIGH IF S3.1 IS FIXED FIRST): Confirm "58 days interest" wording in Cell 6 raw_pdf_noise

- **Plan file**: `plans/topic_02_nlp_preprocessing.md`
- **Line**: 260 (inside `raw_pdf_noise` triple-quoted string)
- **Current**: contains "early repayment charge (ERC) of up to 58 days interest" - this is CORRECT.
- **Action**: NO CHANGE. This is the canonical ground-truth phrasing.
  Listed here so the build agent does NOT touch it when fixing T6.
- **Severity**: NONE (verification only)
- **Justification**: discrepancies_4_5_6.md S3.1 confirms T2 is the
  ground-truth source. T6 must align to T2, not the other way around.
- **NOTEBOOK STATUS (2026-04-25)**: ALREADY FIXED IN NOTEBOOK (verification only)
  Evidence: T2 Cell 6 raw_pdf_noise contains "early repayment charge (ERC) of up to 58 days interest" verbatim. No change needed.

#### T2-FIX-005 (LOW): Gate the Cell 6 getpass

- **Plan file**: `plans/topic_02_nlp_preprocessing.md`
- **Line**: 250
- **Current**:
  ```python
  os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
  client = OpenAI()
  ```
- **Replacement**:
  ```python
  if "OPENAI_API_KEY" not in os.environ:
      os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
  client = OpenAI()
  ```
- **Severity**: LOW
- **Justification**: Avoids a re-prompt for students who ran T1 first.
  Source: discrepancies_1_2_3.md S3.7.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T2 Cell 6 reads `os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")` with no `if "OPENAI_API_KEY" not in os.environ` guard.

#### T2-FIX-006 (MEDIUM, ONLY IF T3-FIX-003 NOT APPLIED): Make T2 chunks cover both PDFs

- **Plan file**: `plans/topic_02_nlp_preprocessing.md`
- **Cell**: Cell 21 (around line 836)
- **Current**: `chunks = chunk_text(cleaned_text, chunk_size=1500, overlap=200)`
  where `cleaned_text` is loan FAQ only.
- **Replacement (one option)**: chunk both the loan FAQ and the credit
  card T&C, concatenate. Adjust line 836 area accordingly.
- **Severity**: MEDIUM
- **Justification**: T3 Cell 22 line 837-838 claims T2's chunks give
  "richer context" but they only contain personal loan content.
  Source: discrepancies_1_2_3.md S3.4. Either fix here OR fix the claim
  in T3 (T3-FIX-003).
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK (or apply T3-FIX-003 instead)
  Evidence: T2's chunks list still derives from `cleaned_text` (loan FAQ only).

### 4.3 Topic 3 - First Chatbot

#### T3-FIX-001 (LOW): Drop dead sagemaker import OR fix comment

- **Plan file**: `plans/topic_03_first_chatbot.md`
- **Lines**: 138-143
- **Current**: `import sagemaker; from sagemaker import get_execution_role; sess = sagemaker.Session(); role = get_execution_role()` plus comment "we use it for role/region; no S3 access needed this topic".
- **Replacement**: delete the block, OR keep it and rewrite the comment
  to "kept for setup-cell parity across topics; T3 makes no AWS calls".
- **Severity**: LOW
- **Justification**: Same as T1-FIX-003. Source: discrepancies_1_2_3.md S1.5.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T3 Cell 3 contains `import sagemaker; from sagemaker import get_execution_role; sess = sagemaker.Session(); role = get_execution_role()` with comment "we use it for role/region; no S3 access needed this topic".

#### T3-FIX-002 (HIGH): Rename T3's expanded PRODUCT_SNIPPET

- **Plan file**: `plans/topic_03_first_chatbot.md`
- **Lines**: 154-164 (variable definition) PLUS Cells 6, 8, 13, 15, 17,
  18, 22 (every consumer).
- **Current**: `PRODUCT_SNIPPET = """..."""` defined as loan + credit card.
- **Replacement**:
  ```python
  BARCLAYS_PRODUCT_SNIPPET = """..."""   # expanded version of T1's loan-only snippet, includes credit card
  ```
  And replace every `PRODUCT_SNIPPET` reference in T3 with
  `BARCLAYS_PRODUCT_SNIPPET`.
  ALSO update T3 line 8 (carryover claims): drop `PRODUCT_SNIPPET` from
  the carryover list.
- **Severity**: HIGH
- **Justification**: Same name as T1's narrower snippet. T3 silently
  overridess T1 when both notebooks share a kernel. T3 Cell 13 demo
  ("late payment fee on credit card?") only works with T3's expanded
  version; if T1's narrower version stays in scope, the demo gives
  "I don't have that information". Source: discrepancies_1_2_3.md S3.1.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T3 Cell 3 still defines the variable as `PRODUCT_SNIPPET` (expanded loan + credit card text). All consumers (Cells 11, 13, 15, 17, 18, 22) reference `PRODUCT_SNIPPET`, not `BARCLAYS_PRODUCT_SNIPPET`.

#### T3-FIX-003 (MEDIUM): Rephrase the chunks "richer context" claim OR remove it

- **Plan file**: `plans/topic_03_first_chatbot.md`
- **Lines**: 837-838 (around Cell 22)
- **Current**: claims "students who completed Topic 2 get richer context automatically".
- **Replacement**:
  ```
  Note: T2's chunks list contains personal loan FAQ text only.
  Credit card questions in Lab 2 may fall through to the system
  prompt's general knowledge if T2's chunks are loaded.
  ```
- **Severity**: MEDIUM
- **Justification**: T2 Cell 21 only chunks the loan FAQ. The "richer
  context" claim is misleading. Source: discrepancies_1_2_3.md S3.4.
  Pair with T2-FIX-006 if you prefer the upstream fix.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T3 Cell 21 still says "Students who ran Topic 2 will have a richer `chunks` list available" (also in Cell 3 comment) and Cell 21 markdown frames T2 chunks as superior context.

#### T3-FIX-004 (HIGH for stretch): Add GPT4O pricing constants to Cell 3

- **Plan file**: `plans/topic_03_first_chatbot.md`
- **Cell**: Cell 3, after the BARCLAYS_PRODUCT_SNIPPET block (around line 165)
- **Add**:
  ```python
  GPT4O_INPUT_PRICE_PER_1K  = 0.0025
  GPT4O_OUTPUT_PRICE_PER_1K = 0.0100
  ```
- **Severity**: HIGH for the stretch path; LOW overall
- **Justification**: T3 Cell 25 stretch lines 1047-1049 reference these
  constants as if they live in scope. T1 defines them in Cell 18 but they
  do not survive into T3 unless the student ran T1 in the same kernel.
  Source: discrepancies_1_2_3.md S3.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T3 Cell 3 does not define `GPT4O_INPUT_PRICE_PER_1K` or `GPT4O_OUTPUT_PRICE_PER_1K`. Cell 25 stretch markdown still references them by name as if available.

#### T3-FIX-005 (LOW for stretch): Resolve the ask_with_cost_tracking carryover claim

- **Plan file**: `plans/topic_03_first_chatbot.md`
- **Line**: 8 (carryover variables list)
- **Current**: lists `ask_with_cost_tracking()` as a Topic 1 carryover.
- **Replacement (Option A)**: drop `ask_with_cost_tracking()` from the
  carryover claim.
- **Replacement (Option B)**: redefine a small cost wrapper inline in T3
  Cell 3 so the stretch lab works in isolation.
- **Severity**: LOW (stretch only)
- **Justification**: T3 never imports or redefines this function.
  A fresh kernel hits NameError. Source: discrepancies_1_2_3.md S3.3.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (carryover claim is plan-side; built notebook does not surface it as an executable carryover list, only stretch references)

#### T3-FIX-006 (LOW): Add MODEL constant for cross-topic continuity

- **Plan file**: `plans/topic_03_first_chatbot.md`
- **Cell**: Cell 3 (around line 150)
- **Add**:
  ```python
  MODEL = "gpt-4o"
  ```
  AND replace inline `model="gpt-4o"` calls in Cells 6, 8, 11, 15 with
  `model=MODEL`.
- **Severity**: LOW
- **Justification**: Same as T1-FIX-004. Source: discrepancies_1_2_3.md S3.5.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T3 Cell 3 has no `MODEL = "gpt-4o"` constant; Cells 13, 15 use inline `model="gpt-4o"`.

### 4.4 Topic 4 - Prompt Engineering

#### T4-FIX-001 (MEDIUM): Drop dead sagemaker import

- **Plan file**: `plans/topic_04_prompt_engineering.md`
- **Lines**: 157-164
- **Current**:
  ```python
  import sagemaker
  from sagemaker import get_execution_role
  sess = sagemaker.Session()
  role = get_execution_role()
  print(f"SageMaker region : {sess.boto_region_name}")
  print(f"Execution role   : ...{role[-30:]}")
  ```
- **Replacement**: delete the block.
- **Severity**: MEDIUM
- **Justification**: T4 makes zero AWS calls. `get_execution_role()`
  raises in any non-SageMaker kernel. Source: discrepancies_4_5_6.md S1.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T4 Cell 3 contains the full sagemaker boilerplate (`import sagemaker; from sagemaker import get_execution_role; sess = sagemaker.Session(); role = get_execution_role(); print(...)`).

#### T4-FIX-002 (LOW): Update line 38 variable-names list

- **Plan file**: `plans/topic_04_prompt_engineering.md`
- **Line**: 38
- **Current**: `Variable names classify_intent, classify_with_schema, refine_prompt are new.`
- **Replacement**: `Variable names classify_intent, classify_with_schema, score_prompt, best_prompt are new.`
- **Severity**: LOW
- **Justification**: `refine_prompt` is never defined in T4. The actual
  iterative section uses `score_prompt(system_prompt, test_battery)` and
  variables `prompt_v0/v1/v2/best_prompt`. Source: discrepancies_4_5_6.md S3.5.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (this is a plan-file documentation line; built notebook does not contain the line 38 variable-names list).

#### T4-FIX-003 (LOW): Fix the false T5 integration claim

- **Plan file**: `plans/topic_04_prompt_engineering.md`
- **Lines**: 29-30
- **Current**: "Coordination with Topic 5: classify_intent() and
  classify_with_schema() are called by name in Topic 5 to route
  memory-augmented conversations to the right handler."
- **Replacement**: "Coordination with Topic 9 (capstone):
  classify_intent() and classify_with_schema() are called by name in
  Topic 9 to route customer queries to the right handler. Topic 5 uses
  these names by reference only."
- **Severity**: LOW
- **Justification**: T5 contains zero calls to either function (one
  prose mention only). T9 does call them. Source: discrepancies_4_5_6.md S3.6.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T4 Cell 0 markdown still says "The classify_intent() and classify_with_schema() functions you build today are called directly by Topic 5 (Conversation Memory) to route queries." This T5 routing claim is false; T9 is the actual consumer.

#### T4-FIX-004 (LOW): Add MODEL constant

- **Plan file**: `plans/topic_04_prompt_engineering.md`
- **Cell**: Cell 3
- **Add**: `MODEL = "gpt-4o"`
- **Action**: replace inline `model="gpt-4o"` literals at lines 358,
  408, 619, 707, etc. with `model=MODEL`.
- **Severity**: LOW
- **Justification**: Same as T1-FIX-004 / T3-FIX-006.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T4 Cell 3 does not define `MODEL`; later cells use inline `model="gpt-4o"` literals.

#### T4-FIX-005 (MEDIUM): Align openai pin to 2.x family

- **Plan file**: `plans/topic_04_prompt_engineering.md`
- **Line**: 140 (install) and line 34 (rationale)
- **Current**: `"openai==1.51.0"` plus rationale "structured outputs require this".
- **Replacement (recommended)**: `"openai>=2.30.0,<3"` (matches T7-T9).
  Drop the 1.51.0 commentary on line 34. Structured outputs work fine
  on openai 2.x via the chat completions API; the Responses API is the
  newer path.
- **Severity**: MEDIUM
- **Justification**: Mid-day SDK swap (T1/T3 install 2.32.0; T4 forces
  1.51.0; T5/T6 install >=1.30.0; T7-T9 install >=2.30.0) creates pip
  resolver churn and risks import-cache issues. Source: discrepancies_4_5_6.md S4.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T4 Cell 2 reads `!pip install -q "openai==1.51.0" "numpy<2"` with the rationale comment "openai 1.51.0: first version with fully stable json_schema strict mode".

#### T4-FIX-006 (LOW): Add emoji to title

- **Plan file**: `plans/topic_04_prompt_engineering.md`
- **Line**: 79 (Cell 0 title)
- **Current**: `# Prompt Engineering - Day 2 Opener`
- **Replacement**: add a leading emoji matching course style (T5 / T6
  use one). Suggestion: `# 🎯 Prompt Engineering - Day 2 Opener` (or
  whatever the visual style of T5/T6 uses).
- **Severity**: LOW
- **Justification**: T4 verification checklist requires the emoji; T5
  and T6 have one. Source: discrepancies_4_5_6.md S5.5.
- **NOTEBOOK STATUS (2026-04-25)**: ALREADY FIXED IN NOTEBOOK
  Evidence: T4 Cell 0 title is `# 🎯 Prompt Engineering - Day 2 Opener` (emoji present).

### 4.5 Topic 5 - Conversation Memory

#### T5-FIX-001 (HIGH): Quote install line specs

- **Plan file**: `plans/topic_05_conversation_memory.md`
- **Line**: 127
- **Current**:
  ```
  !pip install -q openai>=1.30.0 tiktoken==0.9.0 "numpy<2"
  ```
- **Replacement**:
  ```
  !pip install -q "openai>=1.30.0" "tiktoken==0.9.0" "numpy<2"
  ```
- **Severity**: HIGH for re-builds (current built notebook already has
  the quotes; the plan does not).
- **Justification**: Bash/zsh parses `>=1.30.0` as a stdout redirect
  to a file literally named `=1.30.0`. The `pip install` command then
  installs `openai` UNPINNED and creates the spurious file. Source:
  discrepancies_4_5_6.md S4.1.
- **NOTEBOOK STATUS (2026-04-25)**: ALREADY FIXED IN NOTEBOOK
  Evidence: T5 Cell 2 reads `!pip install -q "openai>=1.30.0" "tiktoken==0.9.0" "numpy<2"` (all specs quoted). Plan-side fix may still be needed if rebuild is planned.

#### T5-FIX-002 (MEDIUM): Drop the false MODEL carryover claim

- **Plan file**: `plans/topic_05_conversation_memory.md`
- **Lines**: 24, 985 (continuity choices and verification checklist)
- **Current**: claim that `MODEL` carries from Topic 4.
- **Replacement**: "client, OPENAI_API_KEY carry over from Topic 4;
  MODEL is defined fresh in this topic for the first time."
- **Severity**: MEDIUM
- **Justification**: T4 does not define a top-level MODEL constant.
  T5 line 151 already defines it locally; only the continuity claim is
  wrong. Source: discrepancies_4_5_6.md S3.3.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (carryover claim is plan-side; built T5 notebook does not contain an executable carryover claim block).

#### T5-FIX-003 (MEDIUM): Reuse T3's BARCLAYS_SYSTEM_PROMPT verbatim

- **Plan file**: `plans/topic_05_conversation_memory.md`
- **Lines**: 154-160
- **Current**: 50-word redefinition with comment "this is the same
  persona we built in Topic 3".
- **Replacement**: copy T3 lines 331-355 verbatim. Update line 154
  comment to: "Reusing the T3 system prompt verbatim so the persona
  stays stable across topics."
- **Severity**: MEDIUM
- **Justification**: T3's prompt has explicit Persona, Constraints,
  Format sections and refusal patterns. T5's redefinition silently drops
  all three. T5's refusal demos pass only by accident. Source:
  discrepancies_4_5_6.md S3.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T5 Cell 3 defines a 4-sentence concatenated string ("You are a Barclays Product Knowledge Assistant. You help customer service agents... Be factual, concise, and never give personalised financial advice."). T3 Cell 8 defines a triple-quoted multi-paragraph version with explicit Persona / Constraints / Format / Refusal sections. They are not the same string.

#### T5-FIX-004 (LOW): Fix the classify_intent return-type claim

- **Plan file**: `plans/topic_05_conversation_memory.md`
- **Line**: 9
- **Current**: "A working classify_intent(text) function returning a
  JSON-schema structured output."
- **Replacement**: "A working classify_intent(text) returning a string
  category and classify_with_schema(text) returning a structured dict."
- **Severity**: LOW (documentation only)
- **Justification**: T4 line 352 defines `classify_intent` returning a
  plain string. The structured-output function is `classify_with_schema`.
  Source: discrepancies_4_5_6.md S3.4.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (return-type claim is in plan markdown only; built T5 notebook does not contain a Cell 0 line stating the schema return type).

### 4.6 Topic 6 - RAG Foundations

#### T6-FIX-001 (MEDIUM): Drop dead sagemaker import

- **Plan file**: `plans/topic_06_rag_foundations.md`
- **Lines**: 147-152
- **Current**: `import sagemaker; from sagemaker import get_execution_role; ...`
- **Replacement**: delete the block.
- **Severity**: MEDIUM
- **Justification**: Same as T4-FIX-001. T6 makes zero AWS calls.
  Source: discrepancies_4_5_6.md S1.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T6 Cell 3 still contains `import sagemaker; from sagemaker import get_execution_role; sess = sagemaker.Session(); role = get_execution_role(); print(...)` with comment "we keep this around for the optional S3 stretch lab at the end".

#### T6-FIX-002 (HIGH): Fix BARCLAYS_DOCS[0] ERC value

- **Plan file**: `plans/topic_06_rag_foundations.md`
- **Line**: 213 (inside BARCLAYS_DOCS[0])
- **Current**: `"... No arrangement fee. Early repayment allowed with 30-day interest."`
- **Replacement**: `"... No arrangement fee. Early repayment allowed with an early repayment charge of up to 58 days interest."`
- **Severity**: HIGH
- **Justification**: Two upstream sources both encode 58 days as the
  correct value:
  - `plans/topic_02_nlp_preprocessing.md` line 260 raw_pdf_noise demo
    uses "58 days interest"
  - `plans/test_data_setup.md` lines 305-307 synthetic PDF generator
    uses "58 days interest" AND its docstring (line 281) explicitly
    says "Match BARCLAYS_DOCS[0] in Topic 6 (APR 6.5%, 1-35K GBP, 1-5
    years, 58 days ERC)"
  T6 is the only file that says 30 days. This is the highest-leverage
  one-token fix in the entire course. Source: discrepancies_4_5_6.md S3.1.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T6 Cell 5 BARCLAYS_DOCS[0] reads "Barclays Personal Loan: ... No arrangement fee. Early repayment allowed with 30-day interest." (still says 30-day, not 58 days).

#### T6-FIX-003 (LOW): Add APR to BARCLAYS_DOCS[1]

- **Plan file**: `plans/topic_06_rag_foundations.md`
- **Line**: 214 (inside BARCLAYS_DOCS[1], the Rewards Card)
- **Current**: no APR mentioned for the Rewards Card.
- **Replacement**: insert "Representative 27.9% APR variable." into the
  string.
- **Severity**: LOW (asymmetric content rather than contradiction)
- **Justification**: Synthetic Credit Card PDF includes "27.9 percent
  APR variable" and Topic 2 raw_pdf_noise references "27.9% p.a.
  variable". Adding the APR to T6 makes the RAG retrieval demo more
  realistic. Source: discrepancies_4_5_6.md S5.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T6 Cell 5 BARCLAYS_DOCS[1] (Rewards Credit Card) does not include "27.9% APR" or any APR value in the doc string.

#### T6-FIX-004 (LOW, OPTIONAL): Add Tier 3 stretch hint pointing to barclays_chunks.json

- **Plan file**: `plans/topic_06_rag_foundations.md`
- **Cell**: Tier 3 stretch lab brief
- **Add**: one paragraph: "Stretch data: load
  `s3://barclays-prompt-eng-data/barclays_chunks.json` if you want to
  test against the real Barclays product text. See
  `setup/INSTRUCTOR_SETUP.ipynb` Cell 10 for the load snippet."
- **Severity**: LOW
- **Justification**: `setup/INSTRUCTOR_SETUP.ipynb` already uploads this
  file but no topic plan currently points students at it. Source:
  discrepancies_4_5_6.md S1.4.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T6 stretch lab brief does not reference `barclays_chunks.json` or `setup/INSTRUCTOR_SETUP.ipynb` Cell 10.

### 4.7 Topic 7 - Advanced RAG + Web Search

#### T7-FIX-001 (HIGH): Replace BARCLAYS_DOCS with T6's canonical list

- **Plan file**: `plans/topic_07_advanced_rag_web_search.md`
- **Lines**: 253-273 (the 6-doc list)
- **Current**: 6 different products with different rates than T6.
- **Replacement**: copy T6 lines 212-219 (7 docs) verbatim. Drop T7's
  invented products (Premier Current Account, Rainy Day Saver,
  Barclaycard Platinum Cashback Plus, Travel Wallet) UNLESS T7 demo
  cells specifically reference them.
- **Severity**: HIGH
- **Justification**: T7 line 83 explicitly claims "Same inline
  BARCLAYS_DOCS list from Topic 6". Currently false. The setup
  notebooks were aligned to T6's list, not T7's. Source:
  discrepancies_7_8_9.md S3.1.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T7 Cell 5 BARCLAYS_DOCS contains different products from T6 (Premier Current Account, Rainy Day Saver, etc. with comment "Topic 6 carryover: the same Barclays product corpus" - which is false). T6 Cell 5 has a 7-doc list starting with the personal loan; T7 Cell 5 has a different invented list with different APRs and product names.

#### T7-FIX-002 (LOW, OPTIONAL): Add stretch hint pointing to barclays_chunks.json

- **Plan file**: `plans/topic_07_advanced_rag_web_search.md`
- **Cell**: Tier 2 hard lab brief (around Cell 25)
- **Add**: one paragraph pointing at `barclays_chunks.json` as in T6-FIX-004.
- **Severity**: LOW
- **Justification**: Source: discrepancies_7_8_9.md S1.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T7 Tier 2 hard lab brief does not reference `barclays_chunks.json`.

### 4.8 Topic 8 - Ethical Guardrails

#### T8-FIX-001 (MEDIUM): Fix the carryover claims block

- **Plan file**: `plans/topic_08_ethical_guardrails.md`
- **Lines**: 7-9
- **Current**: claims that
  - T3 defines `MODEL = "gpt-4o"` (false; only T5 does)
  - T3's `create_chatbot` is `(system_prompt, model=MODEL)` (missing the `context=""` param)
  - T4 "refines BARCLAYS_SYSTEM_PROMPT" (false; T4 never touches it)
  - T5 BarclaysChat has `self.client` attribute (false; T5 uses module-global client)
- **Replacement**: rewrite to:
  - "From T3: `BARCLAYS_SYSTEM_PROMPT`,
    `create_chatbot(system_prompt, context='', model='gpt-4o')`."
  - "From T4: `classify_intent(text) -> str`,
    `classify_with_schema(text) -> dict`."
  - "From T5: `MODEL = 'gpt-4o'`,
    `BarclaysChat(system_prompt)` with attributes `system_prompt`,
    `history` (uses module-global `client`)."
- **Severity**: MEDIUM
- **Justification**: Source: discrepancies_4_5_6.md S3.8.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (the carryover claims block exists in the plan file lines 7-9; the built T8 notebook Cell 0 is a free-form intro markdown that does not enumerate the false `MODEL`/`self.client` claims line by line).

#### T8-FIX-002 (LOW): Switch %pip to !pip and align openai pin

- **Plan file**: `plans/topic_08_ethical_guardrails.md`
- **Line**: 80
- **Current**: `%pip install --quiet "openai==2.32.0" "numpy<2"`
- **Replacement**: `!pip install -q "openai>=2.30.0" "numpy<2"`
- **Severity**: LOW
- **Justification**: T1-T7 and T9 all use `!pip install -q`. T8 is the
  only `%pip` user. The version pin alignment also resolves the
  inconsistency in Section 3.4. Source: discrepancies_7_8_9.md S4.1, S4.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T8 Cell 1 reads `%pip install --quiet "openai==2.32.0" "numpy<2"` (uses `%pip` and pins `==2.32.0`, while T9 uses `>=2.30.0`).

#### T8-FIX-003 (LOW): Fix line 10 misreport of T6's openai pin

- **Plan file**: `plans/topic_08_ethical_guardrails.md`
- **Line**: 10
- **Current**: "Pins chromadb==1.5.8, openai>=1.30.0, numpy<2."
- **Replacement**: "Pins chromadb==1.5.8, openai>=2.30.0, numpy<2."
- **Severity**: LOW
- **Justification**: T6 plan is the source of truth; after T4-FIX-005
  / T6 will use 2.x. Source: discrepancies_7_8_9.md S4.3.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (this is a plan-file documentation line; built notebook does not surface this carryover statement).

#### T8-FIX-004 (LOW): Trim the misleading T6 dependency claim

- **Plan file**: `plans/topic_08_ethical_guardrails.md`
- **Line**: 10 (the "What this topic builds on" block)
- **Current**: lists T6's `embed_texts()`, `chromadb.PersistentClient`,
  `collection`, `retrieve()`, `rag_answer()`, `chunks` as dependencies.
- **Replacement**: "T6 (RAG foundations): no direct dependency in this
  notebook, but the guardrails defined here are designed to wrap the
  rag_answer() helper when integrated in T9."
- **Severity**: LOW
- **Justification**: T8 install line does not include chromadb. T8
  Cell 22 uses `_toy_chat` calling `client.chat.completions.create`
  directly with no RAG. Source: discrepancies_7_8_9.md S3.5.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (this is a plan-file dependency block; built notebook does not contain a "What this topic builds on" enumeration listing T6 names).

### 4.9 Topic 9 - Capstone

#### S2-FIX-001 (HIGH BLOCKER): Resolve capstone scope conflict

- **Files affected**: `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` and/or
  `plans/topic_09_capstone.md` and/or `plans/TOPICS.md`
- **Conflict summary**:
  - CORE lines 25-34, 84 say T9 is the "Barclays Transaction Query
    Agent" using Banking77 + tool-calling + built INDEPENDENTLY.
  - T9 plan lines 1, 8, 126 deliver "Production Customer Service
    Assistant" that polishes/integrates T1-T8.
  - TOPICS.md agrees with the T9 plan.
- **Three remediation options** (user must pick one):
  - **Option A**: Treat CORE as authoritative. Rewrite T9 plan to be a
    new notebook that imports nothing from prior topics, loads
    Banking77 via `from datasets import load_dataset; ds = load_dataset("PolyAI/banking77")`,
    defines 3 tools (`lookup_policy`, `lookup_transaction`,
    `escalate_to_human`), uses gpt-4o function calling, runs on 5
    Banking77 intents. Add `"datasets>=2.20.0"` to T9 install line.
  - **Option B**: Treat TOPICS.md / current T9 plan as authoritative.
    Edit CORE lines 27-34 and line 84 to drop Banking77 and replace the
    capstone description with the integrate-and-harden version.
  - **Option C** (hybrid): Keep current T9 plan as MAIN capstone; add a
    short optional `week_09_optional_transaction_agent.ipynb` that
    delivers the Banking77 tool-augmented agent as a take-home for
    advanced students. Preserves both intentions.
- **Severity**: HIGH BLOCKER
- **Justification**: Without this decision, `/build-topic-notebook 9`
  cannot start. Source: discrepancies_7_8_9.md S2.1.
- **NOTEBOOK STATUS (2026-04-25)**: NEEDS USER DECISION (de facto Option B taken: T9 was built as "Production Customer Service Assistant" integrating T1-T8, no Banking77, no `datasets` dependency). CORE manifest still says "Transaction Query Agent" using Banking77; this manifest divergence remains unresolved.

#### T9-FIX-001 (HIGH): Rename ChromaDB collection to barclays_products

- **Plan file**: `plans/topic_09_capstone.md`
- **Line**: 500
- **Current**: `name="barclays_capstone"`
- **Replacement**: `name="barclays_products"`
  Add a comment: "Same name as T6/T7 so the existing local collection
  is reused if present; the get_or_create_collection call is idempotent."
- **Severity**: HIGH
- **Justification**: T6 and T7 use `barclays_products`. T9 ignores the
  populated collection and creates an empty one, wasting embedding cost
  and leaving two divergent collections on the EBS. Source:
  discrepancies_7_8_9.md S3.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 reads `collection = chroma_client.get_or_create_collection(name="barclays_capstone", configuration={"hnsw": {"space": "cosine"}})`. T6/T7 use `barclays_products`.

#### T9-FIX-002 (HIGH): Replace BARCLAYS_DOCS with T6's canonical list

- **Plan file**: `plans/topic_09_capstone.md`
- **Lines**: 332-348
- **Current**: 5-doc invented list (Personal Loan, Barclaycard Platinum,
  card freeze FAQ, Everyday Saver, Money Worries team).
- **Replacement**: copy T6 lines 212-219 (7 docs) verbatim. If T9 needs
  the operational docs (card freeze FAQ, Money Worries team) for its
  5-query battery in Cell 23, ADD them at indices 7-8 with comment:
  "Indices 0-6 are the T6 carryover; indices 7-8 are capstone-specific
  operational docs."
- **Severity**: HIGH
- **Justification**: T9 line 28 claims "Same inline BARCLAYS_DOCS list
  from Topic 6". Currently false. T9 Cell 23 Query 1 ("2-year fixed
  mortgage rate?") relies on web_search recovery because T9's BARCLAYS_DOCS
  has no mortgage doc. Source: discrepancies_7_8_9.md S3.1.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 BARCLAYS_DOCS contains a different invented 5-doc list (Personal Loan with 1000-50000 GBP, Barclaycard Platinum, card freeze FAQ, ...) that does not match T6's 7-doc list (T6 uses 1000-35000 GBP loan, Rewards Credit Card, etc.).

#### T9-FIX-003 (HIGH IF Option A) OR T9-FIX-003 (LOW): Add datasets package

- **Plan file**: `plans/topic_09_capstone.md`
- **Line**: 249
- **Current**: `!pip install -q "openai>=2.30.0" "tenacity==9.1.4" "tiktoken==0.9.0" "chromadb==1.5.8" "numpy<2"`
- **Replacement (only if S2-FIX-001 picks Option A or C)**:
  ```
  !pip install -q "openai>=2.30.0" "tenacity==9.1.4" "tiktoken==0.9.0" "chromadb==1.5.8" "datasets>=2.20.0" "numpy<2"
  ```
- **Severity**: HIGH if Option A or C; NONE if Option B
- **Justification**: Banking77 loads via `datasets.load_dataset("PolyAI/banking77")`.
  Source: discrepancies_7_8_9.md S2.2.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (Option B was effectively chosen during the build; T9 Cell 3 install line `!pip install -q "openai>=2.30.0" "tenacity==9.1.4" "tiktoken==0.9.0" "chromadb==1.5.8" "numpy<2"` has no `datasets` and no Banking77 usage in the notebook).

#### T9-FIX-004 to T9-FIX-006: Cell 5 surface contract drift versus T4/T5

#### T9-FIX-004 (MEDIUM): Make Cell 5 BARCLAYS_SYSTEM_PROMPT match T3 verbatim

- **Plan file**: `plans/topic_09_capstone.md`
- **Lines**: 351-357
- **Replacement**: copy T3 lines 331-355 verbatim. Same change as T5-FIX-003.
- **Severity**: MEDIUM
- **Justification**: Source: discrepancies_4_5_6.md S3.2,
  discrepancies_7_8_9.md (cross-reference).
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 redefines `BARCLAYS_SYSTEM_PROMPT` with its own short version (paired with the "compact reference implementation" reimport pattern). It is not a verbatim copy of T3 Cell 8 multi-paragraph version.

#### T9-FIX-005 (MEDIUM): Drop the false MODEL carryover claim from T1

- **Plan file**: `plans/topic_09_capstone.md`
- **Line**: 41
- **Current**: lists `MODEL` as a Topic 1 carryover.
- **Replacement**: drop `MODEL` from the T1 carryover list. State that
  T9 uses the T5-defined `MODEL = "gpt-4o"` (or, after T1-FIX-004 / T3-
  FIX-006 / T4-FIX-004, the T1-defined MODEL).
- **Severity**: MEDIUM
- **Justification**: T1 does not define MODEL until T1-FIX-004 is
  applied. Source: discrepancies_4_5_6.md S3.3.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (the T1 carryover list is in plan-file metadata; T9 Cell 6 lists expected names but does not assert origin topic incorrectly).

#### T9-FIX-006 (LOW): Drop the false MODEL claim ALSO if it appears elsewhere

- **Plan file**: `plans/topic_09_capstone.md`
- **Line**: any other carryover lists that mention MODEL
- **Action**: same as T9-FIX-005, applied to every carryover claim.
- **Severity**: LOW
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (no other false MODEL carryover claims surface in the built T9 notebook beyond the plan-side metadata).

#### T9-FIX-007 (MEDIUM): Make Cell 5 classify_with_schema match T4

- **Plan file**: `plans/topic_09_capstone.md`
- **Lines**: 410-418 (inline reimpl)
- **Current**: 2-field return `{intent, confidence (number)}`.
- **Replacement**: 3-field return matching T4 lines 588-608:
  `{intent, confidence (string enum: high|medium|low), rationale (one-sentence)}`.
  Replace the inline reimpl with a verbatim copy of T4's definition
  (CLASSIFICATION_SCHEMA + classify_with_schema).
- **Severity**: MEDIUM
- **Justification**: T9 line 19 promises "imports each prior name by
  re-executing a compact reference implementation". The current reimpls
  are NEW implementations with different contracts, breaking the
  capstone integration premise. Source: discrepancies_7_8_9.md S3.3,
  discrepancies_4_5_6.md S3.7.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 still has the inline `classify_with_schema` reimpl that does not match T4's 3-field schema (intent / confidence string-enum / rationale).

#### T9-FIX-008 (MEDIUM): Make Cell 5 count_tokens_in_messages match T5

- **Plan file**: `plans/topic_09_capstone.md`
- **Lines**: 438-447 (inline reimpl)
- **Current**: 4 tokens per message + 2 priming tokens.
- **Replacement**: 3 tokens per message + 3 priming tokens (T5 lines
  531-572). Matches OpenAI Cookbook formula for gpt-4o (o200k_base).
- **Severity**: MEDIUM
- **Justification**: T9's formula is off by 1 in both dimensions.
  Source: discrepancies_7_8_9.md S3.3, discrepancies_4_5_6.md S5.4.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 still contains the inline `count_tokens_in_messages` reimpl with the 4+2 formula that diverges from T5's 3+3 OpenAI Cookbook formula.

#### T9-FIX-009 (MEDIUM): Make Cell 5 BarclaysChat match T5

- **Plan file**: `plans/topic_09_capstone.md`
- **Line**: 474
- **Current**: `__init__(self, system_prompt: str = BARCLAYS_SYSTEM_PROMPT, max_tokens: int = 3000)`
- **Replacement**: `__init__(self, system_prompt)` matching T5 lines 457-460.
  Drop `max_tokens` parameter (or move to a separate config). Use module-
  global `client` (T5 pattern) not `self.client`.
- **Severity**: MEDIUM
- **Justification**: Source: discrepancies_4_5_6.md S3.7.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 still defines `BarclaysChat` with the extra `max_tokens` param and diverges from T5's `__init__(self, system_prompt)` signature.

#### T9-FIX-010 (HIGH): Make detect_pii match T8 GuardrailResult

- **Plan file**: `plans/topic_09_capstone.md`
- **Lines**: ~599+ (inline reimpl) PLUS Cell 22 line ~1407 (consumer)
  PLUS Cell 23 (assertions)
- **Current**: `detect_pii(text: str) -> list` returning list of dicts;
  Cell 22 consumes the list directly.
- **Replacement (recommended Option 1, preserve T8)**: copy T8 line 152
  / 165-166 GuardrailResult and copy T8's detect_pii verbatim. Update
  Cell 22 to consume `.passed`, `.redacted_text`, `.details`.
- **Severity**: HIGH
- **Justification**: A student running T8 -> T9 in the same kernel sees
  detect_pii silently overwritten with a different return type. T9 in
  isolation works only because the reimpl shadows T8. Source:
  discrepancies_7_8_9.md S3.3.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 defines `def detect_pii(text: str) -> list:` returning a list of `{type, match}` dicts. T8 returns a `GuardrailResult` dataclass with `.passed`, `.redacted_text`, `.details` attributes.

#### T9-FIX-011 (HIGH): Make should_escalate match T8 GuardrailResult

- **Plan file**: `plans/topic_09_capstone.md`
- **Lines**: 640 (inline reimpl) PLUS line 1409 (consumer)
- **Current**: `def should_escalate(text: str) -> tuple` consumed as
  `should_esc, esc_reason = should_escalate(user_message)`.
- **Replacement**: copy T8 line 18 signature
  `should_escalate(query: str, history: list) -> GuardrailResult`. Update
  Cell 22 line 1409 to:
  ```python
  esc_result = should_escalate(user_message, session.messages)
  if not esc_result.passed:
      reason = esc_result.details.get("reason", "vulnerability_indicator")
      ...
  ```
- **Severity**: HIGH
- **Justification**: Same as T9-FIX-010. Source: discrepancies_7_8_9.md S3.4.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 5 defines `def should_escalate(text: str) -> tuple:` returning `(True, f"vulnerability indicator: {marker}")`. T8 signature is `should_escalate(query: str, history: list) -> GuardrailResult`.

#### T9-FIX-012 (LOW): Add web_search per-call cost note

- **Plan file**: `plans/topic_09_capstone.md`
- **Cell**: Cell 15 (cost calculator demo) OR Cell 25 (wrap-up)
- **Add**:
  ```
  This calculator covers per-token cost only. The OpenAI hosted
  web_search tool adds a per-call fee on top (consult the current
  OpenAI pricing page). When route_query returns 'hybrid' or 'web',
  add the per-call web_search fee to your log_record manually.
  ```
- **Severity**: LOW
- **Justification**: Cost log under-reports any hybrid/web query.
  Source: discrepancies_7_8_9.md S5.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Cell 4 defines pricing constants (GPT4O_INPUT_PRICE_PER_1K, GPT4O_OUTPUT_PRICE_PER_1K) but no per-call web_search fee note appears in the cost calculator demo or wrap-up cells.

#### T9-FIX-013 (LOW, OPTIONAL): Add Tier 3 stretch hint pointing to barclays_chunks.json

- **Plan file**: `plans/topic_09_capstone.md`
- **Cell**: Tier 3 open-ended lab brief (Cell 24)
- **Add**: same hint as T6-FIX-004 / T7-FIX-002.
- **Severity**: LOW
- **Justification**: Source: discrepancies_7_8_9.md S1.2.
- **NOTEBOOK STATUS (2026-04-25)**: STILL NEEDS FIX IN NOTEBOOK
  Evidence: T9 Tier 3 lab brief does not reference `barclays_chunks.json`.

### 4.10 TOPICS.md

#### TOPICS-FIX-001 (LOW): Remove all Anthropic references

- **Plan file**: `plans/TOPICS.md`
- **Lines**: 25, 29, 35, 39, 110, 117 (per discrepancies_1_2_3.md S2.1)
- **Action**: replace every Anthropic mention with OpenAI-only equivalents.
  T1 plan and T3 plan correctly use OpenAI only (no Anthropic anywhere).
- **Severity**: LOW
- **Justification**: CORE line 46 and line 76 forbid Anthropic in
  student notebooks. TOPICS.md is the only file that still references
  it. Source: discrepancies_1_2_3.md S2.1.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (TOPICS.md is plan metadata, not a notebook).

#### TOPICS-FIX-002 (LOW): Drop the (fitz) parenthetical from T2 library list

- **Plan file**: `plans/TOPICS.md`
- **Line**: 81
- **Current**: `pymupdf (fitz), beautifulsoup4, re, textwrap`
- **Replacement**: `pymupdf, beautifulsoup4, re, textwrap`
- **Severity**: LOW
- **Justification**: T2 plan uses `import pymupdf` (the current
  recommended import). The legacy `import fitz` alias still works but
  is deprecated. Source: discrepancies_1_2_3.md S2.2.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (TOPICS.md is plan metadata, not a notebook).

### 4.11 setup/ notebooks

#### TEST-SETUP-FIX-001 (LOW): Add numpy<2 to install line

- **Plan file**: `plans/test_data_setup.md`
- **Line**: 135
- **Current**: `!pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3`
- **Replacement**: `!pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3 "numpy<2"`
- **Built notebook**: `setup/INSTRUCTOR_TEST_SETUP.ipynb` Cell 2 (would
  also need this if you want to apply the fix)
- **Severity**: LOW
- **Justification**: pymupdf 1.27.2.2 does not declare strict numpy<2.
  CORE line 80 mandates the pin. Source: discrepancies_1_2_3.md S4.5.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (this is a setup notebook fix, outside the 9 topic notebooks scope; not re-verified in this round).

#### TEST-SETUP-FIX-002 (LOW, OPTIONAL): Add 4 more synthetic PDFs to mirror full T6 BARCLAYS_DOCS

- **Plan file**: `plans/test_data_setup.md`
- **New cells**: 4 additional generators (`make_mortgage`,
  `make_student_account`, `make_business_account`, `make_travel_pack`)
  matching T6 docs 3-6.
- **Severity**: LOW
- **Justification**: Currently the test bucket only mirrors T6 docs 0-2.
  Doing all 7 would let students test T6/T7/T9 stretch labs against the
  test bucket without loss of coverage. Source: discrepancies_4_5_6.md S5.1.
- **NOTEBOOK STATUS (2026-04-25)**: NOT APPLICABLE (this is a setup notebook fix, outside the 9 topic notebooks scope).

#### INSTRUCTOR-SETUP-FIX-NONE

`plans/instructor_setup.md` and `setup/INSTRUCTOR_SETUP.ipynb` need NO
changes. The PDF_MANIFEST already covers the 2 required PDFs (T2) plus
the 2 optional ones (T6 stretch). The Cell 12 topic readiness summary
is correct for all 9 topics.

---

## 5. Recommended Global Remediation Order

This is the order in which to apply the fixes. It minimises the
number of times any one plan file is opened and groups by what unblocks
what.

### 5.1 Phase 1 - Unblock the build pipeline (HIGH severity, do first)

1. **S2-FIX-001** - User decision on T9 capstone scope. Until this is
   resolved, T9 cannot be built. Three options (A/B/C) documented.
2. **T2-FIX-001** - Add `"openai==2.32.0"` to T2 Cell 2 install line.
   One-line fix; unblocks T2 self-contained execution.
3. **T5-FIX-001** - Quote `"openai>=1.30.0"` in T5 Cell 2 plan install
   line. Two-character fix; unblocks future T5 re-build.
4. **T6-FIX-002** - One-token fix to T6 BARCLAYS_DOCS[0]: replace
   "30-day interest" with "an early repayment charge of up to 58 days
   interest". Unblocks T6/T7/T9 RAG demo consistency with synthetic PDFs.
5. **T9-FIX-001** - One-character fix to T9 ChromaDB collection name:
   `barclays_capstone` -> `barclays_products`.
6. **T7-FIX-001 + T9-FIX-002** - Replace BARCLAYS_DOCS in T7 Cell 5 and
   T9 Cell 5 with verbatim copies of T6 lines 212-219.
7. **T9-FIX-010 + T9-FIX-011** - Make T9 inline `detect_pii` and
   `should_escalate` match T8's GuardrailResult contract. Update T9
   Cell 22 consumers.

### 5.2 Phase 2 - Continuity correctness (MEDIUM severity)

8. **T3-FIX-002** - Rename T3's expanded snippet to
   `BARCLAYS_PRODUCT_SNIPPET`. Update Cells 6, 8, 13, 15, 17, 18, 22.
9. **T5-FIX-003 + T9-FIX-004** - Make T5 and T9 `BARCLAYS_SYSTEM_PROMPT`
   verbatim copies of T3's canonical version.
10. **T2-FIX-002** - Rewrite T2 Cell 3 misleading "no OpenAI key needed"
    comment.
11. **T4-FIX-001 + T6-FIX-001** - Drop dead sagemaker imports from T4
    Cell 3 and T6 Cell 3.
12. **T8-FIX-001** - Fix T8 carryover claims block (lines 7-9).
13. **T5-FIX-002 + T9-FIX-005** - Drop false MODEL carryover claims.
14. **T9-FIX-007 + T9-FIX-008 + T9-FIX-009** - Make T9 inline reimpls
    of `classify_with_schema`, `count_tokens_in_messages`, and
    `BarclaysChat` byte-for-byte copies of T4 / T5 originals.
15. **T3-FIX-004 + T3-FIX-005** - Make T3 stretch lab self-contained
    by adding GPT4O pricing constants to Cell 3.

### 5.3 Phase 3 - Quality of life (LOW severity)

16. **T1-FIX-001 + T1-FIX-002** - Quote T1 install line; drop redundant
    `import openai`.
17. **T1-FIX-003 + T3-FIX-001** - Drop dead sagemaker imports from T1
    and T3 (matches T4-FIX-001 + T6-FIX-001 pattern).
18. **T1-FIX-004 + T3-FIX-006 + T4-FIX-004** - Add `MODEL = "gpt-4o"`
    constant to T1, T3, T4. Replace inline `model="gpt-4o"` literals.
19. **T2-FIX-003** - Add test-bucket switch comment to T2 Cell 3 above
    `S3_BUCKET`.
20. **T2-FIX-005** - Gate the T2 Cell 6 getpass with `if "OPENAI_API_KEY" not in os.environ`.
21. **T2-FIX-006 OR T3-FIX-003** - Pick one: chunk both PDFs in T2, OR
    rephrase T3's "richer context" claim.
22. **T4-FIX-002 + T4-FIX-003** - Fix T4 documentation: replace
    refine_prompt with the actual names; drop the false T5 integration claim.
23. **T4-FIX-005** - Align openai pin family across T4/T5/T6 with T7-T9.
24. **T4-FIX-006** - Add emoji to T4 title.
25. **T5-FIX-004** - Fix classify_intent return-type claim in T5 line 9.
26. **T6-FIX-003** - Add 27.9% APR to T6 BARCLAYS_DOCS[1].
27. **T8-FIX-002 + T8-FIX-003 + T8-FIX-004** - T8 cosmetic install /
    documentation fixes.
28. **T9-FIX-012 + T9-FIX-013** - T9 web_search cost note + Tier 3 stretch hint.
29. **T6-FIX-004 + T7-FIX-002** - Optional Tier 3 / Tier 2 stretch
    hints pointing at `barclays_chunks.json`.
30. **TOPICS-FIX-001 + TOPICS-FIX-002** - Clean up TOPICS.md.
31. **TEST-SETUP-FIX-001** - Add `numpy<2` to test_data_setup install line.
32. **TEST-SETUP-FIX-002** - Add 4 more synthetic PDFs (optional, only
    if you want full T6 BARCLAYS_DOCS coverage in the test bucket).

### 5.4 What NOT to fix

- `setup/INSTRUCTOR_SETUP.ipynb`: no changes needed. Already correct.
- `plans/instructor_setup.md`: no changes needed. Already correct.
- T2 Cell 6 raw_pdf_noise (line 260): do NOT change "58 days interest"
  to anything else; this is the canonical ground truth.
- T2 Cell 8 fallback string: keep as-is (could be expanded for richer
  fallback per discrepancies_1_2_3.md S5.3 but is not a defect).
- T6 ChromaDB collection name `barclays_products`: correct, leave alone.
- T5 `count_tokens_in_messages` formula: T5 is correct (3+3); T9 is wrong (4+2).
- T8 `GuardrailResult` dataclass: correct teaching asset; keep it; T9
  must adapt to it (T9-FIX-010, T9-FIX-011), not the other way around.

---

## 6. Things Confirmed Correct (Do Not Touch)

This section lists every contract that the audit confirmed is right.
Build agents should treat these as load-bearing.

### 6.1 AWS / S3 contracts

- C-AWS-1: `S3_BUCKET = "barclays-prompt-eng-data"` (T2 Cell 3) matches
  `setup/INSTRUCTOR_SETUP.ipynb` Cell 2 byte-for-byte.
- C-AWS-2: `REGION = "us-east-2"` matches across T2, INSTRUCTOR_SETUP,
  INSTRUCTOR_TEST_SETUP byte-for-byte.
- C-AWS-3: S3 keys `barclays_personal_loan_faq.pdf` and
  `barclays_credit_card_tnc.pdf` match across T2, INSTRUCTOR_SETUP
  PDF_MANIFEST, INSTRUCTOR_TEST_SETUP ALL_PDFS dict.
- C-AWS-4: T2 Cell 8 `load_pdf_from_s3` has try/except S3 fallback;
  notebook still runs if S3 is unreachable.
- C-AWS-5: SageMaker execution role pattern in T2 Cell 3 is correct
  (boto3 client created with no explicit credentials; the role provides
  them).
- C-AWS-6: INSTRUCTOR_SETUP Cell 3 permission probe (put + delete a
  test object) is the right idempotency pattern.
- C-AWS-7: INSTRUCTOR_TEST_SETUP Cell 9 bucket-creation pattern
  (create + delete_public_access_block + put_bucket_policy) is the
  current correct path for AWS in 2025/2026.
- C-AWS-8: INSTRUCTOR_TEST_SETUP Cell 14 `requests.get` verification
  with `b"%PDF"` header check is the right post-upload smoke test.

### 6.2 Library / model contracts

- C-LIB-1: All plans use `gpt-4o` for chat completions. No Anthropic
  SDK is imported anywhere (CORE line 76 satisfied).
- C-LIB-2: All plans pin `numpy<2` (CORE line 80 satisfied) EXCEPT
  `plans/test_data_setup.md` which is missing the pin (TEST-SETUP-FIX-001).
- C-LIB-3: All plans use `text-embedding-3-small` for embeddings;
  T6, T7, T9, INSTRUCTOR_SETUP all consistent.
- C-LIB-4: `chromadb==1.5.8` with `PersistentClient(path="./chroma_db")`
  and `configuration={"hnsw": {"space": "cosine"}}` is consistent across
  T6 and T7. T9 has the same config but the wrong collection name
  (T9-FIX-001).
- C-LIB-5: All plans use `getpass + os.environ["OPENAI_API_KEY"]`
  (CORE line 77 satisfied).
- C-LIB-6: T6 install line orders `chromadb` before `openai`. Correct
  (avoids typing-extensions downgrade).

### 6.3 Cross-topic contracts confirmed correct

- C-XT-1: `INTENT_CATEGORIES` (5 categories: account_inquiry,
  card_services, loans, investments, general) defined in T4, used by T9
  line 381. Exact match.
- C-XT-2: T6 `embed_texts`, `add_to_store`, `retrieve`, `rag_answer`
  signatures match what T9 line 51-56 declares it imports.
- C-XT-3: T7 names `web_search_barclays`, `extract_citations`,
  `route_query`, `hybrid_answer`, `vector_confidence` match T9 line 59,
  530+ usage.
- C-XT-4: T5 `count_tokens_in_messages` formula matches the OpenAI
  Cookbook (3 tokens per message + 3 priming, gpt-4o / o200k_base).
- C-XT-5: T5 `truncate_history` correctly preserves index 0 (system
  message) unconditionally.
- C-XT-6: T8 `GuardrailResult` dataclass is well-defined and consistent
  across T8's four guardrail functions.
- C-XT-7: T7 web_search call shape
  `client.responses.create(tools=[{"type": "web_search", "filters": {"allowed_domains": ["barclays.co.uk"]}}])`
  matches the current OpenAI Responses API and matches what T9 line 530
  expects.
- C-XT-8: All plans pass the AI-tells scan: no em dashes, no en dashes,
  no Unicode multiplication, no smart quotes, no emoji outside markdown
  headers.
- C-XT-9: None of T1-T9 create new files on disk other than
  `./chroma_db` (introduced by T6, reused by T7 and T9). No EBS state
  beyond that, no DynamoDB, no RDS.
- C-XT-10: Day-1 plans (T1, T2, T3) make no HTTP calls (no `requests.get`,
  no `urllib`, no `huggingface_hub`, no `datasets.load_dataset`). Apart
  from Topic 2's S3 read of two PDFs, day-1 is fully offline-capable.
  Source: discrepancies_1_2_3.md S5.4.

### 6.4 Synthetic PDF / inline-data alignment confirmed correct

- C-SYN-1: T1 PRODUCT_SNIPPET numbers (loan range, APR, term, fee)
  match `plans/test_data_setup.md` Cell 5 synthetic PDF byte-for-byte.
- C-SYN-2: T2 Cell 6 raw_pdf_noise "58 days interest" matches synthetic
  PDF "58 days interest" matches T6 BARCLAYS_DOCS[0] AFTER T6-FIX-002.
- C-SYN-3: T2 Cell 6 raw_pdf_noise "27.9% p.a. variable" matches
  synthetic Credit Card PDF "27.9 percent APR variable". T6 BARCLAYS_DOCS[1]
  will match after T6-FIX-003.
- C-SYN-4: Synthetic Savings PDF "3.75 percent AER" matches T6
  BARCLAYS_DOCS[2] "3.75% AER" byte-for-byte.
- C-SYN-5: INSTRUCTOR_SETUP chunking pipeline (chunk_size=1500, overlap=200,
  sentence-boundary heuristic) is byte-identical to T2 Cell 21 and to
  INSTRUCTOR_TEST_SETUP Cell 12. Pre-built `barclays_chunks.json` has the
  same chunk shape as student-generated `chunks`.

### 6.5 Day-1 lab tier distribution confirmed correct

- C-LAB-1: T1 has Tier 1 lab. T2 has Tier 1 + Tier 2 (hard).
  T3 has Tier 1 + Tier 3 (open-ended). One Tier 2 and one Tier 3 per day.
- C-LAB-2: Safety-net cells present for every lab whose output feeds
  downstream: T1 Cell 14, T2 Cell 11 + Cell 24, T3 Cell 11 + Cell 18.
  T3 Cell 24 (Tier 3) correctly has no safety-net.

---

## 7. Appendix - Source Files

### 7.1 The three per-day audits this document consolidates

- `plans/aws_setup/discrepancies_1_2_3.md` (909 lines)
- `plans/aws_setup/discrepancies_4_5_6.md` (725 lines)
- `plans/aws_setup/discrepancies_7_8_9.md` (560 lines)

Each was produced via the `/research` protocol (3 cycles, no web search,
internal file analysis only).

### 7.2 Plan files audited (source of truth)

- `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md` (locked course manifest)
- `plans/TOPICS.md` (per-topic manifest)
- `plans/instructor_setup.md` (course-day S3 populator)
- `plans/test_data_setup.md` (pre-course personal-bucket synthetic PDF generator)
- `plans/topic_01_foundations.md`
- `plans/topic_02_nlp_preprocessing.md`
- `plans/topic_03_first_chatbot.md`
- `plans/topic_04_prompt_engineering.md`
- `plans/topic_05_conversation_memory.md`
- `plans/topic_06_rag_foundations.md`
- `plans/topic_07_advanced_rag_web_search.md`
- `plans/topic_08_ethical_guardrails.md`
- `plans/topic_09_capstone.md`

### 7.3 Built setup notebooks (no plan-vs-built drift)

- `setup/INSTRUCTOR_SETUP.ipynb` (built from `plans/instructor_setup.md`, 13 cells)
- `setup/INSTRUCTOR_TEST_SETUP.ipynb` (built from `plans/test_data_setup.md`, 16 cells)

### 7.4 Built topic notebooks (consulted only for plan-vs-built parity check)

- `exercises/topic_01_foundations/topic_01_foundations.ipynb`
- `exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb`
- `exercises/topic_03_first_chatbot/topic_03_first_chatbot.ipynb`
- `exercises/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb`
- `exercises/topic_05_conversation_memory/topic_05_conversation_memory.ipynb`
- `exercises/topic_06_rag_foundations/topic_06_rag_foundations.ipynb`
- `exercises/topic_07_advanced_rag_web_search/topic_07_advanced_rag_web_search.ipynb`
- `exercises/topic_08_ethical_guardrails/topic_08_ethical_guardrails.ipynb`
- `exercises/topic_09_capstone/topic_09_capstone.ipynb`

(All 9 topics are now built and were re-verified against this audit on 2026-04-25. Per-fix NOTEBOOK STATUS annotations are inline in Section 4.)

### 7.5 No web sources consulted

Per the user's instruction, all three sub-audits ran the `/research`
protocol with `no web search`. This consolidation file follows the same
constraint.

---

**End of AWS_AUDIT.md**. To apply any fix above, open the referenced
plan file at the referenced line, replace the "Current" text with the
"Replacement" text, and re-run `/build-topic-notebook N` (or apply the
edit directly to the built notebook if you do not want to regenerate).

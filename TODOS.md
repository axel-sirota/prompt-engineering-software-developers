# TODOS and Known Issues
# Barclays - Generative AI: Prompt Engineering for Software Developers
#
# Entries are added by /fixes and resolved entries are moved to AUDIT.md.

---

## [RESOLVED] SETUP-FIX-005: INSTRUCTOR_TEST_SETUP.ipynb fails with AccessDenied on delete_public_access_block when run in SageMaker - 2026-04-27

**Topic**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**Reported**: 2026-04-27
**Description**: Running INSTRUCTOR_TEST_SETUP.ipynb inside SageMaker Studio fails with
AccessDenied on s3.delete_public_access_block because the SageMaker execution role lacks
s3:PutBucketPublicAccessBlock. The notebook is designed to run locally with personal AWS
credentials, not in SageMaker. Fix: add a SageMaker environment detection guard in Cell 1
(setup cell 6848e038) that stops execution with a clear message if running inside SageMaker.
Also update the Cell 0 intro markdown to make "run locally, NOT in SageMaker" more prominent.
**Files**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**Status**: resolved
**Fix applied**: Cell 3993f0db (intro markdown) rewritten with prominent "!! RUN THIS LOCALLY
- NOT IN SAGEMAKER !!" heading. Cell 6848e038 (setup code) now detects SageMaker environment
via SM_TRAINING_ENV env var, SAGEMAKER_METRICS_DIRECTORY env var, or /opt/ml directory
presence, and raises EnvironmentError immediately with a clear message explaining why and
how to run locally instead.

---

## [RESOLVED] SETUP-FIX-003: TOPICS.md not updated after TEST-SETUP-FIX-002 BankingDocPDF fix - 2026-04-27

**Topic**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: TEST-SETUP-FIX-002 (BankingDocPDF self.epw fix) was applied to the notebook
and recorded in AUDIT.md and TODOS.md but plans/TOPICS.md was never updated to reflect it.
**Status**: resolved
**Fix applied**: Added a "Setup Notebooks" section to plans/TOPICS.md documenting both
INSTRUCTOR_TEST_SETUP.ipynb and INSTRUCTOR_SETUP.ipynb with their fix histories. Also
corrected the Summary table (Topics 07 and 08 were showing "planned" instead of "done").

---

## [RESOLVED] SETUP-FIX-004: INSTRUCTOR_SETUP.ipynb PDF manifest - add all confirmed real Barclays URLs - 2026-04-27

**Topic**: setup/INSTRUCTOR_SETUP.ipynb
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: Research confirmed 5 additional real public Barclays URLs beyond the 4
already in the manifest. Replace/extend the manifest with all confirmed URLs.
**Status**: resolved
**Fix applied**: Expanded PDF_MANIFEST in cell c5628693 from 4 to 6 entries: added
barclays_isa_instant_terms.pdf (Instant Cash ISA, 1.00% AER) and barclays_isa_guide.pdf
(Guide to ISAs). Added "required" field to distinguish Topic 2 mandatory files from
optional Tier 3 stretch docs. Added BROWSER_HEADERS to bypass Barclays CDN 403 bot
blocking. Updated download_url_and_upload_s3 (cell ffa56033) to pass browser headers.
Updated markdown cell 6281076a and summary cell 330104f6 to reflect 6 docs.

---

## [RESOLVED] T5-FIX-003: T5 BARCLAYS_SYSTEM_PROMPT replaced with T3 full canonical version - 2026-04-27

**Topic**: topic_05_conversation_memory
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: T5 setup cell (2db98a99) defined a short 4-sentence BARCLAYS_SYSTEM_PROMPT.
T3's canonical version has four named sections (Persona, Constraints, Format, Refusal). Using
a shorter version in T5 means the persona drifts mid-course. Replace with T3 verbatim.
**Files**: exercises/topic_05_conversation_memory/topic_05_conversation_memory.ipynb,
           solutions/topic_05_conversation_memory/topic_05_conversation_memory.ipynb
**Status**: resolved
**Fix applied**: Cell 2db98a99 in both exercise and solution notebooks updated with the T3
full multi-paragraph prompt (Persona/Constraints/Format blocks). Comment updated to state
"Reusing the T3 system prompt verbatim so the persona stays stable across topics."

---

## [RESOLVED] T9-FIX-004: T9 BARCLAYS_SYSTEM_PROMPT replaced with T3 full canonical version - 2026-04-27

**Topic**: topic_09_capstone
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: T9 continuity cell (1454f38d9b71) used a short 3-sentence BARCLAYS_SYSTEM_PROMPT
that did not match T3's canonical 4-section version. Students arriving at the capstone would
see a weaker system prompt than the one they built in T3.
**Files**: exercises/topic_09_capstone/topic_09_capstone.ipynb,
           solutions/topic_09_capstone/topic_09_capstone.ipynb
**Status**: resolved
**Fix applied**: BARCLAYS_SYSTEM_PROMPT in cell 1454f38d9b71 of both notebooks replaced with
T3's full canonical version (triple-quoted string, Persona/Constraints/Format sections).

---

## [RESOLVED] T9-FIX-007: T9 classify_with_schema uses T4's 3-field schema verbatim - 2026-04-27

**Topic**: topic_09_capstone
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: T9 continuity cell defined CLASSIFICATION_SCHEMA with only 2 fields (intent,
confidence as number). T4's canonical schema has 3 fields: intent (enum), confidence (string
enum: high|medium|low), rationale (one-sentence string). The mismatch breaks the strict-mode
schema and produces different output shape than students built in T4.
**Files**: exercises/topic_09_capstone/topic_09_capstone.ipynb,
           solutions/topic_09_capstone/topic_09_capstone.ipynb
**Status**: resolved
**Fix applied**: CLASSIFICATION_SCHEMA and classify_with_schema in cell 1454f38d9b71 replaced
with T4's verbatim 3-field schema (confidence is string enum not number, rationale added).

---

## [RESOLVED] T9-FIX-008: T9 count_tokens_in_messages uses T5's 3+3 formula - 2026-04-27

**Topic**: topic_09_capstone
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: T9 continuity cell used 4 tokens per message + 2 priming tokens, which is
wrong for gpt-4o (o200k_base). T5's canonical formula is 3 tokens per message + 3 priming
tokens (per OpenAI Cookbook for o200k_base encoding). This caused token budget calculations
to differ between T5 and T9.
**Files**: exercises/topic_09_capstone/topic_09_capstone.ipynb,
           solutions/topic_09_capstone/topic_09_capstone.ipynb
**Status**: resolved
**Fix applied**: count_tokens_in_messages in cell 1454f38d9b71 replaced with T5's verbatim
implementation: tokens_per_message=3, uses tiktoken.encoding_for_model() with KeyError fallback
to o200k_base, priming tokens=3 at end.

---

## [RESOLVED] T9-FIX-009: T9 BarclaysChat.__init__ aligned to T5 signature - 2026-04-27

**Topic**: topic_09_capstone
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: T9 BarclaysChat.__init__ had signature __init__(self, system_prompt, max_tokens=3000)
and stored self.client = client (T5 uses module-global client). T5's canonical signature is
__init__(self, system_prompt) with no max_tokens param. This created a constructor mismatch
students would notice when comparing T5 and T9 code.
**Files**: exercises/topic_09_capstone/topic_09_capstone.ipynb,
           solutions/topic_09_capstone/topic_09_capstone.ipynb
**Status**: resolved
**Fix applied**: BarclaysChat in cell 1454f38d9b71 updated to T5 signature. max_tokens=3000
added as a class-level attribute so production_assistant can still read chat_state.max_tokens
via getattr(). Callers in ccf4e7d60f5a and eddf92cc860d updated to use
BarclaysChat(system_prompt=BARCLAYS_SYSTEM_PROMPT) without max_tokens kwarg. Lab 4 solution
cell 0a3564a3637d also updated.

---

## [RESOLVED] T3-FIX-002: Rename PRODUCT_SNIPPET to BARCLAYS_PRODUCT_SNIPPET - 2026-04-25

**Topic**: topic_03_first_chatbot
**Reported**: 2026-04-25
**Description**: Variable `PRODUCT_SNIPPET` in Topic 3 setup cell collides with Topic 1's
narrower `PRODUCT_SNIPPET` (loan only). Topic 3 defines an expanded version (loan + credit card).
Rename it `BARCLAYS_PRODUCT_SNIPPET` everywhere in both exercise and solution notebooks.
**Files**: exercises/topic_03_first_chatbot/topic_03_first_chatbot.ipynb,
           solutions/topic_03_first_chatbot/topic_03_first_chatbot.ipynb
**Status**: RESOLVED - 2026-04-25

---

## [RESOLVED] T3-FIX-003: Rephrase the "richer context" claim about Topic 2 chunks - 2026-04-25

**Topic**: topic_03_first_chatbot
**Reported**: 2026-04-25
**Description**: A cell in the notebook claims Topic 2 chunks give "richer context" for
the chatbot. This is inaccurate - Topic 2 chunks contain personal loan FAQ text only, so
credit card questions would fall through to general knowledge. Replace with accurate note.
**Files**: exercises/topic_03_first_chatbot/topic_03_first_chatbot.ipynb,
           solutions/topic_03_first_chatbot/topic_03_first_chatbot.ipynb
**Status**: RESOLVED - 2026-04-25

---

## [RESOLVED] T3-FIX-004: Add GPT4O pricing constants to Cell 3 - 2026-04-25

**Topic**: topic_03_first_chatbot
**Reported**: 2026-04-25
**Description**: `GPT4O_INPUT_PRICE_PER_1K` and `GPT4O_OUTPUT_PRICE_PER_1K` are referenced
in the stretch lab cell but were not defined in the setup cell, causing NameError for students.
Add both constants to Cell 3 after the OpenAI client setup block.
**Files**: exercises/topic_03_first_chatbot/topic_03_first_chatbot.ipynb,
           solutions/topic_03_first_chatbot/topic_03_first_chatbot.ipynb
**Status**: RESOLVED - 2026-04-25

---

## [RESOLVED] T4-FIX-001: Drop dead sagemaker import block from Cell 3 - 2026-04-25

**Topic**: topic_04_prompt_engineering
**Reported**: 2026-04-25
**Resolved**: 2026-04-27
**Description**: Cell 3 (the setup/imports cell) contains a sagemaker import block
(import sagemaker, get_execution_role, sess, role, print statements). Topic 4 makes
zero AWS calls. Remove these lines from Cell 3. Keep all other imports intact.
Do NOT delete the cell - only remove the sagemaker lines from within it.
**Files**: exercises/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb,
           solutions/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb
**Status**: RESOLVED
**Fix applied**: Verified cell 65e8dd4b in both exercise and solution notebooks. The
sagemaker import block (import sagemaker, get_execution_role, sess, role assignment,
and print statements) was already absent from both notebooks - the cell contains only
os, json, getpass, OpenAI imports, client instantiation, and INTENT_CATEGORIES. The
fix had been applied prior to this session. No notebook edit was required.

---

## [RESOLVED] T4-FIX-003: Fix false T5 routing claim in Cell 0 - 2026-04-25

**Topic**: topic_04_prompt_engineering
**Reported**: 2026-04-25
**Resolved**: 2026-04-27
**Description**: Cell 0 (the title/intro markdown cell) contains text claiming that
classify_intent() and classify_with_schema() are called by Topic 5 (Conversation Memory).
This is false - Topic 5 does not call them; Topic 9 (Capstone) does.
Find the sentence(s) referencing Topic 5 as the consumer of these functions and change
them to reference Topic 9 instead.
**Files**: exercises/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb,
           solutions/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb
**Status**: RESOLVED
**Fix applied**: Two cells contained Topic 5 references. Cell 49b3c8d8 (title/intro,
Cell 0) already correctly referenced "Topic 9 (Capstone)" - that fix had been applied
prior to this session. Cell 90ecf431 (wrap-up, Cell 26) still contained "Connection
to Topic 5 (Conversation Memory)" with incorrect claims that classify_intent() and
classify_with_schema() would be called in Topic 5. That section was rewritten in both
exercise and solution notebooks to correctly say "Connection to Topic 9 (Capstone)"
and describe the correct usage: called at the start of each turn in the capstone
orchestrator to route queries.

---

## [RESOLVED] T4-FIX-005: Change openai==1.51.0 to openai==2.32.0 in Cell 2 - 2026-04-25

**Topic**: topic_04_prompt_engineering
**Reported**: 2026-04-25
**Resolved**: 2026-04-27
**Description**: Cell 2 (pip install cell) currently pins openai==1.51.0. This causes
a mid-session SDK downgrade when students run T1 (installs 2.32.0) then T4 (downgrades
to 1.51.0). Change openai==1.51.0 to openai==2.32.0 in Cell 2. Also update any comment
in that cell that says this version is required for json_schema strict mode to note that
2.32.0 is the course-wide canonical pin and structured outputs work fine on 2.x.
**Files**: exercises/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb,
           solutions/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb
**Status**: RESOLVED
**Fix applied**: Verified cell ccd27dca in both exercise and solution notebooks. The pin
was already `openai==2.32.0` (not 1.51.0) and the comment already read "openai 2.32.0:
course-wide canonical pin - structured outputs work fine on 2.x". The fix had been
applied prior to this session. No notebook edit was required.

---

## [RESOLVED] topic_01 plan file has bare --- separators and research URLs not as https:// links - 2026-04-25

**Topic**: topic_01_foundations
**Reported**: 2026-04-25
**Resolved**: 2026-04-25
**Description**: /verify 1 found two issues in plans/topic_01_foundations.md:
  1. 25 bare --- separator lines throughout the plan file (between cell sections). These
     violate the no-bare-separator rule from CLAUDE.md.
  2. Research source URLs in the RESEARCH VALIDATED section were formatted as plain text in
     a table, not as https:// hyperlinks. The verify-research checker found 0 URLs, failing
     the minimum-3-sources check.
**Status**: resolved
**Fix applied**: Removed all 25 bare --- separators using a python3 script that skipped lines
inside code fences. Replaced the RESEARCH VALIDATED table with a numbered list of 7 properly
formatted https:// URLs. Both checks now pass: AI-tells CLEAN, 7 https:// URLs found.

---

## [RESOLVED] T9-FIX-012: Add web_search per-call cost note to T9 end-to-end test cell - 2026-04-27

**Topic**: topic_09_capstone
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: The end-to-end test cell (eddf92cc860d) computes total_cost from per-token
pricing only. The OpenAI hosted web_search tool adds a per-call fee on top of token cost.
Students running hybrid or web routes would undercount total cost. Add a comment note
clarifying that the calculator is token-only and instructing students to add the per-call
web_search fee manually when route is 'hybrid' or 'web'.
**Files**: exercises/topic_09_capstone/topic_09_capstone.ipynb,
           solutions/topic_09_capstone/topic_09_capstone.ipynb
**Status**: resolved
**Fix applied**: Added a NOTE comment block at the end of cell eddf92cc860d in both exercise
and solution notebooks: "NOTE: this calculator covers per-token cost only. The OpenAI hosted
web_search tool adds a per-call fee on top (consult the current OpenAI pricing page). When
route_query returns 'hybrid' or 'web', add the per-call web_search fee to your log_record manually."

---

## [RESOLVED] T9-FIX-013: Add barclays_chunks.json stretch hint to T9 Tier 3 lab brief - 2026-04-27

**Topic**: topic_09_capstone
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: The Tier 3 lab brief markdown cell (b59e17a9a1df) describes four production
extension options but gives no hint about the real Barclays data available in S3 for
testing extensions against richer content. Students implementing Option A (semantic cache)
or Option D (eval queue) would benefit from testing with real product text chunks.
**Files**: exercises/topic_09_capstone/topic_09_capstone.ipynb,
           solutions/topic_09_capstone/topic_09_capstone.ipynb
**Status**: resolved
**Fix applied**: Added a "Stretch data:" paragraph at the end of cell b59e17a9a1df in both
exercise and solution notebooks: "Stretch data: load s3://barclays-prompt-eng-data/barclays_chunks.json
if you want to test your extension against real Barclays product text. See setup/INSTRUCTOR_SETUP.ipynb
Cell 10 for the load snippet."

---

## [RESOLVED] TEST-SETUP-FIX-002: INSTRUCTOR_TEST_SETUP.ipynb FPDFException in make_personal_loan_faq - 2026-04-27

**Topic**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**Reported**: 2026-04-27
**Description**: Running cell ab7e0f75 (make_personal_loan_faq) raises:
FPDFException: Not enough horizontal space to render a single character
The error is in BankingDocPDF.qa_block -> multi_cell(0, 5, answer, align="J").
align="J" (justified) fails in fpdf2 on SageMaker (Python 3.12) when the font/margin
combination leaves insufficient horizontal rendering space. Fix: change align="J" to
align="L" in qa_block and body_text methods of BankingDocPDF (cell 829e9f16).
**Files**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**Status**: resolved
**Fix applied**: Changed multi_cell(0, ...) to multi_cell(self.epw, ...) in both body_text()
and qa_block() methods of BankingDocPDF (cell 829e9f16). self.epw (effective page width =
page width minus both margins) is independent of cursor x position, avoiding the zero-width
computation that causes FPDFException with align="J". Tested locally - all 4 PDFs generate
without error.

---

## [RESOLVED] T02-FIX-001: Cell ca6eb189 Mermaid diagram uses legacy fitz.open label - 2026-04-27

**Topic**: topic_02_nlp_preprocessing
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: Cell ca6eb189 Mermaid diagram node D uses label "fitz.open\nPyMuPDF opens PDF"
but Cell 8 code correctly uses pymupdf.open(). Students will see the diagram contradict the
code. Fix: change the node label from fitz.open to pymupdf.open in both exercise and solution.
**Files**: exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb,
           solutions/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb
**Status**: resolved
**Fix applied**: Node D label changed from "fitz.open\nPyMuPDF opens PDF" to
"pymupdf.open\nPyMuPDF opens PDF" in cell ca6eb189 of both exercise and solution notebooks.

---

## [RESOLVED] T02-FIX-002: Solution Cell 4e17ec14 pip install missing openai==2.32.0 - 2026-04-27

**Topic**: topic_02_nlp_preprocessing
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: Exercise Cell 4e17ec14 includes "openai==2.32.0" in the pip install line.
Solution Cell 4e17ec14 omits it. The solution should match the exercise install line exactly
so running just the solution notebook does not miss the openai dependency.
**Files**: solutions/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb
**Status**: resolved
**Fix applied**: Added "openai==2.32.0" to the pip install line in cell 4e17ec14 of the
solution notebook. Both notebooks now have identical install lines.

---

## [RESOLVED] TEST-SETUP-FIX-001: Add numpy<2 to INSTRUCTOR_TEST_SETUP.ipynb pip install - 2026-04-27

**Topic**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**Reported**: 2026-04-27
**Resolved**: 2026-04-27
**Description**: Cell 2 (6848e038) of INSTRUCTOR_TEST_SETUP.ipynb installs fpdf2, pymupdf,
requests, and boto3 but omits "numpy<2". The test setup notebook generates synthetic PDFs
and chunks them using pymupdf. If numpy 2.x is present in the environment, chromadb (used
in later notebook runs) will break. Adding numpy<2 here ensures the instructor's test
environment stays compatible with all course notebooks.
**Files**: setup/INSTRUCTOR_TEST_SETUP.ipynb
**Status**: resolved
**Fix applied**: Changed pip install line in cell 6848e038 from:
  !pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3
to:
  !pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3 "numpy<2"

# AUDIT - Fix History

| Date | Fix ID | Description | Files | Status |
|------|--------|-------------|-------|--------|
| 2026-04-25 | T2-FIX-002 | Rewrite misleading no-OpenAI-key comment in Cell 3 | topic_02 exercise + solution | resolved - cell id 3599d0ac |
| 2026-04-25 | T2-FIX-003 | Add test-bucket switch comment above S3_BUCKET in Cell 3 | topic_02 exercise + solution | resolved - cell id 3599d0ac |
| 2026-04-25 | T2-FIX-005 | Gate Cell 6 getpass with os.environ check | topic_02 exercise + solution | resolved - cell id 2cb33b37 |
| 2026-04-27 | ALL-FIX-001 | Pin sagemaker==2.232.1 + boto3 in all pip install cells | all 9 exercise + 9 solution notebooks | resolved - see detail below |
| 2026-04-27 | ALL-FIX-002 | Normalize char-by-char source arrays to line arrays in all 211 code cells | all 9 exercise + 9 solution notebooks | resolved - see detail below |
| 2026-04-27 | ALL-FIX-005 | Fix S3 bucket name in T02, remove boto3 narrative, replace s3:// hints with HTTPS | T02 (ex+sol), T06/T07/T09 (ex+sol) | resolved - see detail below |
| 2026-04-29 | T679-FIX-001 | Filesystem-only handoff: replace inline BARCLAYS_DOCS with S3 PDF download + PyMuPDF chunking | T06/T07/T09 (ex+sol) | resolved - see detail below |

## 2026-04-29 - T679-FIX-001: filesystem-only handoff via S3 PDF download

**Topics affected**: topic_06_rag_foundations, topic_07_advanced_rag_web_search, topic_09_capstone (all exercise + solution = 6 notebooks)

**Problem**: Topics 6, 7, 9 sourced their Barclays product corpus from inline Python string literals (`BARCLAYS_DOCS = [...]` with 7-9 hardcoded product summaries). This violated the project rule that handoff between topics must be via the filesystem, not via Python globals or inline-replicated content. Each notebook should download the canonical PDFs from the public `barclays-prompt-eng-data` S3 bucket (the same files Topic 2 downloads), run the same PyMuPDF clean+chunk pipeline inline, and use the resulting chunks. Topic 5 had previously been observed loading from globals which is similarly wrong (left as-is in this fix; T5 does not need a corpus).

**Root cause**: When T6 was first authored, the inline `BARCLAYS_DOCS` was a teaching shortcut to avoid the S3 download in the demo. T7 and T9 copied the same shortcut. Over time this drifted into "the way the corpus is provided" instead of "a fallback when S3 is unreachable", and T9's continuity cell hardcoded 9 docs (7 from T6 + 2 capstone-specific operational notes) which compounded the problem.

**Fix**: Replaced the inline BARCLAYS_DOCS literal in three exercise+solution pairs with a self-contained download-clean-chunk block reusing Topic 2's exact pattern:
- `_load_pdf_from_s3(key)` via `requests.get` over HTTPS (no IAM credentials needed since the bucket is public-read).
- `_clean_pdf_text(text)` with the same regex pipeline as T2 cell `afb66ad6`: hyphen line-break fix, page-header strip, whitespace collapse, decoration-line removal.
- `_chunk_text(text, chunk_size=1500, overlap=200)` with the same sentence-boundary aware chunker as T2 cell `258f7171`.

Two PDFs are downloaded: `barclays_personal_loan_faq.pdf` and `barclays_credit_card_tnc.pdf` (already uploaded to the bucket per `instructor_setup.md`). On any S3 failure the loop prints `[FAIL] {key}: {e}` per file and falls back to a 3-string inline corpus with a visible `!!! WARNING` banner so students immediately know the real corpus did not load.

For T9 specifically, only the Topic 2 BARCLAYS_DOCS block at the top of the continuity cell `1454f38d9b71` was swapped. All downstream T3-T8 helpers (BARCLAYS_SYSTEM_PROMPT, create_chatbot, classify_with_schema, BarclaysChat, count_tokens_in_messages, embed_texts, retrieve, hybrid_answer, detect_pii, should_escalate, etc.) and the final `print("continuity setup complete")` were preserved byte-for-byte. The 2 capstone-specific operational docs (freeze card, Money Worries) are appended inline AFTER the S3 chunks because they are short procedural snippets not present in the S3 PDFs.

Pip install cells were updated in all three topics (`eaea89ff` in T6, `a9a50792` in T7, `2e65d42c1a7d` in T9) to add `pymupdf==1.27.2.2` and `requests` to the install line.

**Cell IDs modified**:
- T06 ex+sol: `eaea89ff` (pip), `23611394` (BARCLAYS_DOCS block)
- T07 ex+sol: `a9a50792` (pip), `5935a476` (BARCLAYS_DOCS block + helpers + collection setup)
- T09 ex+sol: `2e65d42c1a7d` (pip), `1454f38d9b71` (Topic 2 chunks block only - downstream T3-T8 helpers untouched)

**Char-by-char source-array bug**: After NotebookEdit calls, the modified cells reverted to char-by-char source storage (the same bug fixed in ALL-FIX-002). Ran the inline normalization snippet (`source = ''.join(items).splitlines(keepends=True)`) on every modified cell after each batch of edits. Final source-array sanity check: zero char-by-char cells across all 6 notebooks.

**Validation**:
- T06 exercise: 1 false-positive error (Tier 3 lab has no `None` placeholder by design - this is correct per CLAUDE.md tier rules).
- T06 solution: clean.
- T06 pair check: 27 cells matched on both sides, types match.
- T07 exercise: clean (1 warning about no labs detected - false positive, labs are present).
- T07 solution: clean.
- T07 pair check: 27 cells matched on both sides, types match.
- T09 exercise: clean.
- T09 solution: clean.
- T09 pair check: 29 cells matched on both sides, types match.
- AI-tells scan: clean on all 6 notebooks (no em-dashes, en-dashes, Unicode multiplication, or bare `---` separators).

**Files unchanged**: T08 (no Barclays product corpus needed - guardrails-only). T01-T05 not in scope.

## 2026-04-27 - ALL-FIX-002: char-by-char source storage normalization

**Problem**: The NotebookEdit tool stored cell `source` as a list of individual characters (e.g. `["#", " ", "t", "i", ...]` with 447+ items per cell) instead of the correct nbformat: a list of lines (e.g. `["# comment\n", "!pip install ...\n"]`). SageMaker's Jupyter renderer splits on source array items, so each character was rendered as a separate "token", causing comments to display with embedded newlines and cells to appear garbled.

**Root cause**: The NotebookEdit tool passed the cell source as a single string; the tool internally stored it character-by-character instead of splitting on line boundaries.

**Fix**: Ran a normalization script across all 18 notebooks. For each code cell whose source array was detected as character-by-character (items mostly length 1-2), joined all items into a single string then re-split using `splitlines(keepends=True)` to produce the correct line-per-item format. 211 cells normalized across 18 notebooks.

**Validation**: All 9 topic pairs passed. Remaining validator errors are local-env false positives (chromadb not installed in .venv).

## 2026-04-27 - ALL-FIX-005: S3 bucket name, boto3 narrative, and s3:// stretch hints

**Topics affected**: topic_02 (exercise + solution), topic_06, topic_07, topic_09 (exercise + solution each)

**Problem 1**: T02 `S3_BUCKET = "barclays-pe-test-axel-7342"` was the personal test bucket. Production bucket is `barclays-prompt-eng-data`.

**Problem 2**: T02 Mermaid diagram (cell ca6eb189), env setup markdown (cell 489317d0), learning objectives (cell 2d392fb0), and pip comment (cell 4e17ec14) all referenced "boto3" in the narrative. The actual `load_pdf_from_s3()` implementation uses `requests.get()` over HTTPS - no boto3 SDK call. The narrative was misleading students.

**Problem 3**: T06, T07, T09 (exercise + solution) Tier 3 stretch hints contained `s3://barclays-prompt-eng-data/barclays_chunks.json` - an `s3://` URI requires boto3 SDK and IAM credentials. Students have no IAM write permissions.

**Fix**: All 10 notebooks updated:
- T02 (ex+sol): bucket name changed, Mermaid label changed to `requests.get(url)`, narrative text updated
- T06/T07/T09 (ex+sol): stretch hints changed to HTTPS URL + `requests.get(url).json()` pattern

**Verification**: `grep -rn 'barclays-pe-test-axel'` = 0, `grep -rn 's3://'` = 0, `grep -rn 'boto3 get_object'` = 0. T02 validation: pass (exercise + solution). T06/T07/T09: pre-existing chromadb false positive only.

## 2026-04-27 - ALL-FIX-001: sagemaker SDK version pin across all 18 notebooks

**Topics affected**: topic_01 through topic_09 (all exercise and solution notebooks)

**Problem**: SageMaker Distribution default image ships `sagemaker==3.9.0` (the new refactored SDK). In this version, `get_execution_role` was moved out of the top-level `sagemaker` namespace. Every notebook that runs `from sagemaker import get_execution_role` raises `ImportError: cannot import name 'get_execution_role' from 'sagemaker'` on student machines.

**Root cause**: SageMaker SDK v3 restructured the package layout. The classic v2 API (`sagemaker.get_execution_role`, `sagemaker.Session`) still works as expected in `sagemaker==2.232.1`, the last v2 release.

**Fix**: Added `"sagemaker==2.232.1" "boto3"` to every pip install cell in all 18 notebooks (9 exercise + 9 solution). The `boto3` pin ensures the AWS SDK is present for S3 and SageMaker session operations regardless of the base image.

Cell IDs modified:
- T01: cell at index 2 (prior session)
- T02: cell at index 2 (prior session)
- T03: cell at index 2 (prior session)
- T04 exercise + solution: cell `ccd27dca` (index 2)
- T05 exercise + solution: cell `2a251c5f` (index 2)
- T06 exercise + solution: cell `eaea89ff` (index 2)
- T07 exercise + solution: cell `a9a50792` (index 2)
- T08 exercise + solution: cell `17f37084` (index 1 - this topic's pip cell is at index 1)
- T09 exercise + solution: cell `2e65d42c1a7d` (index 3 - unusual: this cell contains both pip install and all imports; imports were preserved)

**Validation**: All 6 pair checks (T04-T09) passed. T01-T03 pair checks passed in prior session. No cell count changes - only cell content modified.

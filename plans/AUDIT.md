# AUDIT - Fix History

| Date | Fix ID | Description | Files | Status |
|------|--------|-------------|-------|--------|
| 2026-04-25 | T2-FIX-002 | Rewrite misleading no-OpenAI-key comment in Cell 3 | topic_02 exercise + solution | resolved - cell id 3599d0ac |
| 2026-04-25 | T2-FIX-003 | Add test-bucket switch comment above S3_BUCKET in Cell 3 | topic_02 exercise + solution | resolved - cell id 3599d0ac |
| 2026-04-25 | T2-FIX-005 | Gate Cell 6 getpass with os.environ check | topic_02 exercise + solution | resolved - cell id 2cb33b37 |
| 2026-04-27 | ALL-FIX-001 | Pin sagemaker==2.232.1 + boto3 in all pip install cells | all 9 exercise + 9 solution notebooks | resolved - see detail below |
| 2026-04-27 | ALL-FIX-002 | Normalize char-by-char source arrays to line arrays in all 211 code cells | all 9 exercise + 9 solution notebooks | resolved - see detail below |
| 2026-04-27 | ALL-FIX-005 | Fix S3 bucket name in T02, remove boto3 narrative, replace s3:// hints with HTTPS | T02 (ex+sol), T06/T07/T09 (ex+sol) | resolved - see detail below |

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

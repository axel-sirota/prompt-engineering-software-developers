# TODOS - Fixes Tracker

## Topic 2: NLP Preprocessing Fixes

### T2-FIX-002 [x] Rewrite misleading "no OpenAI key" comment in Cell 3
- Files: exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb
         solutions/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb
- Find: `# We do NOT need the OpenAI key for this topic - this is pure document processing.`
- Replace with two-line comment clarifying key is prompted later

### T2-FIX-003 [x] Add test-bucket switch comment above S3_BUCKET in Cell 3
- Files: same as above
- Find: `S3_BUCKET = "barclays-prompt-eng-data"`
- Insert 3-line instructor comment above

### T2-FIX-005 [x] Gate the Cell 6 getpass with an os.environ check
- Files: same as above
- Find: `os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")`
- Replace with if-guard pattern

---

## All Topics: char-by-char source storage bug - 2026-04-27

### ALL-FIX-002 [RESOLVED 2026-04-27] All code cells stored character-by-character in source arrays
- Files: all 9 exercise notebooks + all 9 solution notebooks (all code cells affected)
- Problem: NotebookEdit tool stored cell source as a list of individual characters instead
  of a list of lines. SageMaker Jupyter renders these incorrectly - comments appear with
  embedded newlines, cells display wrong.
- Fix: run a normalization script that joins each cell's source chars into proper line-split
  arrays (each item = one line ending in \n, last item has no trailing \n)
- Status: resolved
- Fix applied: normalized 211 code cells across all 18 notebooks using splitlines(keepends=True)

---

## ALL-FIX-005: S3 passthrough hardening - 2026-04-27

### ALL-FIX-005 [RESOLVED 2026-04-27] Fix S3 bucket name, boto3 references, and s3:// stretch hints
- T02: S3_BUCKET changed from "barclays-pe-test-axel-7342" to "barclays-prompt-eng-data"
- T02: Mermaid diagram label changed from "boto3 get_object" to "requests.get(url)"
- T02: Learning objectives, env setup text, and pip comment updated - no more "boto3" in student-facing narrative
- T06/T07/T09 (ex+sol): stretch hints changed from s3:// URI to HTTPS URL + requests.get() pattern
- All 10 notebooks fixed. Verified 0 remaining occurrences of barclays-pe-test-axel, s3://, boto3 get_object.
- T02 validation: pass. T06/T07/T09: pre-existing chromadb false positive only.
- Status: resolved

---

## Topics 6, 7, 9: filesystem-only handoff via S3 PDF download - 2026-04-29

### T679-FIX-001 [RESOLVED 2026-04-29] Replace inline BARCLAYS_DOCS with S3 PDF download + PyMuPDF chunking

**Topic**: topic_06_rag_foundations, topic_07_advanced_rag_web_search, topic_09_capstone
**Reported**: 2026-04-29
**Description**: Topics 6, 7, 9 currently source their Barclays product corpus from
inline Python strings (BARCLAYS_DOCS = [...] hardcoded 7-9 product summaries). This
breaks the project rule that handoff between topics must be via the filesystem only,
not via Python globals or inline-replicated content. Each notebook should download
the canonical PDFs from the public barclays-prompt-eng-data S3 bucket (the same
files Topic 2 downloads), run the same PyMuPDF clean+chunk pipeline inline, and use
the resulting chunks. Fail loud if S3 is unreachable; small inline fallback only as
a last resort with a visible warning. T8 is unchanged (no product corpus needed).

**Files**:
- exercises/topic_06_rag_foundations/topic_06_rag_foundations.ipynb (cell 5: 23611394, pip cell: eaea89ff)
- solutions/topic_06_rag_foundations/topic_06_rag_foundations.ipynb (mirror)
- exercises/topic_07_advanced_rag_web_search/topic_07_advanced_rag_web_search.ipynb (cell 5: 5935a476, pip cell: a9a50792)
- solutions/topic_07_advanced_rag_web_search/topic_07_advanced_rag_web_search.ipynb (mirror)
- exercises/topic_09_capstone/topic_09_capstone.ipynb (cell 5: 1454f38d9b71, pip cell: 2e65d42c1a7d)
- solutions/topic_09_capstone/topic_09_capstone.ipynb (mirror)

**Status**: resolved
**Fix applied**: Replaced inline BARCLAYS_DOCS literal in T6 (cell 23611394), T7 (cell 5935a476),
T9 (cell 1454f38d9b71 - swapped only the Topic 2 chunks block, preserved all downstream
T3-T8 helpers) with a self-contained S3 download + PyMuPDF clean + chunk pipeline using the
exact pattern from Topic 2 (load_pdf_from_s3 via requests.get, clean_pdf_text regex pipeline,
sentence-boundary chunker chunk_size=1500 overlap=200). Two PDFs downloaded:
barclays_personal_loan_faq.pdf and barclays_credit_card_tnc.pdf. T9 appends 2 capstone-specific
operational docs (freeze card, Money Worries) inline after the S3 chunks since those
short procedural snippets are not in the S3 PDFs. Fail-loud on S3 errors with a 3-string
inline fallback. Pip cells in all 3 topics (eaea89ff, a9a50792, 2e65d42c1a7d) updated to add
pymupdf==1.27.2.2 and requests. Char-by-char source array bug from NotebookEdit fixed by
inline normalization after every edit. AI-tells scan: clean on all 6 notebooks. Pair validation
passed for all 3 topics.

---

## All Topics: sagemaker SDK version fix - 2026-04-27

### ALL-FIX-001 [RESOLVED 2026-04-27] Pin sagemaker==2.232.1 in all pip install cells
- Files: all 9 exercise notebooks + all 9 solution notebooks
- Problem: SageMaker Distribution ships sagemaker==3.9.0 which moved get_execution_role
  out of the top-level namespace. `from sagemaker import get_execution_role` raises ImportError.
- Fix: added `"sagemaker==2.232.1" "boto3"` to every pip install cell across all 18 notebooks.
  T01-T03 done in prior session; T04-T09 (exercise + solution) done 2026-04-27.
  Cell IDs: T04=ccd27dca, T05=2a251c5f, T06=eaea89ff, T07=a9a50792, T08=17f37084, T09=2e65d42c1a7d.
- Status: resolved
- Fix applied: pinned sagemaker==2.232.1 (last classic SDK with get_execution_role at top level)
  and boto3 in all 18 pip install cells. All pair checks passed.

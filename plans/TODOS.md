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

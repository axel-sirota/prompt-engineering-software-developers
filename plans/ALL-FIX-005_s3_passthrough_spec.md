# ALL-FIX-005: S3 passthrough hardening + cross-topic fallback audit
# Date: 2026-04-27

## Summary

Full audit of all 9 exercise + 9 solution notebooks for:
1. boto3 SDK usage (must be zero - students have no IAM credentials)
2. S3 download patterns (must use requests.get HTTPS, not s3:// URIs)
3. Cross-topic passthroughs (every dependency must have a local fallback)

## Audit results

### boto3 SDK usage
No boto3.client(), boto3.resource(), or s3.get_object() calls in any student notebook. CLEAN.

### Cross-topic passthrough map (all have fallbacks)

| From | To | Artifact | Fallback |
|------|-----|---------|---------|
| T02 | T03 | chunks list | hardcoded BARCLAYS_PRODUCT_SNIPPET |
| T02 | T06 | chunks list | 7-doc BARCLAYS_DOCS inline (cell 23611394) |
| T02 | T07 | chunks list | 7-doc BARCLAYS_DOCS inline (cell 5935a476) |
| T02 | T09 | chunks list | 9-doc BARCLAYS_DOCS + chunks=list(BARCLAYS_DOCS) (cell 1454f38d9b71) |
| T06 | T07 | ./chroma_db/ | get_or_create_collection + rebuild if empty |
| T06 | T09 | ./chroma_db/ | collection.count()==0 guard + rebuild |
| T05 | T09 | BarclaysChat | full reimplementation in continuity cell |
| T06-T08 | T09 | all functions | full reimplementation in continuity cell |

All ChromaDB writes go to ./chroma_db/ (local disk, writable in per-student SageMaker).

### Issues requiring fixes

**A - T02 uses personal test bucket** (WRONG)
Cell 3599d0ac: S3_BUCKET = "barclays-pe-test-axel-7342"
Must be: S3_BUCKET = "barclays-prompt-eng-data"

**B - T02 Mermaid diagram says "boto3 get_object"** (WRONG - function uses requests.get)
Cell ca6eb189 (markdown, index 7): diagram label B["boto3 get_object<br/>returns streaming body"]
Plus surrounding text: "boto3 connects us to S3" and "boto3 is the AWS SDK"
Must change diagram label to B["requests.get(url)<br/>returns raw bytes"] and fix text.

**C - T02 learning objectives mention boto3** (WRONG)
Header markdown cell: "Load Barclays product documents from S3 using boto3 and PyMuPDF"
Must change "boto3" to "requests (HTTPS)"

**D - Stretch hints in T06, T07, T09 use s3:// URI** (WRONG - requires boto3)
6 cells (exercise + solution for each topic) contain:
  load `s3://barclays-prompt-eng-data/barclays_chunks.json`
Must change to HTTP URL: https://barclays-prompt-eng-data.s3.us-east-2.amazonaws.com/barclays_chunks.json
with requests.get(url).json() pattern.

## Cell IDs to edit

### T02 exercise (exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb)
- Index 0 (markdown header): fix "boto3" -> "requests (HTTPS)" in learning objectives
- Index 3, cell 3599d0ac (code): fix S3_BUCKET bucket name
- Index 7, cell ca6eb189 (markdown): fix Mermaid label + surrounding text

### T02 solution (solutions/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb)
- Same three cells (verify IDs first)

### T06 exercise (exercises/topic_06_rag_foundations/topic_06_rag_foundations.ipynb)
- Tier 3 lab brief cell (markdown, near index 25): fix s3:// in stretch hint

### T06 solution (solutions/topic_06_rag_foundations/topic_06_rag_foundations.ipynb)
- Same cell

### T07 exercise (exercises/topic_07_advanced_rag_web_search/topic_07_advanced_rag_web_search.ipynb)
- Wrap-up or lab brief markdown cell: fix s3:// in stretch hint (find by grep)

### T07 solution
- Same cell

### T09 exercise (exercises/topic_09_capstone/topic_09_capstone.ipynb)
- Cell b59e17a9a1df (markdown, Tier 3 lab brief): fix s3:// in stretch hint

### T09 solution
- Same cell

## Verification after fix

grep -rn 'barclays-pe-test-axel' exercises/ solutions/  # expect 0
grep -rn 's3://' exercises/ solutions/                  # expect 0
grep -rn 'boto3 get_object' exercises/ solutions/       # expect 0
python validate_notebooks.py exercises/topic_02.../... --type exercise
python validate_notebooks.py solutions/topic_02.../... --type solution
python validate_notebooks.py --pair exercises/topic_02.../... solutions/topic_02.../...
# repeat for T06, T07, T09

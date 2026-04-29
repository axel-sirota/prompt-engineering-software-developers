# Extra Topic 8: Data Engineer Use Case - Cell-by-Cell Plan

## Context

**Course**: Barclays Generative AI - Prompt Engineering for Software Developers
**Audience**: Data engineers at Barclays who already saw Topics 1-9 (or are reading this notebook as a reference). The prior nine topics built a customer-service chatbot. This extra topic shifts the lens: the same primitives (prompt engineering, RAG, guardrails, retries, structured outputs) are equally useful when you have NO chat surface at all and just need to enrich rows of a daily ETL pipeline at scale.
**Prerequisites**: Comfort with the Topics 1-9 stack. Familiarity with pandas. No new conceptual material is introduced; everything here is a re-application of what was taught earlier.
**Format**: This is a SHOWCASE notebook, not a lab notebook. There are no `None # YOUR CODE` placeholders, no Tier 1/2/3 lab system, no verification cells. Each code cell is a runnable, heavily commented demo. Each markdown cell explains the design choice or the production tradeoff.

**Running scenario**: A data engineer at Barclays has a daily ETL job that lands a table of customer support tickets in a warehouse (think Snowflake, Redshift, or S3 Parquet). The team wants to enrich every row with LLM-generated columns: intent classification, urgency, product mentioned, PII-redacted body, RAG-grounded suggested response, and an output-validation flag. The pipeline must be cheap, idempotent, observable, and never log raw PII. We will build that pipeline in one notebook end-to-end.

**Patterns reused from prior topics (named explicitly)**:
- From Topic 2: the S3 -> PyMuPDF -> clean -> chunk pattern. We rebuild `chunks` inline so this notebook is self-contained.
- From Topic 4: few-shot prompting and `response_format={"type": "json_schema"}` Structured Outputs. Structured Outputs is the right primitive for data pipelines because every row produces a typed dict that lands cleanly in a DataFrame column.
- From Topic 6: `embed_texts`, `chromadb.PersistentClient`, `collection`, `retrieve`. We rebuild the vector store inline from the freshly downloaded Barclays PDFs.
- From Topic 7: when web_search is appropriate (live rates) and when it is NOT appropriate (batch enrichment of 100k rows is too slow and too expensive). We discuss this rather than use it.
- From Topic 8: `detect_pii`, `validate_output`, `GuardrailResult`. We re-use the regex registry pattern so logging is GDPR-aligned.
- From Topic 9: tenacity retry with `wait_random_exponential`, per-request cost tracking via `prompt_tokens` and `completion_tokens`, structured JSON-line logging.

**Key insight students leave with**: The chatbot is one consumer of these primitives. A batch enrichment pipeline is another. The same Structured Outputs schema, the same guardrail registry, the same retry policy compose into a row-level `enrich_row(row)` callable that you map over a DataFrame, an Airflow task, or an Apache Beam DoFn.

**Filesystem-only handoff (hard rule)**: This notebook does not import anything from earlier notebooks via Python globals. The only handoff is via the filesystem. The S3 bucket `barclays-prompt-eng-data` is the canonical source. The setup cell downloads the PDFs fresh, runs the Topic 2 extract -> clean -> chunk pipeline inline, and proceeds. If S3 is unreachable, we fall back to a small inline corpus so the notebook still runs end-to-end on a flight or in an offline classroom.

**Decisions locked 2026-04-29 before build**:
1. **Single notebook in `exercises/extra_08_data_engineer_usecase/` only.** No `solutions/` mirror. No labs, no `None # YOUR CODE`, no tier system.
2. **Synthetic 50-row DataFrame inline** (no S3 mock CSV). Pure demonstration of the row-fanout shape.
3. **Batch API: JSONL sketch only**, no `client.batches.create` submit, no 24h wait. Markdown discussion + a `build_batch_jsonl` helper that writes the file to `/tmp` and stops there.
4. **2 PDFs** in the corpus download: `barclays_personal_loan_faq.pdf` and `barclays_credit_card_tnc.pdf` (matching the T6/T7/T9 standard). The previously-listed `barclays_savings_rates.pdf` is dropped for consistency.

**Key decisions:**
- **Environment**: SageMaker Studio JupyterLab on ml.t3.medium, no GPU, Python 3.11
- **Library choices**: openai==2.32.0 (Structured Outputs, Batch API), chromadb==1.5.8, tenacity==9.1.4, pymupdf==1.27.2.2, pandas (pre-installed), tiktoken==0.9.0, boto3 (pre-installed), numpy<2 (mandatory pin)
- **Model choices**: gpt-4o for enrichment, text-embedding-3-small for embeddings; the OpenAI Batch API discussion shows where gpt-4o-mini is appropriate for cost
- **Dataset / source choices**: Barclays product PDFs from S3 bucket `barclays-prompt-eng-data` (the same corpus T2/T6/T9 use); customer-ticket DataFrame is generated inline in the notebook (50 mock rows, no PII handling worry because it is synthetic - but the pipeline is built as if it were real PII)
- **Continuity choices**: the notebook is FULLY self-contained. Filesystem handoff only; rebuild collection inline; no Python global from prior notebooks crosses the boundary.
- **No labs, no safety-net cells, no `None # YOUR CODE`**: every cell is a complete runnable demo.

## Deliverables

- **Showcase notebook (recommended)**: `exercises/extra_08_data_engineer_usecase/extra_08_data_engineer_usecase.ipynb`
- **Solution notebook (only if dual-folder convention is enforced)**: `solutions/extra_08_data_engineer_usecase/extra_08_data_engineer_usecase.ipynb` (byte-identical to the exercise)

Recommendation: ship single notebook in `exercises/` only. Confirm with user before build.

## Diagram Index

| Slug | File | Description |
|------|------|-------------|
| pipeline-overview | plans/extra_08_data_engineer_usecase/diagrams/pipeline-overview.mmd | Top-down ETL view: S3 raw tickets table -> pandas DataFrame -> per-row `enrich_row` fan-out (classification, PII redaction, RAG ground, output validation) -> enriched DataFrame -> warehouse. Side branch shows the Barclays product PDFs pulled fresh from S3, chunked, embedded into ChromaDB, and consumed by the RAG-ground step. Annotations: tenacity retry box around every LLM call; cost tracker accumulator collecting prompt/completion tokens; JSON-line log file written on every row. |
| row-enrichment-fanout | plans/extra_08_data_engineer_usecase/diagrams/row-enrichment-fanout.mmd | Sequence diagram for one row: input row -> detect_pii (returns redacted body) -> classify_with_schema (Structured Outputs, returns dict with intent, urgency, product) -> retrieve(redacted body, collection) -> grounded_answer (gpt-4o with retrieved chunks) -> validate_output (regex + moderation). All five outputs are flattened into new DataFrame columns. Failure of any single step is caught, logged, and the row is marked `enrichment_status="failed"` rather than crashing the whole pipeline (dead-letter pattern). |
| sync-vs-batch | plans/extra_08_data_engineer_usecase/diagrams/sync-vs-batch.mmd | Cost vs latency tradeoff matrix. X-axis: rows per day. Y-axis: cost. Three lines: synchronous Chat Completions (slow, expensive at scale), parallel Chat Completions with thread pool (faster, same unit cost), OpenAI Batch API (24h SLA, 50 percent discount). Annotation: under 1k rows/day stay synchronous; 1k-100k rows use threaded parallel; over 100k rows or non-urgent SLA use Batch. |

## Session Timing

This topic is primarily a self-paced reference / read-through notebook. If used as a guided walkthrough by an instructor:

| Section | Duration | Type |
|---------|----------|------|
| 0 - Setup, S3 fetch, ChromaDB rebuild | 10 min | Demo |
| 1 - The pipeline scenario and DataFrame | 5 min | Demo |
| 2 - Enrichment 1: classification with Structured Outputs | 12 min | Demo |
| 3 - Enrichment 2: PII redaction before logging | 8 min | Demo |
| 4 - Enrichment 3: RAG-grounded suggested answer | 12 min | Demo |
| 5 - Enrichment 4: output validation | 8 min | Demo |
| 6 - Composition: enrich_row and enrich_dataframe with retries and cost tracking | 15 min | Demo |
| 7 - Cost and latency report | 5 min | Demo |
| 8 - When to use the OpenAI Batch API | 8 min | Discussion + minimal code |
| 9 - Production checklist for data engineers | 5 min | Wrap |
| **Total** | **~85 min** | |

---

# MAIN NOTEBOOK - Cell-by-Cell Content (Target: ~28 cells)

## Cell 0 - Markdown: Title and audience framing

```markdown
# Extra Topic: Prompt Engineering for Data Engineers

In Topics 1 to 9 we built a customer-facing chatbot. The chatbot is one shape of LLM application. Today I want to show you a completely different shape that uses the exact same primitives: a batch enrichment pipeline.

## Who this notebook is for

You run a daily ETL job at Barclays. The job lands a table of customer support tickets in the warehouse. Product wants every row to carry six new columns:

1. `intent` - one of `account_inquiry`, `card_services`, `loans`, `investments`, `complaint`, `general`.
2. `urgency` - `low`, `medium`, `high`.
3. `product_mentioned` - the Barclays product name extracted from the body.
4. `body_redacted` - the body with UK PII (NI numbers, sort codes, account numbers, postcodes) replaced by placeholders.
5. `suggested_response` - a draft reply grounded in the Barclays product PDFs.
6. `output_safe` - a boolean from the FCA financial-advice boundary check.

You have 50 rows today, 5,000 next month, 50,000 by Q3. You need a pipeline that is cheap, idempotent, observable, and never logs raw PII.

## What I am NOT going to do

I am not going to introduce new concepts. Every primitive in this notebook is a direct re-use of:

- Structured Outputs (`response_format={"type": "json_schema"}`) from Topic 4
- Embeddings + ChromaDB retrieval from Topic 6
- The PII regex registry and output validator from Topic 8
- Tenacity retry and per-request cost tracking from Topic 9

What is new is the SHAPE: row-level fan-out instead of a chat loop. The same primitives compose differently.

## The hard rule for this notebook

This notebook is fully self-contained. It does NOT import anything from prior notebooks via Python globals. The only handoff is via the filesystem: S3 holds the Barclays product PDFs, this notebook downloads them fresh, runs the Topic 2 extract-clean-chunk pipeline inline, builds the ChromaDB collection inline, and then runs the enrichment pipeline. If S3 is unreachable we fall back to a small inline corpus.
```

## Cell 1 - Markdown: Section 0 - Environment setup

```markdown
## Section 0: Environment setup

Same setup pattern as every other notebook in this course. Pin every library, never hardcode keys, always pin numpy<2 because chromadb 1.x will break on numpy 2.x.
```

## Cell 2 - Code: Install pinned dependencies

```python
%pip install -q "openai==2.32.0" "chromadb==1.5.8" "tenacity==9.1.4" "pymupdf==1.27.2.2" "tiktoken==0.9.0" "numpy<2"
```

## Cell 3 - Code: Imports, SageMaker session, OpenAI key, S3 client

```python
import os
import re
import io
import json
import time
import uuid
import getpass
import logging
import unicodedata
from dataclasses import dataclass, field
from typing import Any, Dict, List, Optional, Callable

import boto3
import pymupdf
import pandas as pd
import sagemaker
from sagemaker import get_execution_role
from openai import OpenAI, OpenAIError, RateLimitError, APIConnectionError, APITimeoutError
from tenacity import retry, stop_after_attempt, wait_random_exponential, retry_if_exception_type
import chromadb

if not os.environ.get("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API key: ")

sess = sagemaker.Session()
role = get_execution_role()
s3 = boto3.client("s3", region_name="us-east-2")
client = OpenAI()

MODEL = "gpt-4o"
EMBED_MODEL = "text-embedding-3-small"
MODERATION_MODEL = "omni-moderation-latest"
S3_BUCKET = "barclays-prompt-eng-data"

GPT4O_INPUT_PRICE_PER_1K = 0.0025
GPT4O_OUTPUT_PRICE_PER_1K = 0.01
EMBED_PRICE_PER_1K = 0.00002

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s", force=True)
log = logging.getLogger("pipeline")
log.info("Pipeline session ready. Role=%s, bucket=%s", role, S3_BUCKET)
```

## Cell 4 - Markdown: Why we rebuild the corpus inline every run

```markdown
### Filesystem-only handoff

I rebuild the Barclays product corpus from scratch in this notebook. I do NOT import `chunks`, `collection`, or any other variable from a prior topic. The reason is operational: in production a daily ETL job runs in its own process; it cannot rely on a developer notebook holding live state. The pipeline must be reproducible from cold storage every single run. Same rule applies to this notebook: cold start, fetch from S3, rebuild.
```

## Cell 5 - Code: Download Barclays PDFs from S3 with graceful fallback

```python
PDF_KEYS = [
    "barclays_personal_loan_faq.pdf",
    "barclays_credit_card_tnc.pdf",
]

INLINE_FALLBACK_DOCS = [
    "Barclaycard Platinum credit card. Representative APR 24.9% variable. Annual fee: none. 0% on purchases for 18 months from account opening.",
    "Barclays Personal Loan. Loans from GBP 1,000 to GBP 50,000. Representative APR 6.5% on amounts of GBP 7,500 to GBP 25,000. Early repayment is permitted.",
    "Barclays Premier Savings. Variable rate of 4.10% AER paid monthly. Minimum opening balance GBP 1. Withdrawals permitted at any time without penalty.",
]

def fetch_pdf_bytes(bucket: str, key: str) -> Optional[bytes]:
    try:
        resp = s3.get_object(Bucket=bucket, Key=key)
        return resp["Body"].read()
    except Exception as exc:
        log.warning("S3 fetch failed for %s: %s", key, exc)
        return None

raw_docs: List[str] = []
for key in PDF_KEYS:
    payload = fetch_pdf_bytes(S3_BUCKET, key)
    if payload is None:
        continue
    with pymupdf.open(stream=payload, filetype="pdf") as pdf:
        text = "\n".join(page.get_text() for page in pdf)
    raw_docs.append(text)
    log.info("Loaded %s (%d chars)", key, len(text))

if not raw_docs:
    log.warning("S3 unreachable; using inline fallback corpus.")
    raw_docs = list(INLINE_FALLBACK_DOCS)

log.info("Corpus size: %d documents", len(raw_docs))
```

## Cell 6 - Code: Clean and chunk (Topic 2 pattern, inline)

```python
def clean_pdf_text(text: str) -> str:
    text = unicodedata.normalize("NFKC", text)
    text = re.sub(r"-\n", "", text)
    text = re.sub(r"[ \t]+", " ", text)
    text = re.sub(r"\n{3,}", "\n\n", text)
    return text.strip()

def chunk_text(text: str, chunk_size: int = 800, overlap: int = 100) -> List[str]:
    text = text.strip()
    if not text:
        return []
    chunks: List[str] = []
    start = 0
    while start < len(text):
        end = min(start + chunk_size, len(text))
        chunks.append(text[start:end])
        if end == len(text):
            break
        start = end - overlap
    return chunks

cleaned = [clean_pdf_text(d) for d in raw_docs]
all_chunks: List[str] = []
for doc in cleaned:
    all_chunks.extend(chunk_text(doc, chunk_size=800, overlap=100))

log.info("Built %d chunks across %d documents", len(all_chunks), len(cleaned))
```

## Cell 7 - Code: Build ChromaDB collection inline

```python
chroma_client = chromadb.PersistentClient(path="./chroma_db_extra08")
collection = chroma_client.get_or_create_collection(
    name="barclays_products_extra08",
    configuration={"hnsw": {"space": "cosine"}},
)

def embed_texts(texts: List[str]) -> List[List[float]]:
    resp = client.embeddings.create(model=EMBED_MODEL, input=texts)
    return [d.embedding for d in resp.data]

if collection.count() == 0:
    embeddings = embed_texts(all_chunks)
    ids = [f"chunk_{i}" for i in range(len(all_chunks))]
    collection.add(ids=ids, documents=all_chunks, embeddings=embeddings)
    log.info("Embedded and stored %d chunks", len(all_chunks))
else:
    log.info("Reusing existing collection with %d chunks", collection.count())
```

## Cell 8 - Markdown: The ETL scenario and pipeline diagram

```markdown
## The pipeline we are building

I am going to generate a small synthetic DataFrame of 50 customer support tickets. In production this would be a daily landing table from your ticketing system. The pipeline reads a row, runs four enrichments, and writes the enriched row to a downstream table.

<!-- DIAGRAM: pipeline-overview.mmd -->

[View diagram](../../plans/extra_08_data_engineer_usecase/diagrams/pipeline-overview.mmd)

The diagram above shows the full flow: the raw ticket lands as one row of the input DataFrame, passes through PII redaction first (so nothing past this point sees raw PII), then classification, then RAG grounding, then output validation. Every LLM call is wrapped in a tenacity retry. Every row produces a single JSON-line log entry containing prompt and completion tokens, latency, and the enrichment status. The Barclays PDFs we just chunked are the knowledge source for the RAG step.
```

## Cell 9 - Code: Generate synthetic input DataFrame

```python
SYNTHETIC_TICKETS = [
    "Hi, my Barclaycard was declined yesterday at Tesco. My account is 12345678 and sort code 12-34-56. Can you help?",
    "What is the current APR on the Barclays Personal Loan for GBP 10,000?",
    "I want to close my Premier Savings account. My NI is AB123456C.",
    "My card was lost on holiday. Please freeze it immediately. Account 87654321.",
    "Can I overpay my personal loan without penalty?",
    "I think I have been scammed. Someone called pretending to be from Barclays.",
    "What is the interest rate on the Premier Savings account today?",
    "Customer SW1A 1AA reports a duplicate direct debit on account 11223344.",
    "Please send me a statement for the last 6 months for sort code 20-00-00 account 99887766.",
    "I would like to apply for a personal loan. What APR can I get?",
]

rows = []
for i in range(50):
    body = SYNTHETIC_TICKETS[i % len(SYNTHETIC_TICKETS)]
    rows.append({
        "ticket_id": str(uuid.uuid4()),
        "received_at": pd.Timestamp.utcnow().isoformat(),
        "body": body,
    })

tickets_df = pd.DataFrame(rows)
log.info("Synthetic input: %d rows", len(tickets_df))
tickets_df.head(3)
```

## Cell 10 - Markdown: Enrichment 1 - structured classification

```markdown
## Enrichment 1: structured classification with Structured Outputs

For a chatbot you can tolerate a free-form response. For a data pipeline you cannot. Every row must produce a typed dict that lands cleanly in DataFrame columns. The right OpenAI primitive here is `response_format={"type": "json_schema", "strict": true}` (Structured Outputs). It guarantees the model returns valid JSON conforming to the schema OR refuses; it never produces malformed text.

I am asking for three fields per ticket: `intent`, `urgency`, `product_mentioned`.

<!-- DIAGRAM: row-enrichment-fanout.mmd -->

[View diagram](../../plans/extra_08_data_engineer_usecase/diagrams/row-enrichment-fanout.mmd)
```

## Cell 11 - Code: classify_with_schema using Structured Outputs

```python
CLASSIFICATION_SCHEMA = {
    "name": "ticket_classification",
    "strict": True,
    "schema": {
        "type": "object",
        "additionalProperties": False,
        "properties": {
            "intent": {
                "type": "string",
                "enum": ["account_inquiry", "card_services", "loans", "investments", "complaint", "general"],
            },
            "urgency": {"type": "string", "enum": ["low", "medium", "high"]},
            "product_mentioned": {"type": "string"},
        },
        "required": ["intent", "urgency", "product_mentioned"],
    },
}

CLASSIFY_SYSTEM = (
    "You classify Barclays customer support tickets. "
    "Return only the schema fields. If no specific product is mentioned, set product_mentioned to 'none'."
)

def classify_with_schema(body: str) -> Dict[str, str]:
    resp = client.chat.completions.create(
        model=MODEL,
        temperature=0,
        response_format={"type": "json_schema", "json_schema": CLASSIFICATION_SCHEMA},
        messages=[
            {"role": "system", "content": CLASSIFY_SYSTEM},
            {"role": "user", "content": body},
        ],
    )
    return json.loads(resp.choices[0].message.content)

print(classify_with_schema("My card was lost on holiday. Please freeze it immediately."))
print(classify_with_schema("What is the APR on the Personal Loan?"))
```

## Cell 12 - Markdown: Enrichment 2 - PII redaction BEFORE anything else

```markdown
## Enrichment 2: PII redaction before logging or model calls

In a chatbot the PII guardrail is the FIRST thing that runs against user input. Same here, with one change in framing: in a batch pipeline the LOG is the bigger risk. A pipeline that runs nightly can quietly write 50,000 raw NI numbers to CloudWatch in a year. That is a reportable GDPR Article 5(1)(c) breach.

So the rule for this pipeline is: redact first, log only the redacted body, never the raw body. The redaction registry is the same one from Topic 8.
```

## Cell 13 - Code: GuardrailResult and detect_pii (Topic 8 pattern)

```python
@dataclass(frozen=True)
class GuardrailResult:
    passed: bool
    reason: str
    redacted_text: Optional[str] = None
    details: Dict[str, Any] = field(default_factory=dict)

PII_PATTERNS: Dict[str, re.Pattern] = {
    "uk_ni_number": re.compile(
        r"\b[A-CEGHJ-PR-TW-Z][A-CEGHJ-NPR-TW-Z]\s?\d{2}\s?\d{2}\s?\d{2}\s?[A-D]\b",
        re.IGNORECASE,
    ),
    "uk_sort_code": re.compile(r"\b\d{2}[-\s]?\d{2}[-\s]?\d{2}\b"),
    "uk_account_number": re.compile(r"(?<!\d)\d{8}(?!\d)"),
    "uk_postcode": re.compile(
        r"\b([Gg][Ii][Rr] 0[Aa]{2}|"
        r"[A-Za-z][0-9][A-Za-z0-9]?\s?[0-9][A-Za-z]{2}|"
        r"[A-Za-z][A-Za-z][0-9][A-Za-z0-9]?\s?[0-9][A-Za-z]{2})\b"
    ),
}

def detect_pii(text: str) -> GuardrailResult:
    if not isinstance(text, str) or not text:
        return GuardrailResult(passed=True, reason="empty", redacted_text=text or "")
    norm = unicodedata.normalize("NFKC", text)
    redacted = norm
    matched: Dict[str, int] = {}
    for label, pattern in PII_PATTERNS.items():
        hits = pattern.findall(redacted)
        if hits:
            matched[label] = len(hits)
            redacted = pattern.sub(f"[REDACTED_{label.upper()}]", redacted)
    if matched:
        return GuardrailResult(passed=False, reason="pii_detected", redacted_text=redacted, details={"matched": matched})
    return GuardrailResult(passed=True, reason="no_pii", redacted_text=redacted)

print(detect_pii("My account is 12345678 and sort code 12-34-56."))
```

## Cell 14 - Markdown: Enrichment 3 - RAG-grounded suggested response

```markdown
## Enrichment 3: RAG-grounded suggested response

The classification step gave us the intent. Now we want a draft reply grounded in the Barclays product PDFs we loaded in Section 0. This is exactly the Topic 6 RAG pattern, applied per row.

A note on web search: in the chatbot, Topic 7 added live web search for queries about current rates. In a BATCH pipeline running 50,000 rows nightly, web search is usually the wrong call. It is slow (1-3 seconds per call), it costs more, and it provides freshness you do not need on a 24-hour pipeline. If a row genuinely needs live data, route it to a separate, smaller "live-data" sub-pipeline. The bulk pipeline stays on the static vector store.
```

## Cell 15 - Code: retrieve and rag_ground

```python
def retrieve(query: str, n_results: int = 3) -> List[str]:
    q_emb = embed_texts([query])[0]
    results = collection.query(query_embeddings=[q_emb], n_results=n_results)
    docs = results.get("documents", [[]])
    return docs[0] if docs else []

GROUND_SYSTEM = (
    "You are a Barclays product knowledge assistant. "
    "Answer the customer's question using ONLY the provided context. "
    "If the answer is not in the context, say 'I do not have that information.' "
    "Do not give personal recommendations; share factual product information only."
)

def rag_ground(redacted_body: str) -> Dict[str, Any]:
    chunks_used = retrieve(redacted_body, n_results=3)
    context = "\n\n---\n\n".join(chunks_used) if chunks_used else "(no context available)"
    user_block = f"Context:\n{context}\n\nCustomer message:\n{redacted_body}"
    resp = client.chat.completions.create(
        model=MODEL,
        temperature=0.2,
        messages=[
            {"role": "system", "content": GROUND_SYSTEM},
            {"role": "user", "content": user_block},
        ],
    )
    answer = resp.choices[0].message.content
    return {
        "answer": answer,
        "n_chunks_used": len(chunks_used),
        "prompt_tokens": resp.usage.prompt_tokens,
        "completion_tokens": resp.usage.completion_tokens,
    }

demo = rag_ground("What is the APR on the Personal Loan?")
print(demo["answer"][:300])
print("chunks_used=", demo["n_chunks_used"], "tokens=", demo["prompt_tokens"], "/", demo["completion_tokens"])
```

## Cell 16 - Markdown: Enrichment 4 - output validation

```markdown
## Enrichment 4: output validation

The model can still produce a personal recommendation by accident, even when grounded. The Topic 8 output validator catches it: regex pass for "you should", "I recommend", "best for you", then OpenAI Moderation API for harmful content. We mark the row `output_safe=False` if validation fails. We do NOT discard the row; we keep it and let downstream review pick it up. Discarding rows silently is the worst outcome in a data pipeline.
```

## Cell 17 - Code: validate_output

```python
RECOMMENDATION_PATTERNS: List[re.Pattern] = [
    re.compile(r"\byou should (open|take|invest|buy|switch|move|consider)\b", re.IGNORECASE),
    re.compile(r"\bi recommend (that you )?\b", re.IGNORECASE),
    re.compile(r"\b(is|are) best for (you|people like you|your situation)\b", re.IGNORECASE),
]

def validate_output(text: str) -> GuardrailResult:
    if not isinstance(text, str) or not text:
        return GuardrailResult(passed=True, reason="empty")
    for pattern in RECOMMENDATION_PATTERNS:
        if pattern.search(text):
            return GuardrailResult(passed=False, reason="recommendation_phrase", details={"pattern": pattern.pattern})
    try:
        mod = client.moderations.create(model=MODERATION_MODEL, input=text)
        if mod.results[0].flagged:
            return GuardrailResult(passed=False, reason="moderation_flagged", details={"categories": dict(mod.results[0].categories)})
    except OpenAIError as exc:
        log.error("Moderation error, failing safe: %s", exc)
        return GuardrailResult(passed=False, reason="moderation_error_fail_safe")
    return GuardrailResult(passed=True, reason="ok")

print(validate_output("The current Personal Loan representative APR is 6.5%."))
print(validate_output("You should take the Personal Loan because it is best for you."))
```

## Cell 18 - Markdown: Composing the row-level pipeline

```markdown
## Composition: enrich_row and enrich_dataframe

Now we wire the four enrichments into one `enrich_row(row)` callable, wrap every LLM call in a tenacity retry, and log a single JSON line per row. Then we map it across the DataFrame.

The retry policy is the Topic 9 policy: `wait_random_exponential(multiplier=1, max=60)` with `stop_after_attempt(6)` for transient OpenAI errors. We do NOT retry on schema validation errors or PII regex errors - those are deterministic and would just fail again.

We catch any exception at the row level and mark `enrichment_status="failed"`. This is the dead-letter pattern: a single bad row never crashes a 50,000-row job.
```

## Cell 19 - Code: tenacity-decorated wrappers and enrich_row

```python
TRANSIENT = (RateLimitError, APIConnectionError, APITimeoutError)

@retry(
    stop=stop_after_attempt(6),
    wait=wait_random_exponential(multiplier=1, max=60),
    retry=retry_if_exception_type(TRANSIENT),
    reraise=True,
)
def safe_classify(body: str) -> Dict[str, str]:
    return classify_with_schema(body)

@retry(
    stop=stop_after_attempt(6),
    wait=wait_random_exponential(multiplier=1, max=60),
    retry=retry_if_exception_type(TRANSIENT),
    reraise=True,
)
def safe_ground(redacted_body: str) -> Dict[str, Any]:
    return rag_ground(redacted_body)

def enrich_row(row: Dict[str, Any]) -> Dict[str, Any]:
    started = time.perf_counter()
    out: Dict[str, Any] = dict(row)
    try:
        pii = detect_pii(row["body"])
        out["body_redacted"] = pii.redacted_text
        out["pii_matched"] = pii.details.get("matched", {})

        cls = safe_classify(out["body_redacted"])
        out.update(cls)

        ground = safe_ground(out["body_redacted"])
        out["suggested_response"] = ground["answer"]
        out["rag_chunks_used"] = ground["n_chunks_used"]
        out["prompt_tokens"] = ground["prompt_tokens"]
        out["completion_tokens"] = ground["completion_tokens"]

        check = validate_output(out["suggested_response"])
        out["output_safe"] = check.passed
        out["output_reason"] = check.reason

        out["enrichment_status"] = "ok"
    except Exception as exc:
        out["enrichment_status"] = "failed"
        out["enrichment_error"] = str(exc)[:200]
        out.setdefault("body_redacted", "")
        out.setdefault("prompt_tokens", 0)
        out.setdefault("completion_tokens", 0)
    finally:
        out["latency_ms"] = int((time.perf_counter() - started) * 1000)
        log.info(json.dumps({
            "ticket_id": out.get("ticket_id"),
            "status": out.get("enrichment_status"),
            "intent": out.get("intent"),
            "output_safe": out.get("output_safe"),
            "latency_ms": out.get("latency_ms"),
            "prompt_tokens": out.get("prompt_tokens", 0),
            "completion_tokens": out.get("completion_tokens", 0),
        }))
    return out
```

## Cell 20 - Code: enrich_dataframe and run on the synthetic data

```python
def enrich_dataframe(df: pd.DataFrame) -> pd.DataFrame:
    enriched_rows = [enrich_row(r) for r in df.to_dict("records")]
    return pd.DataFrame(enriched_rows)

run_started = time.perf_counter()
enriched_df = enrich_dataframe(tickets_df.head(10))
run_seconds = time.perf_counter() - run_started

log.info("Processed %d rows in %.1f s", len(enriched_df), run_seconds)
enriched_df[["ticket_id", "intent", "urgency", "product_mentioned", "output_safe", "enrichment_status", "latency_ms"]].head()
```

## Cell 21 - Markdown: Cost and latency reporting

```markdown
## Cost and latency report

Every row carries `prompt_tokens` and `completion_tokens` from the RAG step (the classification step has small fixed cost we are folding into the same accounting for simplicity in this report). We can compute cost per row, total cost, p50 latency, p95 latency, and the failure rate. These five numbers are exactly what your platform team will ask for before this pipeline goes to production.
```

## Cell 22 - Code: cost and latency report

```python
def cost_report(df: pd.DataFrame) -> Dict[str, Any]:
    completed = df[df["enrichment_status"] == "ok"]
    total_in = completed["prompt_tokens"].sum()
    total_out = completed["completion_tokens"].sum()
    total_cost = (total_in / 1000.0) * GPT4O_INPUT_PRICE_PER_1K + (total_out / 1000.0) * GPT4O_OUTPUT_PRICE_PER_1K
    latencies = df["latency_ms"].tolist()
    latencies.sort()
    p50 = latencies[len(latencies) // 2] if latencies else 0
    p95 = latencies[int(len(latencies) * 0.95)] if latencies else 0
    return {
        "rows_total": len(df),
        "rows_ok": int((df["enrichment_status"] == "ok").sum()),
        "rows_failed": int((df["enrichment_status"] == "failed").sum()),
        "total_input_tokens": int(total_in),
        "total_output_tokens": int(total_out),
        "total_cost_usd": round(total_cost, 6),
        "cost_per_row_usd": round(total_cost / max(1, len(completed)), 6),
        "latency_p50_ms": p50,
        "latency_p95_ms": p95,
    }

print(json.dumps(cost_report(enriched_df), indent=2))
```

## Cell 23 - Markdown: When to use the OpenAI Batch API instead

```markdown
## When to use the OpenAI Batch API

The synchronous pipeline above is the right default up to about 1,000 rows per run. Beyond that, two options exist:

1. Threaded parallel sync calls (e.g. `concurrent.futures.ThreadPoolExecutor` with 8-32 workers) for sub-hour SLAs.
2. The OpenAI Batch API for jobs where a 24-hour SLA is acceptable. The Batch API runs your requests asynchronously over up to 24 hours and returns results at a 50 percent discount on per-token pricing.

Rule of thumb:

- Under 1k rows/day or interactive: stay synchronous.
- 1k-100k rows/day with a sub-hour SLA: parallel synchronous with a thread pool, plus tenacity. Watch your tier rate limits.
- Over 100k rows/day or any non-urgent SLA (overnight backfills, monthly re-classifications, historical re-scoring): use the Batch API.

<!-- DIAGRAM: sync-vs-batch.mmd -->

[View diagram](../../plans/extra_08_data_engineer_usecase/diagrams/sync-vs-batch.mmd)
```

## Cell 24 - Code: Batch API minimal sketch (do not run unless you want to wait)

```python
def build_batch_jsonl(df: pd.DataFrame, path: str) -> str:
    with open(path, "w") as fh:
        for _, row in df.iterrows():
            request = {
                "custom_id": row["ticket_id"],
                "method": "POST",
                "url": "/v1/chat/completions",
                "body": {
                    "model": MODEL,
                    "temperature": 0,
                    "response_format": {"type": "json_schema", "json_schema": CLASSIFICATION_SCHEMA},
                    "messages": [
                        {"role": "system", "content": CLASSIFY_SYSTEM},
                        {"role": "user", "content": row["body"]},
                    ],
                },
            }
            fh.write(json.dumps(request) + "\n")
    return path

batch_path = build_batch_jsonl(tickets_df, "/tmp/batch_classify.jsonl")
log.info("Wrote batch JSONL: %s", batch_path)
```

## Cell 25 - Markdown: Production checklist for data engineers

```markdown
## Production checklist

Before you ship a pipeline like this to a real ETL schedule, walk down this list:

1. **Idempotency**: every row carries a stable `ticket_id`. If the job retries, the downstream merge should be UPSERT on `ticket_id`, not INSERT. Otherwise a transient failure produces duplicate enriched rows.
2. **Retry storms**: tenacity backoff is per-call. If the OpenAI API is fully down, a 50,000-row job with 6 retries each becomes 300,000 calls hammering a degraded service. Add a circuit breaker (e.g. fail the whole job if the first 50 rows fail) before turning on a large schedule.
3. **Dead-letter queue**: rows with `enrichment_status="failed"` should land in a separate table or S3 prefix for manual inspection. Do not silently drop them; do not re-queue them in the same run.
4. **Schema evolution**: the Structured Outputs `CLASSIFICATION_SCHEMA` is checked into source control. When you add a new intent (e.g. `mortgage`), bump a schema version field; do not break downstream consumers.
5. **Monitoring**: emit four metrics per run - rows_total, rows_ok, rows_failed, total_cost_usd. Alarm on `rows_failed / rows_total > 5 percent` and on `total_cost_usd > <budget>`.
6. **PII in logs**: log only redacted bodies and only structural fields (intent, urgency, status). Never log raw `body`. CloudWatch retention is typically months; raw PII in logs is the easiest way to a GDPR incident.
7. **Cost ceiling**: keep `EMBED_PRICE_PER_1K` and the gpt-4o pricing constants in one place. When OpenAI changes prices, you change one file. Consider a hard pre-flight estimate that aborts the run if the projected cost exceeds a budget envelope.
8. **Model versioning**: pin to a specific model snapshot (e.g. `gpt-4o-2024-08-06`) for any production schedule so an upstream model change does not silently shift your enrichment distribution. The `gpt-4o` alias is fine for development; pin for production.
9. **Re-runs and backfills**: a backfill across 6 months of historical tickets should reuse the same `enrich_row` function. The function must be deterministic enough (temperature=0 on classification) that a re-run produces equivalent results. Keep classification temperature at 0; the grounded answer can be slightly higher.
10. **Local dry runs**: every change to the pipeline should run on a 10-row sample before being scheduled. The cost report cell above is exactly that pre-flight.

## Wrap

You have seen the same primitives from the chatbot - Structured Outputs, RAG, PII redaction, output validation, retries, cost tracking - composed into a row-level batch enrichment pipeline. The chatbot and the pipeline share 90 percent of the code; only the SHAPE differs (request/response loop vs row fan-out). When the next request comes in for "we need an LLM somewhere in this ETL", you already have the building blocks.
```

---

# VERIFICATION CHECKLIST

## Environment

- [ ] `%pip install -q "openai==2.32.0" "chromadb==1.5.8" "tenacity==9.1.4" "pymupdf==1.27.2.2" "tiktoken==0.9.0" "numpy<2"` runs clean on ml.t3.medium.
- [ ] `getpass` prompt fires when `OPENAI_API_KEY` not set.
- [ ] `sagemaker.Session()` and `get_execution_role()` succeed; `boto3.client("s3", region_name="us-east-2")` succeeds.
- [ ] `numpy<2` pinned (chromadb 1.x compatibility).

## Self-containment

- [ ] No import from prior topic notebooks; no Python global from prior notebooks referenced.
- [ ] PDFs fetched fresh from `barclays-prompt-eng-data` S3 bucket per run.
- [ ] Inline fallback corpus triggers cleanly when S3 is unreachable.
- [ ] ChromaDB collection rebuilt inline; `get_or_create_collection` reuses cache when present.

## Functional correctness

- [ ] `detect_pii` correctly redacts NI number, sort code, 8-digit account, UK postcode on the demo input.
- [ ] `classify_with_schema` returns a dict with keys `intent`, `urgency`, `product_mentioned`, all conforming to the enum.
- [ ] `rag_ground` returns dict with `answer`, `n_chunks_used`, `prompt_tokens`, `completion_tokens`.
- [ ] `validate_output` blocks "you should take..." style phrases via regex; passes factual product statements.
- [ ] `enrich_row` produces a fully-enriched row with `enrichment_status="ok"` on the happy path.
- [ ] `enrich_row` catches exceptions, marks `enrichment_status="failed"`, never raises.
- [ ] `enrich_dataframe(tickets_df.head(10))` produces a 10-row enriched DataFrame.
- [ ] `cost_report` returns the seven required keys (rows_total, rows_ok, rows_failed, total_cost_usd, cost_per_row_usd, latency_p50_ms, latency_p95_ms).
- [ ] `build_batch_jsonl` writes a valid JSONL batch file to `/tmp/batch_classify.jsonl`.

## Pedagogical structure

- [ ] No `None # YOUR CODE` placeholders anywhere.
- [ ] No tier-labelled labs.
- [ ] Each code cell is a complete runnable demo with heavy single-line comments.
- [ ] Each markdown cell explains WHY before the code, not just WHAT.
- [ ] Three diagram placeholders present (`pipeline-overview`, `row-enrichment-fanout`, `sync-vs-batch`).
- [ ] First-person instructor voice throughout.
- [ ] Cell count between 25 and 30 (target ~28).

## AI-tells

- [ ] No em dashes, no en dashes, no Unicode multiplication, no bare `---` separators in markdown cells.
- [ ] No bullet unicode in code stdout strings.
- [ ] Single-line comments only in code cells.

# RESEARCH VALIDATED (April 2026)

This is a re-application topic. Per user instruction NO new web research was performed. Every architectural decision below cites a previously validated source already documented in the existing topic plans.

## Reused from `plans/topic_02_nlp_preprocessing.md`

- S3 access pattern `boto3.client('s3', region_name='us-east-2').get_object(Bucket=..., Key=...)["Body"].read()` (T2 plan, RESEARCH VALIDATED block).
- PyMuPDF extraction with `pymupdf.open(stream=..., filetype="pdf")` and `page.get_text()` per page (T2 plan).
- Cleaning pipeline (NFKC normalisation, hyphenated line-break repair, whitespace normalisation, blank-line collapse) (T2 plan).
- Fixed-size chunking with 100-character overlap (T2 plan).

## Reused from `plans/topic_04_prompt_engineering.md`

- `response_format={"type": "json_schema", "strict": true}` Structured Outputs as the right primitive when you need a typed dict per row (T4 plan).
- `INTENT_CATEGORIES` enum shape; we extend it with `complaint` for the ticket-classification scenario but reuse the closed-enum pattern (T4 plan).

## Reused from `plans/topic_06_rag_foundations.md`

- `chromadb.PersistentClient(path="./chroma_db")` + `get_or_create_collection(name=..., configuration={"hnsw": {"space": "cosine"}})` (T6 plan).
- `embed_texts` returns vectors via `text-embedding-3-small` (T6 plan).
- `collection.query(query_embeddings=[q_emb], n_results=k)` retrieval contract (T6 plan).
- Augmented-prompt pattern: system prompt that instructs grounding only; user message contains `Context:\n...\n\nCustomer message:\n...` (T6 plan).

## Reused from `plans/topic_07_advanced_rag_web_search.md`

- Decision to NOT use web_search in batch context (live calls are slow + costly; batch jobs do not need sub-day freshness). The vector store is the right primary source. (T7 plan, hybrid router decision diagram and discussion.)

## Reused from `plans/topic_08_ethical_guardrails.md`

- `GuardrailResult` frozen dataclass shape with `passed`, `reason`, `redacted_text`, `details` (T8 plan, Cell 7).
- `PII_PATTERNS` registry: UK NI number (HMRC NIM39110 alphabet rules), UK sort code, 8-digit account number, full UK postcode regex (T8 plan, RESEARCH VALIDATED block citing https://www.gov.uk/hmrc-internal-manuals/national-insurance-manual/nim39110).
- `unicodedata.normalize("NFKC", text)` defence against homoglyphs and full-width digit evasion, per OWASP LLM Top 10 2025 (T8 plan).
- `omni-moderation-latest` is the recommended free moderation model (T8 plan, citing https://help.openai.com/en/articles/4936833-is-the-moderation-endpoint-free-to-use).
- Output recommendation regex registry (you should / I recommend / is best for you) anchored in PERG 8.30B.2G personal-recommendation definition (T8 plan).
- Logging-only-redacted-text rule rests on GDPR Article 5(1)(c) and Article 25 (T8 plan, citing https://gdpr-info.eu/art-5-gdpr/ and https://gdpr-info.eu/art-25-gdpr/).

## Reused from `plans/topic_09_capstone.md`

- tenacity policy: `stop_after_attempt(6)` + `wait_random_exponential(multiplier=1, max=60)` + `retry_if_exception_type((RateLimitError, APIConnectionError, APITimeoutError))` (T9 plan).
- Per-request cost formula: `(prompt_tokens / 1000) * GPT4O_INPUT_PRICE_PER_1K + (completion_tokens / 1000) * GPT4O_OUTPUT_PRICE_PER_1K` with constants `0.0025` and `0.01` (T9 plan, T1 carry-over).
- JSON-line structured logging per request as the production observability primitive (T9 plan, logging-pipeline diagram).
- Dead-letter pattern: catch exception at the row level, mark status, never let one row crash the job (T9 plan, retry-flow diagram annotation).

## Reused from `plans/CORE_TECHNOLOGIES_AND_DECISIONS.md`

- SageMaker ml.t3.medium, Python 3.11, no GPU, us-east-2 region.
- `barclays-prompt-eng-data` S3 bucket, read-only, instructor-loaded.
- gpt-4o default; OpenAI only; no Anthropic SDK.
- Credentials via `getpass` for API keys, `sagemaker.Session()` + `get_execution_role()` for AWS.
- numpy<2 pin in every install cell.

## OpenAI Batch API reference (cited in markdown only, no live API calls in the notebook)

- The Batch API is referenced as a 24-hour-SLA, 50-percent-discount asynchronous lane for high-volume jobs. The same primitive is mentioned in the T9 plan as a stretch direction. No new web fact is asserted here beyond what T9 already validated; if a fresh URL is needed at build time the canonical reference is `https://platform.openai.com/docs/guides/batch` (already implicit in T9 cost-optimisation discussion).

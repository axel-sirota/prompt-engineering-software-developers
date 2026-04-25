# Instructor Setup Notebook - Cell-by-Cell Plan

## Context

**What this is**: A single Jupyter notebook (`INSTRUCTOR_SETUP.ipynb`) that you (the instructor) run ONCE
from your own SageMaker Studio JupyterLab Space before the course begins. It populates the
`barclays-prompt-eng-data` S3 bucket with everything students need to run all 9 topic notebooks.

**When to run**: On the day of the course, as soon as you have SageMaker access. Target: run it while
students are doing their VM setup or the Day 1 introduction. It takes under 10 minutes.

**Why it exists**: Students have READ-ONLY access to `barclays-prompt-eng-data`. You (the instructor)
must pre-populate the bucket with all required files before any student notebook can load from S3.
Without this notebook, Topic 2 Cell 8 fails with `NoSuchKey`, and every S3 load in Topics 2-9 falls
back to inline hardcoded text instead of real Barclays documents.

**IAM permission requirement**: Your instructor SageMaker execution role must have `s3:PutObject` on
`barclays-prompt-eng-data`. The student role (`SageMakerStudentExecutionRole`) only has `s3:GetObject`
and `s3:ListBucket` - those students cannot run this notebook. If your role also only has read access,
ask Datacouch to add `s3:PutObject` for `arn:aws:s3:::barclays-prompt-eng-data/*` to your role.

**No teardown needed**: Datacouch handles environment cleanup after the cohort. This notebook only creates.

**ChromaDB note**: ChromaDB is NOT pre-built in S3. Each student's JupyterLab Space has its own
private EBS volume. Topics 6-9 already rebuild ChromaDB locally from inline BARCLAYS_DOCS fallback
strings if needed. That takes 3-5 seconds and requires no S3 access. Pre-building ChromaDB adds
complexity with no benefit given the 7-string corpus size.

**What this notebook uploads**:

| S3 Key | Source | Used By |
|--------|--------|---------|
| `barclays_personal_loan_faq.pdf` | Barclays and You T&C (Nov 2020) | Topic 2 Cell 8 demo, Cell 10 Lab 1 |
| `barclays_credit_card_tnc.pdf` | Barclaycard Core T&C (2023) | Topic 2 Cell 10 Lab 1, Cell 22 Lab 3 |
| `barclays_savings_rates.pdf` | Rates for Savers (Jan 2026) | Optional: Tier 3 lab stretch in Topic 6 |
| `barclays_isa_terms.pdf` | 1-Year Flexible Cash ISA Issue 79 (Nov 2025) | Optional: Tier 3 lab stretch in Topic 6 |
| `barclays_chunks.json` | Pre-chunked from all 4 PDFs above | Optional: richer fallback for Topics 6-9 |

**Content Mismatch Warning (IMPORTANT - read before the course)**:

The topic notebooks were written with specific illustrative numbers in the inline `BARCLAYS_DOCS`
fallback strings (used in Topics 6-9 when students skip Topic 2). These numbers are FICTIONAL:
- "Representative APR 6.5% for loans" - the real loan docs do not quote a specific APR (personalised)
- "3.75% AER" for savings - the real Rates for Savers PDF shows 1.05-4.20% depending on product
- "0.25% cashback" on Barclaycard Rewards - the real Core T&C does not quote cashback rates (varies)
- "27.9% APR variable" for Barclaycard - the real T&C says "see your statement" (personalised)

This is expected. The real PDFs are richer and more realistic for Topic 2's extraction demo. The
inline BARCLAYS_DOCS strings in Topics 6-9 are deliberately simple for teaching clarity. Do NOT
change the notebooks to remove the inline fallbacks - they are necessary when S3 is unavailable.

When students ask "Why does the loan APR in Topic 2 differ from the number in Topic 6?" the
answer is: Topic 2 uses a real Barclays document (which does not quote personal APRs) while
Topic 6 uses a simplified inline string for teaching purposes.

## Deliverables

- **Notebook**: `INSTRUCTOR_SETUP.ipynb` (at repo root, NOT in exercises/ or solutions/)

## Session Timing (~8 min)

| Section | Duration | Notes |
|---------|----------|-------|
| Section 0: Setup + Permission Check | 2 min | Fails fast if no write access |
| Section 1: Upload PDFs to S3 | 3 min | 4 PDFs, idempotent |
| Section 2: Pre-chunk + Upload chunks.json | 3 min | Requires OpenAI key - SKIP if no key yet |
| Final manifest print | 30 sec | Confirmation of all uploads |
| **Total** | **~8 min** | |

---

# MAIN NOTEBOOK - Cell-by-Cell Content (Target: 14 cells)

## Cell 0 - Markdown: Title and Purpose

**Type**: markdown

**Content**:

```
# Barclays Prompt Engineering Course - Instructor Setup Notebook

Run this notebook ONCE before the course starts to populate the
`barclays-prompt-eng-data` S3 bucket with all documents students need.

## What this does

1. Checks that your SageMaker execution role has write access to S3
2. Downloads 4 real public Barclays PDFs and uploads them to `barclays-prompt-eng-data`
3. (Optional, needs OpenAI key) Pre-chunks the PDFs and uploads `barclays_chunks.json`
4. Prints a manifest confirming all uploads

## When to run

Run this on the day of the course, before students start Topic 2.
Topics 1, 3, 4, 5 do not need S3 access - so you have time to run this
during the intro session or while students set up their VMs.

## Prerequisites

- Your instructor SageMaker execution role must have `s3:PutObject` on `barclays-prompt-eng-data`
- Students have `s3:GetObject` (read-only) - they cannot run this notebook
- Internet access from SageMaker to barclays.co.uk and barclaycard.co.uk (open internet on SageMaker)
```


## Cell 1 - Markdown: Section 0 - Setup and Permission Check

**Type**: markdown

**Content**:

```
## Section 0: Setup and Permission Check

Installing dependencies and verifying your role can write to S3.
This section will STOP with a clear error message if your role lacks write access
so you know immediately whether to contact Datacouch for a permission fix.
```


## Cell 2 - Code: Install and Imports

**Type**: code

**Content**:

```python
# Minimal installs - most are pre-installed on SageMaker Distribution image
# requests: for downloading PDFs from public Barclays URLs (may not be pre-installed)
# pymupdf: for chunking PDFs in Section 2 (pre-install to avoid delays later)
# numpy<2: always pin for SageMaker compatibility
!pip install -q requests "pymupdf==1.27.2.2" "numpy<2"

import boto3
import requests
import json
import os
import re
import textwrap
from io import BytesIO
from botocore.exceptions import ClientError

import sagemaker
from sagemaker import get_execution_role

# SageMaker session gives us the execution role and region
sess = sagemaker.Session()
role = get_execution_role()

S3_BUCKET = "barclays-prompt-eng-data"
REGION = "us-east-2"

# Create S3 client using the SageMaker execution role (credentials are automatic in SageMaker)
s3 = boto3.client("s3", region_name=REGION)

print(f"SageMaker region: {sess.boto_region_name}")
print(f"Execution role: ...{role[-40:]}")
print(f"S3 bucket: s3://{S3_BUCKET}/")
print()
print("Setup complete. Running permission check next.")
```


## Cell 3 - Code: Verify S3 Write Permission

**Type**: code

**Content**:

```python
# Verify that this role can write to the S3 bucket.
# We write a tiny probe object and then delete it.
# If this fails with AccessDenied, contact Datacouch to add s3:PutObject to your role.

PROBE_KEY = "_setup_probe_delete_me.txt"

try:
    s3.put_object(
        Bucket=S3_BUCKET,
        Key=PROBE_KEY,
        Body=b"instructor setup probe",
        ContentType="text/plain",
    )
    s3.delete_object(Bucket=S3_BUCKET, Key=PROBE_KEY)
    print("[OK] Write access confirmed: your role can PutObject to s3://" + S3_BUCKET + "/")
    print("[OK] Delete access confirmed: probe object cleaned up.")
    print()
    print("You are ready to run this notebook. Proceed to Section 1.")

except ClientError as e:
    error_code = e.response["Error"]["Code"]
    if error_code in ("AccessDenied", "403"):
        print("[STOP] AccessDenied: your execution role cannot write to s3://" + S3_BUCKET + "/")
        print()
        print("To fix this, ask Datacouch to add this inline policy to your instructor role:")
        print(json.dumps({
            "Version": "2012-10-17",
            "Statement": [{
                "Effect": "Allow",
                "Action": ["s3:PutObject", "s3:DeleteObject"],
                "Resource": "arn:aws:s3:::" + S3_BUCKET + "/*"
            }]
        }, indent=2))
        raise RuntimeError("Fix S3 permissions before continuing.")
    else:
        print(f"[STOP] Unexpected error: {error_code}")
        print(e)
        raise
```


## Cell 4 - Markdown: Section 1 - Upload Barclays PDFs

**Type**: markdown

**Content**:

```
## Section 1: Upload Real Barclays PDFs to S3

These are publicly available PDFs from barclays.co.uk and barclaycard.co.uk.
No login required. No licence restrictions for educational use.

The 4 PDFs we upload:

1. `barclays_personal_loan_faq.pdf` - "Barclays and You" personal banking T&C (Nov 2020).
   Used in Topic 2 Cell 8 (the main demo: load_pdf_from_s3) and Cell 10 (Lab 1).
   Note: this document covers all personal banking (current accounts, savings, overdrafts,
   loans). It does NOT quote a specific personal loan APR - rates are personalised.
   The Topic 2 notebooks' inline fallback text ("58 days interest ERC") comes from
   a simplified version of the actual ERC wording in this document.

2. `barclays_credit_card_tnc.pdf` - Barclaycard Core T&C (2023).
   Used in Topic 2 Cell 10 (Lab 1: load the credit card doc) and Cell 22 (Lab 3 pipeline).
   Note: this document does not quote specific interest rates (personalised). Students
   will extract the structure of the agreement, not specific APRs.

3. `barclays_savings_rates.pdf` - Rates for Savers (Jan 2026).
   Not required by any topic notebook directly. Made available for Tier 3 stretch labs
   in Topic 6 (students who want richer retrieval content for their open-ended lab).
   Contains real ISA rates (1.05% to 4.20% AER depending on product/term) which differ
   from the 3.75% AER in the inline BARCLAYS_DOCS strings (those are illustrative).

4. `barclays_isa_terms.pdf` - 1-Year Flexible Cash ISA Issue 79 T&C (Nov 2025).
   Not required by any topic notebook directly. Available for stretch labs.
   Contains the 4.00% AER rate current as of November 2025.

Uploads are idempotent: if a file already exists in S3, we skip it and print a message.
```


## Cell 5 - Code: Helper Functions

**Type**: code

**Content**:

```python
def s3_key_exists(bucket: str, key: str) -> bool:
    """
    Check if an S3 key already exists without downloading the content.
    Uses head_object (metadata only) - fast and cheap.
    Returns True if key exists, False if not.
    """
    try:
        s3.head_object(Bucket=bucket, Key=key)
        return True
    except ClientError as e:
        if e.response["Error"]["Code"] == "404":
            return False
        raise  # other errors (e.g. 403) should surface


def download_url_and_upload_s3(url: str, bucket: str, key: str, label: str) -> bool:
    """
    Download a file from a public URL and upload it to S3.
    Does NOT write to local disk - streams via BytesIO for speed.
    Returns True if uploaded, False if skipped (already existed).

    Args:
        url:    full https:// URL to a publicly accessible file
        bucket: S3 bucket name
        key:    S3 object key (the "filename" in the bucket)
        label:  human-readable name for progress output
    """
    # Check first - skip if already uploaded
    if s3_key_exists(bucket, key):
        print(f"  [SKIP] {key} already exists in s3://{bucket}/")
        return False

    print(f"  Downloading {label}...")
    # stream=True means requests downloads in chunks rather than all at once
    response = requests.get(url, timeout=30, stream=True)
    response.raise_for_status()  # will throw for 4xx / 5xx responses

    # Read into memory (PDFs here are all under 2MB - no need for chunked upload)
    pdf_bytes = BytesIO(response.content)
    file_size_kb = len(response.content) / 1024

    print(f"  Uploading to s3://{bucket}/{key} ({file_size_kb:.0f} KB)...")
    # upload_fileobj handles the multipart logic for us
    s3.upload_fileobj(pdf_bytes, bucket, key, ExtraArgs={"ContentType": "application/pdf"})

    print(f"  [OK] s3://{bucket}/{key} uploaded.")
    return True


print("Helper functions defined: s3_key_exists, download_url_and_upload_s3")
```


## Cell 6 - Code: Upload All PDFs

**Type**: code

**Content**:

```python
# The 4 real public Barclays PDFs to upload.
# These URLs were validated working as of April 2026.
# If any URL returns a 404 in the future, check barclays.co.uk or barclaycard.co.uk
# for an updated link and update this list.

PDF_MANIFEST = [
    {
        "key":   "barclays_personal_loan_faq.pdf",
        "url":   "https://www.barclays.co.uk/content/dam/documents/personal/site-hygiene/BAR_9910542_UK%2010.18.pdf",
        "label": "Barclays and You (Personal Banking T&C, Nov 2020)",
        "note":  "Does not quote personalised loan APR rates. Topic 2 fallback text uses simplified ERC wording.",
    },
    {
        "key":   "barclays_credit_card_tnc.pdf",
        "url":   "https://www.barclaycard.co.uk/content/dam/barclaycard/documents/personal/existing-customers/terms-and-conditions-barclaycard-core-2023.pdf",
        "label": "Barclaycard Core T&C (2023)",
        "note":  "Does not quote specific APR rates (personalised). Students extract agreement structure.",
    },
    {
        "key":   "barclays_savings_rates.pdf",
        "url":   "https://www.barclays.co.uk/content/dam/documents/personal/savings/Rates_for_Savers.pdf",
        "label": "Barclays Rates for Savers (Jan 2026)",
        "note":  "Real ISA/savings rates differ from the 3.75% AER in Topic 6 inline strings (illustrative).",
    },
    {
        "key":   "barclays_isa_terms.pdf",
        "url":   "https://www.barclays.co.uk/content/dam/documents/personal/savings/flexible_cash_isa_issue79.pdf",
        "label": "1-Year Flexible Cash ISA Issue 79 T&C (Nov 2025)",
        "note":  "4.00% AER fixed rate. Good for RAG retrieval demo with current rates.",
    },
]

print("Uploading Barclays PDFs to S3...")
print(f"Bucket: s3://{S3_BUCKET}/")
print()

uploaded = 0
skipped = 0
failed = 0

for item in PDF_MANIFEST:
    print(f">> {item['label']}")
    try:
        result = download_url_and_upload_s3(
            url=item["url"],
            bucket=S3_BUCKET,
            key=item["key"],
            label=item["label"],
        )
        if result:
            uploaded += 1
        else:
            skipped += 1
        # Always print the content mismatch note
        print(f"     NOTE: {item['note']}")
    except requests.HTTPError as e:
        print(f"  [FAIL] HTTP error downloading {item['label']}: {e}")
        print(f"         URL may have changed. Check barclays.co.uk for updated PDF link.")
        failed += 1
    except Exception as e:
        print(f"  [FAIL] {item['label']}: {e}")
        failed += 1
    print()

print(f"PDF uploads complete: {uploaded} uploaded, {skipped} skipped (already existed), {failed} failed.")

if failed > 0:
    print()
    print("WARNING: Some PDFs failed to upload.")
    print("Students will fall back to inline text in those topics - the course still works,")
    print("but the Topic 2 document extraction demo will use synthetic text instead of real PDFs.")
```


## Cell 7 - Code: Verify Uploads with Manifest

**Type**: code

**Content**:

```python
# List all objects in the bucket to confirm what is there
print(f"Contents of s3://{S3_BUCKET}/")
print("-" * 60)

try:
    response = s3.list_objects_v2(Bucket=S3_BUCKET)
    objects = response.get("Contents", [])

    if not objects:
        print("  Bucket is EMPTY. Uploads may have failed.")
    else:
        total_mb = 0
        for obj in sorted(objects, key=lambda x: x["Key"]):
            size_kb = obj["Size"] / 1024
            total_mb += obj["Size"] / (1024 * 1024)
            print(f"  {obj['Key']:<45}  {size_kb:>8.1f} KB  {obj['LastModified'].strftime('%Y-%m-%d %H:%M')}")
        print()
        print(f"  Total: {len(objects)} objects, {total_mb:.2f} MB")

except ClientError as e:
    print(f"Could not list bucket: {e}")
    print("Check that your role has s3:ListBucket on arn:aws:s3:::{S3_BUCKET}")

print()
print("Student notebooks load from this bucket using sagemaker execution role (read-only).")
print("The barclays_chunks.json file (Section 2 below) is optional but recommended.")
```


## Cell 8 - Markdown: Section 2 - Pre-chunk PDFs

**Type**: markdown

**Content**:

```
## Section 2: Pre-chunk PDFs and Upload barclays_chunks.json (Optional)

This section runs the same preprocessing pipeline as Topic 2 on all 4 PDFs and
saves the resulting chunks to `barclays_chunks.json` in S3.

Why this is useful:
- Students in Topics 6-9 who want to use real Barclays content (instead of the 7 inline
  strings in BARCLAYS_DOCS) can load barclays_chunks.json from S3 in their Tier 3 labs
- The real chunks will produce more interesting retrieval results - they contain actual
  UK banking legal language, product terms, and withdrawal conditions
- Students get real practice with noisy PDF content rather than clean hand-written strings

Why this is OPTIONAL:
- Topics 6-9 work fine without it using the 7-string BARCLAYS_DOCS fallback
- Running this section requires your OpenAI API key to call the embeddings API for verification
  (the chunking itself does not need OpenAI - only the optional verification step at the end)
- If you are short on time before class, skip this section entirely

SKIP THIS SECTION if:
- You have not set up your OpenAI API key yet
- You are short on time (Topic 2 will still load from the PDFs in Section 1)
- barclays_chunks.json already exists in S3 (the code checks before running)
```


## Cell 9 - Code: Load PDFs from S3 and Chunk Them

**Type**: code

**Content**:

```python
import pymupdf

# Skip this cell if chunks.json already exists in S3
CHUNKS_KEY = "barclays_chunks.json"

if s3_key_exists(S3_BUCKET, CHUNKS_KEY):
    print(f"[SKIP] {CHUNKS_KEY} already exists in s3://{S3_BUCKET}/")
    print("Delete it from S3 and rerun this cell if you want to regenerate.")
    all_chunks = None  # signal to skip the upload cell below

else:
    # --- Text extraction ---
    def load_pdf_from_s3_bytes(bucket: str, key: str) -> str:
        """Download PDF from S3 and return full text via PyMuPDF."""
        obj = s3.get_object(Bucket=bucket, Key=key)
        pdf_bytes = obj["Body"].read()
        doc = pymupdf.open(stream=BytesIO(pdf_bytes), filetype="pdf")
        pages = [page.get_text("text") for page in doc]
        doc.close()
        return "\n\n".join(pages)

    # --- Cleaning (same pipeline as Topic 2 Cell 15) ---
    def clean_pdf_text(text: str) -> str:
        text = re.sub(r"(\w)-\n(\w)", r"\1\2", text)         # dehyphenate
        text = re.sub(r"[ \t]+", " ", text)                   # collapse spaces
        text = text.replace("\r\n", "\n").replace("\r", "\n")
        text = re.sub(r"\n{3,}", "\n\n", text)
        text = re.sub(r"^\s*[-_.=*]{3,}\s*$", "", text, flags=re.MULTILINE)
        text = re.sub(r"^\s*\d+\s*$", "", text, flags=re.MULTILINE)
        return text.strip()

    # --- Chunking (same as Topic 2 Cell 21) ---
    def chunk_text(text: str, chunk_size: int = 1500, overlap: int = 200) -> list:
        if not text:
            return []
        chunks = []
        start = 0
        while start < len(text):
            end = start + chunk_size
            if end < len(text):
                boundary = max(
                    text.rfind(". ", start, end),
                    text.rfind(".\n", start, end),
                    text.rfind("\n\n", start, end),
                )
                if boundary > start + chunk_size // 2:
                    end = boundary + 1
            chunk = text[start:end].strip()
            if chunk:
                chunks.append(chunk)
            start += chunk_size - overlap
        return chunks

    # --- Process all 4 PDFs ---
    all_chunks = []
    for item in PDF_MANIFEST:
        print(f"Processing: {item['label']}")
        try:
            raw_text = load_pdf_from_s3_bytes(S3_BUCKET, item["key"])
            cleaned = clean_pdf_text(raw_text)
            doc_chunks = chunk_text(cleaned, chunk_size=1500, overlap=200)
            # Annotate each chunk with its source document key
            for i, c in enumerate(doc_chunks):
                all_chunks.append({
                    "text": c,
                    "source": item["key"],
                    "chunk_index": i,
                })
            print(f"  {len(doc_chunks)} chunks from {len(raw_text):,} chars raw text")
        except Exception as e:
            print(f"  [SKIP] Could not process {item['key']}: {e}")
        print()

    print(f"Total chunks across all 4 PDFs: {len(all_chunks)}")
    print(f"Average chunk length: {sum(len(c['text']) for c in all_chunks) / max(len(all_chunks), 1):.0f} chars")
    print()
    print("Chunk 0 preview:")
    if all_chunks:
        print(textwrap.fill(all_chunks[0]["text"][:300], width=80))
```


## Cell 10 - Code: Upload chunks.json to S3

**Type**: code

**Content**:

```python
# Upload the pre-chunked JSON to S3 (only runs if Cell 9 produced all_chunks)

if all_chunks is None:
    print("Skipping upload - barclays_chunks.json already exists in S3.")
elif not all_chunks:
    print("No chunks were produced - check that the PDFs were uploaded in Section 1.")
else:
    print(f"Uploading {CHUNKS_KEY} to s3://{S3_BUCKET}/{CHUNKS_KEY} ...")
    chunks_json_bytes = BytesIO(json.dumps(all_chunks, indent=2, ensure_ascii=False).encode("utf-8"))
    s3.upload_fileobj(
        chunks_json_bytes,
        S3_BUCKET,
        CHUNKS_KEY,
        ExtraArgs={"ContentType": "application/json"},
    )
    size_kb = len(json.dumps(all_chunks, ensure_ascii=False)) / 1024
    print(f"[OK] s3://{S3_BUCKET}/{CHUNKS_KEY} uploaded ({size_kb:.0f} KB, {len(all_chunks)} chunks).")
    print()
    print("Students can now load this in Topics 6-9 stretch labs with:")
    print()
    print("  import boto3, json")
    print("  from io import BytesIO")
    print("  s3 = boto3.client('s3', region_name='us-east-2')")
    print("  obj = s3.get_object(Bucket='barclays-prompt-eng-data', Key='barclays_chunks.json')")
    print("  rich_chunks = [item['text'] for item in json.load(obj['Body'])]")
```


## Cell 11 - Markdown: Content Mismatch Reference Card

**Type**: markdown

**Content**:

```
## Content Mismatch Reference Card - What to Tell Students

The notebooks have two types of Barclays content:

### Type A: Real PDFs (from S3, used in Topic 2)

Loaded by `load_pdf_from_s3()` in Topic 2. These are genuine public Barclays documents.

Key characteristics:
- Personal loan APR is NOT quoted (it is personalised - students see this is realistic)
- Credit card interest rates say "see your statement" (not in the T&C document itself)
- Savings rates in Rates_for_Savers.pdf are 1.05% to 4.20% AER depending on product
- ISA T&C quotes 4.00% AER for the 1-Year Fixed Cash ISA (Issue 79, Nov 2025)
- The "Barclays and You" doc covers all personal banking in one 50+ page document
- PDF extraction will produce real legal language, not clean marketing copy

### Type B: Inline Fallback Strings (BARCLAYS_DOCS in Topics 6-9)

Hard-coded in every Topic 6-9 notebook for when students skip Topic 2. These are
deliberately simplified with specific fictional numbers for teaching clarity:
- "6.5% APR" for personal loans (illustrative, not real)
- "3.75% AER" for savings (the real Instant Cash ISA is 1.05-1.06% AER per the Jan 2026 doc)
- "0.25% cashback" for Barclaycard Rewards (this IS the real Rewards cashback rate)
- "27.9% APR variable" for the Rewards card (illustrative, real rate is personalised)

### What to say when students notice the difference

"The inline numbers are simplified examples for teaching the RAG pipeline structure. In
production you would embed the real PDFs - exactly what you did in Topic 2. Topics 6-9
use the simplified set so you can focus on the RAG mechanics, not on reading legal text."
```


## Cell 12 - Code: Final Setup Summary

**Type**: code

**Content**:

```python
# Print a final summary of everything that is in S3
print("=" * 60)
print("INSTRUCTOR SETUP COMPLETE")
print("=" * 60)
print()
print(f"S3 bucket: s3://{S3_BUCKET}/")
print()

EXPECTED_KEYS = [item["key"] for item in PDF_MANIFEST] + [CHUNKS_KEY]

all_ok = True
for key in EXPECTED_KEYS:
    exists = s3_key_exists(S3_BUCKET, key)
    status = "[OK]" if exists else "[MISSING]"
    required = "(required)" if key != CHUNKS_KEY and key != "barclays_savings_rates.pdf" and key != "barclays_isa_terms.pdf" else "(optional)"
    print(f"  {status} {key} {required}")
    if not exists and required == "(required)":
        all_ok = False

print()
if all_ok:
    print("All required files are in S3.")
    print("Students can now run Topic 2 and load real Barclays PDFs from S3.")
else:
    print("WARNING: One or more required files are missing.")
    print("Re-run Section 1 to upload them.")
print()
print("Topic readiness summary:")
print("  Topic 1:  No S3 needed - ready.")
print("  Topic 2:  Needs barclays_personal_loan_faq.pdf + barclays_credit_card_tnc.pdf")
print("  Topic 3:  No S3 needed - ready.")
print("  Topic 4:  No S3 needed - ready.")
print("  Topic 5:  No S3 needed - ready.")
print("  Topic 6:  No S3 needed (uses inline BARCLAYS_DOCS). barclays_chunks.json optional.")
print("  Topic 7:  No S3 needed (uses inline BARCLAYS_DOCS). barclays_chunks.json optional.")
print("  Topic 8:  No S3 needed (uses inline BARCLAYS_DOCS). barclays_chunks.json optional.")
print("  Topic 9:  No S3 needed. Banking77 loads from HuggingFace (public, no S3 needed).")
```

---

# VERIFICATION CHECKLIST

- [ ] Cell 0: title states when to run, what it does, and that students cannot run it
- [ ] Cell 2: `requests`, `pymupdf==1.27.2.2`, `numpy<2` in install; `S3_BUCKET = "barclays-prompt-eng-data"`, `REGION = "us-east-2"`
- [ ] Cell 3: permission check uses `put_object` + `delete_object` probe; prints exact IAM policy JSON on failure; raises RuntimeError to stop execution
- [ ] Cell 5: `s3_key_exists` uses `head_object` in `try/except ClientError` checking code `"404"`; `download_url_and_upload_s3` uses `requests.get(stream=True)` + `BytesIO` + `upload_fileobj`
- [ ] Cell 6: PDF_MANIFEST has 4 entries with correct public URLs (validated April 2026); each entry has a `note` field printed as mismatch warning; upload is idempotent (calls `download_url_and_upload_s3` which checks `s3_key_exists` first)
- [ ] Cell 7: uses `list_objects_v2` to print full bucket manifest with sizes and timestamps
- [ ] Cell 9: chunking pipeline identical to Topic 2 (same `clean_pdf_text`, same `chunk_text` with `chunk_size=1500, overlap=200`); skips if `CHUNKS_KEY` already exists; annotates each chunk with `source` and `chunk_index`
- [ ] Cell 10: uploads `barclays_chunks.json` as `application/json`; prints example load pattern for students
- [ ] Cell 11: markdown reference card for instructor to explain content mismatches to students
- [ ] Cell 12: final checklist with `[OK]` / `[MISSING]` per expected file; topic readiness summary
- [ ] NO hardcoded API keys anywhere in the notebook
- [ ] NO em dashes, NO en dashes, NO bare `---` separators in markdown cells
- [ ] The 2 required S3 keys (`barclays_personal_loan_faq.pdf`, `barclays_credit_card_tnc.pdf`) exactly match the keys used in Topic 2 Cell 8 and Cell 10

---

# RESEARCH VALIDATED (April 2026)

1. https://www.barclaycard.co.uk/content/dam/barclaycard/documents/personal/existing-customers/terms-and-conditions-barclaycard-core-2023.pdf - Barclaycard Core T&C PDF, publicly downloadable without authentication, confirmed April 2026. Does not contain specific APR rates (personalised per customer).

2. https://www.barclays.co.uk/content/dam/documents/personal/site-hygiene/BAR_9910542_UK%2010.18.pdf - "Barclays and You" personal banking T&C, November 2020, publicly downloadable. Covers all retail banking products including loans, savings, current accounts. Does not quote loan APRs.

3. https://www.barclays.co.uk/content/dam/documents/personal/savings/Rates_for_Savers.pdf - "Rates for Savers" January 2026, publicly downloadable. Contains real current savings/ISA AER rates: Instant Cash ISA 1.05-1.06% AER, ISA products from 1.05% to 4.20% AER.

4. https://www.barclays.co.uk/content/dam/documents/personal/savings/flexible_cash_isa_issue79.pdf - 1-Year Flexible Cash ISA Issue 79, November 2025, publicly downloadable. Contains 4.00% AER fixed rate, withdrawal rules (max 3 free withdrawals, 10% of balance each).

5. https://docs.aws.amazon.com/boto3/latest/guide/s3-uploading-files.html - boto3 `upload_fileobj` is the recommended method for streaming uploads; handles multipart automatically for large files.

6. https://codegive.com/blog/boto3_s3_file_exists.php - `head_object` in `try/except ClientError` checking for `"404"` is the recommended idempotent existence check pattern; cheaper than `get_object` as it retrieves metadata only.

7. https://github.com/chroma-core/chroma/issues/1061 - ChromaDB has no official backup/restore API as of April 2026. The files in the PersistentClient directory (SQLite) can be zipped and restored but SQLite locking issues make this unreliable during an active notebook session. Decision: no pre-built ChromaDB in S3.

8. https://aws.amazon.com/blogs/machine-learning/use-amazon-sagemaker-studio-with-a-custom-file-system-in-amazon-efs - Each student's JupyterLab Space has its own private EBS volume. No shared local filesystem between students. This confirms ChromaDB must be rebuilt locally per student, not shared via S3.

9. https://docs.aws.amazon.com/sagemaker/latest/dg/sagemaker-roles.html - SageMaker execution role needs explicit `s3:PutObject` on the bucket for the SETUP notebook to work. The standard `AmazonSageMakerFullAccess` managed policy grants this only for buckets with "sagemaker" in the name. Instructor role must have a custom inline policy for `barclays-prompt-eng-data`.

10. https://www.barclaycard.co.uk/personal/cashback-reward-rules - Barclaycard Rewards cashback is 0.25% (the rate used in the inline BARCLAYS_DOCS in Topics 6-9 is accurate for this product). This is one number that does NOT differ between real and illustrative content.

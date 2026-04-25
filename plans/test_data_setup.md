# Instructor Pre-Course Test Setup - Cell-by-Cell Plan

## Context

**What this is**: A notebook (`INSTRUCTOR_TEST_SETUP.ipynb`) you run in your PERSONAL AWS account
before course day. It creates synthetic Barclays-like PDFs, creates a personal public S3 bucket,
and uploads everything there so you can test all course notebooks weeks before you have access
to the Barclays SageMaker environment.

**The problem it solves**: You cannot access the Barclays AWS account until the day of the course.
The student notebooks (Topic 2 especially) load from `barclays-prompt-eng-data`, which is the
Barclays-managed bucket. Without this notebook you cannot test the full course end-to-end in advance.

**The solution**: Your own public S3 bucket. Make it public-read so notebooks can download from it
with zero authentication (using `requests.get()` or boto3 anonymous config). When course day arrives
you run `INSTRUCTOR_SETUP.ipynb` to populate the Barclays bucket with real Barclays PDFs. The
synthetic PDFs serve only as pre-course test material.

**Why synthetic PDFs and not the real ones?**: The real public Barclays PDFs do not quote specific
loan APRs (they are personalised). The notebooks' demos (Topic 2 Cell 4, Topic 6 Cell 5) were
written around specific numbers (6.5% APR, 27.9% APR, 58 days ERC). Synthetic PDFs can exactly
match those numbers, making the cleaning demo consistent. See the content mismatch note in
`plans/instructor_setup.md` for a full comparison.

**What the synthetic PDFs contain**:

| File | Content matches | Used in |
|------|-----------------|---------|
| `barclays_personal_loan_faq.pdf` | BARCLAYS_DOCS[0] + Cell 6 noise demo text | Topic 2 Cells 6, 8, 9, 10 |
| `barclays_credit_card_tnc.pdf` | BARCLAYS_DOCS[1] rates/terms | Topic 2 Cells 10, 22 |
| `barclays_savings_rates.pdf` | BARCLAYS_DOCS[2] AER/rates | Topic 6 Tier 3 stretch |
| `barclays_isa_terms.pdf` | ISA-style terms page | Topic 6 Tier 3 stretch |
| `barclays_chunks.json` | Pre-chunked from synthetic PDFs above | Topics 6-9 stretch |

**Your personal public S3 bucket name**: Pick a globally unique name. Recommended format:
`barclays-pe-test-<your-initials>-<random-4-digits>` (e.g. `barclays-pe-test-axs-7342`).
S3 bucket names are globally unique across all AWS accounts. Choose and set the
`MY_TEST_BUCKET` constant in Cell 2 of this notebook.

**Public URL format**: After setup, files will be downloadable via:
`https://<MY_TEST_BUCKET>.s3.us-east-2.amazonaws.com/<key>`

**Notebook integration (one-line change)**: To test Topic 2 against your personal bucket,
change ONE constant in Cell 3 of Topic 2:
```python
# FROM (default, for course day)
S3_BUCKET = "barclays-prompt-eng-data"
# TO (for pre-course testing)
S3_BUCKET = "barclays-pe-test-<your-initials>-<digits>"
```
Then change it back before the course. No other cell needs to change.

## Deliverables

- **Notebook**: `INSTRUCTOR_TEST_SETUP.ipynb` (at repo root alongside `INSTRUCTOR_SETUP.ipynb`)
- **Plan file**: this document at `plans/test_data_setup.md`

## Session Timing (~15 min one-time)

| Section | Duration | Notes |
|---------|----------|-------|
| Section 0: Setup + install fpdf2 | 1 min | One-time install |
| Section 1: Generate synthetic PDFs | 2 min | 4 PDFs, local only |
| Section 2: Create personal public S3 bucket | 3 min | One-time, idempotent |
| Section 3: Upload synthetic PDFs | 2 min | Idempotent |
| Section 4: Generate + upload chunks.json | 3 min | Optional, for Topics 6-9 |
| Section 5: Verify public URLs | 2 min | requests.get test |
| Final manifest | 1 min | Confirmation |
| **Total** | **~14 min** | |

---

# MAIN NOTEBOOK - Cell-by-Cell Content (Target: 16 cells)

## Cell 0 - Markdown: Title and Purpose

**Type**: markdown

**Content**:

```
# Barclays Prompt Engineering - Instructor Test Setup

Run this notebook ONCE from your personal AWS account (not the Barclays AWS account)
to set up a public test S3 bucket and generate synthetic test documents.

This lets you test all course notebooks end-to-end before course day.

## What this does

1. Generates synthetic Barclays-like PDF documents using fpdf2
2. Creates your personal S3 bucket (public-read so notebooks can download without auth)
3. Uploads the synthetic PDFs to your personal bucket
4. (Optional) Pre-chunks the PDFs and uploads barclays_chunks.json
5. Verifies that the public URLs are downloadable with requests.get

## When to run

Run this once, several days before the course, from your personal laptop or
any environment where you have your own AWS credentials configured.
You do NOT need SageMaker for this - run it in a local terminal or local Jupyter.

## One-line switch in Topic 2

To test Topic 2 against your personal bucket, change Cell 3 of Topic 2 from:
  S3_BUCKET = "barclays-prompt-eng-data"
to:
  S3_BUCKET = "YOUR_PERSONAL_BUCKET_NAME"

Change it back before the course. All other cells stay the same.
```


## Cell 1 - Markdown: Section 0 - Setup

**Type**: markdown

**Content**:

```
## Section 0: Setup

Installing fpdf2 for PDF generation and pymupdf for chunking verification.
All other libraries (boto3, requests) are standard.
```


## Cell 2 - Code: Install and Configure

**Type**: code

**Content**:

```python
!pip install -q fpdf2 "pymupdf==1.27.2.2" requests boto3

import boto3
import json
import os
import re
from io import BytesIO
from botocore.exceptions import ClientError
from botocore import UNSIGNED
from botocore.config import Config

# ================================================================
# SET THIS TO YOUR PERSONAL BUCKET NAME (globally unique in AWS)
# Recommended format: barclays-pe-test-<initials>-<4 digits>
# Example: barclays-pe-test-axs-7342
# ================================================================
MY_TEST_BUCKET = "barclays-pe-test-CHANGEME"
REGION = "us-east-2"  # Keep same region as course bucket for consistency

if MY_TEST_BUCKET == "barclays-pe-test-CHANGEME":
    raise ValueError("Set MY_TEST_BUCKET to your chosen bucket name before running.")

PUBLIC_BASE_URL = f"https://{MY_TEST_BUCKET}.s3.{REGION}.amazonaws.com"

# Use your local AWS credentials (from ~/.aws/credentials or environment variables).
# This is YOUR personal AWS account, not the Barclays account.
s3 = boto3.client("s3", region_name=REGION)
sts = boto3.client("sts", region_name=REGION)

identity = sts.get_caller_identity()
print(f"AWS account: {identity['Account']}")
print(f"IAM principal: {identity['Arn']}")
print(f"Test bucket: s3://{MY_TEST_BUCKET}/")
print(f"Public URL base: {PUBLIC_BASE_URL}/")
print()
print("If this is a Barclays account ID, STOP and reconfigure to your personal AWS account.")
```


## Cell 3 - Markdown: Section 1 - Generate Synthetic PDFs

**Type**: markdown

**Content**:

```
## Section 1: Generate Synthetic PDFs

The synthetic PDFs are designed to match the content the notebooks expect:

- barclays_personal_loan_faq.pdf: matches BARCLAYS_DOCS[0] in Topic 6 (6.5% APR,
  58 days ERC, 1-35K GBP range, 1-5 year terms). Includes deliberate noise (page
  headers mid-page, page numbers) to make the Topic 2 cleaning demo work.

- barclays_credit_card_tnc.pdf: matches BARCLAYS_DOCS[1] (0.25% cashback, 27.9%
  APR variable, no FX fee, 25 GBP minimum repayment).

- barclays_savings_rates.pdf: matches BARCLAYS_DOCS[2] (3.75% AER instant access,
  variable rate, no minimum deposit).

- barclays_isa_terms.pdf: additional ISA document for Tier 3 stretch labs.

Content mismatch with REAL Barclays PDFs:
- The real Barclays PDFs do NOT quote personalised rates (APR, AER).
- These synthetic PDFs DO quote specific numbers matching the inline fallbacks.
- When we upload real Barclays PDFs to the Barclays bucket on course day, numbers
  in the extracted text will differ from the inline BARCLAYS_DOCS strings in Topics 6-9.
- See plans/instructor_setup.md for the full mismatch reference card.
```


## Cell 4 - Code: Define PDF Generator Helper

**Type**: code

**Content**:

```python
from fpdf import FPDF


class BankingDocPDF(FPDF):
    """
    A minimal FPDF subclass with a Barclays-style header and page number footer.
    The header text includes the document title and a "Page N of M" line, which
    PyMuPDF will extract as text. This creates the deliberate "noise" that the
    Topic 2 cleaning pipeline is designed to remove.
    """

    def __init__(self, doc_title: str, ref_code: str = "BRC/2024/001"):
        super().__init__()
        self.doc_title = doc_title
        self.ref_code = ref_code
        # {nb} is replaced with total page count by fpdf2 on output
        self.alias_nb_pages()

    def header(self):
        # Bold header - matches the kind of PDF noise PyMuPDF extracts
        self.set_font("Helvetica", style="B", size=9)
        self.cell(0, 5, self.doc_title, align="L", new_x="LMARGIN", new_y="NEXT")
        self.set_font("Helvetica", size=8)
        self.cell(0, 4, f"Ref: {self.ref_code}   CONFIDENTIAL", align="L", new_x="LMARGIN", new_y="NEXT")
        self.ln(3)

    def footer(self):
        self.set_y(-12)
        self.set_font("Helvetica", style="I", size=8)
        # "{nb}" is the fpdf2 alias for total pages
        self.cell(0, 5, f"Page {self.page_no()}/{{nb}}", align="C")

    def section_heading(self, text: str):
        self.ln(4)
        self.set_font("Helvetica", style="B", size=11)
        self.cell(0, 7, text, new_x="LMARGIN", new_y="NEXT")
        self.set_font("Helvetica", size=10)

    def body_text(self, text: str):
        self.set_font("Times", size=10)
        self.multi_cell(0, 5, text, align="J")
        self.ln(2)

    def qa_block(self, question: str, answer: str):
        """Print a Q&A pair - the format Topic 2 Cell 8 fallback mimics."""
        self.set_font("Helvetica", style="B", size=10)
        self.multi_cell(0, 5, f"Q: {question}", align="L")
        self.set_font("Times", size=10)
        self.multi_cell(0, 5, f"A: {answer}", align="J")
        self.ln(3)


print("BankingDocPDF class defined.")
```


## Cell 5 - Code: Generate barclays_personal_loan_faq.pdf

**Type**: code

**Content**:

```python
def make_personal_loan_faq() -> bytes:
    """
    Generate a synthetic Barclays Personal Loan FAQ PDF.

    Content is designed to:
    1. Match BARCLAYS_DOCS[0] in Topic 6 (APR 6.5%, 1-35K GBP, 1-5 years, 58 days ERC)
    2. Match the Topic 2 Cell 6 raw-text noise demo (page headers mid-extraction, ERC text)
    3. Be multi-page so the page-number noise is realistic
    """
    pdf = BankingDocPDF(
        doc_title="Barclays Personal Loan - Important Information",
        ref_code="BPL/2024/FAQ-003"
    )

    # Page 1: Product overview + FAQ
    pdf.add_page()
    pdf.section_heading("1. Personal Loan Overview")
    pdf.body_text(
        "Barclays personal loans let you borrow between 1,000 GBP and 35,000 GBP. "
        "Repayment terms range from 1 year to 5 years. "
        "There is no arrangement fee and no early repayment penalty beyond the early "
        "repayment charge described below. "
        "For loans of 7,500 GBP to 15,000 GBP, the representative APR is 6.5 percent. "
        "Your actual rate may differ depending on your credit history and personal circumstances."
    )

    pdf.section_heading("2. Frequently Asked Questions")
    pdf.qa_block(
        "What is the early repayment charge?",
        "If you repay all or part of your loan early, we may charge you an early repayment "
        "charge (ERC) of up to 58 days interest on the amount you repay early. "
        "This is consistent with the Consumer Credit Act 1974 regulations on early repayment."
    )
    pdf.qa_block(
        "How do I make a payment?",
        "You can make payments online via the Barclays app, by phone using Telephone Banking "
        "(0345 744 5445), or by direct debit set up at the time of the loan agreement. "
        "Payments are processed within 2 business days."
    )

    # Page 2: Missed payments + top-up
    pdf.add_page()
    pdf.qa_block(
        "What happens if I miss a payment?",
        "Missing a payment may affect your credit score and we may charge a late payment fee. "
        "Contact us immediately on 0345 744 5445 if you are having difficulty making payments. "
        "We will work with you to find a solution before pursuing formal recovery action."
    )
    pdf.qa_block(
        "Can I top up my loan?",
        "You can apply to borrow more at any time after holding the loan for at least 3 months. "
        "The new loan will replace your existing one. A new credit check will be performed "
        "and the new rate may differ from your existing rate."
    )
    pdf.qa_block(
        "Is there a minimum repayment period?",
        "No. You can repay your loan early at any time, in full or in part, with no minimum "
        "repayment period. An early repayment charge of up to 58 days interest applies on "
        "the amount repaid early. There is no charge for making regular scheduled payments."
    )

    # Page 3: Representative example + legal text
    pdf.add_page()
    pdf.section_heading("3. Representative Example")
    pdf.body_text(
        "Loan amount: 10,000 GBP. Repayment term: 36 months. "
        "Representative APR: 6.5 percent (fixed). Monthly repayment: 305.34 GBP. "
        "Total amount repayable: 10,992.24 GBP. Total interest: 992.24 GBP. "
        "This example is based on a loan of 10,000 GBP at a rate of 6.5 percent APR "
        "representative. 51 percent of customers who applied received this rate or better. "
        "Your rate may vary."
    )
    pdf.section_heading("4. Regulatory Information")
    pdf.body_text(
        "Barclays Bank UK PLC is authorised by the Prudential Regulation Authority and "
        "regulated by the Financial Conduct Authority and the Prudential Regulation Authority "
        "(Financial Services Register No. 759676). "
        "Registered in England. Registered No. 9740322. "
        "Registered Office: 1 Churchill Place, London E14 5HP. "
        "Personal loan products are only available to UK residents aged 18 and over. "
        "This document is for information purposes only and does not constitute a binding offer."
    )

    return bytes(pdf.output())


loan_faq_bytes = make_personal_loan_faq()
print(f"barclays_personal_loan_faq.pdf: {len(loan_faq_bytes) / 1024:.1f} KB")
```


## Cell 6 - Code: Generate barclays_credit_card_tnc.pdf

**Type**: code

**Content**:

```python
def make_credit_card_tnc() -> bytes:
    """
    Generate a synthetic Barclaycard Rewards T&C PDF.

    Content matches BARCLAYS_DOCS[1] in Topic 6:
    - 0.25% cashback
    - 27.9% APR variable (the rate used in the raw-noise demo in Topic 2 Cell 6)
    - No annual fee year 1 then 24 GBP
    - No foreign transaction fee
    - Minimum repayment: 25 GBP or 2.5% of balance
    """
    pdf = BankingDocPDF(
        doc_title="Barclaycard Rewards - Terms and Conditions",
        ref_code="BCR/2024/TNC-012"
    )

    # Page 1: Key rates summary (this is what PyMuPDF will extract as the "noisy" first page)
    pdf.add_page()
    pdf.section_heading("Barclaycard Rewards   INTEREST RATES AND CHARGES")
    pdf.body_text(
        "Purchase rate: 27.9 percent p.a. variable   "
        "Representative 27.9 percent APR variable   "
        "Based on a credit limit of 1,200 GBP   "
        "Annual fee: None for the first year, then 24 GBP per year   "
        "Your minimum payment   See page 2 for full terms"
    )
    pdf.section_heading("1. Cashback")
    pdf.body_text(
        "You earn 0.25 percent cashback on all eligible purchases. "
        "Cashback is credited to your account annually on your account anniversary date, "
        "or on closure of the account if earlier. "
        "Eligible purchases do not include: balance transfers, money transfers, cash advances, "
        "gambling transactions, or purchases subsequently returned or refunded."
    )

    # Page 2: Interest, minimum repayment, foreign transactions
    pdf.add_page()
    pdf.section_heading("2. Interest Charges")
    pdf.body_text(
        "Interest is charged at 27.9 percent APR variable on the outstanding balance. "
        "No interest is charged on purchases if you pay your statement balance in full "
        "and on time each month. Interest accrues daily from the date of the transaction. "
        "We calculate interest using your daily balance and a daily rate of (27.9 / 365) percent."
    )
    pdf.section_heading("3. Minimum Monthly Repayment")
    pdf.body_text(
        "Your minimum monthly repayment is whichever is the greater of: "
        "(a) 25 GBP; or "
        "(b) 2.5 percent of your outstanding statement balance; or "
        "(c) the full balance if it is less than 25 GBP. "
        "Paying only the minimum means it will take longer and cost more to clear your balance. "
        "We strongly encourage you to pay as much as possible above the minimum each month."
    )
    pdf.section_heading("4. Foreign Transactions")
    pdf.body_text(
        "There is no foreign transaction fee on the Barclaycard Rewards card. "
        "Purchases made in a foreign currency are converted to sterling at the "
        "Visa exchange rate on the date the transaction is processed. "
        "You do not pay a non-sterling transaction fee or a currency conversion fee "
        "of any kind on this card."
    )

    # Page 3: Account closure + regulatory
    pdf.add_page()
    pdf.section_heading("5. Annual Fee")
    pdf.body_text(
        "No annual fee is charged in the first year from the date your account is opened. "
        "From year two onwards, an annual fee of 24 GBP is charged to your account "
        "on each anniversary of account opening. You will be notified 30 days in advance "
        "before the first annual fee is applied."
    )
    pdf.section_heading("6. Regulatory Information")
    pdf.body_text(
        "This credit card agreement is regulated by the Consumer Credit Act 1974. "
        "Barclays Bank UK PLC is authorised by the Prudential Regulation Authority and "
        "regulated by the Financial Conduct Authority (FCA) and the Prudential Regulation "
        "Authority. Financial Services Register number: 759676. "
        "Barclaycard is a trading name of Barclays Bank UK PLC."
    )

    return bytes(pdf.output())


cc_tnc_bytes = make_credit_card_tnc()
print(f"barclays_credit_card_tnc.pdf: {len(cc_tnc_bytes) / 1024:.1f} KB")
```


## Cell 7 - Code: Generate Savings and ISA PDFs

**Type**: code

**Content**:

```python
def make_savings_rates() -> bytes:
    """Synthetic savings rates PDF matching BARCLAYS_DOCS[2]: 3.75% AER instant access."""
    pdf = BankingDocPDF(
        doc_title="Barclays Savings Rates - Personal Customers",
        ref_code="BSR/2024/SAV-009"
    )
    pdf.add_page()
    pdf.section_heading("1. Instant Access Savings Account")
    pdf.body_text(
        "Account name: Barclays Instant Access Savings Account. "
        "Interest rate: 3.75 percent AER (Annual Equivalent Rate) / 3.69 percent gross p.a. variable. "
        "No minimum deposit. Withdrawals permitted at any time with no penalty. "
        "Interest is calculated daily on your account balance and credited monthly "
        "on the first business day of each month. "
        "The interest rate is variable and may be changed at any time. "
        "We will give you at least 30 days notice before we reduce the rate."
    )
    pdf.section_heading("2. Higher Rate Saver")
    pdf.body_text(
        "Account name: Barclays Higher Rate Saver. "
        "Interest rate: 4.75 percent AER / 4.65 percent gross p.a. variable. "
        "Minimum balance: 1,000 GBP to earn the higher rate. "
        "Balances below 1,000 GBP earn 0.10 percent AER. "
        "No maximum balance. No penalty for withdrawals. "
        "Interest calculated daily, credited on the first business day of each month."
    )
    pdf.add_page()
    pdf.section_heading("3. Regular Saver Account")
    pdf.body_text(
        "Account name: Barclays Monthly Saver. "
        "Interest rate: 5.12 percent AER / 5.00 percent gross p.a. fixed for 12 months. "
        "Save between 25 GBP and 250 GBP per month by direct debit. "
        "Partial or full withdrawals are not permitted during the 12-month term. "
        "At the end of 12 months the account converts to an instant access account."
    )
    return bytes(pdf.output())


def make_isa_terms() -> bytes:
    """Synthetic ISA T&C PDF for Tier 3 stretch labs."""
    pdf = BankingDocPDF(
        doc_title="Barclays 1-Year Flexible Cash ISA - Terms",
        ref_code="ISA/2024/FLEX-079"
    )
    pdf.add_page()
    pdf.section_heading("1. Account Name and Rate")
    pdf.body_text(
        "Account name: 1 Year Flexible Cash ISA. "
        "Interest rate: 4.00 percent AER / 4.00 percent tax-free p.a. fixed. "
        "Rate fixed for 12-month term from date of account opening. "
        "Minimum deposit: 1 GBP (or 0 GBP if funded by an ISA transfer-in). "
        "Maximum balance: 1,000,000 GBP."
    )
    pdf.section_heading("2. Withdrawals")
    pdf.body_text(
        "A maximum of 3 free withdrawals are allowed during the term. "
        "Each withdrawal is limited to 10 percent of your balance at the time. "
        "Funds withdrawn may be re-deposited in the same tax year without counting "
        "against your annual ISA allowance of 20,000 GBP."
    )
    pdf.add_page()
    pdf.section_heading("3. Maturity")
    pdf.body_text(
        "At the end of the 1-year term the account automatically converts to an "
        "instant access variable rate cash ISA. We will write to you 30 days "
        "before maturity to remind you of your options."
    )
    return bytes(pdf.output())


savings_bytes = make_savings_rates()
isa_bytes = make_isa_terms()
print(f"barclays_savings_rates.pdf: {len(savings_bytes) / 1024:.1f} KB")
print(f"barclays_isa_terms.pdf:     {len(isa_bytes) / 1024:.1f} KB")

# Collect all PDFs for upload
ALL_PDFS = {
    "barclays_personal_loan_faq.pdf": loan_faq_bytes,
    "barclays_credit_card_tnc.pdf":   cc_tnc_bytes,
    "barclays_savings_rates.pdf":      savings_bytes,
    "barclays_isa_terms.pdf":          isa_bytes,
}

print()
print("All synthetic PDFs ready in memory.")
```


## Cell 8 - Markdown: Section 2 - Create Public S3 Bucket

**Type**: markdown

**Content**:

```
## Section 2: Create Personal Public S3 Bucket

Creating a personal bucket in your own AWS account.
We use a bucket policy (not ACLs) for public-read access - this is the required
approach in 2025/2026 since AWS disabled ACLs by default on all new buckets.

The process is:
1. Create the bucket (private by default)
2. Remove the Block Public Access settings (which block bucket policies granting public access)
3. Apply a bucket policy: Principal "*", Action "s3:GetObject" - public-read

IMPORTANT: Use your personal AWS account, NOT the Barclays account.
Account-level Block Public Access settings in corporate accounts override bucket-level
settings and will silently prevent public access even after you apply the bucket policy.
```


## Cell 9 - Code: Create Bucket and Apply Public Policy

**Type**: code

**Content**:

```python
import json

PUBLIC_BUCKET_POLICY = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": f"arn:aws:s3:::{MY_TEST_BUCKET}/*"
        }
    ]
}


def create_public_bucket(bucket: str, region: str):
    """
    Create an S3 bucket and configure it for public-read access via bucket policy.
    Idempotent: safe to re-run if bucket already exists.

    Two-step process required in 2025/2026:
    1. Create bucket (Block Public Access enabled by default)
    2. Delete the Block Public Access config
    3. Apply a bucket policy that grants Principal "*" read access
    """
    # Step 1: Create bucket (skip if it already exists)
    try:
        if region == "us-east-1":
            # us-east-1 does not accept CreateBucketConfiguration
            s3.create_bucket(Bucket=bucket)
        else:
            s3.create_bucket(
                Bucket=bucket,
                CreateBucketConfiguration={"LocationConstraint": region}
            )
        print(f"[OK] Bucket created: s3://{bucket}/")
    except s3.exceptions.BucketAlreadyOwnedByYou:
        print(f"[SKIP] Bucket s3://{bucket}/ already exists and is owned by you.")
    except s3.exceptions.BucketAlreadyExists:
        raise RuntimeError(f"Bucket name '{bucket}' is taken by another AWS account. Choose a different name.")

    # Step 2: Remove Block Public Access settings
    # These are enabled by default on new buckets and prevent bucket policies
    # from granting public access even when Principal is "*"
    s3.delete_public_access_block(Bucket=bucket)
    print(f"[OK] Block Public Access settings removed from s3://{bucket}/")

    # Step 3: Apply public-read bucket policy
    s3.put_bucket_policy(
        Bucket=bucket,
        Policy=json.dumps(PUBLIC_BUCKET_POLICY)
    )
    print(f"[OK] Public-read bucket policy applied to s3://{bucket}/")
    print()
    print(f"Public URL base: https://{bucket}.s3.{region}.amazonaws.com/")


create_public_bucket(MY_TEST_BUCKET, REGION)
```


## Cell 10 - Markdown: Section 3 - Upload PDFs

**Type**: markdown

**Content**:

```
## Section 3: Upload Synthetic PDFs

Uploading all 4 synthetic PDFs to your personal bucket.
Uploads are idempotent: if a file already exists, it is skipped.

After upload, each PDF is accessible at:
  https://<MY_TEST_BUCKET>.s3.us-east-2.amazonaws.com/<key>
```


## Cell 11 - Code: Upload PDFs Idempotently

**Type**: code

**Content**:

```python
def s3_key_exists(bucket: str, key: str) -> bool:
    try:
        s3.head_object(Bucket=bucket, Key=key)
        return True
    except ClientError as e:
        if e.response["Error"]["Code"] == "404":
            return False
        raise


print(f"Uploading synthetic PDFs to s3://{MY_TEST_BUCKET}/")
print()

uploaded = 0
skipped = 0

for key, pdf_bytes in ALL_PDFS.items():
    if s3_key_exists(MY_TEST_BUCKET, key):
        print(f"  [SKIP] {key} already exists. Delete from S3 to force re-upload.")
        skipped += 1
    else:
        s3.put_object(
            Bucket=MY_TEST_BUCKET,
            Key=key,
            Body=pdf_bytes,
            ContentType="application/pdf",
        )
        size_kb = len(pdf_bytes) / 1024
        print(f"  [OK]   {key} ({size_kb:.1f} KB) uploaded.")
        uploaded += 1

print()
print(f"Done: {uploaded} uploaded, {skipped} skipped.")
```


## Cell 12 - Code: Optional - Generate and Upload chunks.json

**Type**: code

**Content**:

```python
import pymupdf

CHUNKS_KEY = "barclays_chunks.json"

if s3_key_exists(MY_TEST_BUCKET, CHUNKS_KEY):
    print(f"[SKIP] {CHUNKS_KEY} already exists. Skipping chunk generation.")
else:
    # Re-extract text from the synthetic PDFs we just generated (in memory)
    def pdf_bytes_to_text(pdf_bytes: bytes) -> str:
        doc = pymupdf.open(stream=BytesIO(pdf_bytes), filetype="pdf")
        pages = [page.get_text("text") for page in doc]
        doc.close()
        return "\n\n".join(pages)

    def clean_text(text: str) -> str:
        text = re.sub(r"(\w)-\n(\w)", r"\1\2", text)
        text = re.sub(r"[ \t]+", " ", text)
        text = text.replace("\r\n", "\n").replace("\r", "\n")
        text = re.sub(r"\n{3,}", "\n\n", text)
        text = re.sub(r"^\s*\d+\s*$", "", text, flags=re.MULTILINE)
        return text.strip()

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

    all_chunks = []
    for key, pdf_bytes in ALL_PDFS.items():
        raw = pdf_bytes_to_text(pdf_bytes)
        cleaned = clean_text(raw)
        doc_chunks = chunk_text(cleaned)
        for i, c in enumerate(doc_chunks):
            all_chunks.append({"text": c, "source": key, "chunk_index": i})
        print(f"  {key}: {len(doc_chunks)} chunks from {len(raw):,} chars")

    chunks_json = BytesIO(json.dumps(all_chunks, indent=2, ensure_ascii=False).encode("utf-8"))
    s3.upload_fileobj(chunks_json, MY_TEST_BUCKET, CHUNKS_KEY, ExtraArgs={"ContentType": "application/json"})
    print(f"\n[OK] {CHUNKS_KEY} uploaded ({len(all_chunks)} total chunks).")
```


## Cell 13 - Markdown: Section 4 - Verify Public URLs

**Type**: markdown

**Content**:

```
## Section 4: Verify Public URLs

Testing that notebooks can download from the public bucket using plain HTTPS.
Two methods are verified:
1. requests.get() - zero AWS dependency, works anywhere
2. boto3 UNSIGNED - works in SageMaker without SageMaker credentials for this bucket
```


## Cell 14 - Code: Verify Public Access

**Type**: code

**Content**:

```python
import requests

print("Verifying public URL access (no AWS credentials used)...")
print()

all_ok = True
for key in ALL_PDFS:
    url = f"{PUBLIC_BASE_URL}/{key}"
    try:
        resp = requests.get(url, timeout=10)
        if resp.status_code == 200 and resp.content[:4] == b"%PDF":
            size_kb = len(resp.content) / 1024
            print(f"  [OK]  {url}")
            print(f"        {size_kb:.1f} KB, valid PDF (starts with %PDF)")
        else:
            print(f"  [FAIL] {url}")
            print(f"         HTTP {resp.status_code}")
            all_ok = False
    except Exception as e:
        print(f"  [FAIL] {url}: {e}")
        all_ok = False

print()
if all_ok:
    print("All public URLs verified. Notebooks can download these files with requests.get().")
    print()
    print("boto3 UNSIGNED download pattern (also works in SageMaker without credentials):")
    print()
    print("  from botocore import UNSIGNED")
    print("  from botocore.config import Config")
    print("  import boto3")
    print(f"  s3_anon = boto3.client('s3', region_name='{REGION}', config=Config(signature_version=UNSIGNED))")
    print(f"  obj = s3_anon.get_object(Bucket='{MY_TEST_BUCKET}', Key='barclays_personal_loan_faq.pdf')")
    print("  pdf_bytes = obj['Body'].read()")
else:
    print("SOME URLS FAILED. Check:")
    print("  1. Did the create_public_bucket() call in Cell 9 succeed?")
    print("  2. Is this a personal AWS account (not corporate with account-level BlockPublicPolicy)?")
    print("  3. Wait 30 seconds - bucket policy propagation can take a moment.")
```


## Cell 15 - Code: Final Summary + Notebook Integration Instructions

**Type**: code

**Content**:

```python
print("=" * 65)
print("PRE-COURSE TEST SETUP COMPLETE")
print("=" * 65)
print()
print(f"Personal test bucket: s3://{MY_TEST_BUCKET}/")
print(f"Public URL base:      {PUBLIC_BASE_URL}/")
print()
print("Files in your test bucket:")

resp = s3.list_objects_v2(Bucket=MY_TEST_BUCKET)
for obj in sorted(resp.get("Contents", []), key=lambda x: x["Key"]):
    size_kb = obj["Size"] / 1024
    print(f"  {obj['Key']:<48}  {size_kb:>7.1f} KB")

print()
print("One-line switch for Topic 2 notebook (Cell 3 of exercises/topic_02...):")
print()
print("  # Change this ONE line to test against your bucket:")
print(f"  S3_BUCKET = '{MY_TEST_BUCKET}'   # test mode")
print()
print("  # Restore this before the course:")
print("  S3_BUCKET = 'barclays-prompt-eng-data'   # production")
print()
print("Topic readiness with your test bucket:")
print("  Topic 2:  Change S3_BUCKET in Cell 3. All load_pdf_from_s3() calls use it.")
print("  Topics 6-9:  No change needed. These use inline BARCLAYS_DOCS (no S3 calls).")
print("               Optional: add your bucket to Tier 3 lab instructions for students.")
print()
print("On course day:")
print("  1. Run INSTRUCTOR_SETUP.ipynb (uploads real Barclays PDFs to Barclays bucket).")
print("  2. Revert Topic 2 Cell 3 to S3_BUCKET = 'barclays-prompt-eng-data'.")
print("  3. Verify Topic 2 Cell 8 loads from the Barclays bucket successfully.")
```

---

# VERIFICATION CHECKLIST

- [ ] Cell 2: raises ValueError if MY_TEST_BUCKET is the default placeholder; prints AWS account ID so you can verify this is your personal account
- [ ] Cell 4: `BankingDocPDF` uses `alias_nb_pages()` so `{nb}` works in footer; `header()` outputs document title and ref code as text (becomes PyMuPDF noise); `multi_cell` used for body (produces line-wrapped text PyMuPDF can extract)
- [ ] Cell 5: personal loan PDF contains EXACTLY "58 days interest", "6.5 percent", "1,000 GBP to 35,000 GBP", "1 year to 5 years" to match Topic 2 Cell 6 demo text and BARCLAYS_DOCS[0]
- [ ] Cell 6: credit card PDF contains EXACTLY "27.9 percent APR variable", "0.25 percent cashback", "25 GBP", "2.5 percent", "24 GBP" to match BARCLAYS_DOCS[1] and Topic 2 Cell 6 raw_pdf_noise demo
- [ ] Cell 7: savings PDF contains "3.75 percent AER" to match BARCLAYS_DOCS[2]; ISA PDF contains "4.00 percent AER"
- [ ] Cell 9: uses delete_public_access_block (not put with all False), then put_bucket_policy with Principal "*"; handles BucketAlreadyOwnedByYou gracefully; raises on BucketAlreadyExists (name taken by another account)
- [ ] Cell 11: upload uses put_object with body bytes + ContentType application/pdf; idempotent via head_object check
- [ ] Cell 12: chunk pipeline matches Topic 2 (same chunk_size=1500, overlap=200, clean_text regex steps); annotates each chunk with source and chunk_index; uploads as application/json
- [ ] Cell 14: verifies public access using requests.get() (no AWS creds) + checks response starts with b"%PDF"; prints boto3 UNSIGNED pattern as alternative
- [ ] Cell 15: prints the exact one-line change needed in Topic 2 Cell 3 for test mode; prints the restore instruction for course day
- [ ] NO hardcoded AWS credentials anywhere
- [ ] NO em dashes, NO en dashes, NO Unicode multiplication, no bare --- in markdown cells
- [ ] ALL synthetic PDF content values match BARCLAYS_DOCS in Topic 6 Cell 5 exactly

---

# RESEARCH VALIDATED (April 2026)

1. https://stackoverflow.com/questions/68757380/make-every-object-in-an-amazon-s3-bucket-publicly-accessible - Public bucket requires two steps: disable Block Public Access, then apply bucket policy with Principal "*". Cannot be done in one step during bucket creation.

2. https://repost.aws/questions/QUv94_RRFHS1SVvF4LOVaP0Q/how-to-create-s3-bucket-with-acl-public-read-parameter - AWS disabled ACL-based public access in 2023. Use delete_public_access_block + put_bucket_policy instead. ACL public-read approach fails with BlockPublicAccessError on new buckets.

3. https://github.com/boto/boto3/issues/1200 - boto3 anonymous access: `boto3.client('s3', config=Config(signature_version=UNSIGNED))` - confirmed working for public buckets. Avoids needing any AWS credentials in the SageMaker student environment.

4. https://stackoverflow.com/questions/62086013/download-file-folder-from-public-aws-s3-with-python-no-credentials - Public S3 objects can be downloaded with `requests.get(url)` with no AWS credentials at all. URL format: `https://<bucket>.s3.<region>.amazonaws.com/<key>`.

5. https://py-pdf.github.io/fpdf2/Tutorial.html - fpdf2 multi-page PDF with custom header/footer: subclass FPDF, override header() and footer() methods, use alias_nb_pages() for total page count, multi_cell() for body text with automatic page breaks.

6. https://py-pdf.github.io/fpdf2/UsageInWebAPI.html - `bytes(pdf.output())` returns the PDF as bytes. Wrap in BytesIO for upload_fileobj, or pass directly to put_object(Body=...).

7. https://stackoverflow.com/questions/69585531/upload-fpdf-object-to-aws-s3-in-python - Confirmed: pass `bytes(pdf.output())` as the Body parameter to `put_object`. The FPDF object itself is not a valid Body type.

8. https://docs.aws.amazon.com/AmazonS3/latest/userguide/configuring-block-public-access-bucket.html - Account-level Block Public Access settings override bucket-level settings. Corporate AWS accounts (like Barclays) typically have account-level BlockPublicPolicy=True, making public buckets impossible without IT intervention. Use a personal AWS account for this test bucket.

9. Topic 6 plan (plans/topic_06_rag_foundations.md) Cell 5: BARCLAYS_DOCS list confirmed - 7 inline strings with specific rates: 6.5% APR (loan), 0.25% cashback and 27.9% APR variable (Rewards card), 3.75% AER (savings), 4.2% (mortgage), 0% OD (student), 8 GBP/month (business), 18 GBP/month (travel). Synthetic PDFs match these values for consistency.

10. Topic 2 plan (plans/topic_02_nlp_preprocessing.md) Cell 6: raw_pdf_noise variable contains "up to 58 days interest" and "27.9% p.a. variable" - synthetic PDFs must contain these exact phrases so the cleaning demo makes narrative sense.

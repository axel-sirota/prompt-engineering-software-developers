# Discrepancies in Topics 7, 8, 9 vs Setup Files and Course Manifest

Audit date: 2026-04-25
Method: /research protocol, 3 cycles, no web search (internal file analysis only)
Scope: cross-check `plans/topic_07_advanced_rag_web_search.md`,
`plans/topic_08_ethical_guardrails.md`, and `plans/topic_09_capstone.md`
against `plans/instructor_setup.md`, `plans/test_data_setup.md`,
`plans/CORE_TECHNOLOGIES_AND_DECISIONS.md`, and the prior topic plans
(T1-T6) for continuity rot.

---

## Executive Summary

The 3 topic plans were produced by parallel research subagents that read
`CLAUDE.md` and `CORE_TECHNOLOGIES_AND_DECISIONS.md` but did NOT have
`plans/instructor_setup.md` or `plans/test_data_setup.md` in their
mandatory-reads list. As a result:

1. The good news: Topics 7, 8, 9 do NOT touch S3 at all, which matches
   `instructor_setup.md` Cell 12 ("Topics 6-9: No S3 needed"). No new
   AWS resources are required by these three topic plans. The instructor
   setup notebook does not need any extra uploads to support T7/T8/T9.
2. The bad news: there is significant continuity rot inside the plans
   themselves. The most damaging items are:
   a. T9 capstone scope contradicts the locked CORE manifest decision.
   b. BARCLAYS_DOCS content drifts across T6, T7, and T9 even though T7
      and T9 each claim "Same inline BARCLAYS_DOCS list from Topic 6".
   c. T9 ChromaDB collection name `barclays_capstone` does not match
      T6 and T7's `barclays_products`, breaking the optional carryover.
   d. T9 inline reimplementations of T8's guardrail functions
      (`detect_pii`, `should_escalate`) have different signatures and
      return types from the real T8 implementation.

This document organises the findings by category, rates severity,
and lists concrete remediation steps. Section 1 (the user's primary
concern) is short because there are essentially no AWS issues. Sections
2 through 5 are the substantive defects.

---

## Section 1: AWS / S3 / Setup-File Alignment

Severity: LOW. T7/T8/T9 are well aligned with the setup files.

### S1.1 - All three plans correctly avoid S3

`rg -n "barclays-prompt-eng-data|S3_BUCKET|boto3|us-east-2|REGION"`
on the three plan files returns ZERO matches in code blocks. The only
hit is a docs URL inside T7's RESEARCH VALIDATED block
(plans/topic_07_advanced_rag_web_search.md line 1000).

This is consistent with `plans/instructor_setup.md` Cell 12, which
states explicitly:
- Topic 7: No S3 needed (uses inline BARCLAYS_DOCS). barclays_chunks.json optional.
- Topic 8: No S3 needed (uses inline BARCLAYS_DOCS). barclays_chunks.json optional.
- Topic 9: No S3 needed. Banking77 loads from HuggingFace (public, no S3 needed).

VERDICT: No new S3 keys, no new bucket policy edits, no new IAM
permissions, and no edits to `INSTRUCTOR_SETUP.ipynb` or
`INSTRUCTOR_TEST_SETUP.ipynb` are required for these three topics.

### S1.2 - Optional `barclays_chunks.json` mention is missing in all 3 plans

Severity: LOW (informational, not a defect).

`plans/instructor_setup.md` lines 36-37 list `barclays_chunks.json` as
"Optional: richer fallback for Topics 6-9" and Cell 10 prints a load
pattern for students. Neither T7 nor T9 nor T8 mentions this fallback in
their stretch / Tier 3 lab brief. If you want students to use the real
chunked Barclays text in stretch labs, add a paragraph to the relevant
stretch sections pointing them at `barclays-prompt-eng-data/barclays_chunks.json`
and the load snippet from `instructor_setup.md` Cell 10.

Specifically:
- T7 Tier 2 hard lab (Cell ~25 area): could load real chunks for the
  hybrid router stretch.
- T9 Tier 3 open-ended lab (Cell 24): could load real chunks instead of
  the inline 5-doc set.

REMEDIATION: optional, low priority. Add a one-line "Stretch data:
load `s3://barclays-prompt-eng-data/barclays_chunks.json` if you want
to test against real Barclays product text" hint to the relevant cells.

### S1.3 - SageMaker / region / credentials pattern

All three plans correctly use `getpass + os.environ` for the OpenAI key
(matches CORE manifest line 77) and do NOT reference AWS execution role
helpers. This is correct because none of T7/T8/T9 touch S3.

VERDICT: No discrepancy.

---

## Section 2: CORE_TECHNOLOGIES_AND_DECISIONS Conflicts

Severity: HIGH. Requires an explicit decision before T9 is built.

### S2.1 - T9 capstone scope contradicts the locked CORE decision

CORE_TECHNOLOGIES_AND_DECISIONS.md lines 25-34:

> Running problem (Topics 1-8): Barclays Product Knowledge Assistant
> ... built up topic by topic from a raw API call to a full RAG +
> memory + guardrails assistant ...
>
> Capstone (Topic 9): Barclays Transaction Query Agent - a
> single-pass tool-augmented agent students build INDEPENDENTLY;
> classifies transaction-related customer intent, calls manually
> coded tools (policy lookup via RAG, mock transaction lookup,
> human escalation), returns a grounded compliant response;
> uses Banking77 HuggingFace dataset for intent examples.

CORE line 84:

> Capstone dataset: Banking77 (HuggingFace PolyAI/banking77)

What T9 actually delivers (plans/topic_09_capstone.md lines 1, 8, 126):

> Topic 9: Capstone - Production Customer Service Assistant
> ... A production-hardened orchestrator named
> `production_assistant(user_message, chat_state)` that runs the
> full pipeline for every customer query: ... hybrid retrieval
> (ChromaDB collection + barclays.co.uk web search via the router
> built in Topic 7) ...

This is the polish-and-integrate version of the T1-T8 chatbot, not the
new tool-augmented Transaction Query Agent CORE describes. The T9 plan
does NOT use Banking77 (zero references), does NOT classify
transaction intent, does NOT define manually coded tools (policy
lookup, mock transaction lookup, human escalation) as discrete tool
functions, and is NOT built "independently" of the T1-T8 chain.

ROOT CAUSE: TOPICS.md lines 318-352 describes Topic 9 as
"Production Customer Service Assistant" with concepts about wiring
together prior components. The subagent followed TOPICS.md
(which it was instructed to read) and CORE was not re-read at the
capstone-scope decision point. TOPICS.md and CORE disagree about what
the capstone IS.

REMEDIATION: requires a user decision before /build-topic-notebook 9
runs. Two options:

OPTION A: Treat CORE as authoritative. Rewrite T9 plan to be a
NEW notebook that:
- imports nothing from prior topics
- loads Banking77 via `from datasets import load_dataset; ds = load_dataset("PolyAI/banking77")`
- defines 3 tools: `lookup_policy(query)` (one-shot RAG over a small
  policy doc set), `lookup_transaction(transaction_id)` (mock dict
  lookup), `escalate_to_human(reason)` (returns a human-handoff record)
- uses gpt-4o function calling / tool-use to dispatch
- runs end-to-end on 5 representative Banking77 intents
- This is a different capstone notebook; the current T9 plan would be
  archived.

OPTION B: Treat TOPICS.md as authoritative. Update CORE lines 27-34
and line 84 to remove Banking77 / Transaction Query Agent and replace
with the integrate-and-harden capstone the current T9 plan describes.
Update plans/TOPICS.md if needed (it already matches T9 plan).

OPTION C (hybrid): keep the current T9 plan as the MAIN capstone and
add a SHORT optional second notebook (`week_09_optional_transaction_agent.ipynb`)
that delivers the Banking77 tool-augmented agent as a take-home for
advanced students. This preserves both intentions without a rewrite.

### S2.2 - T9 install line missing `datasets` package

Severity: HIGH if option A or C is chosen, NONE if option B is chosen.

T9 install line (line 249):
```
!pip install -q "openai>=2.30.0" "tenacity==9.1.4" "tiktoken==0.9.0" "chromadb==1.5.8" "numpy<2"
```

If Banking77 is restored, add `"datasets>=2.20.0"` (the HuggingFace
loader package). The dataset itself is public (no HF auth token
required for PolyAI/banking77).

REMEDIATION: depends on S2.1 decision.

---

## Section 3: Cross-Topic Continuity Rot

Severity: HIGH. These breaks affect students whether or not S3 is used,
and they will surface as mysterious bugs at notebook-build time.

### S3.1 - BARCLAYS_DOCS content drifts across T6, T7, T9

Severity: HIGH. The setup notebooks (`instructor_setup.md`,
`test_data_setup.md`) were carefully aligned with T6's exact
BARCLAYS_DOCS values. T7 and T9 each claim continuity but redefine
the list with different products and different numbers.

T6 ground truth (plans/topic_06_rag_foundations.md lines 212-219, 7 docs):

| # | Product | Key numbers |
|---|---------|-------------|
| 0 | Personal Loan | 1,000-35,000 GBP, 6.5% APR (7,500-15,000 GBP range), 1-5 years |
| 1 | Rewards Credit Card | 0.25% cashback, 24 GBP year 2 fee, 0% FX, 25 GBP / 2.5% min repayment |
| 2 | Savings Account | 3.75% AER instant access |
| 3 | Mortgage | 4.2% from 60% LTV, 2-10 years |
| 4 | Student Account | 0% OD up to 500 GBP year 1, 1000 GBP year 2+ |
| 5 | Business Current Account | free 12 months then 8 GBP/month |
| 6 | Travel Pack | 18 GBP/month |

T7 BARCLAYS_DOCS (plans/topic_07_advanced_rag_web_search.md lines 253-273, 6 docs):
- Doc 0: Personal Loan range changed to 1000-50000 GBP (T6 says 1000-35000). Range note removed.
- Doc 1: NEW product "Premier Current Account" with 16 GBP monthly fee (not in T6).
- Doc 2: NEW product "Rainy Day Saver (Blue Rewards)" 5.12% AER (T6 has Savings Account 3.75% AER).
- Doc 3: NEW product "Barclaycard Platinum Cashback Plus" with 0.5% supermarket and 24.9% APR (T6 has Rewards Card with 27.9% APR variable per test_data_setup.md baseline).
- Doc 4: Mortgage from 5000 GBP, 95% LTV first-time buyers, booking fee 999 GBP (T6 says 4.2% from 60% LTV).
- Doc 5: NEW "Travel Wallet" 19 currencies (T6 has Travel Pack 18 GBP/month).
- Student Account and Business Current Account are MISSING.

T9 BARCLAYS_DOCS (plans/topic_09_capstone.md lines 332-348, 5 docs):
- Doc 0: Personal Loan range 1000-50000 GBP (matches T7, NOT T6), 2-5 years (T6 says 1-5).
- Doc 1: "Barclaycard Platinum" with 0% for 20 months then 24.9% APR, "Cashback not included" (different again from T6 and T7).
- Doc 2: NEW "card freeze instructions" (procedural FAQ, not a product factsheet, not in T6 or T7).
- Doc 3: "Everyday Saver" 1.65% AER (different from T6's 3.75% and T7's 5.12%).
- Doc 4: NEW "Help and Support / Money Worries team" (operational doc, not in T6 or T7).

CONSEQUENCES:
1. test_data_setup.md generates synthetic PDFs that match T6's values
   (6.5% APR, 0.25% cashback, 27.9% APR variable, 3.75% AER) per its
   line 887-889 explicit assertion. Those synthetic PDFs do NOT match
   T7's or T9's content. A student who uses the test bucket plus the
   T7 or T9 inline strings will see TWO different sets of numbers and
   be confused.
2. T7 line 83 explicitly claims "Same inline BARCLAYS_DOCS list from
   Topic 6". This claim is false.
3. T9's BARCLAYS_DOCS does not even support the 5 customer queries in
   T9 Cell 23. For example: Cell 23 Query 1 asks "What is the current
   rate on a 2-year fixed mortgage?", but T9's BARCLAYS_DOCS does not
   contain a mortgage doc. The query relies on web_search to recover.
   Workable, but inconsistent with the "vector retrieval should hit
   product factsheets in BARCLAYS_DOCS" diagram.
4. The T7 cells that reference real-vs-stale numbers (T7 line 83
   "the live barclays.co.uk page shows 6.3% as of March 2026; this
   real divergence is the teaching moment") rely on the inline
   string saying 6.5% specifically. Doc 0 in T7 still says 6.5%, so
   that demo still works.

REMEDIATION:
- Pick T6 as the canonical BARCLAYS_DOCS list and copy it byte-for-byte
  into T7 Cell 5 and T9 Cell 5. Keep all 7 docs.
- Update test_data_setup.md only if the canonical list changes.
- If you want T9 to add NEW docs (card freeze FAQ, Money Worries) for
  its 5-query battery, ADD them at indices 7+ rather than REPLACING
  T6's docs. Document in T9 Cell 5 that "indices 0-6 are the Topic 6
  carryover; indices 7-8 are capstone-specific operational docs".

### S3.2 - T9 ChromaDB collection name does not match T6 and T7

Severity: HIGH. Breaks the optional carryover from T6/T7 explicitly.

T6 (plans/topic_06_rag_foundations.md line 610):
```
name="barclays_products"
```

T7 (plans/topic_07_advanced_rag_web_search.md lines 41, 292):
```
name="barclays_products"
```
T7 also has a verification checklist line 933: "The ChromaDB collection
is named `barclays_products` and uses cosine distance".

T9 (plans/topic_09_capstone.md line 500):
```
name="barclays_capstone"
```

CONSEQUENCE: T9's `chromadb.PersistentClient(path="./chroma_db")` lives
on the same EBS as T6 and T7. If a student ran T6 or T7 first, they
have a populated `barclays_products` collection. T9 ignores it and
creates an empty `barclays_capstone`, then re-embeds inline docs. This
wastes 5 seconds of OpenAI embedding cost per student and contradicts
T9's claim that it "imports by exact name from prior topics".

Worse: if T9 students re-embed their own (different) BARCLAYS_DOCS
into `barclays_capstone`, the persistent client now holds TWO
collections with overlapping but different content. Diagnostic noise
for any debugging session.

REMEDIATION: change T9 Cell 5 line 500 from `name="barclays_capstone"`
to `name="barclays_products"`. Add a comment: "Same name as T6/T7 so
the existing local collection is reused if present; the
get_or_create_collection call is idempotent and an empty new collection
is created if not".

### S3.3 - T9 inline `detect_pii` signature does NOT match T8 actual

Severity: HIGH. Type/return mismatch between two notebooks both named
in the capstone integration contract.

T8 actual (plans/topic_08_ethical_guardrails.md line 15):
```
detect_pii(text: str) -> GuardrailResult
```
T8 line 152: "Our `detect_pii` function will return a `GuardrailResult`
carrying both a `passed` flag and a `redacted_text` string".
T8 line 165-166 defines `@dataclass(frozen=True) class GuardrailResult`.

T9 inline reference impl (plans/topic_09_capstone.md lines 599+):
```
def detect_pii(text: str) -> list:
    ...
    # returns a list of {"type": ..., "match": ...} dicts
```
T9 line 1407: `pii_findings = detect_pii(user_message)` - assumes a
list, iterates over individual finding dicts.

CONSEQUENCE: A student who ran T8 has `detect_pii` returning
`GuardrailResult(passed=False, redacted_text="...", details=[...])`.
That same student opening T9 and running Cell 5 OVERWRITES `detect_pii`
with a function returning a list of dicts. If T9 Cell 22's
`production_assistant` were edited to use the T8 GuardrailResult API,
it would TypeError. Reading the existing T9 code that consumes the
list-of-dicts version, the only "production" code path is the inline
reimpl, so T9 in isolation works. But the contract is broken.

REMEDIATION: Pick one signature and apply to both plans. Recommended:

OPTION 1 (preserve T8): Update T9 Cell 5 reference impl to return
`GuardrailResult` exactly as T8 defines it, and update T9 Cell 22 to
consume `.passed`, `.redacted_text`, `.details`. Update T9 Cell 23
assertions to use `.details` instead of dict keys.

OPTION 2 (preserve T9 simplicity): Update T8 to return list-of-dicts
instead of GuardrailResult. This is a bigger rewrite of T8 since the
GuardrailResult dataclass appears throughout T8 and is taught as a
concept (Cell 7 demo).

Recommended: OPTION 1. The GuardrailResult dataclass is a teaching
asset in T8 and discarding it weakens T8.

### S3.4 - T9 inline `should_escalate` signature does NOT match T8

Severity: HIGH. Same problem as S3.3.

T8 (line 18):
```
should_escalate(query: str, history: list) -> GuardrailResult
```

T9 (plans/topic_09_capstone.md line 69):
```
should_escalate(text) -> (should: bool, reason: str)
```
T9 line 640: `def should_escalate(text: str) -> tuple:`
T9 line 1409: `should_esc, esc_reason = should_escalate(user_message)`

REMEDIATION: same as S3.3. Pick one signature and update both plans.
If keeping T8's GuardrailResult, change T9 Cell 5 reimpl and
Cell 22 line 1409 to:
```
esc_result = should_escalate(user_message, session.messages)
if not esc_result.passed:
    reason = esc_result.details.get("reason", "vulnerability_indicator")
    ...
```

### S3.5 - T8 wrap_with_guardrails(_toy_chat) does not use T6 RAG

Severity: LOW (misleading documentation, not a defect).

T8 line 10 claims dependency on T6's `embed_texts()`,
`chromadb.PersistentClient`, `collection`, `retrieve()`, `rag_answer()`,
`chunks`. T8 install line 80 does NOT install chromadb. T8 Cell 22
(line 668-715) uses a `_toy_chat` that calls
`client.chat.completions.create` directly with no RAG retrieval.

CONSEQUENCE: T8 works as written (no chromadb is required) but the
"What this topic builds on" claim in line 10 is misleading. The actual
T6 dependency is just: `client = OpenAI()`, `MODEL = "gpt-4o"`,
`BARCLAYS_SYSTEM_PROMPT` (which is actually a T3 artifact, not T6).

REMEDIATION: shorten T8 line 10 to:
"T6 (RAG foundations): no direct dependency in this notebook, but the
guardrails defined here are designed to wrap the rag_answer() helper
when integrated in T9."

---

## Section 4: Install-Line Inconsistencies

Severity: LOW-MEDIUM. Cosmetic but causes real student-side confusion
("why does Topic 8 use %pip when every other topic uses !pip?").

### S4.1 - T8 uses %pip magic; T7 and T9 use !pip shell

T7 line 179: `!pip install -q "chromadb==1.5.8" "openai>=2.30.0" "numpy<2"`
T9 line 249: `!pip install -q "openai>=2.30.0" "tenacity==9.1.4" "tiktoken==0.9.0" "chromadb==1.5.8" "numpy<2"`
T8 line 80: `%pip install --quiet "openai==2.32.0" "numpy<2"`

T1-T6 all use `!pip install -q ...` per established convention.

REMEDIATION: change T8 line 80 to:
```
!pip install -q "openai>=2.30.0" "numpy<2"
```

(Leave chromadb out since T8 does not use it. See S3.5.)

### S4.2 - T8 pins openai==2.32.0 exactly, T7/T9 use openai>=2.30.0

Severity: LOW.

T8 pins exactly `openai==2.32.0`; T7 and T9 use lower-bound `>=2.30.0`.
No conflict at install time (T7 ran first, pinned 2.32.0 would still
satisfy >=2.30.0). But if 2.33+ ships with a breaking change between
now and course day, T7 and T9 would break and T8 would not - or vice
versa. Inconsistent risk profile across topics.

REMEDIATION: align all three on `openai>=2.30.0,<3` or all three on
`openai==2.32.0`. Pick one based on whether you want students to see
the latest features (range pin) or maximum reproducibility (exact pin).

### S4.3 - T8 misreports T6 install pins

Severity: LOW (documentation only).

T8 line 10: "Pins `chromadb==1.5.8`, `openai>=1.30.0`, `numpy<2`."
T6 actual: pins `openai>=2.30.0` (per T7 line 179 which carries this
forward). The `1.30.0` value in T8 line 10 is wrong by a major version.

REMEDIATION: edit T8 line 10 to read `openai>=2.30.0`.

---

## Section 5: Web-Search Cost Note (Missing from Plans)

Severity: LOW (cost estimate accuracy).

T7 uses the OpenAI Responses API web_search tool with allowed_domains
filtering. As of April 2026, the OpenAI hosted web_search tool incurs
a per-call price (currently around 25 USD per 1000 calls) that is
SEPARATE from the per-token cost of the surrounding chat completion.

T9 Cell 15-16 builds a cost calculator (`compute_cost_usd`) that
multiplies prompt_tokens and completion_tokens by per-1k token prices.
This calculator does NOT add the per-call web_search fee for queries
that route to the hybrid_answer path.

CONSEQUENCE: the T9 cost log will systematically under-report total
cost on any query that hit web_search. This is fine for the teaching
demo but worth flagging in the wrap-up cell as a known production
gap and homework extension.

REMEDIATION: add a paragraph to T9 Cell 15 (cost calculator demo) or
Cell 25 (wrap-up) noting:
"This calculator covers per-token cost only. The OpenAI hosted
web_search tool adds a per-call fee on top (consult the current
OpenAI pricing page). When `route_query` returns 'hybrid' or 'web',
add the per-call web_search fee to your log_record manually."

---

## Section 6: Things That Are CORRECT (Leave Alone)

To save build-time review, here is what the audit confirmed is right:

- C1: T7, T8, T9 do not import or use `boto3`, `S3_BUCKET`,
  `barclays-prompt-eng-data`, `us-east-2`, or any AWS API. CORRECT
  per `instructor_setup.md` Cell 12 topic readiness summary.
- C2: All three plans use `gpt-4o` for chat completions; no Anthropic
  SDK is imported anywhere. CORRECT per CORE manifest line 76.
- C3: All three plans pin `numpy<2`. CORRECT per CORE manifest line 80.
- C4: All three plans use `text-embedding-3-small` for embeddings.
  CORRECT and consistent with T6.
- C5: T7 and T9 use `chromadb.PersistentClient(path="./chroma_db")`
  with `configuration={"hnsw": {"space": "cosine"}}` (the new ChromaDB
  config API). CORRECT and consistent with T6.
- C6: T7 has a continuity-rebuild cell (Cell 5 lines 290-299) that
  reuses an existing collection if present and embeds inline
  BARCLAYS_DOCS only when empty. CORRECT pattern per CORE line 37.
- C7: T9 has a full inline reference implementation of all prior-topic
  helpers (Cell 5 lines 332-695) so the notebook can run standalone.
  CORRECT per CORE line 37.
- C8: T7 web_search call signature
  `client.responses.create(tools=[{"type": "web_search",
  "filters": {"allowed_domains": ["barclays.co.uk"]}}])` matches the
  current OpenAI Responses API contract and is consistent with what
  T9 expects (T9 line 530 uses the same shape).
- C9: T7 names `web_search_barclays`, `extract_citations`,
  `route_query`, `hybrid_answer`, `vector_confidence` are referenced
  by T9 Cell 5 by exact name (T9 lines 59, 530+). CORRECT.
- C10: T8 GuardrailResult dataclass is well defined and the four
  guardrail functions consistently return it. CORRECT in isolation
  (the cross-topic mismatch with T9 is documented in S3.3 and S3.4).
- C11: All three plans pass the AI-tells scan: no em dashes, no en
  dashes, no Unicode multiplication, no smart quotes, no emoji.
- C12: All three install lines pin `numpy<2`. CORRECT.
- C13: None of the three plans create new files on disk other than
  ChromaDB persistence under `./chroma_db` (already used by T6) and
  in-memory log records. No new EBS state, no new S3 keys, no new
  DynamoDB tables, no new RDS schemas. CORRECT.

---

## Section 7: Recommended Remediation Order

If you can only do a subset, do them in this order. The first three
items unblock /build-topic-notebook for T7, T8, T9. The rest are
quality-of-life improvements.

1. RESOLVE S2.1 (T9 capstone scope conflict). User decision required:
   Option A (rewrite T9 to Banking77 transaction agent), Option B
   (update CORE to match current T9 plan), or Option C (keep T9 as
   main + add optional Banking77 notebook). Without this decision,
   T9 build is blocked.

2. FIX S3.2 (T9 ChromaDB collection name): one-character edit in
   T9 Cell 5 line 500: `barclays_capstone` -> `barclays_products`.

3. FIX S3.1 (BARCLAYS_DOCS drift): copy T6 lines 212-219 verbatim
   into T7 Cell 5 lines 253-273 and T9 Cell 5 lines 332-348. If T9
   needs additional operational docs (card freeze, Money Worries),
   append at indices 7-8 with a clear comment.

4. FIX S3.3 + S3.4 (T9 inline guardrail signatures): rewrite T9
   Cell 5 reimpls of `detect_pii` and `should_escalate` to match
   T8's GuardrailResult contract. Update T9 Cell 22 and Cell 23
   to consume `.passed`, `.redacted_text`, `.details` fields.

5. FIX S4.1 + S4.3 (T8 install line): change `%pip install --quiet`
   to `!pip install -q` and update the T8 Context line that misreports
   T6's openai pin.

6. FIX S4.2 (openai version pin): align T7, T8, T9 on the same
   openai SDK version specifier.

7. FIX S3.5 (T8 misleading T6 dependency claim): trim T8 line 10
   to clarify the actual dependency (BARCLAYS_SYSTEM_PROMPT and the
   OpenAI client only).

8. ADD S5 (web_search cost note): one paragraph in T9 Cell 15 or 25.

9. ADD S1.2 (optional barclays_chunks.json hint): one line each in
   T7 Tier 2 hard lab brief and T9 Tier 3 open-ended lab brief.

After applying items 2-9, T7 and T8 are ready for
/build-topic-notebook. Item 1 (S2.1) is the only blocker for T9 and
requires a user decision rather than an automatic fix.

---

## Appendix: Files Cross-Referenced

- plans/instructor_setup.md (the on-day S3 populator)
- plans/test_data_setup.md (the pre-day personal-bucket synthetic PDF generator)
- plans/CORE_TECHNOLOGIES_AND_DECISIONS.md (locked course manifest)
- plans/TOPICS.md (per-topic manifest)
- plans/topic_06_rag_foundations.md (BARCLAYS_DOCS ground truth)
- plans/topic_07_advanced_rag_web_search.md (audited)
- plans/topic_08_ethical_guardrails.md (audited)
- plans/topic_09_capstone.md (audited)

No web sources consulted (per user instruction).

# Course Topic Manifest
# Barclays - Generative AI: Prompt Engineering for Software Developers
# Last updated: 2026-04-27
#
# Status values: not_started | planned | in_progress | done
# Edit the status field manually or let /run-research-topic, /build-topic-notebook, /fixes update it.

---

## Topic 01 - Foundations

- **Status**: done
- **Day**: Day 1
- **Slug**: `topic_01_foundations`
- **Exercise**: `exercises/topic_01_foundations/topic_01_foundations.ipynb`
- **Solution**: `solutions/topic_01_foundations/topic_01_foundations.ipynb`
- **Plan**: `plans/topic_01_foundations.md`

### Concepts
- What is an LLM (decoder-only transformer, next-token prediction)
- How text generation works (temperature, sampling)
- Hallucinations and limitations
- Tasks LLMs excel at (summarization, classification, extraction, Q&A, code)
- Context windows and token limits (GPT-4o: 128K, Claude Sonnet 4.6: 200K)
- API options: OpenAI (gpt-4o, o4-mini) and Anthropic (claude-sonnet-4-6, claude-haiku-4-5)
- Tokens and cost (input vs output pricing)

### Labs
- Call the OpenAI API and the Anthropic API; compare responses for the same prompt
- Experiment with temperature and observe output variation
- Measure token counts and estimate cost for a sample workload

### Learning Objectives
- Explain what an LLM is and how it generates text
- Call both the OpenAI and Anthropic Python SDKs successfully
- Estimate token cost for a given interaction

### Key Libraries
- openai, anthropic

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 1` to produce the plan
- [x] Run `/build-topic-notebook 1` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check

### Open Issues
- [x] Plan file has bare --- separators and research URLs not as https:// links (resolved 2026-04-25)

---

## Topic 02 - NLP Preprocessing for RAG

- **Status**: done (fixes T2-FIX-001/002, T2-FIX-002, T2-FIX-003, T2-FIX-005 applied 2026-04-25/27)
- **Day**: Day 1
- **Slug**: `topic_02_nlp_preprocessing`
- **Exercise**: `exercises/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb`
- **Solution**: `solutions/topic_02_nlp_preprocessing/topic_02_nlp_preprocessing.ipynb`
- **Plan**: `plans/topic_02_nlp_preprocessing.md`

### Concepts
- Why preprocessing matters (noise degrades retrieval quality)
- Text extraction from PDFs using PyMuPDF
- Text extraction from HTML using BeautifulSoup
- Text cleaning (boilerplate removal, whitespace normalization, structure preservation)
- Chunking strategies (fixed-size, sentence, paragraph, semantic)
- Chunk size tradeoffs (precision vs context)

### Labs
- Extract text from a sample Barclays-style PDF using PyMuPDF
- Extract text from an HTML page using BeautifulSoup
- Implement a chunking function and compare chunk sizes
- Clean and normalize a raw extracted text sample

### Learning Objectives
- Extract clean text from PDFs and HTML
- Implement at least two chunking strategies and explain the tradeoff
- Build a preprocessing pipeline ready to feed a vector store

### Key Libraries
- pymupdf (fitz), beautifulsoup4, re, textwrap

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 2` to produce the plan
- [x] Run `/build-topic-notebook 2` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check
- [x] /fixes 2026-04-27: T02-FIX-001 (Mermaid diagram fitz.open -> pymupdf.open in cell ca6eb189),
      T02-FIX-002 (solution Cell 4e17ec14 pip install aligned to add openai==2.32.0)

---

## Topic 03 - Your First Chatbot

- **Status**: done
- **Day**: Day 1
- **Slug**: `topic_03_first_chatbot`
- **Exercise**: `exercises/topic_03_first_chatbot/topic_03_first_chatbot.ipynb`
- **Solution**: `solutions/topic_03_first_chatbot/topic_03_first_chatbot.ipynb`
- **Plan**: `plans/topic_03_first_chatbot.md`

### Concepts
- Chat Completions API structure (messages list, roles)
- System, user, and assistant roles
- Crafting effective system prompts (personality, constraints, domain)
- Basic single-turn conversation flow (receive query, call API, return response)

### Labs
- Build a basic single-turn chatbot function using the OpenAI API
- Write a system prompt that defines a Barclays Customer Service Assistant persona
- Test the assistant with sample banking queries

### Learning Objectives
- Use the Chat Completions API with system/user/assistant roles correctly
- Write a system prompt that constrains assistant behavior to a domain
- Build a working single-turn chatbot function

### Key Libraries
- openai, anthropic

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 3` to produce the plan
- [x] Run `/build-topic-notebook 3` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check
- [x] /fixes 2026-04-25: T3-FIX-002 (PRODUCT_SNIPPET -> BARCLAYS_PRODUCT_SNIPPET),
      T3-FIX-003 (rephrase inaccurate "richer context" claim),
      T3-FIX-004 (add GPT4O pricing constants to setup cell)

---

## Topic 04 - Prompt Engineering

- **Status**: done
- **Day**: Day 2
- **Slug**: `topic_04_prompt_engineering`
- **Exercise**: `exercises/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb`
- **Solution**: `solutions/topic_04_prompt_engineering/topic_04_prompt_engineering.ipynb`
- **Plan**: `plans/topic_04_prompt_engineering.md`

### Concepts
- Prompt structure and anatomy (context, instruction, input, output format)
- Few-shot learning (including examples in prompts)
- Output formatting and control (JSON mode, structured outputs, markdown)
- Task-specific prompting patterns (summarization, classification, extraction, Q&A)
- Iterative prompt refinement (systematic testing)

### Labs
- Build a few-shot prompt for intent classification (banking query categories)
- Use JSON mode to extract structured data from a customer message
- Write a summarization prompt and iterate to improve consistency
- Build an extraction prompt to pull account details from unstructured text

### Learning Objectives
- Design few-shot prompts for classification and extraction tasks
- Use JSON mode to get machine-parseable structured outputs
- Systematically iterate and improve a prompt through testing

### Key Libraries
- openai, anthropic, json

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 4` to produce the plan
- [x] Run `/build-topic-notebook 4` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check
- [x] /fixes 2026-04-27: T4-FIX-001 (sagemaker imports already removed - confirmed clean),
      T4-FIX-003 (wrap-up cell 90ecf431 Topic 5 -> Topic 9 corrected in both notebooks),
      T4-FIX-005 (openai pin already 2.32.0 - confirmed correct)

### Open Issues
- [x] T4-FIX-001: Remove dead sagemaker import block from Cell 3 (resolved 2026-04-27 - already clean)
- [x] T4-FIX-003: Fix false Topic 5 routing claim (resolved 2026-04-27 - wrap-up cell corrected)
- [x] T4-FIX-005: Pin openai to 2.32.0 in Cell 2 (resolved 2026-04-27 - already correct)

---

## Topic 05 - Conversation Memory

- **Status**: done (fixes T5-FIX-003 applied 2026-04-27)
- **Day**: Day 2
- **Slug**: `topic_05_conversation_memory`
- **Exercise**: `exercises/topic_05_conversation_memory/topic_05_conversation_memory.ipynb`
- **Solution**: `solutions/topic_05_conversation_memory/topic_05_conversation_memory.ipynb`
- **Plan**: `plans/topic_05_conversation_memory.md`

### Concepts
- Why memory matters (stateless API, context persistence across turns)
- Managing conversation history (append each turn, send full history)
- Context window constraints and truncation strategies
- Summarization for long conversations (compress old turns, preserve key context)

### Labs
- Build a multi-turn chatbot that maintains full conversation history
- Implement a sliding-window truncation strategy
- Implement a summarization strategy that compresses old turns when the context gets long
- Test the assistant across a 10-turn conversation about a banking product

### Learning Objectives
- Build a stateful multi-turn chatbot by managing the messages list manually
- Implement at least one context management strategy (truncation or summarization)
- Explain the tradeoff between truncation and summarization

### Key Libraries
- openai, anthropic

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 5` to produce the plan
- [x] Run `/build-topic-notebook 5` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check

### Open Issues
- [x] T5-FIX-003: BARCLAYS_SYSTEM_PROMPT replaced with T3 full canonical version (resolved 2026-04-27)

---

## Topic 06 - RAG Foundations

- **Status**: done (T679-FIX-001 filesystem-only handoff applied 2026-04-29)
- **Day**: Day 2
- **Slug**: `topic_06_rag_foundations`
- **Exercise**: `exercises/topic_06_rag_foundations/topic_06_rag_foundations.ipynb`
- **Solution**: `solutions/topic_06_rag_foundations/topic_06_rag_foundations.ipynb`
- **Plan**: `plans/topic_06_rag_foundations.md`

### Concepts
- What is RAG (retrieval-augmented generation)
- Embeddings (dense vectors, semantic similarity)
- Vector stores and similarity search (ChromaDB, cosine distance)
- Chunking for retrieval revisited (chunk size affects precision and context quality)
- The full RAG pipeline (query, embed, retrieve, augment prompt, generate)

### Labs
- Embed a set of document chunks using the OpenAI embeddings API
- Build a ChromaDB collection and add the embedded chunks
- Write a retrieval function (query -> embed -> search -> return top-k chunks)
- Augment the chatbot system prompt with retrieved context and test accuracy

### Learning Objectives
- Embed text chunks and store them in ChromaDB
- Implement a full RAG pipeline end-to-end
- Explain how retrieval quality affects generation quality

### Key Libraries
- openai, chromadb, numpy

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 6` to produce the plan
- [x] Run `/build-topic-notebook 6` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check

---

## Topic 07 - Advanced RAG and Web Search

- **Status**: done (T679-FIX-001 filesystem-only handoff applied 2026-04-29)
- **Day**: Day 3
- **Slug**: `topic_07_advanced_rag_web_search`
- **Exercise**: `exercises/topic_07_advanced_rag_web_search/topic_07_advanced_rag_web_search.ipynb`
- **Solution**: `solutions/topic_07_advanced_rag_web_search/topic_07_advanced_rag_web_search.ipynb`
- **Plan**: `plans/topic_07_advanced_rag_web_search.md`

### Concepts
- Limitations of document-only RAG (stale data, no current rates/promotions)
- Web search integration via OpenAI and Anthropic built-in tools
- Domain-restricted search (filter to barclays.co.uk only)
- Hybrid retrieval strategies (routing logic: vector store vs web search)
- Source prioritization and confidence scoring (when vector and web results conflict)

### Labs
- Integrate OpenAI web search tool to answer a query about current Barclays rates
- Implement domain restriction so only barclays.co.uk results are used
- Build a router that decides whether to use vector store, web search, or both
- Handle source conflicts with a simple prioritization rule

### Learning Objectives
- Add web search to an existing RAG pipeline using the OpenAI tools API
- Restrict search results to a specific domain
- Implement hybrid retrieval routing logic

### Key Libraries
- openai, chromadb, anthropic

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 7` to produce the plan
- [ ] Run `/build-topic-notebook 7` to build exercise notebook
- [ ] Build solution notebook after exercise approved
- [ ] Run `/validate-notebooks --pair ...` final check

---

## Topic 08 - Ethical Guardrails

- **Status**: done (ALL-FIX-001 sagemaker pin applied 2026-04-27)
- **Day**: Day 3
- **Slug**: `topic_08_ethical_guardrails`
- **Exercise**: `exercises/topic_08_ethical_guardrails/topic_08_ethical_guardrails.ipynb`
- **Solution**: `solutions/topic_08_ethical_guardrails/topic_08_ethical_guardrails.ipynb`
- **Plan**: `plans/topic_08_ethical_guardrails.md`

### Concepts
- Input guardrails: PII detection (account numbers, National Insurance numbers, sort codes)
- Input guardrails: sanitization (malformed input, injection attempts)
- Input guardrails: prompt injection detection and blocking
- Output guardrails: financial advice boundaries (factual product info only, no personalized advice)
- Output guardrails: bias and fairness (no discrimination on protected characteristics)
- Output guardrails: escalation triggers (complaints, vulnerability indicators, complex disputes)

### Labs
- Build a PII detector using regex patterns for UK financial identifiers
- Build a prompt injection detector (rule-based + LLM-as-judge)
- Write an output validator that flags financial advice boundary violations
- Build an escalation detector that identifies queries requiring human handoff

### Learning Objectives
- Detect and handle PII in customer inputs
- Detect and block prompt injection attempts
- Validate LLM outputs against financial advice and fairness constraints
- Implement an escalation trigger for sensitive queries

### Key Libraries
- openai, anthropic, re

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 8` to produce the plan
- [x] Run `/build-topic-notebook 8` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check

---

## Topic 09 - Capstone: Production Customer Service Assistant

- **Status**: done (T679-FIX-001 filesystem-only handoff applied 2026-04-29)
- **Day**: Day 3
- **Slug**: `topic_09_capstone`
- **Exercise**: `exercises/topic_09_capstone/topic_09_capstone.ipynb`
- **Solution**: `solutions/topic_09_capstone/topic_09_capstone.ipynb`
- **Plan**: `plans/topic_09_capstone.md`

### Concepts
- Error handling for LLM applications (API failures, rate limits, timeouts, retry logic)
- Cost optimization (caching, model tiering, prompt optimization)
- Logging and observability (query, retrieval, generation, and cost logging)
- Latency optimization (streaming responses, parallel retrieval, caching)

### Labs
- Wire together all components: system prompt, conversation memory, RAG pipeline, web search, input guardrails, output guardrails
- Add retry logic with exponential backoff for API failures
- Add a cost tracker that logs token usage per request
- Add streaming output so responses appear progressively in the notebook
- End-to-end test: run 5 representative customer queries through the complete assistant

### Learning Objectives
- Integrate all course components into a single working assistant
- Add production-grade error handling, logging, and cost tracking
- Demonstrate the complete assistant handling realistic banking queries end-to-end

### Key Libraries
- openai, anthropic, chromadb, re, time, logging

### Manifest (what needs to happen)
- [x] Run `/run-research-topic 9` to produce the plan
- [x] Run `/build-topic-notebook 9` to build exercise notebook
- [x] Build solution notebook after exercise approved
- [x] Run `/validate-notebooks --pair ...` final check

### Open Issues
- [x] T9-FIX-004: BARCLAYS_SYSTEM_PROMPT replaced with T3 full canonical version (resolved 2026-04-27)
- [x] T9-FIX-007: CLASSIFICATION_SCHEMA and classify_with_schema aligned to T4's 3-field schema (resolved 2026-04-27)
- [x] T9-FIX-008: count_tokens_in_messages corrected to T5's 3+3 formula (resolved 2026-04-27)
- [x] T9-FIX-009: BarclaysChat.__init__ aligned to T5 signature, max_tokens as class attr (resolved 2026-04-27)
- [x] T9-FIX-012: web_search per-call cost note added to end-to-end test cell eddf92cc860d (resolved 2026-04-27)
- [x] T9-FIX-013: barclays_chunks.json stretch hint added to Tier 3 lab brief b59e17a9a1df (resolved 2026-04-27)

---

## Setup Notebooks

### INSTRUCTOR_TEST_SETUP.ipynb

- **Status**: done
- **File**: `setup/INSTRUCTOR_TEST_SETUP.ipynb`
- **Purpose**: Run once from personal AWS account to generate synthetic PDFs and upload to personal public S3 bucket for pre-course testing.

#### Fixes Applied
- [x] TEST-SETUP-FIX-001: numpy<2 added to pip install line in cell 6848e038 (2026-04-27)
- [x] TEST-SETUP-FIX-002: BankingDocPDF body_text() and qa_block() changed multi_cell(0,...) to multi_cell(self.epw,...) to fix FPDFException with align="J" (2026-04-27)

### INSTRUCTOR_SETUP.ipynb

- **Status**: done
- **File**: `setup/INSTRUCTOR_SETUP.ipynb`
- **Purpose**: Run on course day morning inside SageMaker to upload real Barclays PDFs to barclays-prompt-eng-data S3 bucket.

#### Fixes Applied
- [x] SETUP-FIX-004: PDF manifest updated with all confirmed public Barclays URLs including savings rates, ISA terms, guide to ISAs; personal loan URL reviewed (2026-04-27)

---

## Summary

| # | Topic | Day | Status |
|---|-------|-----|--------|
| 01 | Foundations | Day 1 | done |
| 02 | NLP Preprocessing for RAG | Day 1 | done |
| 03 | Your First Chatbot | Day 1 | done |
| 04 | Prompt Engineering | Day 2 | done |
| 05 | Conversation Memory | Day 2 | done |
| 06 | RAG Foundations | Day 2 | done |
| 07 | Advanced RAG and Web Search | Day 3 | done |
| 08 | Ethical Guardrails | Day 3 | done |
| 09 | Capstone | Day 3 | done |
| E8 | Extra: Data Engineer Use Case | Extra | planned |

---

## Extra Topic 08 - Data Engineer Use Case

- **Status**: planned
- **Day**: Extra (post-course showcase / reference)
- **Slug**: `extra_08_data_engineer_usecase`
- **Exercise**: `exercises/extra_08_data_engineer_usecase/extra_08_data_engineer_usecase.ipynb`
- **Solution**: `solutions/extra_08_data_engineer_usecase/extra_08_data_engineer_usecase.ipynb` (optional - recommendation is single notebook in exercises/ only since there are no labs)
- **Plan**: `plans/extra_08_data_engineer_usecase.md`

### Concepts (re-application of T1-T9 primitives, no new material)
- Structured Outputs as the right primitive for batch enrichment pipelines
- Row-level fan-out: classification, PII redaction, RAG-grounded answer, output validation
- Filesystem-only handoff: rebuild corpus from S3 inline every run
- tenacity retry + dead-letter pattern for row-level resilience
- Per-request cost and latency reporting
- When to use the OpenAI Batch API vs synchronous calls vs threaded parallel sync

### Format
- SHOWCASE notebook for data engineers, NOT a lab notebook
- No `None # YOUR CODE` placeholders, no Tier 1/2/3 labs, no verification cells
- Every cell is a runnable, heavily commented demo

### Key Libraries
- openai==2.32.0, chromadb==1.5.8, tenacity==9.1.4, pymupdf==1.27.2.2, tiktoken==0.9.0, pandas, boto3, numpy<2

### Manifest (what needs to happen)
- [x] Plan written at `plans/extra_08_data_engineer_usecase.md`
- [ ] User decision: single notebook in `exercises/` only, or dual-folder convention
- [ ] Build showcase notebook
- [ ] Build diagrams (`pipeline-overview`, `row-enrichment-fanout`, `sync-vs-batch`)
- [ ] Final validation pass (AI-tells scan, structure check)

### Open Questions
- [ ] Confirm single-notebook recommendation vs dual exercises/solutions folder layout

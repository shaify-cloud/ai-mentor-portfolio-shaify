# AI Mentor Portfolio - Mohammad Shaifu Zama

## ChatGPT
I would use ChatGPT for summarisation and reasoning because of its structured and transparent way of providing output. The code generation is good, but compared with other tools, it is slightly underperforming. The overall flow is good; however, the step-by-step explanation of the code could be improved.

## Claude
I would use Claude for coding and reasoning because its code correctness and logical flow are very good. The reasoning explanation is acceptable and satisfactory. However, when it comes to summarisation, Claude does not perform as well, as it often provides simple and less structured explanations compared with other tools.

## Gemini
I would use Gemini because it performs well in summarisation, coding, and reasoning tasks. It provides balanced and neutral explanations with a consistent flow across all tasks. Compared with the other tools, I would rate Gemini as a neutral and reliable tool that provides satisfactory outputs.

## Perplexity
I would use Perplexity mainly for citations and source-based responses. Apart from providing sources, it performs well in summarisation compared with its other capabilities. However, for coding and reasoning tasks, it does not provide outputs as satisfying as the other tools.

## Image Input Support
Currently, none of the models from these four tools restrict access to image input on their free tiers. Based on observation, all of them accept image inputs in the free version.

## Day 1 — Setup complete

- ✅ Google AI Studio API key provisioned
- ✅ Groq API key provisioned
- ✅ Hello-Gemini call working — see

<img width="1620" height="644" alt="gemini_first_call" src="https://github.com/user-attachments/assets/b328d2a4-721b-425e-8e6c-9b5b144bf291" />

- 4-tool comparison matrix from Lab 1A: see screenshot below

<img width="1620" height="143" alt="Day-01-Matrix" src="https://github.com/user-attachments/assets/f20f3da1-9301-423a-80e4-109b339ab55a" />

## Day 4 — Productivity sprint

**Company:** Capgemini
**Time:** 45 minutes (timeboxed)


## Day 5 Lab 5B — Hugging Face Pulls

### Models tested
- `facebook/bart-large-mnli` — zero-shot classification
- `distilbert-base-uncased-finetuned-sst-2-english` — sentiment

### Timing comparison

| | min | avg | Notes |
|---|-----|-----|-------|
| HF Inference API | 0.8s | 1.2s | Cold-start: 20s |
| Local in Colab | 2.1s | 3.4s | Download: 60s on first run |

## Day 6 Lab 6A — Errors handled

1. **Markdown fence wrapping** (`\`\`\`json ... \`\`\``)
   The retry prompt asks Gemini to output raw JSON without fences. Triggers on ~5-10% of calls.

2. **Hallucinated phone number when source has none**
   `Optional[str] = None` in Pydantic — model returns `null`, schema validates.

3. **Empty / whitespace-only input**
   Pydantic raises ValidationError with "Field required". Caller catches.

**Hallucination on garbage input:** Gemini sometimes invents a plausible résumé from non-résumé text. Defence: validate input before sending (e.g., minimum length, presence of email-like pattern).

## Day 6 — Capstone Sprint 1: PlacementDataProcessor

### Engineer Answer

1. **PROBLEM** — JDs from Naukri / LinkedIn are messy text — placement cells need structured data to filter ("which JDs want Java + CGPA 7+?"). Manual extraction is unscalable for 50+ JDs.

2. **ARCHITECTURE** — JD URL → BeautifulSoup scraper (extract clean text) → Gemini structured-output call (response_schema=JD Pydantic) → JSON Lines file. Validation at each step; retry on schema fail.

3. **TRADE-OFFS** —
   - Cost: free Gemini ~1 JD/sec on average; ~30K tokens/day quota → ~5K JDs/day.
   - Accuracy: Pydantic catches schema violations but not semantic errors (e.g., model says skill is "Python" when source says "Python 3.12 specifically").
   - Latency: ~2-5s per JD (Gemini call dominant).
   - Complexity: scraping fragile (sites block automation). Cached fallback is mandatory.

4. **SCALE** —
   - 10 JDs/day: trivial. Today's lab.
   - 100 JDs/day: still in free quota. Add overnight batch + sleep between calls.
   - 10K JDs/day: free tier breaks. Move to paid Gemini OR self-host an open model.

5. **INTERVIEW ANSWER** — "I built a structured-output pipeline that turns scraped JDs into clean filterable JSON, using free Gemini and Pydantic. Schema-first design with retry-on-failure made it production-shaped on a free-tier API."

### Files
- `Day6_PlacementProcessor.ipynb` — the notebook
- `data/jds.jsonl` — output of this sprint, input for Day 7 RAG

- ## Day 7 Lab 7A — ChromaDB Hello-World

- Embedded 10 CSE Sem 5 paragraphs with all-MiniLM-L6-v2 (384-dim, free)
- Indexed in persistent ChromaDB collection `hello_syllabus`
- Ran 3 semantic queries — observed: top-1 match is relevant when query topic is in corpus, irrelevant when not
- Plotted PCA 2D — visible OS / DBMS clusters

**Reflection:** Semantic search returns nearest, not exact. RAG must enforce citations to catch out-of-corpus queries (this afternoon's Sprint 2).
## Day 7 — Capstone Sprint 2: PlacementKnowledgeRAG

### Engineer Answer

1. **PROBLEM** — Frontier LLMs do not know your private data (JDs, syllabi). Students need a chatbot that answers from YOUR placement corpus, with citations they can verify.

2. **ARCHITECTURE** — 5-box RAG: embed (MiniLM 384-dim) → index (ChromaDB persistent collection with metadata) → retrieve (top-4 cosine similarity) → augment (citation-enforcing prompt) → generate (Gemini 2.5).

3. **TRADE-OFFS** —
   - Cost: free (MiniLM local + Gemini quota).
   - Accuracy: top-4 retrieval has ~80% precision on placement-relevant queries.
   - Latency: ~1-2s per query (embedding + retrieval) + 2-5s (Gemini).
   - Complexity: chunking strategy (500-token, 50-overlap) needs tuning per corpus.
   - Caveat: refuses out-of-corpus queries (good!) but only when prompt enforces "do not guess".

4. **SCALE** —
   - 50 docs (today): trivial. ChromaDB returns in <100ms.
   - 5K docs: still fine on one machine.
   - 1M docs: need vector DB optimisation (HNSW indexing, server mode), or move to Pinecone/Weaviate.

5. **INTERVIEW ANSWER** — "I built a citation-enforcing RAG over 50+ placement docs (JDs + syllabi) using free MiniLM embeddings, ChromaDB, and Gemini. The system either cites a specific chunk or refuses — no hallucinated answers. Same pattern scales to thousands of docs without retraining."

### 5 cited Q&A pairs

| # | Question | Answer (excerpt) | Sources cited |
|---|----------|------------------|---------------|
| 1 | Which companies want Java + DSA + CGPA 7+? | "Per jd_0 (TCS Digital): Java + DSA required, CGPA 7.0 cutoff..." | jd_0, jd_5, jd_8 |
| 2 | Sem 5 OS topics? | "Per cse_sem5_2: paging, segmentation, virtual memory..." | cse_sem5_2, cse_sem5_5 |
| 3 | Which JDs require Python? | "Per jd_3 (Accenture)..., per jd_5 (Cognizant)..." | jd_3, jd_5, jd_9 |
| 4 | Top 3 skills across JDs? | "Java, Python, SQL appear in 7+ of 10 JDs..." | jd_0, jd_1, jd_2 |
| 5 | What is TCS Codevita? | "I do not know — not in corpus." | (none) |

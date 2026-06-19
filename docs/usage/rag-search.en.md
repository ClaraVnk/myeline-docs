# RAG search

RAG search is the heart of Myeline: ask a natural-language question,
get a synthesised answer **with source citations**.

## Pipeline

```
User question
  ↓
Embedding (on-prem: bge-m3 local Ollama — Cloud: Mistral EU API)
  ↓
ChromaDB hybrid search (RRF + MMR over the selected scopes)
  ↓
HyDE / multi-query / contextual retrieval (auxiliary LLM)
  ↓
Cross-encoder reranking (auxiliary LLM)
  ↓
Synthesis (local Ollama OR external BYOK API depending on edition)
  ↓
Answer + citations [1], [2], [3]…
```

**Embedding depends on the edition**:

- In **sovereign / hybrid on-prem**: always **local** (bge-m3 via
  Ollama, on your infra).
- On the **hosted Cloud**: through the **Mistral embedding API** (EU /
  France) — included platform key on Pro, **your own key** on Free (a
  Mistral or OpenAI key; Claude/Gemini can't index).

Synthesis:

- In pure sovereign: local Ollama (Mistral-Nemo, Llama 3.1, Mixtral
  depending on what you host)
- In sovereign-hybrid: local Ollama by default, or per-organisation
  switch to Mistral / Claude / OpenAI / Gemini with your key (BYOK)
- On the Cloud: included Mistral (Pro) or your BYOK key (Free)

## Citations

Every answer cites its sources as clickable `[1]`, `[2]`. Each
source displays:

- **Document title** or article title
- **Author / source** when available
- **Date** when available
- **Excerpt** (~500-token chunk) that contributed to the answer
- **Relevance score**

If an answer has no citation, the LLM didn't find support in the
documents — either the base lacks the information, or the question
is too vague.

## Strict mode

Toggle in the UI: when enabled, **the LLM is forced to answer only
based on retrieved sources** (explicit system prompt + post-validation).
If no source covers the question, the answer is: "I couldn't find
the information in the available sources."

Recommended for:

- Regulatory / legal research
- Medical research
- Any context where a hallucination would be a risk

## Single-document

From the library, clicking a document opens a chat **scoped to that
single document**. Useful for:

- Querying a long report ("what are the recommendations?")
- Comparing sections ("do the conclusions of part 3 contradict
  part 1?")
- Extracting numerical data ("what is the 2024 revenue mentioned?")

## Writing tips

- **Precise questions** > vague ones. "What are the explicit-consent
  exceptions in GDPR art. 6?" works better than "tell me about
  GDPR".
- **Context**: state the domain if your sources cover several
  topics ("in French employment law, …").
- **Iterate**: refine the question across multi-turns (see
  [Conversations](conversations.md)) rather than dumping everything
  at once.

## Known limits

- Documents > 200 pages: chunking can dilute links between distant
  sections. Prefer single-document chat with contextual retrieval
  enabled.
- Complex tables: imperfect PDF extraction (limits of
  `pdfplumber`/`docx` parsers). Prefer a CSV export when possible.
- Mixed languages in the same base: `bge-m3` is multilingual (100+
  languages), but retrieval quality degrades when FR and EN mix in
  the same question.

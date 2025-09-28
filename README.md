# EasyT9N: RAG-Enhanced Translation System

**EasyT9N** is an end-to-end project that compares **baseline LLM translation** against a **RAG-enhanced translation** pipeline.

For demo purposes, the externalized strings of a press-on nail ecommerce web app was chosen as the sample text. The domain choice is based on the fact that nail-art terminology is nuanced and brand-sensitive (e.g., Gel-X, shapes, finishes), and often mistranslated unless given explicit context. This repo shows how **retrieval-augmented generation** (glossary + translation memory) drives better consistency and correctness in **French (fr)**, **Japanese (ja)**, and **Italian (it)**.

---

## Highlights

- **Baselines:** Gemini 2.5 Flash, GPT-4o Mini, Claude 3.5 Sonnet (fast, cost-effective, temperature-enabled).
- **RAG Pipeline:**
  - Embeddings: **`intfloat/multilingual-e5-base`** (picked from MTEB for multilingual retrieval vs. size).
  - Vector DB: **Chroma** (persistent local index).
  - **Glossary + TM retrieval** → prompt constraints (DNT, terminology) → translation.
- **Evaluations & Reports:**
  - DNT preservation, **HTML tag fidelity**, glossary adherence.
  - Retrieval precision (relevance of retrieved terms).
  - **Runtime, token usage, estimated cost** (from metered API usage).

---

## Security

- All API keys are kept in **`.env`** so secrets never leave your machine.
- The notebooks avoid printing secrets and degrade gracefully if a provider key is missing.

---

## Repo Structure

```
.
├─ data/
│  ├─ en.json                     # source strings
│  ├─ glossary.csv                # domain terms + DNT flags + per-locale mappings
│  └─ translation_memory.csv      # curated TM
├─ translations/
│  ├─ baseline/                   # baseline outputs per model, per language
│  └─ rag/                        # RAG outputs per language
├─ eval/
│  ├─ baseline/                   # baseline run metrics snapshots (per-lang json)
│  ├─ rag_run_summary.csv         # metered tokens/runtime/cost (RAG)
│  ├─ rag_vs_baseline_comparison.csv
│  ├─ rag_comprehensive_evaluation.csv
│  └─ final_comparison.csv        # quality + runtime/cost side-by-side
├─ .env                           
├─ Baseline.ipynb                 # run baseline models and save outputs
└─ RAG_Enhanced.ipynb  		      # full RAG pipeline + evaluation
```

---

## Data Formats

### `data/en.json`
- Nested JSON of source strings. The pipeline flattens to `(path, text)` pairs and preserves paths in outputs.

### `data/glossary.csv` (example columns)

| source_term     | fr                | ja           | it                | dnt   |
|-----------------|-------------------|--------------|-------------------|-------|
| press-on nails  | press-on nails    | ネイルチップ | press-on nails    | FALSE |
| NaiLit          | NaiLit            | NaiLit       | NaiLit            | TRUE  |

- `dnt=TRUE` keeps the term verbatim (case-sensitive) in the target.

### `data/translation_memory.csv` (example columns)

| tm_id   | src_lang | tgt_lang | src_text                                                | tgt_text                                                | domain | quality  | frozen |
|---------|----------|---------|---------------------------------------------------------|---------------------------------------------------------|--------|----------|--------|
| tm-0001 | en       | fr      | {strong}NaiLit{/strong} …                               | {strong}NaiLit{/strong} …                               | ui     | approved | FALSE  |

- **Exact TM matches win first**; otherwise the RAG prompt uses glossary constraints.

---

## Requirements

- **Python**: `>= 3.13`

Install dependencies from `requirements.txt`:

```bash
pip install -r requirements.txt
```

---

## Setup

1) Create & activate a virtual environment
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
pip install -U pip
```

2) Install dependencies
```bash
pip install -r requirements.txt
```

3) Create `.env` (not tracked by git)
```env
# OpenAI
OPENAI_API_KEY=sk-...

# Anthropic
ANTHROPIC_API_KEY=...

# Google (google-genai)
GOOGLE_API_KEY=...

# Optional hints
OPENAI_BASELINE_MODEL=gpt-4o-mini
BASELINE_MODEL_HINT="gpt openai claude gemini"
RAG_MAX_WORKERS=4
OPENAI_RAG_MODEL=gpt-4o-mini
```

4) Put your data in `data/`:
- `en.json` (source strings)
- `glossary.csv` (DNT + mappings)
- `translation_memory.csv` (optional but recommended)

---

## How to Run

### A) Baselines
Open **`Baseline.ipynb`** and run all cells:
- Produces `translations/baseline/<…>/{fr,ja,it}.json`
- Saves baseline metrics to `eval/baseline/<…>/metrics_{lang}.json`

### B) RAG Pipeline
Open **`RAG_Enhanced_Translation_System.ipynb`** and run all cells:
- Builds/updates a persistent **Chroma** index from `glossary.csv`
- Uses **`intfloat/multilingual-e5-base`** embeddings to retrieve glossary terms
- Translates with prompt constraints (DNT, glossary mappings), preserving **HTML tags**
- Saves outputs to `translations/rag/{lang}.json`
- Logs metered **tokens**, **runtime**, and **estimated cost** to `eval/rag_run_summary.csv`

### C) Evaluation & Reports
The RAG notebook writes:
- `eval/rag_vs_baseline_comparison.csv` – coverage & similarity diagnostics
- `eval/rag_comprehensive_evaluation.csv` – DNT/glossary/tags/retrieval + BLEU/chrF + semantic similarity
- `eval/final_comparison.csv` – consolidated **quality + runtime/cost** table with deltas

---

## Methodology

- **Baselines:** Gemini 2.5 Flash, GPT-4o Mini, Claude 3.5 Sonnet were chosen for speed, cost, and temperature control. All results are kept for transparent comparison.
- **RAG:** Exact TM match → otherwise retrieve glossary terms (semantic + lexical boost) → enforce DNT and tag fidelity in prompts.
- **Why nail art?** The domain has many locale-specific terms. Without retrieval, LLMs often normalize or mistranslate specialized phrasing. RAG narrows the model’s options to the correct vocabulary.

---

## Known Limitations

- Very short strings can be tricky for retrieval; tune `top_k`/`min_score`.
- `sentence-transformers` will bring in a compatible `torch`; GPU acceleration depends on your environment.

---

## Extending

- Add more locales or domain TMs (e.g., “ui” vs “marketing”).
- Add **style profiles** per locale (tone, formality).
- Swap in newer MTEB-ranked embedding models as they land.
- Sample segments for **post-edit distance** to estimate MTPE effort saved.


---

## Acknowledgments

- `intfloat/multilingual-e5-base` (Hugging Face) for multilingual retrieval.
- **Chroma** for simple, persistent vector search.
- **OpenAI / Anthropic / Google** SDKs for generation.

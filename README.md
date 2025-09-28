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
└─ RAG_Enhanced.ipynb             # full RAG pipeline + evaluation
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
|---------|----------|----------|---------------------------------------------------------|---------------------------------------------------------|--------|----------|--------|
| tm-0001 | en       | fr       | {strong}NaiLit{/strong} …                               | {strong}NaiLit{/strong} …                               | ui     | approved | FALSE  |

- **Exact TM matches win first**; otherwise the RAG prompt uses glossary constraints.

---

## Setup

**Requirements**  
- Python **3.13+**  
- [JupyterLab](https://jupyter.org/install) for running notebooks  
- [uv](https://github.com/astral-sh/uv) as the package/dependency manager  

**1) Install uv**  
On macOS/Linux:  
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

On Windows (PowerShell):  
```powershell
irm https://astral.sh/uv/install.ps1 | iex
```

**2) Create a project venv and install deps**  
```bash
uv venv
source .venv/bin/activate       # Windows: .venv\Scripts\activate
uv pip install -r requirements.txt
```

**3) Install JupyterLab if not present**  
```bash
uv pip install jupyterlab
```

**4) Add API keys to `.env`**  
```dotenv
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
GOOGLE_API_KEY=...
```

Or just edit the `.env` file provided separately.

**5) Prepare data files**  
To run the demo, ensure the following are present:  
- `data/en.json` (source text)  
- `data/glossary.csv` (EN→FR/JA/IT + DNT flag)  
- `data/translation_memory.csv`

---

## How to Run (Baseline → RAG → Compare)

### Launch JupyterLab
```bash
jupyter lab
```
Open the `notebooks/` folder.

### A) Baselines
Open **`Baseline.ipynb`** and run all cells:  
- Translate EN→FR/JA/IT without retrieval  
- Produces `translations/baseline/<…>/{fr,ja,it}.json`  
- Saves baseline metrics to `eval/baseline/<…>/metrics_{lang}.json`  

### B) RAG Pipeline
Open **`RAG_Enhanced.ipynb`** and run all cells:  
- Embed glossary into Chroma  
- Retrieve glossary terms per source segment  
- Inject them as constraints at inference time  
- Outputs: `translations/rag/{fr,ja,it}.json`, `eval/rag_comprehensive_evaluation.csv`  

### C) Comparison
The RAG notebook writes:  
- Evaluates DNT, glossary adherence, retrieval precision, tag preservation  
- Saves:  
  - `eval/rag_vs_baseline_comparison.csv`  
  - `eval/final_comparison.csv`  

---

## Troubleshooting

- If API requests are throttled: reduce `MAX_WORKERS` to 2–3.  
- If glossary changes aren’t applied: delete `.chroma/` and rerun RAG.  
- If wrong baseline folder loads: adjust `BASELINE_MODEL_HINT`.  

---

## Acknowledgments

- `intfloat/multilingual-e5-base` (Hugging Face) for multilingual retrieval.

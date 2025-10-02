# EasyT9N: RAG-Augmented LLM Translation System

EasyT9N is an end-to-end project comparing baseline LLM translation with a RAG-enhanced pipeline that integrates glossary and translation memory retrieval. It demonstrates how retrieval-augmented generation improves consistency in multilingual localization.

For demo purposes, the externalized strings of a press-on nail ecommerce web app was chosen as the sample text. The domain choice is based on the fact that nail-art terminology is nuanced and brand-sensitive (e.g., Gel-X, shapes, finishes), and often mistranslated unless given explicit context. This repo shows how **retrieval-augmented generation** (glossary + translation memory) drives better consistency and correctness in **French (fr)**, **Japanese (ja)**, and **Italian (it)**.

---

## Highlights
- **Baselines**: Gemini 2.5 Flash, GPT-4o Mini, Claude 3.5 Sonnet  
- **RAG Pipeline**:  
  - Embeddings: `intfloat/multilingual-e5-base` (strong multilingual retrieval performance)  
  - Vector DB: Chroma (persistent local index)  
  - Glossary + TM retrieval → prompt constraints → LLM translation  
- **Evaluation**:  
  - DNT preservation, glossary adherence, HTML tag fidelity  
  - Retrieval precision  
  - Runtime, latency, token usage, cost estimates  
- **Security**:  
  - API keys stored in `.env` (ignored by Git)  

---

## Repo Structure

```
.
├─ data/
│  ├─ en.json                     # source strings
│  ├─ glossary.csv                # domain terms + DNT flags + mappings
│  └─ translation_memory.csv      # curated TM entries
├─ translations/
│  ├─ baseline/                   # baseline outputs (per model/lang)
│  └─ rag/                        # RAG outputs (per lang)
├─ eval/
│  ├─ baseline/                   # baseline metrics snapshots
│  ├─ rag_run_summary.csv         # tokens/runtime/cost
│  ├─ rag_vs_baseline_comparison.csv
│  ├─ rag_comprehensive_evaluation.csv
│  └─ final_comparison.csv
├─ .env                           # store API keys here (ignored by git)
├─ Baseline.ipynb                 # naive baseline pipeline
└─ RAG_Enhanced.ipynb             # glossary retrieval + RAG pipeline
```

---

## Data Formats

**`data/en.json`**  
- Nested JSON of source strings.  
- Flattened into `(path, text)` pairs during preprocessing.  

**`data/glossary.csv`** (example)  
| source_term   | fr               | ja         | it             | dnt   |  
|---------------|-----------------|------------|----------------|-------|  
| press-on nails | press-on nails | ネイルチップ | press-on nails | FALSE |  
| NaiLit        | NaiLit          | NaiLit      | NaiLit         | TRUE  |  

- `dnt=TRUE` → preserve verbatim in target.  

**`data/translation_memory.csv`** (example)  
| tm_id | src_lang | tgt_lang | src_text                  | tgt_text                  | domain | quality   | frozen |  
|-------|----------|----------|---------------------------|---------------------------|--------|-----------|--------|  
| tm-001| en       | fr       | {strong}NaiLit{/strong} … | {strong}NaiLit{/strong} … | ui     | approved  | FALSE  |  

---

## Setup

### Requirements
- Python 3.13+  
- [uv](https://github.com/astral-sh/uv) as package manager  
- JupyterLab for running notebooks  

### Installation

1. **Install uv**  
   - macOS/Linux:  
     ```bash
     curl -LsSf https://astral.sh/uv/install.sh | sh
     ```  
   - Windows (PowerShell):  
     ```powershell
     irm https://astral.sh/uv/install.ps1 | iex
     ```  

2. **Create a venv and install dependencies**  
   ```bash
   uv venv
   source .venv/bin/activate    # Windows: .venv\Scripts\activate
   uv pip install -r requirements.txt
   ```  

3. **Install JupyterLab if missing**  
   ```bash
   uv pip install jupyterlab
   ```  

4. **Add API keys** in a `.env` file (Optional if you would like to use your own; .env file includes readily available API Keys):  
   ```
   OPENAI_API_KEY=...
   ANTHROPIC_API_KEY=...
   GOOGLE_API_KEY=...
   ```  

5. **Prepare data files**  
   Ensure the following are present in `data/`:  
   - `en.json` (source)  
   - `glossary.csv` (glossary with dnt flag)  
   - `translation_memory.csv`  

---

## How to Run

Launch JupyterLab:  
```bash
jupyter lab
```

### A) Baseline
- Open `Baseline.ipynb`  
- Run all cells to translate EN→FR/JA/IT without retrieval  
- Outputs to: `translations/baseline/{lang}.json`  
- Metrics saved to: `eval/baseline/metrics_{lang}.json`  

### B) RAG Pipeline
- Open `RAG_Enhanced.ipynb`  
- Run all cells:  
  - Embed glossary into Chroma  
  - Retrieve constraints per source segment  
  - Inject into prompts → translation  
- Outputs to: `translations/rag/{lang}.json`  
- Evaluation saved to: `eval/rag_comprehensive_evaluation.csv`  

### C) Comparison
The RAG notebook also produces:  
- `eval/rag_vs_baseline_comparison.csv`  
- `eval/final_comparison.csv`  

---

## Troubleshooting

- **API quota exceeded**: Reduce `MAX_WORKERS` to 1–2.  
- **Glossary not applied**: Delete `.chroma/` and rerun RAG.  
- **Baseline mismatch**: Adjust `BASELINE_MODEL_HINT`.  
- **Progress bars missing**: Install `ipywidgets` for JupyterLab.  

---

## Reproducibility Options

This project can be distributed in two ways:  
1. **With results included**: all translations and evaluation CSVs pre-computed for quick review.  
2. **Clean run**: Please delete all files under /EasyT9N/eval and /EasyT9N/translations to reproduce a clean run.  

---

## Acknowledgments
- `intfloat/multilingual-e5-base` (Hugging Face) for multilingual embeddings.  
- ChromaDB for lightweight vector search.  

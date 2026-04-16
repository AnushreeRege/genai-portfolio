# 📚 Semantic Course Matching — RAG Pipeline
### IBM Granite Embeddings · Milvus · Docling · LangChain · LLM Auditor

---

## Overview

A Retrieval-Augmented Generation (RAG) pipeline that semantically matches university course modules across institutions — designed to support academic credit transfer evaluation. Given a source module from University A, the system retrieves the most similar modules from University B and uses an LLM auditor to verify and explain the match.

## Pipeline

```
PDF Module Handbooks (2 universities)
        │
        ▼
┌─────────────────┐
│  Docling Parser │  extracts structured module docs from PDFs
│                 │  handles tables, page breaks, layout edge cases
└────────┬────────┘
         ▼
┌─────────────────┐
│ Deduplication   │  removes cross-listed duplicates by module code
└────────┬────────┘
         ▼
┌─────────────────┐
│ Milvus Vector   │  stores embeddings per module
│ Database        │  IBM Granite 107M multilingual embeddings
└────────┬────────┘
         ▼
┌─────────────────┐
│ Retriever +     │  semantic search + cross-encoder re-ranking
│ Re-ranker       │
└────────┬────────┘
         ▼
┌─────────────────┐
│  LLM Auditor    │  verifies match quality with natural language
│                 │  reasoning (Ollama / Granite)
└────────┬────────┘
         ▼
┌─────────────────┐
│ HTML + CSV      │  similarity report for academic review
│ Report Export   │
└─────────────────┘
```

## Setup

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Start Milvus (via Docker)
docker run -d --name milvus \
  -p 19530:19530 \
  milvusdb/milvus:latest

# 3. Start Ollama
ollama serve
ollama pull granite3.3:8b

# 4. Place PDF handbooks in the project folder
#    - 07_Modul_Handbook.pdf  (Stuttgart)
#    - target handbook PDF    (Darmstadt)

# 5. Launch notebook
jupyter notebook RAG_Project_Final.ipynb
```

## Key Technical Challenges Solved

**Complex PDF parsing** — University module handbooks have highly irregular layouts. Docling was used to handle:
- Tables split across page breaks
- Module headers dropped by the parser (detected via unique course codes)
- Mixed plain-text and table-markdown module titles
- HTML entities in titles (`&amp;` → `&`)

**Multilingual embeddings** — Course titles and descriptions are in German. IBM Granite 107M multilingual embeddings handle cross-lingual semantic similarity correctly.

**Re-ranking** — Initial vector search retrieves candidates by cosine similarity; a cross-encoder re-ranker then re-scores them for higher precision before passing to the LLM auditor.

## Output

- `similarity_report.html` — visual report showing matched module pairs with similarity scores and LLM reasoning
- `similarity_report.csv` — structured data for academic processing

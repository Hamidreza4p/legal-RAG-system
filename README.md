# ⚖️ Agentic RAG-Based Legal Question Answering System

> Ask a legal question in plain language — get a source-grounded answer drawn from civil law documents across Italy, Slovenia, and Estonia. Three retrieval architectures (Single-Agent, Multi-Agent, Hybrid) let you trade off between simplicity, scalability, and precision.

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square&logo=python)
![Streamlit](https://img.shields.io/badge/Streamlit-UI-FF4B4B?style=flat-square&logo=streamlit)
![FAISS](https://img.shields.io/badge/Vector_DB-FAISS-orange?style=flat-square)
![OpenAI](https://img.shields.io/badge/LLM-GPT--4o--mini-412991?style=flat-square&logo=openai)
![Docker](https://img.shields.io/badge/Docker-ready-2496ED?style=flat-square&logo=docker)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

---

## 🧠 What It Does

Legal research is slow, jurisdiction-dependent, and requires navigating dense document corpora. This system addresses that by combining semantic retrieval over a structured legal corpus with LLM reasoning — producing answers that are grounded in actual legal sources, not hallucinated.

A user submits a natural language question. The system decides which retrieval pipeline to activate, embeds the query, searches FAISS indexes of civil code articles and court case decisions, and passes the retrieved context to GPT-4o-mini to generate a final answer with source citations.

Three architectural modes are supported, each suited to different use cases:

| Mode | Best For |
|------|----------|
| Single-Agent RAG | General queries, fast responses |
| Multi-Agent RAG | Cross-jurisdiction comparative questions |
| Hybrid RAG | Structured queries requiring high precision |

---

## 📂 Legal Corpus

The corpus covers **civil law** across three jurisdictions, stored as structured JSON documents:

```
Contest_Data/
├── Italy/
│   ├── Inheritance_italy/       # Civil code articles
│   ├── Divorce_italy/
│   └── Italian_cases_json_processed/   # Court decisions
├── Slovenia/
│   ├── Inheritance_slovenia/
│   ├── Divorce_slovenia/
│   └── Slovenian_cases_json_processed/
└── Estonia/
    ├── Inheritance_estonia/
    ├── Divorce_estonia/
    └── Estonian_cases_json_processed/
```

Document types: **Civil Code Articles** and **Court Case Decisions**

---

## ⚙️ System Architecture

### Pipeline Overview
```
User Question
     ↓
Pipeline Router (single / multi-agent / hybrid)
     ↓
Query Embedding (all-MiniLM-L6-v2, 384-dim)
     ↓
FAISS Vector Search (per country / legal domain)
     ↓
Context Assembly + LLM Reasoning (GPT-4o-mini)
     ↓
Answer + Sources + Metadata
```

### Single-Agent RAG
One LLM instance handles the full pipeline: decides if retrieval is needed, selects relevant jurisdictions, triggers FAISS search, re-ranks documents, and generates the answer.
Key file: `backend/rag_single_agent.py`

### Multi-Agent RAG
A supervisor–worker pattern. A Supervisor Agent interprets the query and coordinates country-specific Sub-Agents, each of which retrieves from its own FAISS index and generates a partial answer. The supervisor synthesizes all partial answers into a final response.
Key file: `backend/rag_multiagent.py`

### Hybrid RAG
A logic-driven pipeline (not fully agentic). Applies metadata-based filtering by country and legal area before vector search, then makes a single LLM call for answer generation. Offers the most predictable and precise behavior.
Key file: `backend/hybrid_rag.py`

---

## 🔬 Embeddings & Vector Storage

| Component | Detail |
|-----------|--------|
| Embedding model | `sentence-transformers/all-MiniLM-L6-v2` |
| Embedding size | 384 dimensions |
| Inference | Local, no external API dependency |
| Vector store | FAISS (separate indexes per country and legal domain) |
| Index files | `index.faiss` (similarity index), `index.pkl` (metadata + mappings) |

---

## 🖥️ Streamlit Interface

The app is split into three pages:

1. **Configuration** — select LLM, embedding model, and add document folders
2. **Vector DB Builder** — scan folders and build FAISS indexes
3. **Chatbot Q&A** — submit legal questions and receive grounded answers

---

## 🚀 Quick Start

### Local (no Docker)

```bash
git clone https://github.com/Hamidreza4p/legal-RAG-system.git
cd legal-RAG-system

python -m venv .venv
source .venv/bin/activate  # Windows: .\.venv\Scripts\Activate.ps1

pip install --upgrade pip
pip install -r requirements.txt

# Create .env with your OpenAI key
echo "OPENAI_API_KEY=sk-your-key-here" > .env

streamlit run app.py
```

### Docker

```bash
docker build -t legal-rag-app .

docker run -p 8501:8501 \
  --env-file .env \
  -v /path/to/your/legal_corpus:/data \
  legal-rag-app
```

Then open `http://localhost:8501`.

---

## 🗂️ Project Structure

```
├── app.py                    # Streamlit entry point
├── backend/
│   ├── rag_pipeline.py       # Routing logic
│   ├── rag_single_agent.py   # Single-agent architecture
│   ├── rag_multiagent.py     # Multi-agent architecture
│   └── hybrid_rag.py         # Hybrid architecture
├── pages/                    # Streamlit multi-page setup
├── Contest_Data/             # Legal corpus (JSON)
├── Dockerfile
├── requirements.txt
└── .env.example
```

---

## 🛠️ Tech Stack

| Layer | Tools |
|-------|-------|
| LLM | GPT-4o-mini (OpenAI API) |
| Embeddings | sentence-transformers/all-MiniLM-L6-v2 (local) |
| Vector DB | FAISS |
| Frontend | Streamlit |
| Containerization | Docker |

---

## 📸 Screenshots

*Coming soon*

---

## 👥 Authors

Developed as part of the **Text Mining** course at Università degli Studi di Napoli Federico II.

# NovaAI 🧠

**An agentic, self-correcting RAG platform with hybrid retrieval and real-time streaming.**

NovaAI routes user queries through an intent-aware pipeline, retrieves context using a hybrid of vector, keyword, and graph search, and validates its own answers with a reflection loop before streaming them back to the client — with citations and confidence scores attached.

---

## ✨ Highlights

- **Intent-aware routing** — an LLM router classifies each query as general chat, research/RAG, or a tool request, and dispatches accordingly instead of forcing every message through the same pipeline.
- **Hybrid retrieval** — combines vector search, BM25, and graph search, merges results, and reranks them for relevance.
- **Context grading + adaptive re-retrieval** — a grader checks whether retrieved context is actually good enough; if not, the query is rewritten and re-run through retrieval instead of generating on weak context.
- **Self-correcting generation** — a reflection/critic node reviews the draft answer, and a hallucination checker sends the pipeline back to retrieval if the answer isn't well-grounded.
- **Trustworthy output** — final answers include auto-generated citations and a confidence score.
- **Real-time delivery** — responses are streamed to the client over WebSockets.

---

## 🏗️ Architecture

```
User Query
   │
   ▼
Intent Router (LLM) ──▶ General Chat
   │              └───▶ Tool Request
   ▼
Research / RAG
   │
   ▼
Query Rewriter ──▶ Planner
                      │
                      ▼
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
  Vector Search   BM25 Search   Graph Search
        └─────────────┼─────────────┘
                       ▼
                Merge Retrieval
                       │
                       ▼
                   Reranker
                       │
                       ▼
                Context Grader ──(rewrite)──▶ back to Query Rewriter
                       │ (good)
                       ▼
               Generation Node
                       │
                       ▼
             Reflection / Critic
                       │
                       ▼
           Hallucination Checker ──(no)──▶ back to Merge Retrieval
                       │ (yes)
                       ▼
               Citation Generator
                       │
                       ▼
             Confidence Estimator
                       │
                       ▼
          ⚡ WebSocket Stream Output
```

The pipeline is graph-based (built with **LangGraph**), so retrieval and generation aren't a single linear pass — the system can loop back and retry when context is weak or an answer looks unsupported, rather than committing to a bad response.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| API / Backend | Python, FastAPI |
| Orchestration | LangGraph |
| LLM Serving | Ollama (local models) |
| Retrieval | Vector Search, BM25, Graph Search |
| Real-time Delivery | WebSockets |

---

## 📂 Project Structure

```
NovaAI/
├── app/
│   ├── main.py                # FastAPI entrypoint + WebSocket routes
│   ├── graph/                 # LangGraph pipeline definition
│   │   ├── router.py          # Intent routing node
│   │   ├── retrieval.py       # Vector / BM25 / graph search + merge + rerank
│   │   ├── grading.py         # Context grader
│   │   ├── generation.py      # Answer generation node
│   │   ├── reflection.py      # Critic + hallucination checker
│   │   └── output.py          # Citation generator + confidence estimator
│   ├── models/                # Ollama client / model config
│   └── utils/
├── requirements.txt
└── README.md
```

*(Adjust this section to match your actual repo layout.)*

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- [Ollama](https://ollama.com) installed and running locally
- A pulled model (e.g. `ollama pull llama3`)

### Installation

```bash
git clone https://github.com/segfault-00/NovaAI.git
cd NovaAI
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
```

### Running

```bash
uvicorn app.main:app --reload
```

The API will be available at `http://localhost:8000`, with a WebSocket endpoint for streaming responses at `ws://localhost:8000/ws`.

*(Update commands/ports to match your actual setup.)*

---

## 🧭 Roadmap

- [ ] Add evaluation harness for retrieval relevance and hallucination rate
- [ ] Support additional vector store backends
- [ ] Add authentication for WebSocket connections
- [ ] Dockerize the full stack

---

## 📄 License

[MIT](LICENSE) — or update to whichever license you're using.

---

## 🙋 About

Built by [segfault-00](https://github.com/segfault-00) as a hands-on exploration of agentic RAG systems — combining hybrid retrieval, self-correction loops, and real-time delivery in a single production-style pipeline.

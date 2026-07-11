# 🧠 Agent-Based Intelligent Tutor for Network Security

A fully local, privacy-preserving AI tutoring system built on a Retrieval-Augmented Generation (RAG) architecture. Two intelligent agents help students master Network Security concepts through interactive Q&A and quiz sessions — with zero external data transmission.

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square&logo=python)
![LangChain](https://img.shields.io/badge/LangChain-0.1+-green?style=flat-square)
![Llama3](https://img.shields.io/badge/Llama_3-Ollama-orange?style=flat-square)
![ChromaDB](https://img.shields.io/badge/ChromaDB-local-purple?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

---

## 🔒 Privacy First

All computation runs **entirely on your local machine**. No data is sent to any external server.

> ✅ Privacy verified via **Wireshark** — 100% of traffic confirmed on `127.0.0.1` loopback. Zero external API calls during operation.

---

## 🚀 Features

### 🤖 Agent 1 — Q&A Tutor Agent
- Answers course-related questions with accurate responses grounded in your study materials
- Retrieves the most relevant document chunks using **top-k FAISS / ChromaDB vector search**
- Provides **exact source citations** (e.g., `Lecture_1_slides.pdf:4–6`)
- Prevents hallucination by anchoring every response to retrieved context

### 📝 Agent 2 — Quiz Agent
- Generates quiz questions directly from your study material
- Supports **Multiple Choice (MCQ)**, **True/False**, and **Open-Ended** formats
- Uses **LLM-based grading** to evaluate open-ended answers with cited feedback
- Provides instant explanations and rationales for every question

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────┐
│              Chainlit UI (Browser)           │
│           http://localhost:8000              │
└──────────────────┬──────────────────────────┘
                   │
       ┌───────────┴───────────┐
       │                       │
  ┌────▼────┐             ┌────▼────┐
  │  Q&A    │             │  Quiz   │
  │  Agent  │             │  Agent  │
  └────┬────┘             └────┬────┘
       │                       │
  ┌────▼───────────────────────▼────┐
  │           RAG Pipeline           │
  │                                  │
  │  ┌─────────┐   ┌─────────────┐  │
  │  │  FAISS  │   │  ChromaDB   │  │
  │  │ (dense) │   │  (persist)  │  │
  │  └─────────┘   └─────────────┘  │
  │                                  │
  │  ┌──────────────────────────┐   │
  │  │  HuggingFace MiniLM      │   │
  │  │  Sentence Transformers   │   │
  │  └──────────────────────────┘   │
  └─────────────────┬────────────────┘
                    │
            ┌───────▼───────┐
            │   Llama 3 via  │
            │    Ollama      │
            │  (127.0.0.1)  │
            └───────────────┘
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.10+ |
| LLM | Llama 3 via Ollama (fully local) |
| RAG Framework | LangChain |
| Vector Store | ChromaDB (persistent) + FAISS |
| Embeddings | HuggingFace MiniLM Sentence Transformers |
| Document Parsing | pypdf, Unstructured loaders |
| UI | Chainlit |
| Privacy Verification | Wireshark |

---

## 📋 Requirements

- Python 3.10+
- 8 GB RAM (recommended)
- Local storage ≥ 1 GB
- [Ollama](https://ollama.ai) installed

---

## ⚙️ Installation

```bash
# 1. Clone the repository
git clone https://github.com/yogendravarmaa7/ai-tutor-network-security.git
cd ai-tutor-network-security

# 2. Create virtual environment
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Pull Llama 3 via Ollama
ollama pull llama3
```

---

## 🚦 Usage

```bash
# Step 1 - Build local vector database from your documents
python src/ingest.py

# Step 2 - Launch the tutor
chainlit run src/app.py -w
```

Open your browser at `http://localhost:8000`

---

## ⚙️ Flow of Execution

### Message Handling
The `@cl.on_message` decorator triggers whenever a new message is received in the chat UI.

### Main Function
```python
@cl.on_message
async def main(message: cl.Message):
    response = llm(message.content)
    await cl.Message(content=response).send()
```

### LLM Function
- Configures local embeddings
- Connects to local vector store (`db/`)
- Retrieves the most relevant context chunks
- Generates and returns answers with citations

### loadResourceDocuments()
- Loads and vectorizes lecture slides, textbooks, and quizzes
- Uses Chroma for persistent storage and future reuse

---

## 💬 Example Interactions

**Q&A Tutor Agent:**
```
Prompt:  "Explain the CIA triad and give one example for each component."

Response: "Confidentiality protects sensitive data from unauthorized access.
           Integrity ensures data accuracy and prevents tampering.
           Availability maintains system accessibility for authorized users.
           (Source: Lecture_1_slides.pdf:4–6)"
```

**Quiz Agent:**
```
Prompt:  "Generate 3 MCQ and 2 open-ended questions about symmetric encryption."

Response: Generates questions with correct answers, rationales, and citations.
```

---

## 📁 Project Structure

```
ai-tutor-network-security/
│
├── src/
│   ├── app.py           # Main Chainlit application
│   ├── ingest.py        # Document loader and vector DB builder
│   ├── quiz_agent.py    # Quiz Agent - generates and grades questions
│   └── response.py      # LLM handler with citation support
│
├── db/                  # Local ChromaDB vector database
├── captures/            # Wireshark captures confirming local traffic
├── requirements.txt
└── README.md
```

---

## 🔬 Privacy Verification

Network traffic was captured using **Wireshark** during a full tutoring session.

**Finding:** 100% of all traffic was bound to `127.0.0.1` loopback interface. No packets were sent to any external IP address. The system operates completely offline.

---

## 🐛 Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| No responses in chat | Vector DB not built | Run `python src/ingest.py` first |
| API Key error | OpenAI key not set | Use `.env` file or switch to Llama via Ollama |
| Slow inference | Large document chunks | Reduce chunk size in `ingest.py` |
| External traffic warning | Optional tool misconfigured | Disable Google Search API |

---

## ⚡ Command Summary

```bash
# Environment setup
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Build local vector database
python src/ingest.py

# Launch the tutor
chainlit run src/app.py -w
```

---

## 🔮 Future Enhancements

- Adaptive question difficulty based on student performance
- Graph-based topic visualization
- Multi-user session support with session logs
- Live Wireshark dashboard integration in Chainlit UI

---

## 👤 Author

**Yogendra Varma Addepalli**
- 🌐 Portfolio: [yogendravarmaa7.github.io](https://yogendravarmaa7.github.io)
- 💼 LinkedIn: [linkedin.com/in/yaddepalli](https://linkedin.com/in/yaddepalli)
- 📧 Email: addepalliyogendravarma@gmail.com

---

## 📄 License

This project is licensed under the MIT License.

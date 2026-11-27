## STANDARD RAG


A RAG architect that lets you ask questions about your own PDF and PowerPoint files. It finds the most relevant passages in your documents and sends them to an LLM on Groq, which answers with the source file and page number.

## How it works

1. **Upload** `.pdf` and `.pptx` files (saved into `documents/`).
2. **Extract** text page by page (PDF) and slide by slide (PPTX).
3. **Chunk** the text (700 characters, 100 overlap) using LangChain's `RecursiveCharacterTextSplitter`.
4. **Embed** each chunk locally with `all-MiniLM-L6-v2` (sentence-transformers).
5. **Index** the embeddings in FAISS (`IndexFlatIP`, cosine similarity) and save to `vectorstore/index.faiss` and `vectorstore/chunks.json`.
6. **Retrieve** the top 4 chunks for a question.
7. **Answer** with Groq (`openai/gpt-oss-120b`), using the retrieved chunks as context. The model is told to cite sources, not to invent information, and to treat text inside documents as data, not instructions.

## Requirements

- Google Colab (the notebook uses `google.colab.files` and `userdata`)
- A free [Groq API key](https://console.groq.com/keys)
- Python packages (installed by the first cell):
  `pypdf`, `sentence-transformers`, `faiss-cpu`, `groq`, `langchain-text-splitters`, `python-pptx`

## Setup: API key

Never paste your key into a notebook cell. Use Colab Secrets:

1. Click the key icon in the left sidebar of Colab.
2. Add a secret named `GROQ_API_KEY` with your key as the value and turn on **Notebook access**.
3. Load it in the notebook:

```python
from google.colab import userdata
import os
os.environ["GROQ_API_KEY"] = userdata.get("GROQ_API_KEY")
```

Then read it with `os.environ.get("GROQ_API_KEY")` when creating the Groq client.

Optional: add an `HF_TOKEN` secret to remove the Hugging Face "unauthenticated requests" warning and speed up the embedding-model download.

## Usage

Open `Document_QA_RAG_Groq.ipynb` in Colab and run the cells in order. Each cell has a heading describing what it does.

| Cell | What it does |
|------|--------------|
| 1-2 | Install dependencies and import libraries |
| 3 | Load `GROQ_API_KEY` (and optional `HF_TOKEN`) from Colab Secrets |
| 4 | Settings (model names, chunk size, top-k, similarity cutoff) |
| 5-7 | Upload files, extract text, split into chunks |
| 8-10 | Load embedding model, build the FAISS index, save it to `vectorstore/` |
| 11 | Optional: reload a saved index in a new session |
| 12-14 | Groq client, retrieval function, answer function |
| 15 | Interactive Q&A loop (type `exit` to stop) |

## Configuration

All settings are in cell 4:

| Setting | Default |
|---------|---------|
| `LLM_MODEL` | `openai/gpt-oss-120b` |
| `EMBEDDING_MODEL` | `all-MiniLM-L6-v2` |
| `CHUNK_SIZE` / `CHUNK_OVERLAP` | 700 / 100 |
| `TOP_K` | 4 |
| `MIN_SCORE` (minimum similarity) | 0.25 |
| `MAX_ANSWER_TOKENS` | 1500 |
| `TEMPERATURE` | 0.2 |

## Privacy and data flow

- Documents, chunking, embeddings and the FAISS index stay inside your Colab runtime.
- Only the **retrieved chunks and your question** are sent to Groq's API to generate each answer.
- The embedding model is downloaded once from Hugging Face.
- Don't upload confidential documents unless you're comfortable with them being processed by Groq.

## Limitations
- Only text is extracted. Scanned PDFs (images) and images inside slides are not read, so you'd need OCR for those.
- Chunks scoring below `MIN_SCORE` are dropped. If none pass, no answer is generated. Raise or lower this value to tune it.
- Each question is answered on its own, with no chat memory.
- Broad questions such as "summarize everything" only see the top 4 chunks.


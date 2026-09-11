# Local Chatbot with RAG

Fully local console chatbot. Runs LLM via **Llamafile**, optional **RAG** (Retrieval-Augmented Generation) over your own documents using FAISS + local embeddings. No API keys, no internet required at inference time.

## How it works

- `indexer.py` — builds a searchable index from documents in `Files/`. Extracts text from `.pdf`, `.md`, `.xml`, splits into overlapping chunks, embeds each chunk with a local Llamafile embedding model, and saves vectors + chunks + FAISS index to `Index/faiss_index.pkl`.
- `main.py` — starts a local Llamafile LLM server, loads the FAISS index (if RAG enabled), and runs an interactive chat loop in the console. Each query is optionally rewritten using chat history, matched against the index for relevant context, injected into the prompt, then streamed back as a response.

Console output is color-coded: green = LLM answer, red/yellow = debug/prompt info, blue = RAG sources.

## Setup

1. Download [Llamafile](https://github.com/Mozilla-Ocho/llamafile/releases) (matching the version referenced in `main.py`/`indexer.py`, e.g. `llamafile-0.8.13.exe`) and place the executable in the project root.
2. Download a model in `.gguf` format from Hugging Face (an instruction-tuned model for chat, an embedding model like `gte-large` for RAG).
3. Install Python dependencies:
   ```
   pip install -r requirements.txt
   ```
4. Update model paths in `main.py` (`LLM_NAME`, `EMBEDDING_NAME`) and `indexer.py` (`EMBEDDING_NAME`) to point to your downloaded `.gguf` files.

## Usage

### Chat without RAG

```
python main.py
```

`ENABLE_RAG = False` by default in `main.py`. Just starts the LLM server and chats.

### Chat with RAG (your own documents)

1. Put source documents (`.pdf`, `.md`, `.xml`) into a `Files/` folder.
2. Build the index:
   ```
   python indexer.py
   ```
   This starts the embedding server, extracts and chunks text, embeds it, and writes `Index/faiss_index.pkl`.
3. In `main.py`, set `ENABLE_RAG = True`.
4. Run:
   ```
   python main.py
   ```
   Each query is embedded and matched against the index; chunks scoring under `RAG_THRESH` are injected as context, and source file paths are printed with the answer.

## Key configuration (`main.py`)

| Setting | Purpose |
|---|---|
| `LLM_NAME` / `LLM_PORT` | Chat model path and server port |
| `EMBEDDING_NAME` / `EMBEDDING_PORT` | Embedding model path and server port |
| `ENABLE_RAG` | Toggle retrieval on/off |
| `RAG_THRESH` | Max cosine distance for a chunk to count as relevant |
| `RAG_N_RETURN` | Number of chunks to inject as context |
| `REWRITE_RAG_QUERIES` | Rephrase follow-up questions using prior turn as context before retrieval |
| `SYSTEM_PROMPT` | Persona/instructions prepended to every prompt (German by default) |
| `HISTORY_MSG_N` | How many past turns to keep in the conversation window |

## Requirements

- Windows (scripts call `llamafile-*.exe` directly; no `./` prefix or `.exe` handling for other OSes)
- Python packages: `scikit-learn`, `langchain_community`, `numpy`, `faiss-cpu`, `PyPDF2`, `PyCryptodome`, plus `tqdm` and `faiss` (used by `indexer.py` but missing from `requirements.txt`)

## Notes / known limitations

- Windows-only as written (hardcoded `.exe` calls, backslash model paths).
- `indexer.py` and `main.py` currently reference different embedding models (`gte-large` vs `qwen2-1_5b-instruct`) and Llamafile binary versions (`0.8.12` vs `0.8.13`) — keep these in sync for RAG to work correctly.
- No automatic cleanup of spawned Llamafile server processes; stop them manually after use.

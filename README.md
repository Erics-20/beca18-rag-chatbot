# Beca 18 RAG Chatbot

> HW03 · Applied Data Science

A retrieval-augmented generation (RAG) chatbot that answers questions about the **Beca 18** scholarship program using the official PRONABEC regulation as its sole source of truth.

---

## Purpose and source document

**Beca 18** is Peru's flagship scholarship program managed by PRONABEC (Programa Nacional de Becas y Crédito Educativo). It funds higher education for students from low-income households at accredited universities and technical institutes nationwide.

This chatbot is grounded exclusively on **RDE N° 033-2026** — the official regulation that governs eligibility requirements, scholarship modalities, economic benefits, student obligations, and conditions for cancellation. The document is a 40+ page legal text written in Spanish and is stored in `data/beca18_reglamento.pdf`.

---

## Pipeline summary

The notebook implements a full RAG pipeline in ten sequential sections. The source PDF is extracted page by page with PyPDF, cleaned (headers/footers removed, whitespace normalised), and tagged with `[PAGE N]` markers. The cleaned text is tokenised with tiktoken and split into 400-token chunks with 60-token overlap using LangChain's `RecursiveCharacterTextSplitter`. Each chunk is embedded with `gemini-embedding-001` (768 dimensions, `RETRIEVAL_DOCUMENT` task type) and stored in a persistent ChromaDB collection using cosine distance. At query time, the user's question is embedded with the `RETRIEVAL_QUERY` task type, the top-k most similar chunks are retrieved, and `gemini-2.5-flash` generates a grounded answer citing specific page numbers — refusing to answer if the context is insufficient.

---

## Installation and setup

### 1. Clone the repository

```bash
git clone https://github.com/Erics-20/beca18-rag-chatbot.git
cd beca18-rag-chatbot
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure your Gemini API key

Create a `.env` file in the project root (it is already listed in `.gitignore` and will never be committed):

```bash
cp .env.example .env
```

Then open `.env` and replace the placeholder with your actual key:

```
GEMINI_API_KEY=your_actual_key_here
```

You can obtain a free API key at [aistudio.google.com](https://aistudio.google.com).

> **Free-tier limits:** `gemini-embedding-001` allows 1,000 embedding requests per day and ~100 per minute. The notebook handles rate limits automatically with exponential backoff and detects daily quota exhaustion immediately so it fails fast with a clear message.

### 4. Source PDF

`data/beca18_reglamento.pdf` is already committed to the repository and is included automatically when you clone. No manual download or upload is required.

---

## How to run the notebook

### Google Colab (recommended)

1. Open `notebooks/beca18_rag_chatbot.ipynb` in [Google Colab](https://colab.research.google.com).
2. Run **Section 1** — it clones the repository (including the PDF from `data/`) and installs all dependencies. When prompted, **restart the session** (`Runtime > Restart session`), then continue from Section 2.
3. Run **Section 2** — upload your `.env` file when the file picker appears.
4. Run **Sections 3–9** in order to verify the environment, extract and chunk the PDF, build the ChromaDB index, and test the search and generation functions.
5. Run **Section 10** to launch the chat interface.

> **Re-running:** ChromaDB indexing (Section 7) is idempotent — if the collection is already populated it skips embedding entirely, so re-running the notebook after a kernel restart costs no API calls for indexing.

### Local Jupyter

```bash
jupyter notebook notebooks/beca18_rag_chatbot.ipynb
```

Run all cells in order. Section 1 detects that it is not running in Colab and skips the `git clone`; it reads `requirements.txt` and `.env` from the project root automatically.

---

## How to use the chat interface

After running Section 10 an interactive widget panel appears:

| Element | Description |
|---|---|
| **Text box** | Type your question about Beca 18 in Spanish or English |
| **Ask button** | Embeds the question, retrieves context, and generates a grounded answer |
| **Clear button** | Resets the input and output areas |
| **k slider** (1–10) | Controls how many chunks are retrieved from ChromaDB; higher k gives broader context, lower k gives more focused answers |
| **Answer panel** | Displays the model's response with page citations |
| **Source fragments** | Expandable accordion listing each retrieved chunk with its page number and cosine distance score |

The model is instructed to answer **only** from the retrieved passages and will respond with *"The document does not contain information about this topic."* for questions outside the scope of the regulation.

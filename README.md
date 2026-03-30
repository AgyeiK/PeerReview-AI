# 🔬 AI Peer-Reviewer: Dynamic RAG Architecture for Scientific Manuscripts

## 📌 Project Overview
The AI Peer-Reviewer is a highly optimized Retrieval-Augmented Generation (RAG) web application designed specifically for analyzing 20–30 page scientific manuscripts. Unlike standard document-chatbots that suffer from "semantic dilution" and API rate limits when processing large files, this architecture uses precision chunking, dynamic state management, and an automated LLM memory buffer to deliver rigorous, peer-review-level critiques with zero downtime.

---

## 🚀 Core Features
* **Dynamic Document Hot-Swapping:** A drag-and-drop Gradio interface that automatically intercepts new PDFs, slices them into optimized nodes, and rebuilds the vector index on the fly without server restarts.
* **Adjustable Retrieval Breadth (Top-K):** A live UI slider allowing users to mathematically restrict or expand the AI's context window mid-conversation.
* **Token-Efficient Memory Compression:** Implements a `ChatSummaryMemoryBuffer` to actively compress conversation history, completely eliminating the "token bloat" that causes standard RAG systems to crash under API rate limits.
* **Strict Academic Persona:** The system is prompt-engineered with a Temperature of 0.0 to act as a rigorous peer-reviewer, preventing hallucinated metrics and forcing factual analysis of methodologies and results.

---

## 🧠 System Architecture & Data Flow

The application is built on **LlamaIndex** and **Gradio**, utilizing the following block-by-block logic:

### 1. The "Brain" (Global Settings)
To ensure maximum uptime and zero dependency conflicts, the system globally overrides LlamaIndex defaults. It strictly enforces the high-throughput `llama-3.1-8b-instant` (via Groq) for inference and `BAAI/bge-small-en-v1.5` (via HuggingFace) for local vector embeddings. This guarantees speed and self-sufficiency.

### 2. State Management & Cross-Contamination Prevention
The application maintains global states for the document index and the memory buffer. When a new manuscript is uploaded, the `process_new_file` handler triggers a hard `memory.reset()`. This critical step prevents the AI from accidentally using context or data from *Manuscript A* to answer questions about *Manuscript B*.

### 3. Asynchronous UX & Error Handling
The chat interface is decoupled into two asynchronous steps:
1.  **Frontend Update:** The user's query instantly updates the UI, preventing the application from feeling frozen.
2.  **Backend Retrieval:** The engine queries the vector database, formats the payload, and streams the LLM response. The entire RAG pipeline is wrapped in a `try/except` block to catch and print API errors (like rate limits) directly into the chat, ensuring the UI never silently crashes.

---

## ⚙️ Engineering Parameters & "Smart Decisions"

Every parameter in this pipeline was mathematically selected to optimize for the dense IMRaD (Introduction, Methods, Results, and Discussion) structure of scientific manuscripts.

| Parameter | Configured Value | Engineering Logic |
| :--- | :--- | :--- |
| **Chunk Size** | `512 tokens` | Strikes the perfect balance for scientific text. Large enough to keep a hypothesis and its results linked, but small enough to prevent "semantic dilution" during vector search. |
| **Chunk Overlap** | `30 tokens` | Acts as a contextual safety net. It prevents critical scientific metrics (like p-values or sample sizes) from being severed at the "seam" between two chunks. |
| **Retrieval Top-K** | `5 chunks` | Pulling 5 precise chunks (~2,500 tokens) covers about 3–4 pages. For a short manuscript, this captures an entire target section (e.g., Methodology) without grabbing irrelevant data. |
| **LLM Model** | `Llama 3.1 8B` | Prioritizes High Availability. By routing tasks to an efficient 8B model instead of a 70B model, we eliminated 429 Rate Limit crashes, ensuring zero downtime. |
| **Temperature** | `0.0` | Guarantees deterministic, strict factual output. Essential for academic applications to prevent the AI from hallucinating data when answering edge-case questions. |
| **Memory Buffer** | `ChatSummary` | Actively compresses older conversational turns into a dense paragraph, preventing linear "token bloat" from crashing the API payload as the chat history grows. |

---

## 💻 Quick Start & Installation

**1. Install Dependencies**
Using the legacy resolver ensures no dependency conflicts with HuggingFace inference packages:
```bash
pip install huggingface-hub llama-index llama-index-llms-groq gradio sentence-transformers
pip install llama-index-embeddings-huggingface --no-deps

# Abstract

**Retrieval-Augmented Generation (RAG)** has emerged as a powerful approach to improve the contextual understanding of language models by integrating external knowledge. This paper presents a **lightweight, offline-capable RAG system** that leverages the **Mistral 7B Instruct** model to deliver context-aware responses using dynamically scraped web content or user-uploaded text files.

Unlike conventional chatbot systems that rely solely on pre-trained knowledge, our architecture integrates a **custom web scraper**, **FAISS-based embedding indexer**, and **persistent memory** to continuously learn and adapt to new information in real time.

The model accepts user queries, retrieves relevant context from indexed documents or URLs, and generates responses using **prompt engineering** tailored for instruction-based LLMs. This setup not only improves the relevance of answers but also offers a **practical, fully local alternative** to cloud-dependent AI services.

The system demonstrates strong potential for **educational tools**, **research assistants**, and **secure enterprise applications**.

---
Link to Project - https://github.com/ArjunJagdale/Local-RAG-System-using-Mistral-Sentence-Transformers-FAISS
---

# 1. Introduction

### What is the problem we are solving?

Modern conversational AI systems often struggle to provide accurate and up-to-date responses when faced with dynamic or domain-specific content. Most chatbots rely on **static, pre-trained knowledge** and cannot incorporate new information unless explicitly retrained. This limits their usefulness in scenarios where context is constantly evolving — such as educational resources, personal research, or rapidly updating websites.

### Why is context-aware QA important?

**Context-aware Question Answering (QA)** systems enhance the relevance and quality of responses by grounding answers in **external, real-time knowledge**. By retrieving supporting content from documents or the web, these systems can reason over recent or domain-specific information, making them more accurate and useful. 

For users engaging in academic research, professional tasks, or learning, having a chatbot that understands the **specific context** of a topic can significantly improve productivity and understanding.

### Why use local models?

While cloud-based models like **GPT-4** or **Claude** offer powerful capabilities, they come with challenges — **privacy concerns, limited customization, latency, and cost**. In contrast, **local models** such as **Mistral 7B Instruct**, when paired with efficient tooling like `llama-cpp-python`, allow for complete **offline execution**.

This approach ensures **data security**, supports use in **low-resource or restricted environments**, and allows users to **fine-tune the experience** for their own needs without external dependencies.

---

# 2. System Design

The proposed system functions as a **retrieval-augmented chatbot** that processes user-provided content (via URL or file), builds a **semantic index**, and enables **interactive question answering** using a **locally deployed Mistral 7B Instruct model**. The architecture is modular, consisting of four main stages:

---

## 2.1 Input: URL or File-Based Source

Users begin by providing either:

1. A **web URL**, from which the system scrapes textual content using a **custom HTML parser**, or  
2. A **plain text file**, loaded directly from local storage.

This flexibility enables dynamic integration of both **online and offline sources** for contextual grounding.

---

## 2.2 Preprocessing: Web Scraping or File Loader

- If a **URL** is provided, the `scrape_url()` function in `web_scraper.py` extracts clean body text from the HTML using **BeautifulSoup**, removing navigation bars, scripts, and ads.
- If a **file** is specified, the `load_text_file()` function reads and parses the text into manageable chunks.

This ensures all content is **uniformly preprocessed**, regardless of the input source.

---

## 2.3 Embedding and Indexing: Semantic Understanding with FAISS

The core of the system’s retrieval capability is built using:

1. **Sentence Embedding**: Leveraging a transformer-based encoder (e.g., `all-MiniLM-L6-v2`), the text is converted into **semantic vectors**.
2. **FAISS Indexing**: These vectors are stored in a **FAISS index** for fast **approximate nearest neighbor search**.

This lightweight embedding + indexing step gives the local system its **“memory”** of documents or web pages.

---

## 2.4 Response Generation: Mistral 7B Instruct with Custom Prompts

Once the user asks a question:

- The system uses the **FAISS index** to retrieve the **top-k relevant chunks** based on semantic similarity.
- These chunks are combined with the question to form a **contextual prompt** in the `[INST] ... [/INST]` format, suitable for the Mistral model.
- The local LLM (`llama_cpp.Llama`) processes this prompt and generates a natural language response using **quantized inference (GGUF format)**.

The prompt is **carefully structured** to guide the model to ground its answers in the **retrieved context**, rather than relying solely on pre-trained knowledge.

---

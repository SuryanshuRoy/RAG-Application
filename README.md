# 📚 Local RAG Document Question Answering System

A **local Retrieval-Augmented Generation (RAG)** application that allows users to ask questions about the contents of their documents.

The system processes a PDF or text document, splits it into smaller chunks, converts those chunks into vector embeddings, stores them in **ChromaDB**, and retrieves the most relevant information when a user asks a question. An **Ollama-hosted Llama 3.1 model** then generates an answer using only the retrieved document context.

The application is designed to provide **grounded document-based answers** while keeping the entire workflow local.

---

## 🚀 Features

* 📄 Supports **PDF and text documents**
* ✂️ Automatically splits documents into manageable chunks
* 🧠 Generates embeddings using `nomic-embed-text`
* 🗄️ Stores embeddings in **ChromaDB**
* 🔎 Performs similarity-based document retrieval
* 🤖 Uses **Llama 3.1 through Ollama** for answer generation
* 📌 Retrieves the **top 3 relevant chunks** for each question
* 🛡️ Uses a grounded prompt that prevents

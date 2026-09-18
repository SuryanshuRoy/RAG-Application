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
* 🛡️ Uses a grounded prompt that prevents the LLM from intentionally speculating beyond the document
* 💻 Runs locally without requiring a cloud-based LLM API
* 📊 Displays retrieved chunks, distance scores, and page information

---

# 🏗️ Architecture

The application follows a standard **Retrieval-Augmented Generation architecture**.

```text
                 ┌─────────────────────┐
                 │    User Document    │
                 │    PDF / TXT File   │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │   Document Loader   │
                 │ PyPDFLoader /       │
                 │ TextLoader          │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │   Text Chunking     │
                 │ Chunk Size: 1000    │
                 │ Overlap: 200        │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │    Embeddings       │
                 │ nomic-embed-text    │
                 │      Ollama         │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │     ChromaDB        │
                 │   Vector Database   │
                 └──────────┬──────────┘
                            │
                   User asks a question
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Similarity Search    │
                 │      Top 3 Chunks    │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │   Prompt + Context  │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │      Llama 3.1      │
                 │       Ollama        │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │    Final Answer     │
                 └─────────────────────┘
```

---

# 🔄 How the System Works

The application consists of two main stages:

## 1. Document Ingestion and Indexing

When the application starts, the user provides a document path.

The application first checks whether the specified file exists. It then selects the appropriate loader:

* `PyPDFLoader` for PDF files
* `TextLoader` for other supported text files

The loaded document is then divided into smaller chunks.

### Chunking

The application uses `RecursiveCharacterTextSplitter` with:

```text
Chunk Size   = 1000 characters
Overlap      = 200 characters
```

The overlap helps preserve context between neighboring chunks.

### Embeddings

Each chunk is converted into a numerical vector using:

```text
nomic-embed-text
```

through Ollama.

These vectors represent the semantic meaning of the document chunks.

### Vector Database

The generated embeddings and corresponding document chunks are stored in:

```text
./chroma_db
```

using ChromaDB.

---

# 🔎 2. Question Answering

After indexing, the application starts an interactive question-answering loop.

When the user enters a question, the system performs a similarity search against the stored document vectors.

It retrieves the **3 most relevant chunks**:

```python
retriever = vector_db.as_retriever(
    search_kwargs={"k": 3}
)
```

## The application also displays the retrieved chunks along with their distance scores and page information.

# 🤖 Response Generation

The retrieved document chunks are passed to a prompt along with the user's question.

The system prompt instructs the model to:

1. Answer using only the provided context.
2. Avoid speculation.
3. State when the requested information is not available in the document.

The application uses:

```text
Model: llama3.1
Temperature: 0
```

through Ollama.

The final RAG pipeline is built using LangChain's LCEL architecture:

```text
Question
   ↓
Retriever
   ↓
Relevant Documents
   ↓
Context Formatting
   ↓
Prompt
   ↓
Llama 3.1
   ↓
String Output
```

This chain is implemented using `RunnablePassthrough`, the retriever, prompt, LLM, and `StrOutputParser`.

---

# 🛠️ Technology Stack

| Technology                         | Purpose                              |
| ---------------------------------- | ------------------------------------ |
| **Python**                         | Application programming language     |
| **LangChain**                      | RAG pipeline and document processing |
| **PyPDFLoader**                    | PDF document loading                 |
| **TextLoader**                     | Text document loading                |
| **RecursiveCharacterTextSplitter** | Document chunking                    |
| **Ollama**                         | Local model and embedding runtime    |
| **nomic-embed-text**               | Text embedding model                 |
| **Llama 3.1**                      | Large language model                 |
| **ChromaDB**                       | Vector database                      |
| **LCEL**                           | LangChain pipeline composition       |

---

# 📋 Requirements

Before running the project, make sure you have:

* Python 3.9+
* Ollama
* `llama3.1` model installed in Ollama
* `nomic-embed-text` model installed in Ollama
* Required Python packages

---

# ⚙️ Installation

## 1. Clone the Repository

```bash
git clone https://github.com/your-username/your-repository.git
cd your-repository
```

## 2. Create a Virtual Environment

### Windows

```bash
python -m venv venv
venv\Scripts\activate
```

### Linux / macOS

```bash
python3 -m venv venv
source venv/bin/activate
```

## 3. Install Dependencies

Install the required Python packages:

```bash
pip install langchain-community
pip install langchain-text-splitters
pip install langchain-ollama
pip install langchain-chroma
pip install langchain-core
```

---

# 🦙 Ollama Setup

Install Ollama on your system and download the models required by the application.

Pull the embedding model:

```bash
ollama pull nomic-embed-text
```

Pull the language model:

```bash
ollama pull llama3.1
```

Make sure Ollama is running before starting the Python application.

---

# ▶️ How to Run

Run the Python file:

```bash
python "RAG application.py"
```

The application will ask you for the path to your document:

```text
Enter path to your document (e.g., sample.pdf):
```

Enter the path to your PDF or text file.

For example:

```text
sample.pdf
```

or:

```text
documents/my_notes.pdf
```

The application will then:

```text
1. Load the document
2. Split it into chunks
3. Generate embeddings
4. Store the vectors in ChromaDB
5. Start the question-answering system
```

---

# 💬 Asking Questions

Once the RAG system is ready, you will see:

```text
==================================================
 RAG SYSTEM READY: Type your question or 'exit' to quit.
==================================================
```

You can then ask questions about the document.

Example:

```text
Ask a question: What is the main objective of the project?
```

The system first displays the relevant chunks:

```text
--- Performing Similarity Search ---

[Chunk 1] (Distance Score: 0.XXXX | Page: 2)
...
```

It then generates the final response:

```text
--- Generating Grounded Response ---

Final Answer:
...
```

To close the application:

```text
exit
```

You can also use:

```text
quit
```

or:

```text
q
```

The application explicitly handles these commands in its interactive loop.

---

# 📁 Project Structure

A typical project structure can look like this:

```text
RAG-Document-QA/
│
├── RAG application.py
├── README.md
├── requirements.txt
│
├── documents/
│   └── sample.pdf
│
└── chroma_db/
    └── Vector database files
```

The `chroma_db` directory is created/used as the persistent ChromaDB location by the application.

---

# 🧩 Core Components

## `ingest_and_index()`

Responsible for the document ingestion pipeline:

```text
Document
   ↓
Loader
   ↓
Chunks
   ↓
Embeddings
   ↓
ChromaDB
```

It returns the vector database and embeddings used by the application.

---

## `format_docs()`

Combines retrieved document chunks into a single context string that can be passed to the LLM.

---

## `run_qa_loop()`

Responsible for:

* Initializing Llama 3.1
* Creating the system prompt
* Configuring the retriever
* Building the RAG chain
* Accepting user questions
* Performing similarity searches
* Displaying retrieved chunks
* Generating final answers

---

# 🔐 Grounded Answering

One of the important aspects of this project is that the LLM is instructed to answer using the retrieved document context.

The prompt specifically tells the model not to extrapolate or speculate and provides a fallback response when the requested information cannot be found.

This makes the application useful for situations where answers should be based on a specific document rather than general model knowledge.

---

# 🎯 Use Cases

This RAG application can be adapted for:

* 📚 Study material Q&A
* 📄 Research paper analysis
* 🏫 College notes and textbooks
* 📑 Company documentation
* 📖 Books and manuals
* 🧾 Technical documentation
* 🗂️ Internal knowledge bases
* 🔍 Document search and information retrieval

---

# 🧠 Why RAG?

Traditional LLM applications rely primarily on the information encoded within the model.

RAG adds an external knowledge-retrieval step:

```text
User Question
      ↓
Retrieve Relevant Information
      ↓
Provide Information to LLM
      ↓
Generate Answer
```

This allows an LLM to answer questions using a specific knowledge source without requiring the model itself to be retrained for every new document.

---

# 📌 Conclusion

This project demonstrates the fundamental architecture of a **Retrieval-Augmented Generation system** using entirely local components.

It combines document loading, intelligent text chunking, semantic embeddings, vector search, and local LLM generation into a single pipeline. The use of **ChromaDB** enables efficient similarity-based retrieval, while **Ollama's ****`nomic-embed-text`**** and Llama 3.1 models** provide the embedding and generation capabilities.

The project provides a practical foundation for building more advanced document-question-answering applications. It can be expanded with a user interface, multi-document support, better retrieval strategies, source citations, conversational memory, and production deployment.

**In short, this project turns static documents into an interactive knowledge base that users can query using natural language.**

##

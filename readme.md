# Multimodal RAG with Groq, LangChain & ChromaDB

## Setup Instructions

### 1. Add Your API Key

Create a `.env` file in the project root directory.

Example:

```env
GROQ_API_KEY=your_groq_api_key
```

Replace `your_groq_api_key` with your own Groq API key.

---

### 2. Add Your PDF

Create a folder named `Content` in the project root.

```text
project/
│
├── Content/
│   └── your_document.pdf
│
├── .env
├── README.md
└── ...
```

Place the PDF you want to chat with inside the `Content` folder.

---

### 3. Update the File Path

In the notebook or script, update the PDF filename if necessary:

```python
file_path = "./Content/your_document.pdf"
```

For example:

```python
file_path = "./Content/HR Handbook.pdf"
```

---

### 4. Run the Pipeline

After adding your API key and PDF:

1. Run the notebook cells in order.
2. Generate text, table, and image summaries.
3. Create embeddings and store them in ChromaDB.
4. Start asking questions about your document.

Example questions:

```text
What is the leave policy?
```

```text
Describe the workflow diagram.
```

```text
Summarize the employee benefits section.
```

## Overview

This project implements a Multimodal Retrieval-Augmented Generation (RAG) pipeline that allows users to chat with PDF documents containing:

* Text
* Tables
* Images
* Diagrams
* Charts

The system extracts content from PDFs, generates summaries, creates embeddings, stores them in a vector database, and retrieves relevant information to answer user questions.

---

## Architecture

```text
PDF
 │
 ▼
Unstructured PDF Parsing
 │
 ├── Text
 ├── Tables
 └── Images
 │
 ▼
Summarization
 │
 ├── Text Summaries
 ├── Table Summaries
 └── Image Descriptions
 │
 ▼
Embeddings
 │
 ▼
Chroma Vector Database
 │
 ▼
Retriever
 │
 ▼
Groq LLM
 │
 ▼
Answer Generation
```

---

## Tech Stack

### Document Processing

* Unstructured
* PDF2Image
* PyPDF

### LLMs

* Groq
* Llama 3.1 8B Instant
* Llama 4 Scout

### Embeddings

* HuggingFace Embeddings
* all-MiniLM-L6-v2

### Vector Database

* ChromaDB

### Framework

* LangChain

---

## Installation

```bash
pip install -U "unstructured[all-docs]"
pip install -U chromadb
pip install -U langchain langchain-core langchain-community
pip install -U langchain-groq
pip install -U sentence-transformers
pip install -U python-dotenv
pip install -U pillow lxml pdfminer.six pdf2image pypdf pi_heif unstructured_inference
```

---

## Environment Variables

Create a `.env` file:

```env
GROQ_API_KEY=your_groq_api_key
```

---

## PDF Parsing

The PDF is partitioned using the Unstructured library.

```python
chunks = partition_pdf(
    filename=file_path,
    infer_table_structure=True,
    strategy="hi_res",
    extract_image_block_types=["Image"],
    extract_image_block_to_payload=True,
    chunking_strategy="by_title",
    max_characters=3000,
    combine_text_under_n_chars=2000,
    new_after_n_chars=6000,
)
```

### Parameters

| Parameter                 | Purpose                   |
| ------------------------- | ------------------------- |
| strategy="hi_res"         | High quality parsing      |
| infer_table_structure     | Detects tables            |
| extract_image_block_types | Extracts images           |
| by_title                  | Groups content by section |

---

## Content Separation

Extract:

* Text chunks
* Tables
* Images

```python
texts = []
tables = []
images = []
```

---

## Text Summarization

Text chunks are summarized before storage.

Example:

### Original

```text
Employees receive 20 vacation days annually.
```

### Summary

```text
Company offers 20 annual leave days.
```

Benefits:

* Reduced storage
* Better retrieval
* Lower token consumption

---

## Table Summarization

Tables are converted into descriptive summaries.

### Example

| Employee | Salary |
| -------- | ------ |
| John     | 50000  |
| Anna     | 60000  |

Summary:

```text
Employee salary table showing salaries between 50k and 60k.
```

---

## Image Understanding

Images are processed using a vision-capable model.

Model:

```text
meta-llama/llama-4-scout-17b-16e-instruct
```

The model generates descriptions of:

* Charts
* Graphs
* Diagrams
* Figures

### Example

Image:

```text
Transformer Architecture Diagram
```

Generated Description:

```text
The image shows the Transformer architecture with
multi-head attention, residual connections,
feed-forward layers and normalization blocks.
```

---

## Embeddings

Embeddings are generated using:

```text
sentence-transformers/all-MiniLM-L6-v2
```

Example:

```text
"vacation policy"
```

becomes:

```text
[0.21, -0.89, 0.53, ...]
```

---

## Vector Database

ChromaDB stores:

* Text summaries
* Table summaries
* Image descriptions

```python
vectorstore = Chroma.from_texts(
    texts=all_docs,
    embedding=embedding_function,
    collection_name="multi_modal_rag"
)
```

---

## Retriever

Retriever performs semantic search.

```python
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 5}
)
```

The retriever returns the most relevant document chunks for a user query.

---

## RAG Pipeline

```python
rag_chain = (
    {
        "context": retriever,
        "question": RunnablePassthrough()
    }
    | prompt
    | model
    | StrOutputParser()
)
```

Workflow:

```text
User Question
       ↓
Retriever
       ↓
Relevant Context
       ↓
Prompt
       ↓
Groq LLM
       ↓
Answer
```

---

## Running the Chatbot

```python
while True:

    question = input("Ask a question: ")

    if question.lower() == "exit":
        break

    response = rag_chain.invoke(question)

    print(response)
```

---

## Sample Questions

### Text Questions

```text
What is the leave policy?
```

```text
How many vacation days are provided?
```

### Table Questions

```text
What information does the employee salary table contain?
```

### Image Questions

```text
Describe the workflow diagram.
```

```text
What does the transformer architecture image explain?
```

---

## Features

* Multimodal PDF understanding
* Text extraction
* Table extraction
* Image extraction
* Image captioning
* Semantic search
* ChromaDB vector storage
* Groq-powered question answering
* Interactive chatbot interface

---

## Future Improvements

* Persistent Chroma storage
* Streamlit frontend
* Source citations
* Multi-document support
* Hybrid search
* Reranking
* Metadata filtering

---

## Project Structure

```text
project/
│
├── data/
│   └── PDF Files
│
├── notebooks/
│   └── multimodal_rag.ipynb
│
├── .env
│
├── requirements.txt
│
└── README.md
```

---


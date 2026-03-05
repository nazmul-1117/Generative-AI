# Day_017 | 🏗️ Building a RAG System in LangChain

Building a Retrieval-Augmented Generation (RAG) system in LangChain involves orchestrating several core components, typically executed in a sequence using the **LangChain Expression Language (LCEL)**. The entire process is split into two phases: **Indexing** (pre-processing your data) and **Query Execution** (answering the user's question).

-----

## Phase 1: Indexing Workflow (Preparing the Knowledge Base)

This phase only needs to be run when your documents are new or updated.

### Step 1: Document Ingestion

The process starts by loading data from its source into the standardized LangChain `Document` format.

  * **Tools:** **Document Loaders** (e.g., `PyPDFLoader`, `WebBaseLoader`).
  * **Action:** Connect to the source and load all content.
  * **Best Practice:** For large collections, use `DirectoryLoader` with a `glob` pattern to load all files of a specific type (e.g., `*.pdf`) automatically.

### Step 2: Text Chunking

Long documents must be broken into smaller chunks to fit the LLM's context window and improve retrieval precision.

  * **Tools:** **Text Splitters** (e.g., `RecursiveCharacterTextSplitter`).
  * **Action:** Define a `chunk_size` and a `chunk_overlap` and pass the loaded documents to the splitter.
  * **Best Practice:** Use `RecursiveCharacterTextSplitter` as the default, as it tries to preserve semantic units (like paragraphs) before resorting to arbitrary character splitting.

### Step 3: Embeddings and Vector Storage

The processed text chunks are converted into searchable vectors and stored.

  * **Tools:** **Embedding Model** (e.g., `OpenAIEmbeddings`, `HuggingFaceEmbeddings`) and a **Vector Store** (e.g., **Chroma**, **FAISS**, or **Pinecone**).
  * **Action:** The embedding model generates a vector for each chunk. The Vector Store stores these vectors and their associated original text/metadata.
  * **Implementation:** You typically call a function like `VectorStore.from_documents(chunks, embedding_model)` to perform both steps simultaneously.

-----

## Phase 2: Query Execution Workflow (The RAG Chain)

This phase defines the runtime pipeline that runs every time a user asks a question. This is where LCEL excels.

### Step 4: Define the Retriever

The vector store is converted into a **Retriever**, which defines the search strategy (e.g., $k=5$ for the top 5 chunks).

  * **Tools:** `VectorStore.as_retriever()`.
  * **Best Practice:** You can configure the retriever for advanced techniques here, such as **Maximum Marginal Relevance (MMR)** to get more diverse results, or using a **Self-Query Retriever** for metadata filtering.

### Step 5: Construct the RAG Chain using LCEL

The modern LangChain RAG pipeline uses LCEL to connect components with the pipe operator (`|`). A standard RAG chain performs three actions in parallel and sequence:

1.  **Retrieve Context:** The user's question is passed to the retriever to fetch relevant documents.
2.  **Pass Question:** The original question is preserved.
3.  **Generate Answer:** Both the context and the question are combined into a final prompt for the LLM.

The core LCEL structure often looks like this:

```python
from langchain_core.runnables import RunnablePassthrough, RunnableParallel
from langchain_core.output_parsers import StrOutputParser

# 1. Define the Context Augmentation
setup_and_retrieval = RunnableParallel({
    "context": retriever,  # Retrieve context documents based on question
    "question": RunnablePassthrough() # Pass the original question through
})

# 2. Define the Final RAG Chain
rag_chain = (
    setup_and_retrieval
    | qa_prompt_template  # Format the context and question into the prompt
    | llm                 # Send to the Language Model
    | StrOutputParser()   # Output the final string answer
)

# Execution: response = rag_chain.invoke("What are the core RAG steps?")
```

### Step 6: Response Generation and Evaluation

The final step is the generation of the response, often with configuration tips to ensure quality:

  * **LLM Configuration:** Use a low **temperature** (e.g., $0.0$ to $0.3$) to ensure the response is grounded and factual, rather than creative.
  * **Prompt Engineering:** The prompt template must strongly instruct the LLM to **"Answer based ONLY on the provided context"** and to state "I don't know" if the context is insufficient.
  * **Evaluation:** For production, use specialized tools (like LangSmith) to evaluate the **faithfulness** (did it stick to the facts?) and **answer relevance** of the RAG system.

This video provides a comprehensive crash course on building a RAG system using LangChain. [Complete RAG Crash Course With Langchain In 2 Hours](https://www.youtube.com/watch?v=o126p1QN_RI)

http://googleusercontent.com/youtube_content/5


---

## 🧱 Building a RAG System in LangChain (Step-by-Step)

---

## 1️⃣ Install Required Libraries

```bash
pip install langchain langchain-community langchain-openai faiss-cpu pypdf
```

---

## 2️⃣ Load Documents (Data Ingestion)

Example: Load PDFs

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("data/my_document.pdf")
documents = loader.load()
```

Other loaders:

* `TextLoader`
* `CSVLoader`
* `WebBaseLoader`
* `UnstructuredLoader`

---

## 3️⃣ Split Documents into Chunks

Chunking is critical for good retrieval.

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)

chunks = text_splitter.split_documents(documents)
```

---

## 4️⃣ Generate Embeddings

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
```

---

## 5️⃣ Store Embeddings in a Vector Database

Using **FAISS** (local, fast):

```python
from langchain_community.vectorstores import FAISS

vectorstore = FAISS.from_documents(chunks, embeddings)
vectorstore.save_local("faiss_index")
```

---

## 6️⃣ Create a Retriever

```python
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}
)
```

---

## 7️⃣ Set Up the LLM

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0
)
```

---

## 8️⃣ Build the RAG Chain (Modern LangChain – LCEL)

### Option A: Using `RetrievalQA` (Simple)

```python
from langchain.chains import RetrievalQA

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever,
    chain_type="stuff",
    return_source_documents=True
)
```

Query:

```python
response = qa_chain.invoke({"query": "What is Retrieval Augmented Generation?"})

print(response["result"])
```

---

### Option B: Recommended – LCEL (Composable Chains)

```python
from langchain.prompts import ChatPromptTemplate
from langchain.schema import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough

prompt = ChatPromptTemplate.from_template("""
You are a helpful assistant.
Use the following context to answer the question.
If you don't know, say you don't know.

Context:
{context}

Question:
{question}
""")

rag_chain = (
    {
        "context": retriever,
        "question": RunnablePassthrough()
    }
    | prompt
    | llm
    | StrOutputParser()
)

rag_chain.invoke("Explain RAG in simple terms")
```

---

## 🏗️ RAG Architecture Recap

```
User Query
   ↓
Embedding
   ↓
Vector Store (FAISS)
   ↓
Retriever
   ↓
LLM + Context
   ↓
Answer
```

---

## 🧪 Best Practices for Production RAG

### ✅ Chunking

* 500–1000 tokens
* 10–20% overlap

### ✅ Retrieval

* Try `mmr` instead of similarity

```python
search_type="mmr"
```

### ✅ Prompting

* Force grounding:

> “Answer ONLY using the provided context.”

### ✅ Metadata Filtering

```python
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 4, "filter": {"source": "policy.pdf"}}
)
```

---

## ⚡ Advanced RAG Improvements

| Technique            | Benefit                   |
| -------------------- | ------------------------- |
| Re-ranking           | Better document relevance |
| Hybrid search        | Keyword + vector          |
| Query rewriting      | Better retrieval          |
| Multi-step retrieval | Complex questions         |
| Context compression  | Fit more data             |

---

## 🧠 When to Use LangChain for RAG

* Multi-source retrieval
* Tool + agent workflows
* Rapid prototyping
* Enterprise knowledge bots

---

## 📌 Summary

> **LangChain makes RAG modular: load → chunk → embed → retrieve → generate.**

---
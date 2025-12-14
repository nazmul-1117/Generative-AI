# Day_004 | 🧩 LangChain: The Main Components Explained

LangChain's power comes from its modular architecture, which breaks down complex LLM applications into reusable and composable units. While the framework is extensive, the following are the **six fundamental components** you will use to build nearly any application.

---

### 1. Models (LLMs)

Models are the core interfaces that allow LangChain to interact with various large language models (LLMs).

* **LLMs:** The primary interface for text generation. These components wrap APIs for models like OpenAI's GPT, Google's Gemini, or open-source models like Llama, providing a consistent way to send prompts and receive text completions.
* **Chat Models:** Specifically designed for conversational use cases. They handle inputs as a list of messages (e.g., system, human, AI) and return an AI message, making them ideal for building chatbots.
* **Embedding Models:** Essential for converting text into **Vector Embeddings** (numerical representations). These vectors are crucial for measuring semantic similarity, which underpins the **Retrieval-Augmented Generation (RAG)** process.

---

### 2. Prompts

Prompts manage the inputs to the LLM, ensuring the model receives clear instructions and context.

* **Prompt Templates:** Standardized, reusable recipes for generating prompts. Instead of hardcoding every instruction, a template allows you to define the fixed structure and use placeholders for dynamic input (like a user's question or retrieved context).
* **Output Parsers:** Components used to structure the LLM's raw text response into a predictable format, such as JSON, a Python object, or a list. They are vital for reliably chaining the LLM's output into the next application step.
* **Example Selectors:** Used within templates to automatically select high-quality examples of inputs/outputs to include in the prompt (few-shot prompting) based on the current user query.

---

### 3. Chains

Chains are sequences of components (e.g., a Prompt, followed by an LLM, followed by an Output Parser) that are executed in a predetermined order. They allow you to bundle a series of operations into a single, reliable call.

* **LLMChain:** The most basic chain, taking input, formatting it with a prompt template, and passing it to an LLM.
* **Retrieval Chain (e.g., RAG Chain):** A crucial chain that coordinates the steps for RAG, typically involving retrieving documents and then passing them to the LLM for final answer generation.
* **LangChain Expression Language (LCEL):** The underlying framework that allows components (called **Runnables**) to be easily composed using the pipe operator (`|`). This is the modern and preferred way to build chains.

---

### 4. Retrieval (Data Connection)

This set of components connects the LLM to external data, enabling context awareness and RAG.

* **Document Loaders:** Tools to load data from various sources (PDFs, websites, Notion, etc.) into the `Document` format used by LangChain.
* **Text Splitters:** Necessary for breaking down large documents into smaller, manageable chunks suitable for vector embedding and retrieval.
* **Vector Stores:** Databases (e.g., Chroma, Pinecone) designed to store and search the vector embeddings created from your documents.
* **Retrievers:** The interface used to query the Vector Store (or other data sources) and return the most relevant document chunks based on a user's query. 

---

### 5. Agents

Agents empower the LLM to act as a **reasoning engine** that decides which **Tools** to use and in what order to achieve a goal. Unlike Chains, which follow a fixed path, Agents operate dynamically in a loop.

* **Tools:** Functions or APIs (e.g., a Google Search API, a Calculator, a Python code executor) that an Agent can call to gather information or interact with its environment.
* **Agent Executor:** The runtime that coordinates the entire process: taking user input, passing it to the LLM (the **brain**), receiving an **Action** (Tool and inputs), executing the Tool, and feeding the **Observation** back to the LLM until the final answer is formulated. 

---

### 6. Memory

Memory components are used to maintain the history and context of a conversation, giving the LLM the ability to "remember" previous interactions.

* **Conversation Buffer Memory:** Stores the entire history of the conversation (input/output pairs) and inserts it into the prompt.
* **Conversation Summary Memory:** Stores a running summary of the conversation, which is inserted into the prompt instead of the full transcript, saving token space and costs for long conversations.
* **Entity Memory:** Designed to remember specific named entities (like names or places) mentioned in the conversation.

---

## 🔧 **Main Components of LangChain**

LangChain is built from several modular components that help create powerful LLM-based applications. Below are the **core components** and what each one does.

---

## 1️⃣ **Models**

These are the LLMs or embeddings used in your application.

### **Types of Models**

* **LLMs** → Generate text (e.g., GPT, Llama, Claude)
* **Chat Models** → Structured conversation models
* **Embedding Models** → Convert text into vectors for search/RAG

### **Purpose**

* Main intelligence behind your app
* Generate responses, reasoning, and content

---

## 2️⃣ **Prompts**

Prompts define *what the model should do*.

### **Includes:**

* **Prompt templates** (variables + instructions)
* **Example prompts**
* **Prompt chains**

### **Purpose**

* Provide clear, reusable instructions
* Avoid messy, hard-to-manage prompt strings

---

## 3️⃣ **Chains**

A **Chain** links multiple steps together.

> Example:
> Retrieve data → Summarize → Answer question

### **Common Types**

* **LLMChain** → Basic prompt + model
* **SequentialChain** → Multi-step workflow
* **RouterChain** → Choose path dynamically

### **Purpose**

* Build pipeline-like workflows instead of single prompts

---

## 4️⃣ **Agents**

Agents allow the model to **think, choose actions, and call tools** autonomously.

### **An agent can:**

* Decide which tool to use
* Ask for missing information
* Perform multi-step reasoning
* Execute code, search, or query data sources

### **Purpose**

* Introduce autonomy and decision-making

---

## 5️⃣ **Tools**

Tools are external actions or functions the agent can use.

### **Examples of Tools**

* Web search
* Database query
* Code execution
* Calculator
* File reading
* API calls

### **Purpose**

* Extend LLM capabilities beyond text generation
  (LLMs → Action-taking AI)

---

## 6️⃣ **Memory**

Memory stores information about past interactions.

### **Types of Memory**

* **Buffer memory** → Full conversation history
* **Summary memory** → Summaries instead of full text
* **Vector memory** → Semantic long-term memory
* **Key-value memory** → Custom memory store

### **Purpose**

* Maintain context
* Support long, natural conversations

---

## 7️⃣ **Document Loaders**

These load data from different sources.

### **Sources include:**

* PDF files
* Word docs
* Text files
* Websites
* Databases
* Cloud storage (S3, GDrive)

### **Purpose**

* Bring your external documents into LangChain

---

## 8️⃣ **Text Splitters**

These break documents into smaller chunks.

### **Why Needed?**

* LLMs have context limits
* Improve retrieval quality
* Efficient processing

### **Types**

* Character splitter
* Recursive splitter
* Token-based splitter

---

## 9️⃣ **Retrievers**

Retrievers fetch the most relevant chunks of information.

### **Examples**

* Vector store retriever
* BM25 retriever
* Hybrid retriever (vector + keyword)

### **Purpose**

* Connect LLMs with your stored knowledge base
* Forms the core of RAG systems

---

## 🔟 **Vector Stores**

Databases that store embeddings of your text.

### **Popular Vector Stores**

* Pinecone
* FAISS
* Chroma
* Weaviate
* Milvus

### **Purpose**

* Fast similarity search
* Enables retrieval-augmented generation (RAG)

---

## 🧩 **Putting It All Together**

Below is how components typically interact:

```
Documents → Loaders → Splitters → Embeddings → Vector Store
                                   ↓
                              Retriever
                                   ↓
                                Chain
                        (Prompt + Model)
                                   ↓
                                 Output
```

If using agents:

```
User → Agent → (Tools + Memory + Model) → Output
```

---

## 📝 Summary (Very Short)

| Component          | What it Does                      |
| ------------------ | --------------------------------- |
| **Models**         | Generate text or embeddings       |
| **Prompts**        | Instructions for models           |
| **Chains**         | Multi-step workflows              |
| **Agents**         | Autonomous decision-making        |
| **Tools**          | External functions models can use |
| **Memory**         | Stores conversation context       |
| **Doc Loaders**    | Load data from files/web          |
| **Text Splitters** | Break documents into chunks       |
| **Retrievers**     | Fetch relevant data               |
| **Vector Stores**  | Store embeddings for search       |

---


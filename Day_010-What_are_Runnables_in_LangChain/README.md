# Day_010 | 🏃 What are Runnables in LangChain?

A **Runnable** is the fundamental building block and standardized interface across all components in LangChain. Essentially, a Runnable is **any object that takes an input, processes it, and returns an output**, adhering to a consistent protocol.

Every core component in LangChain—from a **Prompt Template** and a **Chat Model** to an **Output Parser** or a **Retriever**—is a Runnable.

### Why We Need Runnables

The introduction of the Runnable interface solved major problems in the LangChain ecosystem, enabling reliable and scalable application development:

| Reason | Description |
| :--- | :--- |
| **Standardized Execution** | Before Runnables, every component had its own execution method (`.predict()`, `.run()`, `.get_relevant_documents()`). Runnables unify this with a standard set of methods: `.invoke()`, `.batch()`, and `.stream()`. |
| **Composability (LCEL)** | They enable the **LangChain Expression Language (LCEL)**, allowing components to be easily connected using the pipe operator (`|`) to form complex chains. This makes workflows readable and less brittle. |
| **Modularity and Reusability** | Since every component conforms to the same interface, you can effortlessly swap out a model (e.g., from GPT-4 to Llama 3) or an Output Parser without rewriting the rest of the chain logic. |
| **Advanced Features** | Runnables automatically gain support for advanced features like **streaming** (for real-time chat), **parallel execution** (`.batch()` and asynchronous methods), and **observability** (via LangSmith). |

---

## 🛠️ How Runnables are Implemented in LangChain

Runnables are implemented through the `Runnable` base class and several specialized helper classes, primarily used in building complex workflows.

### 1. The Core Interface Methods

All Runnables expose a core set of synchronous and asynchronous execution methods:

| Method | Purpose | Use Case |
| :--- | :--- | :--- |
| **`.invoke(input)`** | Executes the runnable with a single input and returns a single output. | Standard sequential execution. |
| **`.batch([inputs])`** | Executes the runnable with a list of inputs, often running them **concurrently** for efficiency. | Processing a list of documents or user queries at once. |
| **`.stream(input)`** | Executes the runnable and yields the output in chunks as it's generated. | Real-time output display in chatbots and UIs. |

### 2. Implementation in Components

All built-in LangChain components automatically implement the Runnable interface:

* **Prompt Template:** Takes a dictionary of variables as input and outputs a formatted prompt (a message list).
* **Chat Model:** Takes a message list as input and outputs an `AIMessage` object.
* **Output Parser:** Takes the raw `AIMessage` content as input and outputs a parsed Python object (e.g., a dictionary or Pydantic model).

### 3. Key Runnable Types (Building Blocks)

LangChain provides specialized Runnables to manage data flow and logic within chains:

| Runnable Type | Description | LCEL Use Case |
| :--- | :--- | :--- |
| **`RunnableLambda`** | Wraps any standard Python function or `lambda` into a Runnable. | Custom preprocessing, data cleaning, or simple calculations within a chain. |
| **`RunnableParallel`** | Executes multiple Runnables simultaneously on the same input and aggregates their results into a dictionary. | Running a web search and a document retrieval concurrently on a user query.  |
| **`RunnablePassthrough`** | Passes the input received from the previous step *unchanged* to the next step, sometimes adding new keys. | Preserving the original user query for a later step (e.g., in a RAG chain). |
| **`RunnableBranch`** | Implements conditional logic (`if/elif/else`), routing the input to different chains based on a specified condition (often checked by a `RunnableLambda`). | Building a multi-purpose router agent. |

Understanding how to use these core runnable types is essential for building complex, customized chains in LangChain.

You can learn more about how to create custom Runnables in this video: [LangChain Runnables - Create Custom Runnables](https://www.youtube.com/watch?v=pJ0B_3x0RkU).


http://googleusercontent.com/youtube_content/2



---

## 1️⃣ What are **Runnables** in LangChain?

In LangChain, **Runnables** are **abstractions for anything that can be "run" with input to produce output**. Think of them as **building blocks in a computational pipeline**.

* They generalize many things in LangChain:

  * Language models (LLMs)
  * Prompts
  * Chains
  * Tools
  * Text transformations

Instead of directly calling an LLM or a chain, you wrap it as a **Runnable**. Then you can **compose, execute, and manage these computations in a uniform way**.

**Key idea:**
A `Runnable` is **anything you can `.invoke()` or `.batch()` on**.

---

## 2️⃣ Why do we need Runnables?

Runnables exist to solve some problems:

1. **Uniform interface**:
   Previously, using LLMs, chains, and prompts had slightly different ways of execution. Runnables give a **single interface**:

   ```python
   output = my_runnable.invoke("Hello")
   ```

2. **Composition and pipelines**:
   You can **chain Runnables** together, like a production line:

   ```python
   result = (runnable1 | runnable2 | runnable3).invoke(input_data)
   ```

   This is much cleaner than manually passing outputs around.

3. **Batching support**:
   Many models benefit from batch processing. Runnables have built-in support:

   ```python
   outputs = my_runnable.batch(["input1", "input2", "input3"])
   ```

4. **Flexibility for advanced features**:
   Runnables make it easier to add:

   * Caching
   * Tracing/logging
   * Async execution
   * Retry logic

---

## 3️⃣ How are Runnables implemented in LangChain?

LangChain’s **Runnable API** is designed in Python as a **base class** called `Runnable`. The key methods are:

### **a) Basic Runnable class structure**

```python
from langchain.core.runnables import Runnable

class MyRunnable(Runnable):
    def invoke(self, input: str) -> str:
        # Core logic
        return input.upper()
    
    def batch(self, inputs: list[str]) -> list[str]:
        # Optional batch method
        return [self.invoke(i) for i in inputs]
```

### **b) Composition operators**

* `|` → sequential composition
* `&` → parallel composition
* `>>` → also sequential, but for pipelines

Example:

```python
r1 = MyRunnable()
r2 = MyRunnable()

pipeline = r1 | r2
result = pipeline.invoke("hello")  # passes through r1 then r2
```

### **c) Built-in Runnables**

LangChain provides Runnable wrappers for:

* **LLMs**: `LLMRunnable`
* **Chains**: `ChainRunnable`
* **Text transformations**: `TextTransformRunnable`
* **Tools**: `ToolRunnable`

Example with an LLM:

```python
from langchain.chat_models import ChatOpenAI
from langchain.core.runnables import LLMRunnable

llm = ChatOpenAI(model="gpt-4")
llm_runnable = LLMRunnable(llm)

response = llm_runnable.invoke("Translate to French: Hello")
```

### **d) Async and batch support**

```python
# Async
response = await llm_runnable.ainvoke("Translate to French: Hello")

# Batch
responses = llm_runnable.batch(["Hello", "Goodbye"])
```

---

### ✅ Summary

* **Runnables** = abstract executable units in LangChain.
* **Purpose** = unify interface, enable composition, batching, and add features like logging, caching, retries.
* **Implementation** = Python `Runnable` base class with:

  * `invoke()` → single execution
  * `batch()` → batch execution
  * Composition operators (`|`, `&`, etc.)
  * Wrappers for LLMs, chains, tools, and transformations.

Think of them like **LEGO blocks**: each block can be run, combined, and reused in pipelines, giving you a lot of flexibility and consistency.

---


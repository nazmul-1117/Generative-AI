# Day_007 | 📊 Structured Output in LangChain

**Structured Output** is a critical capability in LangChain that enables developers to guarantee that the output of an LLM adheres to a predefined, machine-readable format, such as JSON. This transforms the model's response from free-form, unpredictable text into clean, typed data that can be used directly by software.

LangChain provides the `with_structured_output` method on ChatModels to easily enforce a schema.

-----

### Structured Output vs. Non-Structured Output (Example)

The difference between the two is reliability and direct programmatic utility.

| Aspect | Non-Structured Output (Raw Text) | Structured Output (e.g., JSON) |
| :--- | :--- | :--- |
| **Prompt:** "What is the capital of France and its population?" | **Response:** "The capital of France is Paris. Its metropolitan population is about 12.3 million people." | **Response:** `{"city": "Paris", "country": "France", "population_millions": 12.3}` |
| **Utility** | Requires manual or complex regex/parsing code to extract "Paris" and "12.3". Parsing often **fails** if the model changes the wording. | The data is immediately available as a Python dictionary or Pydantic object, ready for database storage or API calls. |
| **Reliability** | **Low**. The model's wording can vary, breaking extraction logic. | **High**. The output is guaranteed to follow the schema's keys and data types. |

-----

### Why We Need Structured Output

Structured output is a game-changer for building reliable, production-ready AI applications.

1.  **Reliability and Consistency:** It eliminates the variability inherent in LLM text generation. The output is predictable, allowing downstream systems to rely on specific keys and data types.
2.  **Machine Integration:** Data in structured formats (like JSON) can be consumed directly by APIs, databases (e.g., inserting a new row), and other microservices **without writing custom parsing code**.
3.  **Type Safety and Validation:** When using tools like Pydantic, the output is not just JSON, but a fully validated Python object. If the LLM returns an integer where a string is expected, validation fails, and the system can handle the error immediately (often by retrying).
4.  **Simpler Prompting:** You no longer need extensive, wordy instructions (e.g., "Respond ONLY in JSON format. Do not include any introductory text,...") as the structured output mechanism handles the formatting guidance automatically.

-----

## 🛠️ LLM Support for Structured Output

The ability to generate reliable structured output depends on whether the model provider supports the feature natively.

| Model Capability | Description | Closed-Source Examples | Open-Source Examples |
| :--- | :--- | :--- | :--- |
| **Native Structured Support** (Preferred) | The model (via its API) is given the output schema (usually JSON Schema) and uses techniques like **Function Calling** or **JSON Mode** to enforce the structure at the token generation level. **Most reliable.** | GPT-4o, Gemini 2.5 series, Claude 3/3.5, Mistral Large/Medium. | Llama 3 (via its API or using libraries like Outlines/Instructor), Nemotron-mini. |
| **Non-Native Support** (Fallback) | The model is prompted with strong instructions and examples to produce JSON, and the LangChain **Output Parser** attempts to clean up the raw text. **Less reliable** and prone to failure. | Older GPT-3.5 models, most smaller self-hosted models without constrained decoding frameworks. | Older Llama models, general models hosted on Hugging Face without specific structured generation tools. |

-----

## 💾 Data Formats for `with_structured_output`

The LangChain method `.with_structured_output(schema)` accepts various ways to define the output schema, with **Pydantic** being the most feature-rich option.

| Data Format | LangChain Input Type | Output Data Type | Key Benefit |
| :--- | :--- | :--- | :--- |
| **Pydantic Model** | A Python class inheriting from `pydantic.BaseModel`. | A Pydantic object instance. | **Strict Validation,** Type Coercion, Field Descriptions (which help the LLM), and automatic handling of complex/nested schemas. |
| **TypedDict** | A Python class defined using `typing.TypedDict`. | A standard Python `dict`. | Simplicity and lightweight—useful when you need a simple structure and don't require the full validation features of Pydantic. |
| **JSON Schema** | A standard Python `dict` representing a JSON Schema specification. | A standard Python `dict`. | **Interoperability**—useful when the schema is already defined externally or needs to conform to a universal API standard. |

**Example using Pydantic (The Recommended Approach):**

```python
from pydantic import BaseModel, Field
from langchain_openai import ChatOpenAI

# 1. Define the desired structure using a Pydantic Model
class TaskList(BaseModel):
    """A list of tasks extracted from a user request."""
    primary_task: str = Field(description="The main task the user wants to accomplish.")
    required_tools: list[str] = Field(description="A list of external tools needed for the task.")
    priority_level: int = Field(description="The priority of the task, 1-10.")

# 2. Bind the schema to the model
structured_llm = ChatOpenAI(model="gpt-4o-mini").with_structured_output(TaskList)

# 3. Invoke the model
result = structured_llm.invoke("I need to schedule a meeting with John, check the weather, and then send an email. Make it high priority.")

# The result is a guaranteed TaskList object, ready for use
print(result.primary_task)  # Output: schedule a meeting...
print(result.priority_level) # Output: 9 (or similar)
```

---

## 🧱 **Structured Output in LangChain**

LangChain provides the method **`with_structured_output()`** that forces or strongly guides the LLM to return data in a fixed schema (e.g., JSON with predefined fields).
This removes ambiguity and makes LLM responses **machine-processable**.

Example:

```python
model = ChatOpenAI(model="gpt-4.1-mini").with_structured_output(MySchema)
```

---

## 📌 **1. Structured Output vs Non-Structured Output (with example)**

### **Non-Structured Output (Default LLM Behavior)**

The model returns free text—good for human reading, bad for downstream automation.

#### 🔹 Example Prompt:

> Extract the company name and revenue from this text:
> “TechNova generated $4.2M in revenue in 2023.”

### 🔹 Typical LLM Response:

```
TechNova made $4.2 million in revenue in 2023.
```

This is unstructured:\
❌ difficult to parse reliably\
❌ formatting may vary\
❌ ambiguous data types

---

## **Structured Output (Strict Schema Enforcement)**

LangChain enforces schema using `with_structured_output()`.

### Example Schema (TypedDict)

```python
class CompanyInfo(TypedDict):
    company: str
    revenue: float
```

### Structured Output Model Call:

```python
model = ChatOpenAI().with_structured_output(CompanyInfo)
result = model.invoke("TechNova generated $4.2M in revenue in 2023.")
```

### Result:

```json
{
  "company": "TechNova",
  "revenue": 4.2
}
```

Machine-friendly:\
✅ guaranteed field names\
✅ guaranteed types\
✅ eliminates parsing errors

---

## 📌 **2. Why We Need Structured Output**

| Reason                                    | Explanation                                                          |
| ----------------------------------------- | -------------------------------------------------------------------- |
| **Reliable automation**                   | When LLM output feeds into other systems (ETL, RAG pipelines, APIs). |
| **Prevents hallucinated formats**         | Ensures model sticks to required structure.                          |
| **Type safety**                           | Numbers stay numbers, arrays stay arrays—no messy parsing.           |
| **Integration with databases**            | JSON → SQL inserts, object stores, data pipelines.                   |
| **Easier testing**                        | Deterministic format → stable unit tests.                            |
| **Higher confidence for production apps** | Output shape is guaranteed, reducing failures.                       |

Structured output is essential when LLMs automate workflows.

---

## 📌 **3. LLMs That Support Structured Output (and Models That Don’t)**

There are 2 categories:

---

### ✅ **A. Models That Natively Support Structured Output / JSON Mode**

These models can be forced to output JSON that conforms to a schema.

### **Closed-Source Models**

| Provider      | Model                         | Support                      | Notes                    |
| ------------- | ----------------------------- | ---------------------------- | ------------------------ |
| **OpenAI**    | GPT-4.1, 4.1-mini, o3-mini    | Full JSON + function calling | Best schema adherence    |
| **Anthropic** | Claude 3.5 Sonnet / 3.5 Haiku | JSON mode + tool use         | Very strict              |
| **Google**    | Gemini 1.5 Flash / Pro        | JSON mode                    | Strong structured output |

---

### **Open-Source Models**

| Model                             | Support                  | Notes                                       |
| --------------------------------- | ------------------------ | ------------------------------------------- |
| **Llama-3 (Meta)**                | Good JSON reliability    | Works well with LangChain structured output |
| **Mistral-Large / Mistral-Small** | JSON mode                | High adherence, open-weight                 |
| **Qwen-2.5**                      | JSON mode                | Great for structured extraction             |
| **Phi-3**                         | “JSON mode” prompt-based | Lightweight, but can drift slightly         |

These models explicitly support or perform well with structured output when using LangChain.

---

### ❌ **B. Models That DO NOT Reliably Support Structured Output**

| Model                                    | Reason                                       |
| ---------------------------------------- | -------------------------------------------- |
| GPT-3.5 (legacy)                         | Often breaks JSON; formatting inconsistent   |
| Older Llama 2/3 fine-tuned variants      | Tune may distort formatting                  |
| Tiny open-source models (<2B parameters) | Memory too small for strict schema adherence |
| Models without JSON mode                 | Harder to enforce object structure           |

But even these can *attempt* structured output with LangChain’s schema enforcement—they’re just less reliable.

---

## 📌 **4. Data Formats Supported by `with_structured_output()`**

LangChain accepts multiple schema definitions.
The most common:

---

### **A. Python TypedDict**

Useful for simple flat structures.

```python
class Person(TypedDict):
    name: str
    age: int
```

---

### **B. Pydantic Models (V1 or V2)**

Best for validated, nested, complex schemas.

```python
from pydantic import BaseModel

class Product(BaseModel):
    name: str
    price: float
    tags: list[str]
```

---

### **C. JSON Schema**

For maximum flexibility or when generating schemas dynamically.

```python
schema = {
    "type": "object",
    "properties": {
        "topic": {"type": "string"},
        "sentiment": {"type": "string"},
    },
    "required": ["topic", "sentiment"]
}
```

Use:

```python
model = llm.with_structured_output(schema)
```

---

### **D. Tool / Function Calling Schema**

LangChain can convert structured output into a function call format.

```python
tool_schema = {
  "name": "extract",
  "description": "Extract company details",
  "parameters": {...}
}
```

---

## 📌 **5. Summary Table**

| Feature               | Structured Output            | Non-Structured Output |
| --------------------- | ---------------------------- | --------------------- |
| Machine readable      | ✅                            | ❌                     |
| JSON guaranteed       | ✅                            | ❌                     |
| Downstream automation | Easy                         | Hard                  |
| Schema validation     | Yes                          | No                    |
| Best for              | Data pipelines, agents, APIs | Creative text, essays |

---


## **1. Pydantic**

**Type:** Python class-based data validation library

| Feature                      | Details                                                                                                                                                                        |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Type enforcement**         | Strong. Uses Python type hints and validates at runtime.                                                                                                                       |
| **Runtime validation**       | ✅ Full runtime validation. Ensures data conforms to types and constraints.                                                                                                     |
| **IDE support**              | ✅ Excellent. Autocomplete, type checking, linting.                                                                                                                             |
| **Serialization**            | ✅ Easy: `.dict()` / `.json()` methods.                                                                                                                                         |
| **Flexibility**              | High. Supports default values, nested models, validators, complex types.                                                                                                       |
| **Use case for chat models** | Ideal for structured outputs where you need **guaranteed correctness** before further processing. E.g., extracting entities, generating structured JSON from a model response. |

**Example:**

```python
from pydantic import BaseModel

class UserInfo(BaseModel):
    name: str
    age: int
    email: str

# Validate model output
user = UserInfo.parse_obj(model_output)
```

---

## **2. TypedDict**

**Type:** Python typing construct for dictionary-like objects

| Feature                      | Details                                                                                                                                  |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Type enforcement**         | ⚠️ Only at **static type checking** time (mypy, Pyright). Not enforced at runtime.                                                       |
| **Runtime validation**       | ❌ None by default (requires extra libraries like `typeguard`).                                                                           |
| **IDE support**              | ✅ Good for autocomplete and type hints.                                                                                                  |
| **Serialization**            | Limited: works like normal dicts, `.json()` not automatic.                                                                               |
| **Flexibility**              | Moderate. Can describe nested dictionaries, optional keys.                                                                               |
| **Use case for chat models** | Useful when you **trust model output** and want IDE assistance for structured data. Not safe if you need guaranteed runtime correctness. |

**Example:**

```python
from typing import TypedDict

class UserInfo(TypedDict):
    name: str
    age: int
    email: str

# Just a type hint; no runtime check
def process_user(user: UserInfo):
    ...
```

---

## **3. JSON Schema**

**Type:** Language-agnostic schema for JSON

| Feature                      | Details                                                                                                                                        |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Type enforcement**         | ✅ Strong, if you use a validator library (e.g., `jsonschema`).                                                                                 |
| **Runtime validation**       | ✅ Can validate any JSON data at runtime.                                                                                                       |
| **IDE support**              | ⚠️ Limited in Python; better tooling in JS/TS.                                                                                                 |
| **Serialization**            | ✅ Directly serializable as JSON; works across languages.                                                                                       |
| **Flexibility**              | High. Can express required/optional fields, enums, arrays, nested objects.                                                                     |
| **Use case for chat models** | Perfect for **cross-platform structured outputs** or APIs. Often used with **OpenAI function-calling** to ensure the model outputs valid JSON. |

**Example:**

```python
from jsonschema import validate

schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer"},
        "email": {"type": "string"},
    },
    "required": ["name", "age", "email"]
}

validate(instance=model_output, schema=schema)
```

---

## **Summary Comparison**

| Feature              | Pydantic                           | TypedDict                | JSON Schema                                  |
| -------------------- | ---------------------------------- | ------------------------ | -------------------------------------------- |
| Runtime validation   | ✅ Yes                              | ❌ No                     | ✅ Yes                                        |
| Static type checking | ✅ Yes                              | ✅ Yes                    | ⚠️ Limited in Python                         |
| IDE autocomplete     | ✅ Excellent                        | ✅ Good                   | ⚠️ Limited                                   |
| Serialization        | ✅ Easy                             | ⚠️ Manual                | ✅ Native JSON                                |
| Flexibility          | High                               | Moderate                 | High                                         |
| Best for             | Python apps with strict validation | Lightweight typing hints | Cross-language APIs, schema-based validation |

---

**Key takeaway:**

* Use **Pydantic** if you want **Python-native, runtime-checked structured outputs**.
* Use **TypedDict** if you want **lightweight static typing** with minimal overhead.
* Use **JSON Schema** if you need **language-agnostic validation** or want to integrate with external APIs (e.g., OpenAI function-calling).

---
# Day_020 | 🌦️ End-to-End AI Agent using LangChain (ReAct) + Hugging Face | Weather Application

![image](assets/573b688fbac605dde5807bf57b725bd5.gif)

An **AI Agent** is a system that uses a Large Language Model (LLM) as a reasoning engine to determine which actions to take and in what order. Unlike a **Chain**, where the sequence of steps is hard-coded, an **Agent** dynamically decides its path based on the user's input and the results of its own previous actions.

In 2025, the industry standard for building production-grade agents has shifted from the legacy `AgentExecutor` to **LangGraph**, which offers better control, state management, and reliability.

---

## 🏗️ The 5 Core Components of a Modern Agent

To build an end-to-end agent, you must coordinate these five fundamental pillars:

| Component | Role | 2025 Industry Standard |
| --- | --- | --- |
| **1. The Brain** | The reasoning engine that plans and selects tools. | **LLMs with Native Tool Calling** (e.g., GPT-4o, Claude 3.5 Sonnet). |
| **2. The Hands** | External functions (APIs, DBs, Search) the agent can invoke. | **LangChain Tools** (decorated with `@tool`). |
| **3. The Memory** | Persistence of conversation history and internal "thoughts." | **LangGraph State** (using `TypedDict` and `Checkpointers`). |
| **4. The Strategy** | The logic of how the agent thinks (e.g., ReAct: Reason + Act). | **Prompt Engineering** + **Iterative Loops**. |
| **5. The Engine** | The orchestration layer that manages the execution loop. | **LangGraph** (replaces the older `AgentExecutor`). |

---

## 🛠️ Step-by-Step: Building an Agent with LangGraph

The modern "End-to-End" workflow follows a structured graph-based approach.

### 1. Define the Tools

Create functions that the agent can use to interact with the world. LangChain uses these to generate a JSON schema that the LLM understands.

```python
from langchain_core.tools import tool

@tool
def get_weather(city: str):
    """Returns the current weather for a specific city."""
    return f"The weather in {city} is 22°C and sunny."

```

### 2. Initialize the Model & Bind Tools

You must "bind" the tools to the model so the LLM knows it has permission to request their execution.

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o")
llm_with_tools = llm.bind_tools([get_weather])

```

### 3. Define the Agent State

In LangGraph, you define a **State** object (usually a list of messages) that gets passed between different nodes in your graph.

### 4. Construct the Graph (Nodes & Edges)

A standard agent graph consists of two main nodes:

* **The Agent Node:** Calls the LLM to decide the next action.
* **The Tool Node:** Executes the tools requested by the LLM.

### 5. Orchestrate the Logic (The ReAct Loop)

The graph uses **Conditional Edges** to decide the flow:

1. **If** the LLM provides a final answer \rightarrow **End**.
2. **If** the LLM requests a tool call \rightarrow Route to **Tool Node**, then loop back to the **Agent Node**.

---

## 🚀 Advanced Production Features

For a truly "end-to-end" system, you must implement the following production-grade layers:

* **Observability (LangSmith):** Essential for debugging. It allows you to see every "thought," tool call, and token spent in real-time.
* **Human-in-the-Loop:** In high-stakes environments (e.g., sending an email or deleting data), you can configure the graph to **Interrupt** and wait for human approval before executing a tool.
* **Persistence (Checkpointers):** This allows the agent to "remember" the state of a conversation across different sessions or threads, making it truly stateful.

### Why use LangGraph over standard Chains?

While **Chains** are great for predictable, linear tasks (e.g., "Summarize this PDF"), **Agents (via LangGraph)** are built for open-ended complexity where the LLM might need to try multiple steps, correct its own errors, or loop back to gather more data until the goal is achieved.

Would you like me to generate a complete, runnable Python code example for a basic LangGraph search agent?

[LangChain Agents in 2025 | Full Tutorial for v0.3](https://www.youtube.com/watch?v=Gi7nqB37WEY)
This video provides a complete walk-through of the modern LangChain agent architecture, focusing on tool integration and the latest orchestration patterns for 2025.

---

## Weather Application [ChatGPT.com](https://chatgpt.com/)

---

## 1. Project Overview

This project demonstrates how to build an **AI Agent** that:

* Understands natural language queries
* Reasons step-by-step using **ReAct (Reason + Act)**
* Calls external tools (weather API)
* Uses an **open-source Hugging Face LLM**
* Returns accurate weather information

---

## 2. Tech Stack

| Layer                 | Technology                               |
| --------------------- | ---------------------------------------- |
| Agent Framework       | LangChain                                |
| Agent Type            | ReAct Agent                              |
| LLM                   | Hugging Face (Mistral / Llama / FLAN-T5) |
| Tool                  | Weather API (OpenWeatherMap)             |
| Language              | Python                                   |
| Deployment (Optional) | FastAPI / Streamlit                      |

---

## 3. Architecture

```
User Query
   ↓
ReAct Agent (LangChain)
   ↓
LLM (Hugging Face)
   ↓
Tool Decision (Weather Tool)
   ↓
Weather API
   ↓
Final Answer
```

---

## 4. ReAct Agent Concept

**ReAct = Reason + Act**

The agent:

1. Thinks about what the user wants
2. Decides which tool to call
3. Executes the tool
4. Observes the result
5. Produces a final response

Example reasoning (hidden from user):

```
Thought: User wants weather for Delhi
Action: get_weather
Observation: 32°C, clear sky
Final Answer: It's 32°C and clear in Delhi
```

---

## 5. Project Structure

```
weather-ai-agent/
│
├── app.py
├── agent.py
├── tools.py
├── requirements.txt
├── README.md
└── .env
```

---

## 6. Installation

### 6.1 Create Virtual Environment

```bash
python -m venv venv
source venv/bin/activate
```

### 6.2 Install Dependencies

```bash
pip install langchain langchain-community transformers huggingface_hub requests python-dotenv
```

---

## 7. Hugging Face Model Setup

### Recommended Models

* `mistralai/Mistral-7B-Instruct`
* `google/flan-t5-large`
* `meta-llama/Llama-2-7b-chat-hf`

### Example (FLAN-T5 – lightweight)

```python
from langchain_community.llms import HuggingFacePipeline
from transformers import pipeline

llm_pipeline = pipeline(
    "text2text-generation",
    model="google/flan-t5-large",
    max_length=512
)

llm = HuggingFacePipeline(pipeline=llm_pipeline)
```

---

## 8. Weather Tool Implementation

### `tools.py`

```python
import requests
import os
from langchain.tools import tool

API_KEY = os.getenv("OPENWEATHER_API_KEY")

@tool
def get_weather(city: str) -> str:
    """
    Get current weather for a city
    """
    url = (
        f"http://api.openweathermap.org/data/2.5/weather"
        f"?q={city}&appid={API_KEY}&units=metric"
    )

    response = requests.get(url).json()

    if response.get("cod") != 200:
        return "City not found"

    temp = response["main"]["temp"]
    desc = response["weather"][0]["description"]

    return f"Temperature: {temp}°C, Condition: {desc}"
```

---

## 9. ReAct Agent Implementation

### `agent.py`

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub
from langchain_community.llms import HuggingFacePipeline
from transformers import pipeline

from tools import get_weather

# Load ReAct Prompt
prompt = hub.pull("hwchase17/react")

# HuggingFace LLM
hf_pipeline = pipeline(
    "text2text-generation",
    model="google/flan-t5-large",
    max_length=512
)

llm = HuggingFacePipeline(pipeline=hf_pipeline)

# Tools
tools = [get_weather]

# Create Agent
agent = create_react_agent(llm=llm, tools=tools, prompt=prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True
)
```

---

## 10. Main Application

### `app.py`

```python
from agent import agent_executor

def run_weather_agent():
    print("🌦️ Weather AI Agent (type 'exit' to quit)\n")

    while True:
        query = input("You: ")

        if query.lower() == "exit":
            break

        response = agent_executor.invoke({"input": query})
        print("Agent:", response["output"])

if __name__ == "__main__":
    run_weather_agent()
```

---

## 11. Example Run

```text
You: What's the weather in Paris today?
Thought: I should get weather information
Action: get_weather
Action Input: Paris
Observation: Temperature: 18°C, Condition: cloudy
Final Answer: It's currently 18°C with cloudy skies in Paris.
```

---

## 12. Environment Variables

### `.env`

```env
OPENWEATHER_API_KEY=your_api_key_here
```

Load it:

```python
from dotenv import load_dotenv
load_dotenv()
```

---

## 13. Enhancements (Next Steps)

✅ Multi-day forecast
✅ City auto-detection
✅ Voice input
✅ Streamlit UI
✅ Memory (conversation history)
✅ Agent routing (weather + news + maps)

---

## 14. Key Takeaways

* ReAct enables **tool-driven reasoning**
* LangChain simplifies agent orchestration
* Hugging Face allows **open-source LLMs**
* This architecture scales to **any AI assistant**

---

## 15. Want Extras?

I can:

* Add **FastAPI backend**
* Convert to **Streamlit Web App**
* Use **Llama-3 or Mistral**
* Add **agent memory**
* Provide **system design diagram**
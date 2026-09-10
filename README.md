# LangGraph Tool Calling Agent 🤖

A practical **AI agent project built with LangGraph, LangChain, and OpenAI** that demonstrates how an LLM can decide when to call an external tool and then continue the conversation using the tool result.

The project uses a simple addition tool to demonstrate the complete **LLM → Tool → LLM** agent workflow.

---

## 📌 Project Overview

This project demonstrates how to build a tool-calling agent using **LangGraph**.

The agent is built around a state graph where:

1. A user provides a message.
2. The message is sent to the LLM.
3. The LLM determines whether a tool is required.
4. If a tool call is requested, LangGraph routes the request to the tool.
5. The tool executes the requested operation.
6. The tool result is added back to the message state.
7. The LLM receives the result and generates the final response.

The project uses a simple `add()` function as the tool so that the focus remains on understanding the **agent architecture and tool-calling workflow**.

---

## 🏗️ Agent Architecture

```text
                User
                  │
                  ▼
        ┌──────────────────┐
        │   LangGraph      │
        │     State        │
        └────────┬─────────┘
                 │
                 ▼
        ┌──────────────────┐
        │   LLM Node       │
        │  ChatOpenAI       │
        └────────┬─────────┘
                 │
          tools_condition
             /       \
            /         \
       No tool       Tool call
          │              │
          │              ▼
          │       ┌──────────────┐
          │       │   ToolNode   │
          │       │    add()     │
          │       └──────┬───────┘
          │              │
          │              │ Tool result
          │              ▼
          │       ┌──────────────┐
          └──────►│     LLM      │
                  └──────┬───────┘
                         │
                         ▼
                    Final Answer
```

---

## 🧠 Technologies Used

* **Python**
* **LangChain**
* **LangGraph**
* **OpenAI**
* **ChatOpenAI**
* **Pydantic / TypedDict**
* **python-dotenv**
* **LangSmith**
* **uv / pyproject.toml**

---

## 🔑 Key Concepts Demonstrated

### 1. LangGraph StateGraph

The project uses `StateGraph` to define the agent workflow.

The graph contains two main nodes:

```text
call_llm_model
tools
```

The graph starts at the LLM node and can conditionally move to the tool node.

---

### 2. Agent State

The agent maintains its conversation using a typed state:

```python
class State(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]
```

The `messages` field stores the conversation history.

`add_messages` allows new messages to be added to the existing state rather than replacing the previous messages.

---

### 3. Custom Tool

The project defines a simple addition tool:

```python
@tool
def add(a: float, b: float):
    """Add two number"""
    return a + b
```

The `@tool` decorator allows LangChain/LangGraph to expose the Python function as an LLM-callable tool.

For example, if the user asks:

```text
What is 25 + 17?
```

the LLM can decide to call the `add` tool with:

```text
a = 25
b = 17
```

The tool returns:

```text
42
```

The result is then passed back to the LLM.

---

## 🔄 Tool Calling Workflow

The important part of the project is the interaction between the LLM and the tool.

```text
User Request
     ↓
ChatOpenAI
     ↓
Does the request require a tool?
     ↓
   Yes
     ↓
ToolNode
     ↓
add(a, b)
     ↓
Tool Result
     ↓
ChatOpenAI
     ↓
Final Response
```

This creates the basic foundation of an **agentic workflow**.

---

## 🕸️ LangGraph Workflow

The graph is constructed using:

```python
builder = StateGraph(State)
```

The LLM node is added:

```python
builder.add_node("call_llm_model", call_llm_model)
```

The tool node is added:

```python
builder.add_node("tools", tool_node)
```

The graph starts with the LLM:

```python
builder.add_edge(START, "call_llm_model")
```

The conditional routing is handled using:

```python
builder.add_conditional_edges(
    "call_llm_model",
    tools_condition
)
```

Finally, the tool sends its result back to the LLM:

```python
builder.add_edge("tools", "call_llm_model")
```

This creates the loop:

```text
LLM → Tool → LLM
```

---

## 📊 LangSmith Tracing

The project also uses **LangSmith** for tracing and observing LLM execution.

Environment variables are loaded using:

```python
from dotenv import load_dotenv

load_dotenv()
```

LangSmith tracing is enabled through environment configuration.

This makes it possible to inspect the agent execution and understand:

* LLM calls
* Tool calls
* Execution flow
* Inputs and outputs
* Agent behavior

LangSmith is particularly useful when debugging more complex LangGraph agents.

---

## ⚙️ Environment Variables

Create a `.env` file in the project directory.

Example:

```text
OPENAI_API_KEY=your_openai_api_key
LANGSMITH_API_KEY=your_langsmith_api_key
```

**Do not upload the `.env` file to GitHub.**

Your `.gitignore` should contain:

```text
.env
.venv/
__pycache__/
```

API keys and other secrets should always remain private.

---

## 📦 Installation

Clone the repository:

```bash
git clone https://github.com/srikar-maddala/langgraph-tool-calling-agent.git
```

Move into the project directory:

```bash
cd langgraph-tool-calling-agent
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

---

## ▶️ Running the Project

Make sure your `.env` file contains the required API keys.

Then run the Python application:

```bash
python Agent.py
```

The exact execution command may vary depending on the final project structure.

---

## 🧪 Example Concept

For a request such as:

```text
Add 15 and 27.
```

the agent can follow this process:

```text
User
 ↓
LLM
 ↓
Recognizes that addition is required
 ↓
Calls add(15, 27)
 ↓
Tool returns 42
 ↓
LLM receives tool result
 ↓
Final response: 42
```

The important learning outcome is not the addition itself, but understanding **how an LLM can select and execute a tool through a graph-based agent workflow**.

---

## 📚 What I Learned

Through this project, I practiced:

* Building agents with LangGraph
* Creating a `StateGraph`
* Managing agent state
* Working with message-based state
* Creating custom LangChain tools
* Binding tools to an LLM
* Using `ToolNode`
* Conditional tool routing
* Building LLM → Tool → LLM workflows
* Using OpenAI models with LangChain
* Using environment variables securely
* LangSmith tracing and observability
* Understanding the foundation of agentic AI systems

---

## 🚀 Future Improvements

Possible extensions include:

* Add multiple tools
* Add weather/search tools
* Add MCP-based tools
* Add structured tool outputs
* Add memory
* Add human-in-the-loop workflows
* Add LangGraph persistence
* Add LangGraph Studio
* Add FastAPI as an API layer
* Add evaluation of agent responses
* Containerize the agent using Docker
* Deploy the agent to the cloud
* Build a more advanced Agentic RAG system

---

## 🎯 Project Purpose

This project is part of my learning journey toward **AI Engineering and Agentic AI development**.

The main goal is to understand how modern LLM applications move beyond simple prompt → response interactions by allowing models to:

```text
Reason about a task
      ↓
Select a tool
      ↓
Execute the tool
      ↓
Receive the result
      ↓
Continue the workflow
      ↓
Generate a response
```

This forms a fundamental building block for more advanced **AI agents, MCP applications, RAG systems, and LLM-based automation**.

---

## 👨‍💻 Author

**Srikar Maddala**

MSc Artificial Intelligence & Robotics
Hochschule Hof, Germany

### Areas of Interest

* Artificial Intelligence
* Machine Learning
* Generative AI
* LLM Applications
* Agentic AI
* LangGraph
* MCP
* AI Engineering
* Backend Development
* MLOps / LLMOps

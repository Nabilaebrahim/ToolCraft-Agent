# ToolCraft Agent: LangGraph-Powered Tool-Calling AI Agent

ToolCraft Agent is an autonomous, tool-augmented conversational AI agent built using **LangGraph**, **LangChain**, and **Groq Cloud API** (`ChatGroq`). The agent dynamically determines whether to respond directly using LLM knowledge or leverage specialized external tools (such as Wikipedia and arXiv) through conditional graph execution before generating accurate, well-grounded answers.

---

## 📌 Project Overview & Objectives

Traditional Large Language Models (LLMs) often hallucinate factual details or lack access to up-to-date domain knowledge. The goal of **ToolCraft Agent** is to create a modular, reactive agent workflow that:
- Maintains conversational state across interactions using message reducers.
- Evaluates incoming queries and identifies whether tool retrieval is required.
- Dynamically executes external tools (e.g., Wikipedia search, arXiv scholarly paper lookup).
- Routes tool responses back to the model for final synthesis and response formulation.
- Runs with low latency by leveraging Groq's LPU acceleration infrastructure.

---

## 🏗️ Architecture & Core Concepts

The workflow is constructed as a state machine using **LangGraph**:






### Components:
1. **Agent State (`State`)**:
   A `TypedDict` containing a `messages` key annotated with `add_messages` from `langgraph.graph.message`, ensuring new messages (user inputs, AI tool calls, tool responses) append smoothly without overwriting conversation history.
2. **LLM Node (`chatbot`)**:
   Invokes Groq LLM (e.g., `qwen/qwen3.8-27b`) bound with available tools using `.bind_tools(tools)`.
3. **Tools Node (`tools`)**:
   Prebuilt `ToolNode` executing the invoked tool functions and returning tool output messages back to the graph.
4. **Conditional Routing (`tools_condition`)**:
   Inspects the output of the `chatbot` node. If the model generates a `tool_calls` request, execution branches to `tools`; otherwise, it finishes at `END`.
5. **Cyclic Feedback Edge**:
   An edge from `tools` back to `chatbot` allows the model to ingest tool findings and generate the final user-facing response (or chain further tool calls if needed).

---

## 🛠️ Tools & Technologies Used

- **Frameworks & Libraries**:
  - `langgraph`: Orchestration of cyclic, stateful multi-actor agent workflows.
  - `langchain` / `langchain_core` / `langchain_community`: Agent abstractions, wrappers, and tool connectors.
  - `langchain_groq` & `groq`: Ultra-fast inference with Groq Cloud models.
  - `wikipedia`: Wikipedia API client for encyclopedic queries.
  - `arxiv`: arXiv API client for scholarly scientific literature searches.
- **Hardware & Environment**:
  - Google Colab (Python 3.13 / GPU T4 / Secrets Manager).

---

## 🔑 How to Get and Configure Your API Key

The agent uses **Groq Cloud** for fast LLM inference (and/or Google Cloud / external services). Follow the step-by-step guide below:

### Step 1: Obtain the API Key from Groq Cloud Console
1. Navigate to [Groq Console](https://console.groq.com/).
2. Sign in or create a new account (you can sign in with Google or GitHub).
3. In the left sidebar, click on **API Keys**.
4. Click **Create API Key**.
5. Give your key an identifiable name (e.g., `toolcraft-agent-key`) and click **Submit**.
6. **Copy and save** the generated key immediately (it will not be shown again).

*(Note: If connecting to Google Cloud APIs or Vertex AI, generate an API key via [Google Cloud Console](https://console.cloud.google.com/) -> **APIs & Services** -> **Credentials** -> **Create Credentials** -> **API Key**).*

### Step 2: Store the Key Securely in Google Colab (Secrets Manager)
To follow security best practices, never hardcode API keys into notebook cells:

1. Open your notebook in **Google Colab**.
2. On the left sidebar, click the **Secrets** icon (🔑 *Key icon*).
3. Click **Add new secret**.
4. Enter the Name: `GROQ_API_KEY`
5. Enter the Value: Paste your secret API key.
6. Enable the toggle **Notebook access** to allow the notebook to read the secret.

```
+-----------------------------------------------------------+
| 🔑 Secrets (Google Colab Left Sidebar)                    |
| Name: GROQ_API_KEY                                        |
| Value: gsk_**************************************         |
| [X] Notebook access (Enabled)                             |
+-----------------------------------------------------------+
```

### Step 3: Accessing the Key in Code
In the notebook cells, retrieve the key securely via `google.colab.userdata`:

```python
from google.colab import userdata

groq_api_key = userdata.get("GROQ_API_KEY")
```



---

## 🏆 Accomplishments So Far

- [x] Configured modern Python environment with LangGraph, LangChain, and Groq SDK.
- [x] Integrated and verified external API tools (`WikipediaQueryRun` & `ArxivQueryRun`).
- [x] Handled Wikimedia user-agent compliance and rate limits.
- [x] Built stateful message-passing schema using `TypedDict` and `add_messages`.
- [x] Configured secure credential access in Google Colab with `userdata.get("GROQ_API_KEY")`.
- [x] Implemented conditional branching logic (`tools_condition`) for autonomous tool selection.
- [x] Validated execution: tested general conversational queries (no tools invoked) and factual queries like *"what is RLHF."* (Wikipedia tool invoked and synthesized successfully).

---

## 🔮 Future Enhancements & Roadmap

- **Multi-Tool Binding**: Enable simultaneous tool access across arXiv, DuckDuckGo web search, Python REPL, and custom calculators.
- **Memory & Checkpointing**: Integrate `MemorySaver` or persistent database checkpoints (`SqliteSaver`) for continuous multi-turn dialogue memory.
- **Human-in-the-Loop (HITL)**: Introduce breakpoint conditions before executing sensitive tools.
- **Streaming UI**: Deploy an interactive frontend using Streamlit or Chainlit.

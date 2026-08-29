# 🤖 Agentic AI — Multi-Agent System Project — Revision & Interview Notes

> Condensed from: *"Agentic AI - Multi Model Agent Course | LangGraph | LangChain | Elasticsearch Full Stack Project"*

---

## 📌 Table of Contents
1. [Project Overview](#1-project-overview)
2. [High-Level Architecture](#2-high-level-architecture)
3. [Orchestration Layer Explained](#3-orchestration-layer-explained)
4. [Tech Stack & Setup](#4-tech-stack--setup)
5. [Choosing the Right LLM Model](#5-choosing-the-right-llm-model)
6. [Backend Folder Structure](#6-backend-folder-structure)
7. [Key Python/Pydantic Concepts Used](#7-key-pythonpydantic-concepts-used)
8. [Redis — Memory & Conversation State](#8-redis--memory--conversation-state)
9. [Elasticsearch — Data Ingestion & Search](#9-elasticsearch--data-ingestion--search)
10. [Auth, JWT & Logging](#10-auth-jwt--logging)
11. [LangChain vs LangGraph](#11-langchain-vs-langgraph)
12. [Core Agentic Concepts: State, Nodes, Edges](#12-core-agentic-concepts-state-nodes-edges)
13. [The Three Agents: Search, Summary, Supervisor](#13-the-three-agents-search-summary-supervisor)
14. [Full Workflow Execution Trace](#14-full-workflow-execution-trace)
15. [LLM-Based Routing vs Keyword-Based Routing](#15-llm-based-routing-vs-keyword-based-routing)
16. [PDF Ingestion (Bonus Feature)](#16-pdf-ingestion-bonus-feature)
17. [Common Issues Encountered (Debugging Log)](#17-common-issues-encountered-debugging-log)
18. [Quick Revision — Interview Q&A](#18-quick-revision--interview-qa)

---

## 1. Project Overview

- Goal: build a **multi-model, multi-agent Agentic AI system** using **LangChain, LangGraph, Python, Pydantic, Redis, and Elasticsearch**.
- Since most learners don't have GPUs to run models locally, the project uses **Hugging Face's free Inference API** instead of pulling models directly (which can require 8–16GB+ RAM).
- Described as **a "to-do app" for AI learning** — intentionally simple, not solving a real business problem, but teaching core Agentic AI mechanics (nodes, edges, state, orchestration) before moving to advanced projects (RAG, MCP agent communication) in future videos.
- **Redis** → conversation context / state storage. **Elasticsearch** → ingested application-specific data that the LLM doesn't already know, fetched and passed to the model for answering.

---

## 2. High-Level Architecture

```
Frontend (Next.js) --HTTPS + JWT--> API Gateway (FastAPI)
                                        │
                     Middleware: CORS, rate limiting, logging, LangFuse tracing
                                        │
                Routers: health, auth (register/login), chat, CSV ingest
                                        │
                    Service Layer: Auth Service, Token Service, LLM Service, Search Service
                                        │
        ┌───────────────────────────────┴───────────────────────────────┐
        │                                                                │
   Storage Layer                                                  LLM Service (3 Agents)
   - Redis (conversation state)                              - Supervisor Agent
   - Elasticsearch (app data)                                 - Summary Agent
                                                               - Search Agent
```

- **Supervisor agent** decides, based on the query, which agent(s) to call: **search only**, **summary only**, or **both in parallel** (e.g., if asked to summarize something it hasn't fetched yet, it first calls search, then feeds that data to summary).
- **LangFuse** is used for tracing/monitoring — visualizing nodes, edges, arguments passed, and LLM responses.

---

## 3. Orchestration Layer Explained

- **Orchestration** = like a musical **orchestra conductor** — syncing multiple "musicians" (agents) to produce one coherent output.
- **Steps in orchestration:**
  1. **Query Understanding** — parse what the user is asking and its context.
  2. **Query Transformation** — rewrite the query using prior context (e.g., "tell me benefits of that" → resolved using previous turn about LangFuse → "tell me benefits of LangFuse").
  3. **Data Retrieval** — pull relevant documents from Elasticsearch if needed.
  4. **Answer Generation** — LLM processes retrieved data + query and produces a scored answer (higher score = more relevant).
- **Orchestration components:** Query Processor, Router, Context Builder, Prompt Builder (defines each agent's task/instructions), Response Formatter.
- **Data flow:** `User query → (optional) document retrieval → LLM prompting with context → final answer`.

---

## 4. Tech Stack & Setup

| Tool | Purpose |
|---|---|
| **Python 3.14+** | Backend language |
| **FastAPI + Uvicorn** | Backend API server (ASGI) |
| **Pydantic** | Data validation & schema management using type hints |
| **LangChain** | Framework for working with LLMs (abstracts away provider-specific API details) |
| **LangGraph** | Workflow orchestration — built **on top of** LangChain, enables stateful multi-agent workflows |
| **LangFuse** | LLM call tracing, token usage monitoring, observability |
| **Redis** | Conversation context / state storage (via a container) |
| **Elasticsearch** | Full-text search engine for ingested application data |
| **Podman Desktop** | Container management (alternative to Docker Desktop) |
| **Hugging Face Inference API** | Free-tier access to LLMs without needing local GPUs |
| **Next.js** | Frontend (used only for testing backend services, not coded in this video) |
| **PyJWT, passlib** | JWT auth tokens and password hashing |

**Setup steps:**
1. Install Python, Node.js, Podman Desktop.
2. Create a Hugging Face account → Settings → Access Tokens → create token with **inference-only permissions**.
3. Use `podman-compose.yml` to define and spin up Redis + Elasticsearch containers (`podman compose -f podman-compose.yml up -d`).
4. Create Python virtual environment (`python -m venv .venv`), activate it, install dependencies via `requirements.txt` (`pip install -r requirements.txt`).
5. Configure `.env` with app settings, Hugging Face API key, model-per-task assignments, Redis/Elasticsearch config, LangFuse keys, and JWT secret.

---

## 5. Choosing the Right LLM Model

- **Not every model can be used freely** — each model has its own trainer-provided specification (what it's good at).
- **Why model choice matters:** higher accuracy, better performance, cost efficiency, **lower hallucination**, better user experience.
- **Hallucination:** when a model confidently produces an incorrect or non-existent output (e.g., assuming a function like "remove color" exists just because a similar function "remove background" exists). No model achieves **zero hallucination** — the best you can do is **minimize** it via correct model selection.
- **Example model-task mapping used in this project:**
  - **Question Answering / reasoning (general accuracy):** Meta LLaMA 3.2 Instruct
  - **Code generation:** Qwen
  - **Reasoning:** DeepSeek
  - **Summarization:** DeepSeek variant
- **Practical tip:** ask an AI tool (ChatGPT/Claude) to recommend the best model for your specific use case rather than manually reading every model's documentation.

---

## 6. Backend Folder Structure

```
app/
├── agents/           # Agent logic (search, summary, supervisor)
├── config/           # settings.py — loads all .env variables
├── data_ingest/       # CSV/PDF ingestion logic
├── decorators/        # Shared decorators (e.g., LangFuse observe wrapper)
├── dependencies/      # Shared auth dependencies (e.g., get_current_user)
├── logging_config.py  # Custom colored/JSON logger setup
├── memory/            # Redis memory service
├── middleware/         # CORS, rate limiting, security headers
├── models/             # Pydantic schemas (auth, chat, ingest models)
├── prompts/            # Centralized prompt templates (LLMPrompts class)
├── routers/            # API route definitions (auth, health, chat, ingest)
├── services/            # auth_service, token_service, llm_service, search_service
├── state/               # graph_state.py — the shared workflow state
├── utils/               # Utility functions (e.g., password hashing)
├── workflows/           # chat_workflow.py — the orchestration logic
└── main.py              # FastAPI app entrypoint
```

- Every folder needs an `__init__.py` file (even if empty) to be recognized as a **Python package** — this allows imports like `from app.utils import helpers`.

---

## 7. Key Python/Pydantic Concepts Used

- **Pydantic `BaseModel`:** used for data validation and settings management via Python type hints; auto-generates JSON schemas (`model.model_dump()` converts to dict/JSON).
- **`Field`:** adds validation constraints to model fields (e.g., minimum password length).
- **`Literal`:** restricts a field to a specific set of allowed values (e.g., `mode: Literal["fast", "slow"]` — anything else raises a type error).
- **`typing.List`:** used for type-checked arrays/lists (e.g., `List[ChatMessage]`).
- **`dataclass` (from `dataclasses`):** used for `GraphState` — a lighter-weight alternative to Pydantic for defining the shared workflow state.
- **f-strings:** Python 3.6+ string interpolation (e.g., `f"conversation:{conversation_id}"`).
- **`importlib.import_module`:** dynamically imports a module at runtime (used for lazy-loading Redis/Elasticsearch clients only if needed).
- **Async functions (`async def`):** used throughout agents/services since LLM/API calls can take time; must be `await`-ed properly.
- **Decorators (`@observe`):** wrap functions for LangFuse tracing — conditionally imported only if LangFuse is enabled (to avoid performance overhead when disabled).

---

## 8. Redis — Memory & Conversation State

- **`RedisMemoryService` class** manages:
  - `get_messages(conversation_id)` — uses Redis's **`LRANGE`** command to fetch a conversation's message history (JSON-decoded).
  - `append_message(conversation_id, role, content)` — serializes a message payload to JSON and stores it, with a **TTL (time-to-live)** for automatic expiry.
  - `set_value` / `get_value` — general key-value storage (e.g., storing registered user accounts with **TTL = -1**, meaning **no expiry** — so users don't need to re-register).
  - `clear_messages` — deletes a conversation's history.
  - `is_available` (property) — health check returning whether the Redis client initialized successfully.
- **Why Redis here:** used as a lightweight substitute for a full database (no SQL/Mongo needed) since the project is a learning exercise — conversation state and user records are stored directly as Redis key-value/JSON entries.

---

## 9. Elasticsearch — Data Ingestion & Search

- **Indexing analogy:** like a book's index page — instead of scanning every page for "Python," you jump directly to the indexed page number. Elasticsearch does this for text data — it's **"a search engine database optimized for full-text search... like a Google for your application data."**
- **`ensure_index()`** — creates an index (like a database table) with fields marked as `text` (analyzed for full-text search) or `keyword` (stored as-is for exact matching/filtering).
- **`bulk_index_documents()`** — ingests many documents at once via Elasticsearch's `bulk` API.
- **`search(query)`** — uses a **`multi_match`** query across title/snippet/category fields, returns top N (e.g., 5) results ranked by relevance **score**.
- **CSV ingestion:** reads a CSV (`csv.DictReader`) and converts rows into documents for indexing.
- **PDF ingestion (added later):** uses `PyPDF2` to extract text page-by-page, tagging each chunk with page number and file name metadata, then indexes it the same way.
- **File-type detection:** a generic `load_documents_from_file()` dispatches to the correct loader (CSV vs PDF) based on file extension; `load_documents_from_directory()` can recursively ingest a whole folder.

---

## 10. Auth, JWT & Logging

- **Auth flow:** `register` → hash password (via `passlib`'s `CryptContext`) → store in Redis (no expiry) → `login` → verify password → issue a **JWT access token** (`PyJWT`, with configurable secret, algorithm, and expiry minutes).
- **`get_current_user` dependency:** extracts and decodes the JWT from the `Authorization` header to protect routes (e.g., data ingestion should not be open to anyone).
- **Custom logging (`logging_config.py`):**
  - **`JSONFormatter`** — structures logs as JSON (timestamp, agent type, session ID, route, etc.).
  - **`ColoredFormatter`** — adds color-coded console output for readability.
  - **`RotatingFileHandler`** — writes logs to a file, rotating when size exceeds a max byte threshold, keeping backups.
  - **Context vars (`log_context`)** — track session ID / user ID / route across a request so logs are traceable end-to-end.

---

## 11. LangChain vs LangGraph

| LangChain | LangGraph |
|---|---|
| Tool for working with LLMs — abstracts away provider-specific API details | Enables **stateful, complex workflows** with multiple agents |
| Gives flexibility to switch between LLM providers easily | Built **on top of** LangChain (installing LangGraph auto-installs LangChain) |
| Provides composability (chaining agents/steps together) | Provides a shared **graph state** across agents instead of manually passing variables around |
| Provides built-in conversation memory | Provides **checkpoints** and **human-in-the-loop** features (covered in future videos) |

- **Analogy:** LangChain = **LEGO building blocks** (build architecture piece by piece); LangGraph = **a blueprint** for building much larger, complex multi-agent AI workflows.
- **Evolution of the field (as described):** Prompt AI (direct SDK calls) → Prompt Engineering → Generative AI → **Agentic AI** (LLMs acting as autonomous agents that can build/manage entire workflows, e.g., an "AI IT team").

---

## 12. Core Agentic Concepts: State, Nodes, Edges

- **State:** the data that flows through the graph — like **shared memory** all nodes can read from and write to. It is **mutable** and **shared** across nodes.
- **Node:** an individual step in the workflow (an agent, a specific task, an LLM call, a database query, etc.). Receives the current state, performs an action, and can **update and return** the modified state.
- **Edge:** connects nodes and defines execution flow.
  - **Normal edge:** simple sequential flow (Node A → Node B).
  - **Conditional edge:** the supervisor decides dynamically which node/agent to route to next based on the query.
- **Example state evolution:**
  ```
  Initial: {conversation_id, user_message, route: null, summary_output: null}
  → Supervisor node updates route = "summary"
  → Summary node updates summary_output
  → Final node updates final_answer
  ```

---

## 13. The Three Agents: Search, Summary, Supervisor

### Search Agent
- Calls `SearchService.search()` (Elasticsearch) with the user's query.
- Formats top results (title, snippet, score) into a readable block of text.
- Returns an `AgentResult` with `agent="search"`, formatted output, and metadata (result count, index name).

### Summary Agent
- Calls `LLMService.summarize(text, context)` using a dedicated prompt template.
- Uses conversation context (previous messages) to produce a coherent, context-aware summary.
- Returns an `AgentResult` with `agent="summary"`.

### Supervisor Agent
- Decides the **route**: `"search"`, `"summary"`, `"greeting"`, or `"parallel"` (both search + summary).
- Two routing strategies (see section 15): **keyword-based** (simple `if`/`elif` checks) and **LLM-based** (asks the LLM itself to output a routing decision, more robust for ambiguous queries).
- Greeting messages (e.g., "hi", "hello") are handled with a **hardcoded constant reply** — to avoid wasting an LLM call/cost on trivial greetings.

**LLM Service** (shared by agents):
- Wraps the **Hugging Face Router** (OpenAI-SDK-compatible client) to call different models per task capability: `SUMMARIZATION`, `CODE_GENERATION`, `QUESTION_ANSWERING`, `REASONING`.
- `get_model_for_capability()` picks the configured (or default) model for a given task.
- `generate()` is the core method that calls `client.chat.completions.create(...)`.
- Centralized prompt templates live in `LLMPrompts` (a class of static methods: `summarization()`, `code_generation()`, `question_answering()`, `reasoning()`, `chat_summary()`, `routing_decision()`), keeping prompts reusable and easy to maintain.

---

## 14. Full Workflow Execution Trace

Example: user asks *"What are the best AI tools for data analysis?"*

1. `ChatWorkflow.run()` starts — generates (or reuses) a **conversation ID**.
2. Checks **Redis cache** for an identical previous question — if cached, returns immediately (saves LLM cost).
3. Initializes the **graph state**.
4. **Supervisor agent** analyzes the query → decides route = **parallel** (needs both retrieval and synthesis).
5. **Search agent** queries Elasticsearch → returns matched documents.
6. **Summary agent** is called with the search results as context → produces a synthesized answer.
7. Final combined answer is generated, **stored back in Redis** (for caching/context), and returned to the user as a `ChatResponse` (conversation ID, route used, answer, agent, cached flag, context message count).

**Other possible routes:** direct to `search` only (e.g., "find details about LangChain"), direct to `summary` only, or `greeting` (hardcoded reply, no LLM call).

---

## 15. LLM-Based Routing vs Keyword-Based Routing

- **Keyword-based routing (simple):** checks if the lowercased user prompt contains specific keywords (e.g., "search", "find" → search route; "hi"/"hello" → greeting route). Fast but rigid.
- **LLM-based routing (more robust):** sends the query to the LLM itself (using the `question_answering` model) with a **routing decision prompt**, asking it to output **only one word** representing the correct route.
  - Configured with **very low max tokens (e.g., 10–20)** and **temperature 0.0** for consistent, deterministic output.
  - **Fallback:** if the LLM's raw output isn't a recognized valid route (hallucination risk), the system defaults back to **keyword-based routing** or a default **"parallel"** route.
- **Debugging lesson from the video:** even simple one-word routing decisions can hallucinate (e.g., LLM returning an unexpected string instead of a valid route) — reinforcing why **prompt engineering** (being very explicit: *"give only one output"*) and **fallback logic** are essential in production agentic systems.

---

## 16. PDF Ingestion (Bonus Feature)

- Added using `PyPDF2` + `python-multipart` (for file uploads in FastAPI).
- **`load_documents_from_pdf()`** — extracts text page-by-page, storing page number + file name metadata alongside content, so answers can cite their **source** (file + page).
- **`load_documents_from_file()`** — detects file type (PDF vs CSV) and dispatches to the correct loader.
- **`load_documents_from_directory()`** — recursively scans a directory for matching files (using glob patterns) and ingests them all in a batch.
- New endpoint: `POST /api/v1/ingest/batch` — accepts an `UploadFile`, writes it to a temp file, loads + indexes it into Elasticsearch, then cleans up the temp file.
- Search results were updated to include `page_number` and `file_name` so the chat response can show **where** an answer's source information came from.

---

## 17. Common Issues Encountered (Debugging Log)

*(useful for understanding real-world debugging patterns in agentic systems)*

- **Missing `__init__.py`** → module not recognized as a Python package → import errors.
- **Spelling mistakes** in service/class names → `ImportError: cannot import name X`.
- **Forgetting `await`** on an async function call (e.g., `state.load(...)`) → runtime errors.
- **Model not supported by the configured provider** → had to switch Hugging Face **provider** (e.g., to "Featherless AI") for a given model/task in `.env`.
- **LLM routing hallucination** → the routing LLM sometimes returned unexpected text instead of a clean route keyword → fixed via **stricter prompt wording**, **low temperature**, **small max tokens**, and a **fallback to keyword-based routing**.
- **General takeaway:** understanding *why* code works (not just copy-pasting) is what lets you debug issues quickly — even when using AI tools to generate code, a developer must be able to trace and fix failures.

---

## 18. Quick Revision — Interview Q&A

**Q: Why use Hugging Face's free Inference API instead of running models locally?**
A: Running LLMs locally requires significant GPU/RAM resources (8–16GB+); the free Inference API lets you make API calls to hosted models without needing your own hardware.

**Q: What is the difference between LangChain and LangGraph?**
A: LangChain is a general framework for working with LLMs (provider abstraction, chaining, memory). LangGraph is built on top of LangChain and enables **stateful, multi-agent workflows** with a shared graph state, nodes, and edges — better suited for complex agent orchestration.

**Q: What is a "state" in LangGraph?**
A: The data that flows through the graph — a shared, mutable memory that all nodes can read from and write to, carrying information between steps.

**Q: What's the difference between a node and an edge?**
A: A node is an individual workflow step (e.g., an agent or task) that processes and updates the state. An edge connects nodes and defines execution flow — either a fixed sequential path or a conditional path decided dynamically (e.g., by a supervisor agent).

**Q: What role does the Supervisor agent play?**
A: It analyzes the user's query and decides the routing — whether to call the search agent, the summary agent, both in parallel, or return a canned greeting response — orchestrating the other agents.

**Q: Why use Redis in this project?**
A: To store conversational context/state (message history per conversation ID) and cache identical queries, avoiding redundant LLM calls and reducing cost.

**Q: Why use Elasticsearch here?**
A: To store and full-text search application-specific data that the LLM doesn't inherently know, retrieving relevant documents to feed as context to the LLM (a lightweight form of retrieval).

**Q: What is model hallucination, and can it be fully eliminated?**
A: Hallucination is when a model confidently generates an incorrect or non-existent output. It cannot be fully eliminated — only minimized through better model selection, prompt engineering, and fallback mechanisms.

**Q: Why did the project implement both LLM-based and keyword-based routing?**
A: LLM-based routing is more flexible/robust for natural language queries, but LLMs can occasionally hallucinate an invalid route; keyword-based routing serves as a deterministic fallback to guarantee the workflow doesn't break.

**Q: What is the purpose of the `__init__.py` file in a Python folder?**
A: It tells Python that the directory should be treated as an importable package, enabling imports like `from app.utils import helpers`.

**Q: Why is Pydantic used extensively in this project?**
A: For data validation and settings management via Python type hints — ensuring only correctly-typed data flows through API requests/responses and configuration, with automatic JSON schema generation.

**Q: What does LangFuse provide in this system?**
A: Tracing and observability for LLM calls — visualizing node/edge execution, arguments passed, token usage, and responses, useful for debugging and monitoring agent behavior.

---

*Made for quick revision before interviews/exams — condensed from a full-stack Agentic AI build-along covering FastAPI, Redis, Elasticsearch, LangChain/LangGraph, multi-agent orchestration, and LLM routing.*
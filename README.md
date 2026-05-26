# 🤖 MCP-Powered GitHub Agent
An intelligent agent built using the Model Context Protocol (MCP) that connects to an external service (e.g., a DB/ API) to fetch, process, or act on data in real time. MCP provides the agent with structured context, memory, and tool-use capabilities, enabling it to understand user intents, call external endpoints, and return meaningful results.
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/e05940ff-6314-4294-aab1-b74c9c5a0ae9" />

> A LangGraph AI agent connected to GitHub via the **Model Context Protocol (MCP)** — ask natural language questions about any GitHub repository and get real-time, structured answers.

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.1-FF6B35?style=flat-square)](https://langchain-ai.github.io/langgraph)
[![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-6B21A8?style=flat-square)](https://modelcontextprotocol.io)
[![GitHub API](https://img.shields.io/badge/GitHub-API%20v3-181717?style=flat-square&logo=github&logoColor=white)](https://docs.github.com/en/rest)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=flat-square&logo=openai&logoColor=white)](https://openai.com)
[![MongoDB](https://img.shields.io/badge/MongoDB-Optional-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://mongodb.com)

**🌐 Live Demo:** `http://<your-ec2-public-ip>:8000`  
**📖 API Docs:** `http://<your-ec2-public-ip>:8000/docs`  
**🎥 Demo Video:** [Watch 3-min Loom walkthrough](#)

---

## 📌 What It Does

Ask any natural language question about a GitHub user or repository — the LangGraph agent reasons through the query, selects the right MCP tools, calls the GitHub API via the MCP Server, and returns a clear, structured answer.

```
"What are the open issues in my most starred repository?"
        ↓
  LangGraph Agent reasons → selects tools → MCP Client sends requests
        ↓
  MCP Server calls GitHub API → returns structured data
        ↓
  LLM generates natural language answer → streamed to user
```

| Feature | Details |
|---|---|
| 🔗 MCP Protocol | Standardized tool integration between agent and GitHub API |
| 🤖 LangGraph Agent | Multi-step reasoning, tool selection, orchestration |
| 🐙 GitHub Tools | `get_user_repos`, `get_repo_issues`, `get_pull_requests`, `create_issue` |
| ⚡ Streaming | Token-by-token SSE streaming via FastAPI |
| 🔒 Auth | GitHub Personal Access Token via MCP Server |
| 🔄 Reusable | MCP Server tools work with any MCP-compatible agent |
| 💾 Persistence | Optional MongoDB for conversation history |
| 🐳 Containerized | Docker + Docker Compose, production-ready |

---

## 🏗️ Architecture

![MCP-Powered Agent Project — Complete End-to-End Flow](./assets/MCP.png)

The system has 4 main components:

**1. User** — Sends a natural language query via the web interface or API.

**2. MCP Client (AI Agent)**
- **LangGraph Agent** — understands the query, plans required steps, selects tools, orchestrates execution
- **MCP Client** — connects to the MCP Server, sends tool requests, receives responses, returns results to agent
- **LLM (OpenAI GPT-4o)** — analyzes tool results and generates the final natural language answer

**3. MCP Server (GitHub API Wrapper)**
- Exposes tools following the MCP protocol standard
- Validates incoming requests from the MCP Client
- Executes the corresponding GitHub API calls
- Returns structured responses back to the agent

**Available Tools exposed via MCP:**
| Tool | Description |
|---|---|
| `get_user_repos` | Get all repositories of the authenticated GitHub user |
| `get_repo_issues` | Get issues of a repository (filter by state: open/closed) |
| `get_pull_requests` | Get pull requests of a repository |
| `create_issue` | Create a new issue in a repository |

**4. GitHub API** — Repositories, Issues, Pull Requests, Users, and more.

---

## 🔄 Step-by-Step Execution Flow

Using the example query: *"What are the open issues in my most starred repository?"*

```
Step 1  →  User sends query to LangGraph Agent

Step 2  →  Agent plans: (1) get user repos, (2) find most starred, (3) get open issues

Step 3  →  MCP Client calls tool: get_user_repos

Step 4  →  MCP Server executes GET /user/repos on GitHub API → returns repo list

Step 5  →  Agent analyzes list, selects repo with most stars

Step 6  →  MCP Client calls tool: get_repo_issues (state=open)

Step 7  →  MCP Server executes GET /repos/{owner}/{repo}/issues → returns open issues

Step 8  →  MCP Client returns structured issues data to agent

Step 9  →  Agent analyzes issues, prepares context for final answer

Step 10 →  LLM converts structured results into natural language response

Step 11 →  Final answer streamed token-by-token to user
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Agent Framework** | LangGraph (StateGraph + MemorySaver + conditional edges) |
| **Protocol** | MCP — Model Context Protocol |
| **LLM** | OpenAI GPT-4o / Gemini |
| **API** | FastAPI, Pydantic v2, Uvicorn |
| **External Service** | GitHub REST API v3 |
| **Auth** | GitHub Personal Access Token |
| **Database** | MongoDB (optional — conversation history) |
| **Containerization** | Docker, Docker Compose |

---

## 📁 Project Structure

```
mcp-github-agent/
│
├── mcp_server/                      # MCP Server — GitHub API wrapper
│   ├── server.py                    # MCP server entrypoint, tool registration
│   ├── tools/
│   │   ├── get_user_repos.py        # Tool: list all user repositories
│   │   ├── get_repo_issues.py       # Tool: get issues (open/closed filter)
│   │   ├── get_pull_requests.py     # Tool: get pull requests
│   │   └── create_issue.py          # Tool: create a new issue
│   ├── github_client.py             # GitHub API HTTP client (httpx)
│   └── auth.py                      # GitHub PAT authentication handler
│
├── mcp_client/                      # MCP Client — agent side
│   ├── client.py                    # MCP Client connection + tool invocation
│   └── tool_schemas.py              # JSON schemas for all MCP tools
│
├── agent/                           # LangGraph Agent
│   ├── state.py                     # AgentState TypedDict
│   ├── nodes.py                     # agent_node, tool_executor_node
│   ├── edges.py                     # Conditional routing logic
│   ├── tools.py                     # Tool wrappers calling MCP Client
│   └── graph.py                     # StateGraph assembly + checkpointing
│
├── app/                             # FastAPI application
│   ├── main.py                      # App entrypoint, middleware, routers
│   ├── routers/
│   │   ├── query.py                 # POST /query (SSE streaming)
│   │   └── health.py                # GET /health
│   └── schemas.py                   # Pydantic request/response models
│
├── db/                              # Optional MongoDB persistence
│   ├── client.py                    # Motor async client
│   └── conversations.py             # Store queries, tool calls, answers
│
├── tests/
│   ├── test_mcp_server.py
│   ├── test_agent.py
│   └── test_api.py
│
├── Dockerfile
├── docker-compose.yml               # FastAPI + MCP Server together
├── requirements.txt
└── .env.example
```

---

## 🚀 Quick Start

### Prerequisites
- Python 3.11+
- Docker & Docker Compose
- GitHub Personal Access Token ([create one here](https://github.com/settings/tokens))
- OpenAI API key

### 1. Clone and configure

```bash
git clone https://github.com/<your-username>/mcp-github-agent.git
cd mcp-github-agent

cp .env.example .env
# Add your GitHub PAT and OpenAI API key
```

### 2. Run with Docker

```bash
docker-compose up --build
```

API live at `http://localhost:8000`  
Docs at `http://localhost:8000/docs`

### 3. Run without Docker

```bash
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate

pip install -r requirements.txt

# Terminal 1 — start MCP Server
python mcp_server/server.py

# Terminal 2 — start FastAPI app
uvicorn app.main:app --reload --port 8000
```

---

## 🔑 Environment Variables

```bash
# .env.example

# LLM
OPENAI_API_KEY=sk-...

# GitHub
GITHUB_PAT=ghp_...               # GitHub Personal Access Token
GITHUB_USERNAME=your-github-username

# MCP Server
MCP_SERVER_HOST=localhost
MCP_SERVER_PORT=8001

# MongoDB (optional)
MONGODB_URI=mongodb+srv://...

# App
APP_API_KEY=your-secret-key
```

---

## 📡 API Reference

### Ask the Agent

```http
POST /query
Content-Type: application/json
X-API-Key: your-secret-key

{
  "question": "What are the open issues in my most starred repository?",
  "session_id": "session_abc"
}
```

**Streaming response (SSE):**
```
data: {"token": "Your", "done": false}
data: {"token": " most", "done": false}
...
data: {
  "answer": "Your most starred repository is 'ai-research-assistant' (⭐ 142 stars). It has 3 open issues: ...",
  "tools_used": ["get_user_repos", "get_repo_issues"],
  "steps": 4,
  "done": true
}
```

### Create a GitHub Issue

```http
POST /query
Content-Type: application/json

{
  "question": "Create an issue in my repo 'my-project' titled 'Add dark mode support' with label enhancement"
}
```

```json
{
  "answer": "Issue #47 'Add dark mode support' has been created in my-project.",
  "issue_url": "https://github.com/username/my-project/issues/47",
  "tools_used": ["create_issue"]
}
```

### Health Check

```http
GET /health
```
```json
{
  "status": "healthy",
  "mcp_server": "connected",
  "github_api": "authenticated",
  "uptime_seconds": 1800
}
```

---

## 💬 Example Queries

```
"What are the open issues in my most starred repository?"
→ Uses: get_user_repos → get_repo_issues

"List all pull requests in my repo called 'portfolio-website'"
→ Uses: get_pull_requests

"How many repositories do I have and which one has the most forks?"
→ Uses: get_user_repos (analyzes fork count)

"Create an issue in my-project titled 'Fix mobile layout bug'"
→ Uses: create_issue

"Show me closed issues from the last month in ai-research-assistant"
→ Uses: get_repo_issues (state=closed)
```

---

## 🔑 Why MCP Matters

MCP (Model Context Protocol) is Anthropic's open standard for connecting AI agents to external services. Instead of writing one-off API integrations per agent, MCP standardizes the interface — **any MCP-compatible agent can use any MCP server** without rewriting the integration logic.

| Without MCP | With MCP |
|---|---|
| Custom integration per agent | One server, any agent |
| Tightly coupled to one LLM | Works with GPT-4o, Claude, Gemini |
| Hard to extend with new tools | Add a tool in one place |
| No standard request/response | Standardized protocol |

---

## ✅ Key Benefits

- **Standardized tool integration** using MCP — no custom API wrappers per agent
- **Reusable tools** across different agents and LLM providers
- **Separation of concerns** — Agent logic lives in the MCP Client, API logic lives in the MCP Server
- **Easy to extend** — add a new GitHub tool in one file, available to all agents immediately
- **Production-ready** — Docker, FastAPI, auth, structured error handling

---

## 🌐 Common Use Cases

- 🐙 **GitHub Repository Assistant** — query repos, issues, PRs in natural language
- ⚙️ **DevOps Automation** — auto-create issues from monitoring alerts
- 📋 **Project Management Bot** — summarize open issues, tag PRs, generate status reports
- 📊 **Data Retrieval & Reporting** — extract GitHub metrics for dashboards
- 🔌 **Custom API Integrations** — swap GitHub for Notion, Jira, Slack by changing the MCP Server

---

## ⚠️ Known Limitations

- **GitHub rate limits:** Unauthenticated requests limited to 60/hour; PAT raises this to 5,000/hour
- **Repo scope:** Agent currently works within a single user's repositories — org-level support coming
- **Write operations:** `create_issue` is the only write tool; full CRUD coming in future iterations
- **Context window:** Very large repo lists (1,000+ repos) may need pagination handling

---

## 🔭 Future Improvements

- [ ] Add `close_issue`, `merge_pull_request`, `add_label` tools
- [ ] Support GitHub Organizations and team repositories
- [ ] Webhook listener — agent responds to GitHub events automatically
- [ ] Swap GitHub MCP Server for Notion / Jira / Linear MCP Server
- [ ] LangSmith tracing for full agent observability
- [ ] Frontend chat UI (Streamlit or Next.js)

---

## 📚 Resources

- [Model Context Protocol — Official Docs](https://modelcontextprotocol.io)
- [Anthropic MCP Announcement](https://www.anthropic.com/news/model-context-protocol)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph)
- [GitHub REST API v3 Docs](https://docs.github.com/en/rest)
- [CampusX MCP Playlist](#)

---

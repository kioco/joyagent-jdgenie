# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JoyAgent-JDGenie is an open-source, end-to-end multi-agent product for building AI assistants. It achieves 65.12% on GAIA Test set and 75.15% on Validation set, outperforming frameworks like Smolagents, Owl, and AutoAgent.

## Architecture

The system consists of four independent services that communicate via HTTP:

```
UI (React) --> genie-backend (Java) --> genie-tool (Python)
                                       |
                                  genie-client (Python MCP client)
```

### Services and Ports
- **UI**: React frontend on port 3000
- **genie-backend**: Java Spring Boot API on port 8080
- **genie-tool**: Python FastAPI tools service on port 1601
- **genie-client**: Python MCP client on port 8188

### Key Directories
- `ui/`: React 19 + TypeScript + Vite + TailwindCSS v4 frontend
- `genie-backend/src/main/java/com/jd/genie/`: Java Spring Boot backend
  - `agent/agent/`: Agent implementations (BaseAgent, ReActAgent, PlanningAgent)
  - `agent/tool/`: Tool implementations (CodeInterpreterTool, DeepSearchTool, etc.)
  - `service/impl/`: Agent execution services
- `genie-tool/genie_tool/`: Python tool service
  - `tool/`: Tool implementations (ci_agent.py, report.py, deepsearch.py, etc.)
  - `api/`: FastAPI routes
  - `db/`: SQLite database engine
- `genie-client/`: MCP client that proxies MCP server tools

## Development Commands

### Full Stack Start (one script)
```bash
sh Genie_start.sh
```
This runs dependency checks, builds backend, initializes DB, and starts all 4 services.

### Individual Service Commands

**Frontend (React/Vite)**:
```bash
cd ui && sh start.sh
```

**Backend (Java)**:
```bash
cd genie-backend && sh build.sh && sh start.sh
```

**Tool Service (Python)**:
```bash
cd genie-tool
uv sync                    # Install dependencies
source .venv/bin/activate  # Activate venv
uv run python server.py    # Start on port 1601
```

**MCP Client (Python)**:
```bash
cd genie-client
uv venv
source .venv/bin/activate
sh start.sh
```

### First-Time Database Init (genie-tool only needed once)
```bash
cd genie-tool && source .venv/bin/activate && python -m genie_tool.db.db_engine
```

## Agent Patterns

The backend implements multiple agent patterns in `genie-backend/src/main/java/com/jd/genie/agent/agent/`:
- **BaseAgent**: Abstract base with memory management, tool execution, concurrent execution via `executeTools()`
- **ReActAgent**: Extends BaseAgent with `think()` and `act()` pattern
- **PlanningAgent**: Multi-step planning with task decomposition
- **CIAgent** (in genie-tool): Extends smolagents `CodeAgent` for code generation and execution

## Adding Custom Tools

### MCP Tools
Add to `genie-backend/src/main/resources/application.yml`:
```yaml
mcp_server_url: "http://ip:port/sse"
```

### Java Backend Tools
1. Implement `BaseTool` interface:
```java
public interface BaseTool {
    String getName();
    String getDescription();
    Map<String, Object> toParams();
    Object execute(Object input);
}
```
2. Register in `GenieController#buildToolCollection`:
```java
toolCollection.addTool(new YourTool());
```

### Python Tool Service Tools
Add in `genie-tool/genie_tool/tool/` following existing patterns like `report.py`, `deepsearch.py`.

## Configuration Files

- `genie-backend/src/main/resources/application.yml`: LLM API keys, model settings, MCP server URLs
- `genie-tool/.env`: Tool service config (from `.env_template`)
  - `OPENAI_API_KEY`, `OPENAI_BASE_URL`, `DEFAULT_MODEL`
  - `SERPER_SEARCH_API_KEY` for search functionality
- `ui/.env`: Frontend → backend proxy path

## Key Dependencies

- **LLM Integration**: litellm (genie-tool), custom LLM class (genie-backend)
- **Agent Framework**: smolagents for CIAgent code execution
- **Search**: Serper, Jina, Bing, Sogou (configurable)
- **Database**: SQLite via sqlmodel in genie-tool
- **Vector Store**: Qdrant for table RAG
- **Elasticsearch**: ES7 client for table RAG

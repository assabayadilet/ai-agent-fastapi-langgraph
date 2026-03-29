# AI Agent Service — FastAPI + LangGraph + MCP

An AI agent that processes natural language queries and routes them to specialized tools via **LangGraph** and **MCP (Model Context Protocol)**. Built with FastAPI for the web layer, all containerized with Docker.

## Architecture

```
┌─────────────────────────────────────────────────┐
│             Docker Container                    │
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │         FastAPI App                       │  │
│  │  POST /api/v1/agent/query                 │  │
│  └─────────────────┬─────────────────────────┘  │
│                    │                             │
│                    ▼                             │
│  ┌───────────────────────────────────────────┐  │
│  │        LangGraph Agent                    │  │
│  │  Analyzes query → selects tools → runs    │  │
│  └───┬──────────────────────┬────────────────┘  │
│      │                      │                    │
│      ▼                      ▼                    │
│  ┌──────────┐   ┌──────────────────────────┐    │
│  │ Custom   │   │   MCP Server (stdio)     │    │
│  │ Tools    │   │  ├─ list_products        │    │
│  │ ├─ calc  │   │  ├─ get_product          │    │
│  │ └─ fmt   │   │  ├─ add_product          │    │
│  └──────────┘   │  └─ get_statistics       │    │
│                 └──────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Web Framework | FastAPI (async) |
| Agent | LangGraph (tool routing & execution) |
| MCP Server | FastMCP (stdio transport) |
| Storage | JSON-based (`data/products.json`) |
| Containerization | Docker + docker-compose |
| Testing | pytest |
| Language | Python 3.11+ |

## Project Structure

```
├── api/            # FastAPI application
├── agent/          # LangGraph agent + custom tools
├── mcp_server/     # FastMCP server + JSON storage
├── data/           # products.json (persisted via volume)
├── tests/          # pytest tests
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

## Quick Start

### Docker (recommended)

```bash
docker-compose up --build
```

### Local

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn api.main:app --host 0.0.0.0 --port 8000
```

## Usage

```bash
# List products
curl -X POST http://localhost:8000/api/v1/agent/query \
  -H 'Content-Type: application/json' \
  -d '{"query": "Покажи все продукты в категории Электроника"}'

# Get statistics
curl -X POST http://localhost:8000/api/v1/agent/query \
  -H 'Content-Type: application/json' \
  -d '{"query": "Какая средняя цена продуктов?"}'

# Add product
curl -X POST http://localhost:8000/api/v1/agent/query \
  -H 'Content-Type: application/json' \
  -d '{"query": "Добавь новый продукт: Мышка, цена 1500, категория Электроника"}'

# Calculate discount
curl -X POST http://localhost:8000/api/v1/agent/query \
  -H 'Content-Type: application/json' \
  -d '{"query": "Посчитай скидку 15% на товар с ID 1"}'
```

## Tests

```bash
# Docker
docker-compose run --rm app pytest -v

# Local (Python 3.11+)
PYTHONPATH=. pytest -v
```

## Key Design Decisions

- **Mock LLM**: Deterministic, rule-based routing — no external API keys required
- **MCP via stdio**: Agent spawns MCP server per request through stdio pipes
- **Single container**: All components colocated for simplicity
- **Idempotent**: JSON storage with Docker volume persistence

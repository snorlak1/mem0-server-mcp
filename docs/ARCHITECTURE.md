# Architecture Overview

## System Components

### 1. PostgreSQL + pgvector
- **Purpose:** Vector storage for semantic search
- **Version:** PostgreSQL 17 with pgvector extension
- **Configuration:** Auto-adjusts dimensions based on embedding model
- **HNSW:** Automatically disabled for >2000 dimensions

### 2. Neo4j
- **Purpose:** Graph database for relationship tracking
- **Version:** 5.26.4 with APOC plugin
- **Access:** http://localhost:7474 (neo4j/mem0graph)

### 3. Mem0 Server
- **Technology:** FastAPI (Python 3.12)
- **Purpose:** REST API for memory operations
- **Features:**
  - Multi-LLM support (Ollama/OpenAI/Anthropic)
  - Smart embedding dimension detection
  - Automatic HNSW toggle
  - Comprehensive error handling

### 4. MCP Server
- **Technology:** FastMCP with SSE transport
- **Purpose:** Model Context Protocol integration for Claude Code
- **Features:**
  - 5 production-ready tools
  - Project isolation (3 modes)
  - HTTP client with retry logic
  - Health monitoring

## Data Flow

```
Claude Code
    ↓ (MCP Tool Call)
MCP Server
    ↓ (HTTP REST)
Mem0 Server
    ↓ (Query/Store)
PostgreSQL (vectors) + Neo4j (graphs)
    ↑
Ollama/OpenAI (LLM + Embeddings)
```

## Networking

All services communicate via Docker bridge network `mem0_network`:
- Internal DNS resolution
- Isolated from host
- Exposed ports: 5432, 7474, 7687, 8000, 8080

## Storage

### Volumes
- `postgres_data`: PostgreSQL database
- `neo4j_data`: Neo4j graph data
- `./history`: SQLite history files (host mount)

### Data Persistence
- Memories survive container restarts
- Use `./scripts/clean.sh` to remove all data

## Security

- Default passwords (MUST change for production)
- No external network exposure by default
- Health checks for all services
- Proper dependency ordering

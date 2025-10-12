# Configuration Guide

Complete reference for all configuration options in the Mem0 MCP Server.

## Configuration File

All configuration is done via environment variables in `.env` file at the root of the project.

```bash
# Copy the example
cp .env.example .env

# Edit with your settings
nano .env
```

---

## LLM Provider Configuration

### Ollama (Default - Self-Hosted)

**Benefits:**
- 100% self-hosted, no API costs
- Full data privacy
- Works offline
- No rate limits

**Configuration:**

```bash
# Provider selection
LLM_PROVIDER=ollama

# Ollama server address
OLLAMA_BASE_URL=http://192.168.1.2:11434

# LLM model for text generation
OLLAMA_LLM_MODEL=qwen3:8b

# Embedding model for vector search
OLLAMA_EMBEDDING_MODEL=qwen3-embedding:8b

# CRITICAL: Embedding dimensions (must match model)
OLLAMA_EMBEDDING_DIMS=4096
```

**Supported Models:**

| Model | Dimensions | Speed | Quality | Use Case |
|-------|-----------|-------|---------|----------|
| `qwen3-embedding:8b` | 4096 | Slow | High | Production, accuracy-focused |
| `nomic-embed-text` | 768 | Fast | Good | Development, speed-focused |
| `all-minilm:l6-v2` | 384 | Very Fast | Basic | Testing, quick iteration |

**Setup:**

```bash
# On your Ollama server
ollama pull qwen3:8b
ollama pull qwen3-embedding:8b

# Verify
ollama list | grep qwen3

# Test connectivity
curl http://192.168.1.2:11434/api/tags
```

---

### OpenAI (Cloud - Paid)

**Benefits:**
- Fast response times
- High-quality embeddings
- GPT-4 level reasoning
- No local infrastructure

**Configuration:**

```bash
# Provider selection
LLM_PROVIDER=openai

# API key (required)
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxx

# LLM model
OPENAI_LLM_MODEL=gpt-4o

# Embedding model
OPENAI_EMBEDDING_MODEL=text-embedding-3-small

# Embedding dimensions
OPENAI_EMBEDDING_DIMS=1536
```

**Model Options:**

| Model | Context | Speed | Cost | Use Case |
|-------|---------|-------|------|----------|
| `gpt-4o` | 128k | Fast | $$$ | Production |
| `gpt-4o-mini` | 128k | Very Fast | $ | Development |
| `gpt-3.5-turbo` | 16k | Very Fast | $ | Basic tasks |

**Embedding Options:**

| Model | Dimensions | Cost | Performance |
|-------|-----------|------|-------------|
| `text-embedding-3-small` | 1536 | $ | Balanced |
| `text-embedding-3-large` | 3072 | $$ | High quality |
| `text-embedding-ada-002` | 1536 | $ | Legacy (stable) |

**Get API Key:**
1. Visit https://platform.openai.com/api-keys
2. Create new secret key
3. Copy to `.env` file

---

### Anthropic (Cloud - Paid, LLM Only)

**Benefits:**
- Claude 3.5 Sonnet reasoning
- Long context windows (200k tokens)
- Strong safety features

**Configuration:**

```bash
# Provider selection
LLM_PROVIDER=anthropic

# API key (required)
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxx

# Model
ANTHROPIC_MODEL=claude-3-5-sonnet-20241022

# IMPORTANT: Still need embeddings from Ollama or OpenAI
OLLAMA_BASE_URL=http://192.168.1.2:11434
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768
```

**Model Options:**

| Model | Context | Speed | Cost | Use Case |
|-------|---------|-------|------|----------|
| `claude-3-5-sonnet-20241022` | 200k | Fast | $$$ | Production |
| `claude-3-5-haiku-20241022` | 200k | Very Fast | $ | Development |
| `claude-3-opus-20240229` | 200k | Slow | $$$$ | Complex reasoning |

**Note:** Anthropic doesn't provide embeddings, so you must configure Ollama or OpenAI for embeddings.

**Get API Key:**
1. Visit https://console.anthropic.com/settings/keys
2. Create new API key
3. Copy to `.env` file

---

## Database Configuration

### PostgreSQL (Vector Storage)

```bash
# Connection settings
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=mem0
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres123

# Collection name (table name for vectors)
POSTGRES_COLLECTION_NAME=memories

# Version (for pgvector)
POSTGRES_VERSION=17
```

**Important:**
- `POSTGRES_HOST=postgres` uses Docker service name (internal networking)
- Change `POSTGRES_PASSWORD` for production
- `POSTGRES_VERSION=17` ensures latest pgvector features

**External Access:**
To connect from host machine:
```bash
# In docker-compose.yml, expose port
ports:
  - "5432:5432"

# Then connect with
psql -h localhost -p 5432 -U postgres -d mem0
```

---

### Neo4j (Graph Database)

```bash
# Authentication
NEO4J_USER=neo4j
NEO4J_PASSWORD=mem0graph

# Version
NEO4J_VERSION=5.26.4
```

**Important:**
- Change `NEO4J_PASSWORD` for production
- First login may require password change via browser
- APOC plugin is pre-installed

**Browser Access:**
```
http://localhost:7474
Username: neo4j
Password: mem0graph
```

**Bolt Connection:**
```
bolt://localhost:7687
```

---

## Project Isolation Configuration

### Auto Mode (Recommended)

```bash
PROJECT_ID_MODE=auto
```

**How it works:**
- Automatically generates `user_id` from project directory
- Each directory gets unique ID via hash
- Complete isolation between projects

**Example:**
```
/home/user/project-a  →  user_id: "prj_a1b2c3d4"
/home/user/project-b  →  user_id: "prj_e5f6g7h8"
```

**Use when:**
- Working on multiple projects
- Want automatic isolation
- Default behavior desired

---

### Manual Mode

```bash
PROJECT_ID_MODE=manual
DEFAULT_USER_ID=my_custom_project_name
```

**How it works:**
- Uses fixed `user_id` specified in `DEFAULT_USER_ID`
- All operations use this ID
- Manual control over isolation

**Example:**
```
All projects → user_id: "my_custom_project_name"
```

**Use when:**
- Need custom project naming
- Testing specific scenarios
- Sharing memories across directories

---

### Global Mode

```bash
PROJECT_ID_MODE=global
DEFAULT_USER_ID=global_memory
```

**How it works:**
- Single shared memory pool for all projects
- No isolation between directories
- Universal memory access

**Example:**
```
All projects → user_id: "global_memory"
```

**Use when:**
- Personal coding preferences (shared everywhere)
- Team-wide conventions
- Single user, multiple projects
- Prototyping

---

## Server Configuration

### Mem0 API Server

```bash
# Port (internal Docker network)
MEM0_PORT=8000

# Host binding
MEM0_HOST=0.0.0.0

# Workers (for production)
# WORKERS=4
```

**External Access:**
The port 8000 is exposed to host, accessible at:
```
http://localhost:8000
http://localhost:8000/docs  # Swagger UI
http://localhost:8000/redoc # ReDoc
```

---

### MCP Server

```bash
# Port (internal Docker network)
MCP_PORT=8080

# Host binding
MCP_HOST=0.0.0.0

# Timeout for HTTP requests to Mem0 API (seconds)
# REQUEST_TIMEOUT=30
```

**External Access:**
```
http://localhost:8080/sse  # SSE endpoint for Claude Code
```

---

## Performance Tuning

### Embedding Dimensions vs HNSW

pgvector's HNSW index is limited to **2000 dimensions**. The config auto-detects and adjusts:

```bash
# Fast mode: 768 dimensions, HNSW enabled
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768
# → HNSW: ON, Search: Fast, Accuracy: Good

# Accurate mode: 4096 dimensions, HNSW disabled
OLLAMA_EMBEDDING_MODEL=qwen3-embedding:8b
OLLAMA_EMBEDDING_DIMS=4096
# → HNSW: OFF, Search: Slower, Accuracy: High
```

**Recommendation:**
- **Development:** Use `nomic-embed-text` (768d) for speed
- **Production:** Use `qwen3-embedding:8b` (4096d) for accuracy

---

### Ollama Performance

```bash
# Concurrent requests (if Ollama supports)
# OLLAMA_NUM_PARALLEL=2

# Model context size
# OLLAMA_CONTEXT_SIZE=4096

# Keep models loaded in memory
# OLLAMA_KEEP_ALIVE=-1
```

**Pre-warming Models:**
```bash
# Keep models in memory to avoid cold starts
curl http://192.168.1.2:11434/api/generate \
  -d '{"model": "qwen3:8b", "prompt": "Hello", "keep_alive": -1}'
```

---

### Database Performance

```bash
# PostgreSQL (in docker-compose.yml environment)
POSTGRES_MAX_CONNECTIONS=100
POSTGRES_SHARED_BUFFERS=256MB
POSTGRES_EFFECTIVE_CACHE_SIZE=1GB

# Neo4j (in docker-compose.yml environment)
NEO4J_dbms_memory_heap_initial__size=512M
NEO4J_dbms_memory_heap_max__size=2G
NEO4J_dbms_memory_pagecache_size=512M
```

---

## Docker Configuration

### Resource Limits

Create `docker-compose.override.yml`:

```yaml
services:
  mem0:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G

  postgres:
    deploy:
      resources:
        limits:
          memory: 2G

  neo4j:
    deploy:
      resources:
        limits:
          memory: 2G
```

---

### Volume Configuration

**Default volumes:**
```yaml
volumes:
  postgres_data:  # PostgreSQL database
  neo4j_data:     # Neo4j graph data
  ./history:      # SQLite history files (host mount)
```

**Custom locations:**
```yaml
volumes:
  postgres_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/data/postgres

  neo4j_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/data/neo4j
```

---

### Network Configuration

**Default network:**
```yaml
networks:
  mem0_network:
    driver: bridge
```

**Custom network:**
```yaml
networks:
  mem0_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

---

## Security Configuration

### Production Checklist

```bash
# 1. Change all passwords
POSTGRES_PASSWORD=<strong-random-password>
NEO4J_PASSWORD=<strong-random-password>

# 2. Use HTTPS (add reverse proxy)
# Example: nginx with Let's Encrypt

# 3. Restrict network access
# Only expose MCP server (8080), not API server (8000)

# 4. Use secrets management
# Don't commit .env to git
# Use Docker secrets or vault

# 5. Enable authentication
# Add API key middleware to mem0 server

# 6. Firewall rules
# Restrict Ollama access to trusted IPs
```

---

### API Key Protection (Custom Implementation)

Add to `mem0-server/main.py`:

```python
from fastapi import Header, HTTPException

API_KEY = os.getenv("API_KEY", "")

async def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != API_KEY:
        raise HTTPException(status_code=401, detail="Invalid API key")

# Add dependency to routes
@app.post("/memories", dependencies=[Depends(verify_api_key)])
async def create_memory(...):
    ...
```

Then in `.env`:
```bash
API_KEY=your-secret-api-key-here
```

---

## Environment Variables Reference

### Complete List

```bash
# === LLM Provider ===
LLM_PROVIDER=ollama                           # ollama | openai | anthropic

# === Ollama ===
OLLAMA_BASE_URL=http://192.168.1.2:11434
OLLAMA_LLM_MODEL=qwen3:8b
OLLAMA_EMBEDDING_MODEL=qwen3-embedding:8b
OLLAMA_EMBEDDING_DIMS=4096

# === OpenAI ===
OPENAI_API_KEY=sk-proj-xxxx
OPENAI_LLM_MODEL=gpt-4o
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_EMBEDDING_DIMS=1536

# === Anthropic ===
ANTHROPIC_API_KEY=sk-ant-xxxx
ANTHROPIC_MODEL=claude-3-5-sonnet-20241022

# === PostgreSQL ===
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=mem0
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres123
POSTGRES_COLLECTION_NAME=memories
POSTGRES_VERSION=17

# === Neo4j ===
NEO4J_USER=neo4j
NEO4J_PASSWORD=mem0graph
NEO4J_VERSION=5.26.4

# === Mem0 Server ===
MEM0_PORT=8000
MEM0_HOST=0.0.0.0

# === MCP Server ===
MCP_PORT=8080
MCP_HOST=0.0.0.0

# === Project Isolation ===
PROJECT_ID_MODE=auto                          # auto | manual | global
DEFAULT_USER_ID=default                       # Used in manual/global modes

# === Docker ===
COMPOSE_PROJECT_NAME=mem0-mcp
```

---

## Configuration Validation

### Startup Checks

The `scripts/start.sh` script validates:
1. `.env` file exists
2. Docker is running
3. Docker Compose is available
4. Required ports are free (5432, 7474, 7687, 8000, 8080)

### Health Checks

```bash
# Check all services
./scripts/health.sh

# Check specific service
docker compose exec mem0 curl http://localhost:8000/health
```

---

## Configuration Examples

### Example 1: Fast Development Setup

```bash
LLM_PROVIDER=ollama
OLLAMA_BASE_URL=http://192.168.1.2:11434
OLLAMA_LLM_MODEL=qwen3:8b
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768
PROJECT_ID_MODE=global
DEFAULT_USER_ID=dev
```

**Benefits:**
- Fast responses (768d embeddings, HNSW enabled)
- Shared memory across projects
- Self-hosted (no API costs)

---

### Example 2: Production with OpenAI

```bash
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-proj-xxxx
OPENAI_LLM_MODEL=gpt-4o
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_EMBEDDING_DIMS=1536
PROJECT_ID_MODE=auto
POSTGRES_PASSWORD=<strong-password>
NEO4J_PASSWORD=<strong-password>
```

**Benefits:**
- Fast, high-quality responses
- Automatic project isolation
- Secure passwords

---

### Example 3: High Accuracy Self-Hosted

```bash
LLM_PROVIDER=ollama
OLLAMA_BASE_URL=http://192.168.1.2:11434
OLLAMA_LLM_MODEL=qwen3:8b
OLLAMA_EMBEDDING_MODEL=qwen3-embedding:8b
OLLAMA_EMBEDDING_DIMS=4096
PROJECT_ID_MODE=auto
```

**Benefits:**
- High-quality embeddings (4096d)
- Automatic project isolation
- 100% self-hosted
- No API costs

---

### Example 4: Claude + Ollama Embeddings

```bash
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-xxxx
ANTHROPIC_MODEL=claude-3-5-sonnet-20241022
OLLAMA_BASE_URL=http://192.168.1.2:11434
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768
PROJECT_ID_MODE=auto
```

**Benefits:**
- Claude's reasoning (LLM)
- Fast self-hosted embeddings
- Automatic project isolation
- Lower costs than full OpenAI

---

## Troubleshooting Configuration

### Issue: Dimension Mismatch

**Error:** `expected 1536 dimensions, not 4096`

**Fix:**
```bash
# Ensure embedding_dims matches your model
OLLAMA_EMBEDDING_MODEL=qwen3-embedding:8b
OLLAMA_EMBEDDING_DIMS=4096  # Must match model output
```

---

### Issue: HNSW Index Error

**Error:** `column cannot have more than 2000 dimensions for hnsw index`

**Fix:** Already handled automatically! Config disables HNSW for >2000 dims.

---

### Issue: Ollama Not Reachable

**Error:** `Failed to connect to Ollama`

**Fix:**
```bash
# Test connectivity
curl http://192.168.1.2:11434/api/tags

# Check from container
docker compose exec mem0 curl http://192.168.1.2:11434/api/tags

# Verify OLLAMA_BASE_URL in .env
```

---

### Issue: Slow First Request

**Behavior:** First memory takes 60+ seconds

**Cause:** Ollama loading model into memory

**Solutions:**
1. **Pre-warm models** (keep in memory):
   ```bash
   curl http://192.168.1.2:11434/api/generate \
     -d '{"model": "qwen3:8b", "prompt": ".", "keep_alive": -1}'
   ```

2. **Use smaller model**:
   ```bash
   OLLAMA_EMBEDDING_MODEL=nomic-embed-text
   OLLAMA_EMBEDDING_DIMS=768
   ```

3. **Switch to OpenAI**:
   ```bash
   LLM_PROVIDER=openai
   OPENAI_API_KEY=sk-proj-xxxx
   ```

---

## See Also

- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [API.md](API.md) - REST API reference
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues
- [PERFORMANCE.md](PERFORMANCE.md) - Optimization guide

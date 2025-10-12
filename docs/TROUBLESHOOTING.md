# Troubleshooting Guide

## Common Issues

### 0. Authentication Issues

See [AUTHENTICATION.md](AUTHENTICATION.md) for detailed authentication troubleshooting.

**Quick fixes:**

```bash
# Verify environment variables
echo $MEM0_TOKEN
echo $MEM0_USER_ID

# List tokens
python3 scripts/mcp-token.py list

# Test authentication
./tests/test_auth.sh

# Remove and re-add server
claude mcp remove mem0
claude mcp add mem0 http://localhost:8080/mcp/ -t http \
  -H "X-MCP-Token: ${MEM0_TOKEN}" \
  -H "X-MCP-UserID: ${MEM0_USER_ID}"
```

### 1. Services Won't Start

**Symptoms:** Docker containers fail to start or are unhealthy

**Solutions:**
```bash
# Check logs
./scripts/logs.sh

# Check ports
lsof -i :5432  # PostgreSQL
lsof -i :7474  # Neo4j HTTP
lsof -i :7687  # Neo4j Bolt
lsof -i :8000  # Mem0 API
lsof -i :8080  # MCP Server

# Clean and restart
./scripts/clean.sh
./scripts/start.sh
```

### 2. Slow Memory Operations

**Symptoms:** Memory creation takes 60+ seconds

**Causes:**
- Ollama model loading (first request)
- Large embedding dimensions (4096)
- HNSW disabled

**Solutions:**
```bash
# Use smaller embedding model
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768

# Or switch to OpenAI
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...
```

### 3. Can't Connect to Ollama

**Symptoms:** "Failed to connect to Ollama" errors

**Solutions:**
```bash
# Test Ollama connectivity
curl http://192.168.1.2:11434/api/tags

# Verify models are pulled
ollama list | grep qwen3

# Check from inside container
docker compose exec mem0 curl http://192.168.1.2:11434/api/tags

# Update OLLAMA_BASE_URL in .env
```

### 4. Dimension Mismatch Errors

**Error:** `expected 1536 dimensions, not 4096`

**Solution:**
Already fixed in this package! The config auto-detects dimensions.
If you still see this, ensure `.env` has:
```bash
OLLAMA_EMBEDDING_DIMS=4096  # Match your model
```

### 5. HNSW Index Error

**Error:** `column cannot have more than 2000 dimensions for hnsw index`

**Solution:**
Already fixed! HNSW auto-disables for >2000 dims.
For faster search, use smaller model:
```bash
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768
```

### 6. Claude Code Can't Connect

**Symptoms:** MCP tools not available in Claude Code

**Solutions:**
1. Verify endpoints:
   ```bash
   # HTTP Stream (recommended)
   curl -v http://localhost:8080/mcp/

   # SSE (legacy)
   curl -v http://localhost:8080/sse/
   ```

2. Check MCP config with authentication:
   ```json
   {
     "mcpServers": {
       "mem0": {
         "url": "http://localhost:8080/mcp/",
         "transport": "http",
         "headers": {
           "X-MCP-Token": "your-token-here",
           "X-MCP-UserID": "your.email@company.com"
         }
       }
     }
   }
   ```

3. Verify using CLI:
   ```bash
   claude mcp list
   # Should show mem0 with "✓ Connected"
   ```

4. Restart Claude Code after config changes

5. Check MCP server logs:
   ```bash
   ./scripts/logs.sh mcp
   ```

6. Verify authentication:
   ```bash
   echo $MEM0_TOKEN
   echo $MEM0_USER_ID
   ```

### 7. Database Connection Errors

**Symptoms:** "Could not connect to database"

**Solutions:**
```bash
# Check PostgreSQL
docker compose exec postgres pg_isready -U postgres

# Check Neo4j
curl http://localhost:7474

# Restart services
./scripts/restart.sh
```

## Getting Help

1. Check service health: `./scripts/health.sh`
2. View logs: `./scripts/logs.sh`
3. Run tests: `./scripts/test.sh`
4. See ARCHITECTURE.md for system design
5. Check .env configuration

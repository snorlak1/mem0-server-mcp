# 🧠 Mem0 MCP Server - Self-Hosted Memory for AI

A production-ready, self-hosted Model Context Protocol (MCP) server that provides persistent memory for Claude Code and other AI assistants. Built with Docker Compose for easy deployment.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/docker-compose-blue.svg)](https://docs.docker.com/compose/)
[![Python](https://img.shields.io/badge/python-3.12-blue.svg)](https://www.python.org/downloads/)

## ✨ Features

### Core Features
- 🚀 **One-Command Deployment** - Start the entire stack with a single script
- 🔒 **100% Self-Hosted** - No external API dependencies (when using Ollama)
- 🔐 **Token-Based Authentication** - Secure multi-user access with PostgreSQL-backed token management
- 🌐 **Multi-LLM Support** - Works with Ollama, OpenAI, or Anthropic
- 🎯 **Project Isolation** - Automatic memory isolation per project directory
- 📊 **Semantic Search** - Vector-based search with pgvector
- ⚡ **13 MCP Tools** - Complete memory management + intelligence analysis
- 🔌 **Dual Transport Support** - Modern HTTP Stream (recommended) + legacy SSE transport
- 🐳 **Docker Compose** - Easy orchestration of all services
- 🧪 **Comprehensive Tests** - Automated test suite included
- 📝 **Audit Logging** - Track all authentication attempts and token usage

### 🧠 Memory Intelligence System (NEW!)
- 🔗 **Knowledge Graphs** - Link memories with typed relationships (RELATES_TO, DEPENDS_ON, SUPERSEDES, etc.)
- 🕒 **Temporal Tracking** - Track how knowledge evolves over time
- 🏗️ **Architecture Mapping** - Map system components and dependencies
- 📊 **Impact Analysis** - Understand cascading effects of changes
- 📝 **Decision Tracking** - Record technical decisions with pros/cons/alternatives
- 🎯 **Topic Clustering** - Automatically detect knowledge groups
- ⭐ **Quality Scoring** - Trust scores based on validations and citations
- 🚀 **Intelligence Analysis** - Comprehensive health reports with actionable recommendations

## 🏗️ Architecture

```
┌──────────────┐
│  Claude Code │  (Your IDE with MCP client)
└──────┬───────┘
       │ HTTP Stream (recommended): http://localhost:8080/mcp
       │ SSE (legacy): http://localhost:8080/sse
       │ + Token Authentication Headers
       ↓
┌──────────────┐
│  MCP Server  │  Port 8080 (FastMCP)
│  (Python)    │  • 13 MCP Tools (5 core + 8 intelligence)
│              │  • Token Validation
│              │  • Dual Transport Support
└──────┬───────┘
       │ HTTP REST API
       ↓
┌──────────────┐
│ Mem0 Server  │  Port 8000 (FastAPI)
│  (FastAPI)   │  • 28 REST Endpoints (13 core + 15 intelligence)
│              │  • Multi-LLM Support
│              │  • Vector + Graph Storage
│              │  • Memory Intelligence System
└──────┬───────┘
       │
   ┌───┴────┬──────────┬──────┐
   ↓        ↓          ↓      ↓
┌────────┐ ┌────┐  ┌─────┐ ┌──────┐
│Postgres│ │Neo4j│  │Auth │ │Ollama│
│pgvector│ │Graph│  │Token│ │ LLM  │
│Vector  │ │Intel-│  │Store│ │      │
│Search  │ │ligence│ │     │ │      │
└────────┘ └────┘  └─────┘ └──────┘
```

## 🚀 Quick Start (5 Minutes)

### Prerequisites

1. **Docker & Docker Compose** installed
   ```bash
   docker --version
   docker compose version
   ```

2. **Ollama Server** with models (or OpenAI/Anthropic API key)
   ```bash
   # On your Ollama server:
   ollama pull qwen3:8b
   ollama pull qwen3-embedding:8b
   ```

### Installation

```bash
# 1. Clone or copy this directory
cd /path/to/mem0-mcp

# 2. Create configuration file
cp .env.example .env

# 3. Edit .env with your Ollama server address (if needed)
nano .env  # Update OLLAMA_BASE_URL

# 4. Start everything!
./scripts/start.sh
```

That's it! The script will:
- ✅ Start PostgreSQL with pgvector
- ✅ Start Neo4j graph database
- ✅ Start Mem0 REST API server
- ✅ Start MCP server for Claude Code
- ✅ Wait for all services to be healthy

### Setup Authentication

**Step 1: Run Database Migrations**

```bash
./scripts/migrate-auth.sh
```

**Step 2: Create Your Authentication Token**

```bash
python3 scripts/mcp-token.py create \
  --user-id your.email@company.com \
  --name "Your Name" \
  --email your.email@company.com
```

This will output your token and setup instructions. Copy the `MEM0_TOKEN` value.

**Step 3: Add to Your Shell Profile**

Add these lines to `~/.zshrc` or `~/.bashrc`:

```bash
export MEM0_TOKEN='mcp_abc123...'  # Your token from step 2
export MEM0_USER_ID='your.email@company.com'
```

Then reload:
```bash
source ~/.zshrc  # or ~/.bashrc
```

### Connect to Claude Code

**Recommended: Using Claude CLI (Easiest)**

```bash
# Add mem0 server with HTTP Stream transport (recommended)
claude mcp add mem0 http://localhost:8080/mcp/ -t http \
  -H "X-MCP-Token: ${MEM0_TOKEN}" \
  -H "X-MCP-UserID: ${MEM0_USER_ID}"

# Verify it's configured
claude mcp list
```

**Alternative: Manual Configuration**

Add this to your Claude Code MCP configuration file:

**File:** `~/.config/claude-code/config.json`

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

**Legacy SSE Transport (Backward Compatibility)**

```bash
# Using CLI
claude mcp add mem0 http://localhost:8080/sse/ -t http \
  -H "X-MCP-Token: ${MEM0_TOKEN}" \
  -H "X-MCP-UserID: ${MEM0_USER_ID}"
```

**Important:**
- Always include the trailing slash in URLs: `/mcp/` or `/sse/` (not `/mcp` or `/sse`)
- HTTP Stream transport (`/mcp/`) is recommended as it's the modern MCP protocol
- SSE (`/sse/`) is maintained for backward compatibility

Restart Claude Code and you're ready to go!

## 📖 Usage

### Basic Commands

```bash
# Start the stack
./scripts/start.sh

# View logs
./scripts/logs.sh          # All services
./scripts/logs.sh mem0     # Specific service

# Check health
./scripts/health.sh

# Run tests
./scripts/test.sh

# Stop the stack
./scripts/stop.sh

# Restart
./scripts/restart.sh

# Clean all data (⚠️  destructive)
./scripts/clean.sh
```

### Using with Claude Code

Once connected, you can use these commands in Claude Code:

```
"Store this code in memory: [your code snippet]"
"Search my memories for Python functions"
"Show all my stored coding preferences"
"Delete memory with ID [id]"
"Show history of memory [id]"
```

### Available MCP Tools

#### Core Memory Tools (5)

1. **add_coding_preference** - Store code snippets and implementation details
2. **search_coding_preferences** - Semantic search through memories
3. **get_all_coding_preferences** - Retrieve all stored memories
4. **delete_memory** - Delete specific memory by ID
5. **get_memory_history** - View change history

#### Memory Intelligence Tools (8)

6. **link_memories** - Create typed relationships between memories (build knowledge graphs)
7. **get_related_memories** - Graph traversal to discover connected context
8. **analyze_memory_intelligence** 🚀 - **GAME-CHANGER:** Comprehensive intelligence report with health scores, clusters, and recommendations
9. **create_component** - Map system architecture with component nodes
10. **link_component_dependency** - Define dependencies between components
11. **analyze_component_impact** - Analyze cascading effects of changes
12. **create_decision** - Track technical decisions with pros/cons/alternatives
13. **get_decision_rationale** - Retrieve decision context and reasoning

All tools automatically use authentication credentials from your MCP configuration headers.

### Using Memory Intelligence

```
"Link these two memories as related"
"Show me all memories related to authentication"
"Analyze my project's knowledge graph health"
"Create a component called Database with type Infrastructure"
"What would be impacted if I change the Authentication component?"
"Record this decision: Use PostgreSQL, pros: ACID compliance, cons: complexity"
```

## 🔐 Authentication Management

### Token Management

```bash
# List all tokens
python3 scripts/mcp-token.py list

# List tokens for specific user
python3 scripts/mcp-token.py list --user-id john.doe@company.com

# Create a new token
python3 scripts/mcp-token.py create \
  --user-id john.doe@company.com \
  --name "John Doe" \
  --email john.doe@company.com

# Revoke (disable) a token
python3 scripts/mcp-token.py revoke mcp_abc123

# Re-enable a token
python3 scripts/mcp-token.py enable mcp_abc123

# Delete a token permanently
python3 scripts/mcp-token.py delete mcp_abc123

# View audit log
python3 scripts/mcp-token.py audit --days 30

# View user statistics
python3 scripts/mcp-token.py stats john.doe@company.com
```

### Testing Authentication

```bash
./tests/test_auth.sh
```

This tests: missing headers, invalid tokens, user ID mismatches, valid authentication, token revocation, and re-enabling.

## 🔧 Configuration

### LLM Providers

#### Ollama (Default - Free, Self-Hosted)

```bash
LLM_PROVIDER=ollama
OLLAMA_BASE_URL=http://192.168.1.2:11434
OLLAMA_LLM_MODEL=qwen3:8b
OLLAMA_EMBEDDING_MODEL=qwen3-embedding:8b
OLLAMA_EMBEDDING_DIMS=4096
```

**Supported Models:**
- **LLM:** llama3, qwen3, mistral, phi3, etc.
- **Embeddings:** qwen3-embedding (4096d), nomic-embed-text (768d), all-minilm (384d)

#### OpenAI (Cloud - Paid)

```bash
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-proj-...
OPENAI_LLM_MODEL=gpt-4o
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_EMBEDDING_DIMS=1536
```

#### Anthropic (Cloud - Paid, LLM only)

```bash
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-3-5-sonnet-20241022

# Still need embeddings from Ollama or OpenAI:
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768
```

### Project Isolation

Control how memories are isolated per project:

```bash
# Auto mode (recommended) - Auto-detect project from directory
PROJECT_ID_MODE=auto

# Manual mode - Set explicitly per project
PROJECT_ID_MODE=manual
DEFAULT_USER_ID=my_project_name

# Global mode - Share all memories
PROJECT_ID_MODE=global
DEFAULT_USER_ID=shared_memory
```

### Performance Tuning

For faster performance, use smaller embedding models:

```bash
# Fast: 768 dimensions (enables HNSW indexing)
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
OLLAMA_EMBEDDING_DIMS=768

# Slower but more accurate: 4096 dimensions (HNSW disabled)
OLLAMA_EMBEDDING_MODEL=qwen3-embedding:8b
OLLAMA_EMBEDDING_DIMS=4096
```

**Note:** pgvector's HNSW index is limited to 2000 dimensions. For larger dimensions, the system automatically disables HNSW (slower but still functional).

## 📊 Endpoints

### Mem0 REST API (Port 8000)

#### Core Endpoints (13)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check |
| `/docs` | GET | OpenAPI documentation |
| `/memories` | POST | Create memory |
| `/memories` | GET | Get all memories |
| `/memories/{id}` | GET | Get specific memory |
| `/memories/{id}` | PUT | Update memory |
| `/memories/{id}` | DELETE | Delete memory |
| `/memories/{id}/history` | GET | Get history |
| `/search` | POST | Semantic search |
| `/reset` | POST | Reset all memories |
| `/configure` | POST | Configure Mem0 |

#### Memory Intelligence Endpoints (15)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/graph/link` | POST | Link memories with relationships |
| `/graph/related/{id}` | GET | Get related memories (graph traversal) |
| `/graph/path` | GET | Find path between memories |
| `/graph/evolution/{topic}` | GET | Track knowledge evolution |
| `/graph/superseded` | GET | Find obsolete memories |
| `/graph/thread/{id}` | GET | Get conversation thread |
| `/graph/component` | POST | Create component node |
| `/graph/component/dependency` | POST | Link component dependencies |
| `/graph/component/link-memory` | POST | Link memory to component |
| `/graph/impact/{name}` | GET | Analyze component impact |
| `/graph/decision` | POST | Create decision with pros/cons |
| `/graph/decision/{id}` | GET | Get decision rationale |
| `/graph/communities` | GET | Detect memory communities |
| `/graph/trust-score/{id}` | GET | Calculate trust score |
| `/graph/intelligence` | GET | 🚀 **Comprehensive intelligence analysis** |

### MCP Server (Port 8080)

| Endpoint | Description |
|----------|-------------|
| `/mcp` | HTTP Stream endpoint (recommended) |
| `/sse` | SSE endpoint (legacy) |
| `/` | Health check |

### Neo4j Browser (Port 7474)

Access the Neo4j browser at `http://localhost:7474`
- Username: `neo4j`
- Password: `mem0graph`

## 🧪 Testing

```bash
# Run all tests
./scripts/test.sh

# Individual test suites
./tests/test_api.sh                         # REST API tests
./tests/test_mcp.sh                         # MCP server tests
./tests/test_integration.sh                 # Full integration test
./tests/test_memory_intelligence_fixed.sh   # Memory Intelligence integration test
./tests/test_mcp_intelligence.sh            # MCP Intelligence verification
./tests/test_auth.sh                        # Authentication tests
./tests/test_ownership_simple.sh            # Memory ownership tests
```

## 📚 Documentation

Detailed documentation is available in the `docs/` directory:

- **[QUICKSTART.md](docs/QUICKSTART.md)** - Quick start guide with authentication setup
- **[AUTHENTICATION.md](docs/AUTHENTICATION.md)** - Complete authentication guide
- **[SECURITY.md](docs/SECURITY.md)** - Security features and best practices
- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - System design and components
- **[API.md](docs/API.md)** - Complete API reference
- **[MCP_TOOLS.md](docs/MCP_TOOLS.md)** - MCP tools usage guide
- **[CONFIGURATION.md](docs/CONFIGURATION.md)** - All configuration options
- **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)** - Common issues and solutions
- **[PERFORMANCE.md](docs/PERFORMANCE.md)** - Performance optimization

## 🔒 Security

The Mem0 MCP Server implements enterprise-grade security:

### Memory Ownership & Isolation

**All memory operations validate ownership:**
- ✅ Users can only access their own memories
- ✅ Read, update, delete, and history operations are protected
- ✅ Automatic validation at both REST API and MCP tool levels

```bash
# User A cannot access User B's memory
curl "http://localhost:8000/memories/{memory_id}?user_id=user_b"
# Returns: 403 Forbidden - "Access denied"
```


### Production Security Checklist

1. **Change default passwords** in `.env`:
   ```bash
   POSTGRES_PASSWORD=<strong-password>
   NEO4J_PASSWORD=<strong-password>
   ```

2. **Rotate authentication tokens** regularly:
   ```bash
   python3 scripts/mcp-token.py create --user-id user@company.com
   ```

3. **Restrict network access** - Don't expose ports publicly

4. **Use HTTPS** - Add TLS termination via reverse proxy (nginx, Traefik)

5. **Monitor audit logs**:
   ```bash
   python3 scripts/mcp-token.py audit --days 7
   ```

6. **Test security**:
   ```bash
   ./tests/test_ownership_simple.sh
   ./tests/test_auth.sh
   ```

For complete security documentation, see [SECURITY.md](docs/SECURITY.md).

## 🐛 Troubleshooting

### Authentication Issues

**"Missing authentication headers"**
- Ensure `MEM0_TOKEN` and `MEM0_USER_ID` are exported in your shell
- Verify Claude Code config has headers section
- Restart your shell and Claude Code

**"Invalid authentication token"**
- Check token exists: `python3 scripts/mcp-token.py list`
- Verify token is not expired or disabled
- Ensure you're using the correct token value

**"User ID mismatch"**
- Token belongs to different user
- Check which user owns the token: `python3 scripts/mcp-token.py list`
- Create a new token for your user ID

**"Token has been disabled"**
- Token was revoked
- Re-enable: `python3 scripts/mcp-token.py enable <token>`
- Or create a new token

**Server doesn't show in `claude mcp list`**
- Check the URL has a trailing slash: `http://localhost:8080/mcp/` (not `/mcp`)
- Verify environment variables are set: `echo $MEM0_TOKEN $MEM0_USER_ID`
- Remove and re-add: `claude mcp remove mem0` then add again
- Check server is running: `docker compose ps` and `curl http://localhost:8080/`

### Services won't start

```bash
# Check logs
./scripts/logs.sh

# Check health
./scripts/health.sh

# Ensure ports are free
lsof -i :8000  # Mem0 API
lsof -i :8080  # MCP Server
lsof -i :5432  # PostgreSQL
lsof -i :7474  # Neo4j
```

### Slow performance

1. **Use smaller embedding model:**
   ```bash
   OLLAMA_EMBEDDING_MODEL=nomic-embed-text
   OLLAMA_EMBEDDING_DIMS=768
   ```

2. **Switch to OpenAI:**
   ```bash
   LLM_PROVIDER=openai
   OPENAI_API_KEY=sk-...
   ```

3. **Pre-warm Ollama models** - Keep them loaded in memory

### Memory not storing

1. **Check Ollama connectivity:**
   ```bash
   curl http://192.168.1.2:11434/api/tags
   ```

2. **Verify models are available:**
   ```bash
   ollama list
   ```

3. **Check mem0 logs:**
   ```bash
   ./scripts/logs.sh mem0
   ```

See [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for more help.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Mem0](https://mem0.ai) - Memory layer for AI applications
- [Model Context Protocol](https://modelcontextprotocol.io) - MCP specification
- [FastMCP](https://github.com/jlowin/fastmcp) - FastMCP framework
- [pgvector](https://github.com/pgvector/pgvector) - Vector similarity search for Postgres
- [Neo4j](https://neo4j.com) - Graph database

## 📞 Support

- **Documentation:** See the `docs/` directory
- **Issues:** Open an issue on GitHub
- **Questions:** Check [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

---

**Made with ❤️ for the AI community**

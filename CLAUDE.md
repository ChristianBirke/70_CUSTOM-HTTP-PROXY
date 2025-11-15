# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Proof-of-Concept**: Lean HTTP/HTTPS proxy server demonstrating basic request forwarding between clients and servers.

**POC Goals:**
- ✅ Prove core concept works (HTTP forwarding, HTTPS tunneling)
- ✅ Keep it simple (~200 lines of code)
- ✅ Build in 7-13 hours (1-2 days)
- ✅ Safe implementation with basic validation
- ✅ Learn before committing to production features

## Current Approach: Lean POC

**Primary Document**: See `PRD-POC.md` for complete POC requirements

**What's Included** (POC features only):
- HTTP/1.1 forwarding (GET, POST, PUT, DELETE, HEAD)
- HTTPS tunneling via CONNECT method
- Basic error handling (timeouts, connection failures)
- Simple stdout logging
- Docker container
- Basic tests

**What's Excluded** (Future consideration):
- ❌ Caching, compression (premature optimization)
- ❌ CLI process integration (complex, not core)
- ❌ Rate limiting, domain filtering (production features)
- ❌ Prometheus metrics (production features)
- ❌ Load balancing (production features)
- ❌ HTTP/2, HTTP/3 (unnecessary complexity)

## POC Development Plan

**5 Tasks** (replacing the original 20):
1. **Project Setup** (1-2 hours): Basic structure, dependencies
2. **HTTP Forwarding** (2-4 hours): Core proxy functionality
3. **HTTPS Tunneling** (2-3 hours): CONNECT method
4. **Error Handling & Logging** (1-2 hours): Safety and debugging
5. **Docker & Documentation** (1-2 hours): Portability and usage docs

**Total Time**: 7-13 hours

## Current Status

**Location**: Git worktree at `.worktrees/poc-implementation`
- **Branch**: `poc/implementation` (isolated from master)
- **Status**: Ready to start Task #1 (Project Setup)
- **Setup**: Git worktrees configured, `.worktrees/` in .gitignore

**Next Step**: Create `pyproject.toml` and basic project structure

## Technology Stack (POC)

**Primary Language:** Python 3.8+ (3.11+ recommended)

**Core Libraries** (minimal for POC):
- `aiohttp`: Async HTTP server/client framework
- `pyyaml`: Configuration loading
- `pytest` + `pytest-asyncio`: Testing framework

**That's it.** No complex dependencies for POC.

**Package Management:** `uv` (preferred) or `poetry`

**Note**: Additional libraries (pydantic, structlog, prometheus, caching) are deferred to post-POC phases.

## Architecture (POC - Simplified)

### Simple Structure (~200 lines total)

```
src/
└── proxy.py           # Single file with all POC functionality:
                       # - HTTP server setup
                       # - Request forwarding logic
                       # - CONNECT method for HTTPS tunneling
                       # - Basic error handling
                       # - Stdout logging

tests/
└── test_proxy.py      # Basic tests for HTTP/HTTPS forwarding

config.yaml            # Simple configuration (port, timeout, log level)
pyproject.toml         # Minimal dependencies (aiohttp, pyyaml, pytest)
Dockerfile             # Container for portability
README.md              # Usage instructions
```

**Note**: Complex architecture (handlers/, filters/, cache/, etc.) is deferred to post-POC phases.

### Key Patterns (POC - Keep Simple)

1. **Async I/O**: Use `asyncio` and `aiohttp` for concurrent connections
2. **Single File**: All logic in one file (~200 lines) for simplicity
3. **Basic Config**: YAML file with port, timeout, log level
4. **Stdout Logging**: Simple print statements, no complex logging

### Request Flows (POC)

**HTTP Forwarding:**
```
Client → Proxy → Parse Request → Forward to Server → Stream Response → Client
```

**HTTPS Tunneling (CONNECT):**
```
Client → CONNECT request → Proxy → Establish TCP to Server
       → 200 OK → Client
       → Bidirectional tunnel (pass-through encrypted data)
```

## Development Commands

### Project Setup

```bash
# Initialize Python environment (uv preferred)
uv venv
source .venv/bin/activate  # macOS/Linux
uv pip install -e ".[dev]"

# Alternative: Using poetry
poetry install --with dev

# Alternative: Using pip
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

### Code Quality

```bash
# Format code
black src/ tests/
isort src/ tests/

# Lint
ruff check src/ tests/
flake8 src/ tests/

# Type checking
mypy src/

# Security scan
bandit -r src/

# Run all quality checks
black . && isort . && ruff check . && mypy src/ && bandit -r src/
```

### Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=proxy --cov-report=html --cov-report=term-missing

# Run specific test file
pytest tests/unit/test_handlers.py -v

# Run specific test function
pytest tests/unit/test_handlers.py::test_http_get -v

# Run integration tests only
pytest tests/integration/ -v

# Run with asyncio debug mode
pytest --asyncio-mode=auto -v

# Generate coverage report
pytest --cov=proxy --cov-report=html
# View: open htmlcov/index.html
```

### Running the Proxy

```bash
# Development mode (with auto-reload)
python -m proxy.server --config config/proxy.dev.yaml --reload

# Production mode
python -m proxy.server --config config/proxy.prod.yaml

# With specific host/port
python -m proxy.server --host 0.0.0.0 --port 8080

# With environment variable overrides
LOG_LEVEL=debug python -m proxy.server

# Using Docker
docker build -t custom-http-proxy .
docker run -p 8080:8080 -v $(pwd)/config:/app/config custom-http-proxy

# Using Docker Compose
docker-compose up -d
docker-compose logs -f proxy
docker-compose down
```

### Development Workflow

```bash
# Start development server with live reload
./scripts/run-dev.sh

# Run tests in watch mode (requires pytest-watch)
ptw -- -v

# Check proxy health
curl http://localhost:8080/health/liveness
curl http://localhost:8080/health/readiness

# View metrics
curl http://localhost:9090/metrics

# Test basic forwarding
curl -x http://localhost:8080 http://example.com

# Test HTTPS tunneling
curl -x http://localhost:8080 https://example.com

# Test CLI streaming
curl -X POST http://localhost:8080/cli/execute \
  -H "Content-Type: application/json" \
  -d '{"command":"python","args":["script.py"],"stream":true}'
```

## Configuration

### Configuration Files

```
config/
├── proxy.yaml          # Main configuration
├── proxy.dev.yaml      # Development overrides
├── proxy.prod.yaml     # Production overrides
└── filters.yaml        # Filtering rules
```

### Configuration Structure

```yaml
server:
  host: "0.0.0.0"
  port: 8080
  tls:
    enabled: true
    cert: "/path/to/cert.pem"
    key: "/path/to/key.pem"
  timeouts:
    read: 30s
    write: 30s
    idle: 120s

routing:
  default_backend: "http://localhost:8000"
  rules:
    - match: {host: "api.example.com"}
      forward: {target: "http://backend-api:8080"}

security:
  filtering:
    mode: "blocklist"
    domains:
      blocked: ["malicious.com", "*.ads.example.com"]
  rate_limiting:
    enabled: true
    requests_per_second: 100
    burst: 200

cache:
  enabled: true
  max_memory: "256MB"
  default_ttl: 300

cli:
  enabled: true
  max_concurrent: 10
  process_timeout: 60s
  allowed_commands:
    - "/usr/bin/python3"
    - "/usr/bin/node"

logging:
  level: "info"
  format: "json"

metrics:
  enabled: true
  port: 9090
```

### Environment Variables

```bash
# Server configuration
PROXY_HOST=0.0.0.0
PROXY_PORT=8080

# Logging
LOG_LEVEL=info        # debug, info, warn, error
LOG_FORMAT=json       # json, text

# Cache
CACHE_ENABLED=true
CACHE_MAX_MEMORY=256MB

# Security
RATE_LIMIT_ENABLED=true
RATE_LIMIT_RPS=100

# CLI
CLI_ENABLED=true
CLI_MAX_CONCURRENT=10

# Backends
DEFAULT_BACKEND=http://localhost:8000
```

## Testing Strategy

### Test Organization

```
tests/
├── conftest.py              # Shared fixtures
├── unit/                    # Fast, isolated tests
│   ├── test_handlers.py     # Handler logic
│   ├── test_filters.py      # Filtering logic
│   ├── test_cache.py        # Cache operations
│   └── test_streaming.py    # CLI streaming
├── integration/             # End-to-end tests
│   ├── test_proxy_flow.py   # Full request flow
│   ├── test_cli_streaming.py # CLI integration
│   └── test_caching.py      # Cache behavior
└── fixtures/                # Test data
    ├── requests/
    └── responses/
```

### Test Fixtures

Key fixtures defined in `conftest.py`:
- `event_loop`: Async event loop for tests
- `mock_backend`: Mock HTTP backend server
- `proxy_server`: Test proxy instance
- `test_client`: HTTP client for requests
- `test_config`: Test configuration

### Coverage Targets

- **Unit Tests**: 80%+ coverage
- **Integration Tests**: Critical paths covered
- **Performance Tests**: Load testing for 1000+ RPS

## API Reference

### Health Endpoints

```
GET /health/liveness      # Is proxy running?
GET /health/readiness     # Is proxy ready to serve?
GET /health/startup       # Completed initialization?
```

### Metrics Endpoint

```
GET /metrics              # Prometheus format metrics
```

### CLI Execution Endpoint

```
POST /cli/execute
Content-Type: application/json

{
  "command": "python",
  "args": ["script.py"],
  "stdin": "input data",
  "stream": true
}

Response: Transfer-Encoding: chunked
{"type":"stdout","data":"output"}
{"type":"exit","code":0}
```

### Admin Endpoints

```
DELETE /admin/cache?pattern=*.example.com  # Invalidate cache
GET /admin/cache/stats                     # Cache statistics
```

## Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Request Latency (p95) | <50ms | Load testing |
| Throughput | 1000+ RPS | Sustained load |
| Concurrent Connections | 1000+ | Connection pooling |
| Cache Hit Rate | >70% | Cache metrics |
| Memory Usage | <512MB | Container metrics |
| Uptime | 99.9% | Health checks |

## Security Considerations

### Input Validation

All inputs must be validated:
- URL format and length limits (max 8192 chars)
- Header validation (RFC compliance, max 16KB)
- Request body size limits (max 10MB)
- CLI command allowlist

### TLS Configuration

- Minimum TLS 1.2 (prefer TLS 1.3)
- Strong cipher suites only
- Certificate validation for backends
- Support for custom CA certificates

### Rate Limiting

- Token bucket algorithm
- Per-client IP tracking
- Configurable limits and burst
- 429 Too Many Requests response

### Security Headers

Injected into all responses:
- `Strict-Transport-Security`: HSTS enforcement
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Content-Security-Policy`: Configurable CSP

## Troubleshooting

### High Latency

```bash
# Check backend health
curl http://localhost:9090/metrics | grep backend_healthy

# Check cache hit rate
curl http://localhost:9090/metrics | grep cache_hit

# Review slow requests in logs
docker logs proxy | grep '"duration_ms":[0-9]{3,}'
```

### High Error Rate

```bash
# Check error distribution
curl http://localhost:9090/metrics | grep http_requests_total

# Review recent errors
docker logs proxy --since 10m | grep ERROR

# Verify backend connectivity
curl -I http://backend:8080/health
```

### Memory Issues

```bash
# Check memory usage
docker stats proxy

# Check cache size
curl http://localhost:8080/admin/cache/stats

# Review for memory leaks (development)
python -m memory_profiler src/proxy/server.py
```

## Task Master AI Integration

This project uses Task Master AI for project management. **20 tasks are already defined** covering all 4 development phases.

### MCP Server Integration

Task Master AI is integrated via MCP server (`.mcp.json`):
```json
{
  "mcpServers": {
    "task-master-ai": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "task-master-ai"]
    }
  }
}
```

### Key Commands

```bash
# View all tasks
task-master list

# Get next task to work on
task-master next

# View task details
task-master show <id>

# Update task status
task-master set-status --id=<id> --status=done

# Expand task into subtasks
task-master expand --id=<id>

# Parse PRD to generate tasks (already done)
task-master parse-prd .taskmaster/docs/prd.txt
```

### Current Task Status

All 20 tasks are defined in `.taskmaster/tasks/tasks.json`:
1. **Task #1** (pending): Project Setup and Foundation - Ready to start!
2. **Tasks #2-20**: Configuration, HTTP server, handlers, security, performance features

For comprehensive Task Master AI documentation, see `.taskmaster/CLAUDE.md`.

### API Keys Setup

The project requires API keys for Task Master AI functionality. Copy `.env.example` to `.env` and configure:

```bash
# Copy environment template
cp .env.example .env

# Edit with your API keys (at minimum, configure ANTHROPIC_API_KEY)
# See .env.example for all available API providers
```

Required for Task Master AI:
- `ANTHROPIC_API_KEY`: Required for Claude models (primary)
- `PERPLEXITY_API_KEY`: Optional for research features
- Other providers: OpenAI, Google, Mistral, xAI, Groq, OpenRouter, etc.

## Docker Deployment

### Building

```bash
# Build image
docker build -t custom-http-proxy:latest .

# Build with specific Python version
docker build --build-arg PYTHON_VERSION=3.11 -t custom-http-proxy:latest .
```

### Running

```bash
# Run with Docker
docker run -d \
  -p 8080:8080 \
  -p 9090:9090 \
  -v $(pwd)/config:/app/config \
  -e LOG_LEVEL=info \
  --name proxy \
  custom-http-proxy:latest

# Run with Docker Compose
docker-compose up -d

# View logs
docker logs -f proxy

# Execute shell in container
docker exec -it proxy /bin/bash
```

### Health Checks

```bash
# Docker health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health/liveness || exit 1
```

## Production Deployment

### Kubernetes

```yaml
# Deployment with 3 replicas
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-proxy
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: proxy
        image: custom-http-proxy:latest
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health/liveness
            port: 8080
        readinessProbe:
          httpGet:
            path: /health/readiness
            port: 8080
```

### Monitoring

- **Prometheus**: Scrape `/metrics` endpoint
- **Grafana**: Visualize metrics (RPS, latency, errors, cache hit rate)
- **Alerting**: Configure alerts for high error rate, high latency, proxy down

## Code Style Guidelines

### Python Style

- **Formatting**: Black (88 char line length)
- **Import Sorting**: isort (black profile)
- **Type Hints**: Required for all function signatures
- **Docstrings**: Google style for public APIs
- **Naming**: `snake_case` for functions/variables, `PascalCase` for classes

### Async Patterns

```python
# Correct: Use async/await consistently
async def handle_request(request):
    response = await fetch_from_backend(request)
    return response

# Correct: Use asyncio.gather for concurrent operations
results = await asyncio.gather(
    fetch_backend1(),
    fetch_backend2(),
    fetch_backend3()
)

# Correct: Proper async context managers
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.json()
```

### Error Handling

```python
# Use custom exceptions for domain errors
class ProxyError(Exception):
    """Base exception for proxy errors"""

class BackendTimeoutError(ProxyError):
    """Backend server timeout"""

class DomainBlockedError(ProxyError):
    """Domain blocked by filter"""

# Proper error context
try:
    response = await backend.fetch(url)
except asyncio.TimeoutError as e:
    raise BackendTimeoutError(f"Backend {url} timed out") from e
```

### Logging

```python
# Use structured logging
import structlog

logger = structlog.get_logger()

# Log with context
logger.info(
    "request_completed",
    client_ip="192.168.1.100",
    method="GET",
    url="https://example.com",
    status=200,
    duration_ms=45
)
```

## Important Files

- `Product Requirements Document v2.0 (Custom HTTP Proxy).md`: Complete PRD with architecture, API specs, testing strategy
- `.taskmaster/CLAUDE.md`: Task Master AI integration guide
- `.serena/project.yml`: Serena MCP configuration
- `.mcp.json`: MCP server configuration
- `pyproject.toml`: Python project configuration (when created)
- `Dockerfile`: Container image definition (when created)
- `docker-compose.yml`: Multi-container setup (when created)

## Current Project Status

**Phase**: Planning Complete, Development Ready to Start

**Tasks Defined**: 20 tasks covering all 4 development phases
- ✅ PRD parsed and task structure created
- ✅ Task dependencies established
- ✅ Implementation details and test strategies defined
- 📋 Ready to begin Task #1: Project Setup and Foundation

**Next Immediate Steps**:

1. Set up API keys: `cp .env.example .env` and configure at minimum `ANTHROPIC_API_KEY`
2. Get next task: Use Task Master AI MCP tools or run `task-master next`
3. Start Task #1: Create `pyproject.toml` with dependencies
4. Set up project structure (`src/`, `tests/`, `config/`)
5. Continue with tasks in dependency order

**Development Workflow**:
- Use Task Master AI to track progress through all 20 tasks
- Each task includes detailed implementation guidance
- Test strategies defined for validation
- Follow dependency chain for logical development order

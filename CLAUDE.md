# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Custom HTTP/HTTPS proxy server acting as an intermediary between clients and backend servers, providing performance optimization (caching, compression), security (TLS, content filtering), and real-time CLI process execution with streaming support.

**Key Goals:**
- High performance: <50ms latency (p95), 1000+ RPS per instance
- Security: TLS 1.2+, domain filtering, rate limiting, input validation
- Streaming: Real-time CLI process execution with HTTP chunked encoding
- Scalability: Stateless design, horizontal scaling, 99.9% uptime target

## Development Phases

The project follows a 4-phase development approach:

1. **MVP (Phase 1)**: Core HTTP/HTTPS proxy with basic forwarding, CONNECT tunneling, CLI integration
2. **Phase 2**: Security features - domain filtering, rate limiting, ACLs, structured logging
3. **Phase 3**: Performance optimization - caching, compression, connection pooling, metrics
4. **Phase 4**: Advanced features - HTTP/2, multiple CLI processes, Prometheus export, load balancing

## Technology Stack

**Primary Language:** Python 3.8+ (3.11+ recommended)

**Core Libraries:**
- `aiohttp`: Async HTTP server/client framework
- `asyncio`: Event-driven I/O
- `pydantic` / `pydantic-settings`: Configuration with type safety
- `structlog`: Structured logging
- `prometheus-client`: Metrics exposition
- `cryptography` / `pyOpenSSL`: TLS operations
- `aiocache`: Async caching
- `pytest` + `pytest-asyncio`: Testing framework

**Package Management:** `uv` (preferred) or `poetry`

## Architecture

### Component Structure

```
src/proxy/
├── server.py           # Main proxy server, event loop, listener
├── config.py           # Configuration loading/validation (pydantic)
├── logging.py          # Structured logging setup (structlog)
├── metrics.py          # Prometheus metrics exposition
├── handlers/           # Request/response handlers
│   ├── base.py         # Abstract base handler
│   ├── http.py         # HTTP/1.1 request handling
│   ├── https.py        # TLS termination/re-encryption
│   └── connect.py      # CONNECT method (HTTPS tunneling)
├── filters/            # Content filtering & security
│   ├── base.py         # Filter interface (plugin pattern)
│   ├── domain_block.py # Domain blocking
│   ├── malware.py      # Content inspection
│   └── access_control.py # IP-based ACL
├── cache/              # Caching layer
│   ├── memory.py       # In-memory cache (default)
│   └── redis.py        # Redis cache (optional)
├── streaming/          # CLI integration
│   ├── process.py      # Async subprocess management
│   └── chunked.py      # HTTP chunked encoding
├── compression/        # Response compression
│   ├── gzip.py
│   └── brotli.py
├── routing/            # Request routing
│   └── router.py       # Routing logic, load balancing
└── utils/              # Utilities
    ├── headers.py      # Header manipulation
    └── errors.py       # Custom exceptions
```

### Key Design Patterns

1. **Event-driven I/O**: Asyncio-based for high concurrency
2. **Middleware Pattern**: Request/response processing pipeline
3. **Plugin Architecture**: Extensible filters and handlers
4. **Stateless Design**: No in-memory session state (enables horizontal scaling)
5. **Configuration-Driven**: Rules and policies in YAML/TOML config files

### Request Flow

```
Client → Listener → Request Processor → Security Check
                                            ↓ (Pass)
                    Cache Check → (Miss) → Router
                                            ↓
                                    Backend Selection
                                            ↓
                    Backend ← Forward Request
                                            ↓
                    Receive Response ← Backend
                                            ↓
                        Response Handler → Compression
                                            ↓
                                     Inject Headers
                                            ↓
                                      Cache Store
                                            ↓
                Client ← Stream Response ← Logger
```

### HTTPS Tunneling Flow

```
Client → CONNECT request → Security Check
                              ↓ (Pass)
          Establish TCP → Backend Connection
                              ↓
Client ← 200 Connection Established
                              ↓
Client ↔ Bidirectional Tunnel ↔ Backend
```

### CLI Streaming Flow

```
Client → POST /cli/execute → CLI Handler
                                ↓
                         Validate Command
                                ↓
                         Spawn Process
                                ↓
        Client ← Chunked Response ← Stdout
        Client ← Chunked Response ← Stderr
                                ↓
        Client ← Final Chunk (exit code)
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

This project uses Task Master AI for project management. Key commands:

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

# Parse PRD to generate tasks
task-master parse-prd .taskmaster/docs/prd.txt
```

For comprehensive Task Master AI documentation, see `.taskmaster/CLAUDE.md`.

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

## Next Steps

1. Create `pyproject.toml` with dependencies
2. Set up project structure (`src/`, `tests/`, `config/`)
3. Implement MVP Phase 1 (core proxy functionality)
4. Write comprehensive tests (unit + integration)
5. Set up Docker containerization
6. Add monitoring and observability
7. Deploy to staging environment
8. Load testing and performance optimization

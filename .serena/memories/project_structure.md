# Project Structure

## Recommended Directory Layout

```
70_CUSTOM-HTTP-PROXY/
├── .claude/                    # Claude Code configuration
├── .serena/                    # Serena MCP configuration
│   └── project.yml
├── .venv/                      # Python virtual environment (gitignored)
├── src/
│   └── proxy/                  # Main package
│       ├── __init__.py
│       ├── __main__.py         # CLI entry point
│       ├── server.py           # Main proxy server
│       ├── config.py           # Configuration management
│       ├── logging.py          # Logging setup
│       ├── metrics.py          # Metrics collection
│       │
│       ├── handlers/           # Request/response handlers
│       │   ├── __init__.py
│       │   ├── base.py         # Base handler class
│       │   ├── http.py         # HTTP/1.1 handler
│       │   ├── https.py        # HTTPS handler
│       │   └── connect.py      # CONNECT method (tunneling)
│       │
│       ├── filters/            # Content filtering
│       │   ├── __init__.py
│       │   ├── base.py         # Base filter interface
│       │   ├── domain_block.py # Domain blocking
│       │   ├── malware.py      # Malware detection
│       │   └── access_control.py # IP whitelist/blacklist
│       │
│       ├── cache/              # Caching layer
│       │   ├── __init__.py
│       │   ├── base.py         # Cache interface
│       │   ├── memory.py       # In-memory cache
│       │   └── redis.py        # Redis cache (optional)
│       │
│       ├── streaming/          # CLI integration & streaming
│       │   ├── __init__.py
│       │   ├── process.py      # Process execution
│       │   └── chunked.py      # Chunked encoding
│       │
│       ├── compression/        # Response compression
│       │   ├── __init__.py
│       │   ├── gzip.py
│       │   └── brotli.py
│       │
│       ├── routing/            # Request routing
│       │   ├── __init__.py
│       │   └── router.py       # Routing logic
│       │
│       └── utils/              # Utility functions
│           ├── __init__.py
│           ├── headers.py      # Header manipulation
│           └── errors.py       # Custom exceptions
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py             # Pytest fixtures
│   │
│   ├── unit/                   # Unit tests
│   │   ├── test_server.py
│   │   ├── test_handlers.py
│   │   ├── test_filters.py
│   │   ├── test_cache.py
│   │   └── test_streaming.py
│   │
│   ├── integration/            # Integration tests
│   │   ├── test_proxy_flow.py
│   │   ├── test_cli_streaming.py
│   │   └── test_caching.py
│   │
│   └── fixtures/               # Test data
│       ├── requests/
│       └── responses/
│
├── config/                     # Configuration files
│   ├── proxy.yaml              # Main configuration
│   ├── proxy.dev.yaml          # Development config
│   ├── proxy.prod.yaml         # Production config
│   └── filters.yaml            # Filtering rules
│
├── docs/                       # Documentation
│   ├── architecture.md
│   ├── api.md
│   ├── deployment.md
│   └── development.md
│
├── scripts/                    # Utility scripts
│   ├── setup.sh
│   ├── run-dev.sh
│   └── run-tests.sh
│
├── logs/                       # Log files (gitignored)
│   └── .gitkeep
│
├── .gitignore
├── .dockerignore
├── .pre-commit-config.yaml     # Pre-commit hooks
├── pyproject.toml              # Python project config
├── requirements.txt            # Production dependencies
├── requirements-dev.txt        # Development dependencies
├── Dockerfile
├── docker-compose.yml
├── README.md
├── LICENSE
└── CHANGELOG.md
```

## Key Directories Explained

### `/src/proxy/`
Main application package. All production code lives here.

**Core modules**:
- `server.py`: Main proxy server, event loop, listener
- `config.py`: Configuration loading, validation (using pydantic)
- `logging.py`: Structured logging setup (using structlog)
- `metrics.py`: Prometheus metrics exposition

### `/src/proxy/handlers/`
HTTP request/response handling logic.

- `base.py`: Abstract base handler class
- `http.py`: Standard HTTP/1.1 request handling
- `https.py`: TLS termination and re-encryption
- `connect.py`: CONNECT method for HTTPS tunneling

### `/src/proxy/filters/`
Content filtering and security policies.

- `base.py`: Filter interface (plugin pattern)
- `domain_block.py`: URL/domain blocking
- `malware.py`: Malicious content detection
- `access_control.py`: IP-based access control

### `/src/proxy/cache/`
Caching layer for static content.

- `memory.py`: In-memory cache (default)
- `redis.py`: Redis-backed cache (scalable)

### `/src/proxy/streaming/`
CLI process execution and real-time streaming.

- `process.py`: Async subprocess management
- `chunked.py`: HTTP chunked transfer encoding

### `/tests/`
Comprehensive test suite.

- `unit/`: Fast, isolated tests for individual components
- `integration/`: End-to-end tests for complete flows
- `fixtures/`: Reusable test data and mock responses
- `conftest.py`: Shared pytest fixtures and configuration

### `/config/`
YAML or TOML configuration files.

**Example structure** (`proxy.yaml`):
```yaml
server:
  host: 0.0.0.0
  port: 8080
  tls:
    enabled: true
    cert: /path/to/cert.pem
    key: /path/to/key.pem

routing:
  rules:
    - path: /api/*
      backend: http://api.internal:8000
    - path: /static/*
      backend: http://cdn.internal:8080

filters:
  enabled:
    - domain_block
    - access_control
  domain_block:
    blocklist:
      - example-bad.com
      - malware-site.net
  access_control:
    whitelist:
      - 192.168.1.0/24
      - 10.0.0.0/8

cache:
  enabled: true
  backend: memory
  ttl: 3600
  max_size: 1000

logging:
  level: INFO
  format: json
  output: /var/log/proxy/access.log

metrics:
  enabled: true
  endpoint: /metrics
```

### `/docs/`
Project documentation (Markdown).

- `architecture.md`: System architecture, components
- `api.md`: API reference, endpoints
- `deployment.md`: Deployment guide (Docker, K8s)
- `development.md`: Development setup, workflows

### `/scripts/`
Development and deployment scripts.

**Examples**:
- `setup.sh`: Initial project setup
- `run-dev.sh`: Start development server
- `run-tests.sh`: Run full test suite
- `deploy.sh`: Deployment automation

## File Naming Conventions

### Python Files
- **Modules**: `lowercase_with_underscores.py`
- **Packages**: `lowercase` (no underscores if possible)
- **Tests**: `test_<module_name>.py` or `<module_name>_test.py`

### Configuration Files
- **YAML**: `*.yaml` (preferred over `.yml`)
- **TOML**: `pyproject.toml`
- **Environment**: `.env` (gitignored), `.env.example` (template)

### Documentation
- **Markdown**: `*.md` (lowercase, hyphens for spaces)
- Examples: `getting-started.md`, `api-reference.md`

## Important Files

### `pyproject.toml`
Unified Python project configuration (PEP 518).

Contains:
- Package metadata
- Dependencies
- Tool configurations (black, isort, mypy, pytest, etc.)

**Example**:
```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "custom-http-proxy"
version = "0.1.0"
description = "Custom HTTP/HTTPS proxy with caching, filtering, and CLI streaming"
authors = [{name = "Your Name", email = "you@example.com"}]
readme = "README.md"
requires-python = ">=3.8"
dependencies = [
    "aiohttp>=3.9.0",
    "pydantic>=2.0.0",
    "structlog>=23.0.0",
    "prometheus-client>=0.18.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-asyncio>=0.21.0",
    "pytest-cov>=4.1.0",
    "black>=23.0.0",
    "isort>=5.12.0",
    "flake8>=6.1.0",
    "mypy>=1.5.0",
    "bandit>=1.7.5",
]

[tool.black]
line-length = 88
target-version = ['py38', 'py39', 'py310', 'py311']

[tool.isort]
profile = "black"
line_length = 88

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = "-v --cov=proxy --cov-report=term-missing"

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

### `.gitignore`
**Essential entries**:
```gitignore
# Python
__pycache__/
*.py[cod]
*.so
.Python
.venv/
venv/
ENV/
env/
*.egg-info/
dist/
build/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Logs
logs/
*.log

# Environment variables
.env
.env.local

# Coverage
.coverage
htmlcov/
.pytest_cache/

# macOS
.DS_Store

# Docker
docker-compose.override.yml
```

### `Dockerfile`
Multi-stage build for optimized image size.

**Example**:
```dockerfile
FROM python:3.11-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.11-slim

WORKDIR /app
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY src/ ./src/
COPY config/ ./config/

EXPOSE 8080
CMD ["python", "-m", "proxy.server", "--config", "config/proxy.yaml"]
```

## Codebase Organization Principles

1. **Separation of Concerns**: Each module has a single, clear responsibility
2. **Dependency Injection**: Pass dependencies explicitly, avoid global state
3. **Testability**: Write code that's easy to test in isolation
4. **Extensibility**: Use plugin patterns for filters, handlers, caches
5. **Configuration-Driven**: Minimize hardcoded values, use config files

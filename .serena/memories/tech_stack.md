# Technology Stack

## Core Language
- **Python 3.8+** (minimum)
- **Python 3.11+** (recommended for performance)

## Key Python Libraries (Recommended)

### HTTP/Proxy Core
- **aiohttp**: Async HTTP client/server framework
- **asyncio**: Core async I/O support
- **httpx**: Alternative modern HTTP client with HTTP/2 support
- **h11** or **h2**: Low-level HTTP/1.1 and HTTP/2 protocol implementation
- **uvloop**: High-performance event loop (drop-in asyncio replacement)

### TLS/Security
- **cryptography**: TLS certificate handling
- **pyOpenSSL**: SSL/TLS operations
- **certifi**: CA certificate bundle

### Caching & Compression
- **aiocache**: Async caching framework
- **brotli** / **brotlicffi**: Brotli compression
- **gzip** (built-in): GZIP compression

### Process Management
- **asyncio.subprocess**: Async process execution
- **sh** or **subprocess**: CLI tool invocation

### Logging & Monitoring
- **structlog**: Structured logging
- **prometheus-client**: Metrics exposition
- **python-json-logger**: JSON-formatted logs

### Configuration
- **pydantic** / **pydantic-settings**: Config validation with type safety
- **python-dotenv**: Environment variable loading
- **PyYAML** or **toml**: Config file parsing

### Testing
- **pytest**: Testing framework
- **pytest-asyncio**: Async test support
- **pytest-cov**: Coverage reporting
- **pytest-mock**: Mocking utilities
- **httpx** (test client): HTTP testing
- **aioresponses**: Mock aiohttp responses

### Code Quality
- **black**: Code formatter (opinionated)
- **isort**: Import sorting
- **flake8** or **ruff**: Linting
- **mypy**: Static type checking
- **pylint**: Advanced linting
- **bandit**: Security linting

### Development Tools
- **uv** (preferred): Fast Python package installer and resolver
- **poetry** (alternative): Dependency management and packaging
- **pip-tools**: Requirements compilation
- **pre-commit**: Git hooks for code quality

## Architecture Patterns
- **Event-driven I/O**: Asyncio-based for high concurrency
- **Stateless design**: No in-memory session state (scalability)
- **Middleware pattern**: Request/response processing pipeline
- **Plugin architecture**: Extensible filters and handlers
- **Configuration-driven**: Rules and policies in config files

## Deployment Stack
- **Docker**: Containerization
- **Docker Compose**: Local development
- **Kubernetes**: Production orchestration
- **Nginx** or **HAProxy**: Load balancer (optional, external)
- **Prometheus + Grafana**: Monitoring
- **ELK Stack** or **Loki**: Log aggregation

## Development Environment
- **macOS** (Darwin): Current development platform
- **Linux containers**: Deployment target
- **Git**: Version control
- **VS Code** or **PyCharm**: IDE

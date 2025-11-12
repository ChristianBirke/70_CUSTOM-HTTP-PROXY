# Code Style and Conventions

## Python Style Guide
Follow **PEP 8** (Python Enhancement Proposal 8) as the base style guide.

## Code Formatting

### Black (Code Formatter)
- **Line length**: 88 characters (Black default)
- **String quotes**: Double quotes preferred
- **Configuration**: `pyproject.toml`

```toml
[tool.black]
line-length = 88
target-version = ['py38', 'py39', 'py310', 'py311']
include = '\.pyi?$'
```

### isort (Import Sorting)
- **Profile**: black-compatible
- **Multi-line**: vertical hanging indent

```toml
[tool.isort]
profile = "black"
line_length = 88
multi_line_output = 3
include_trailing_comma = true
```

## Type Hints
**REQUIRED** for all function signatures and class attributes.

```python
from typing import Optional, Dict, List, Union
import asyncio

async def handle_request(
    url: str,
    headers: Dict[str, str],
    timeout: Optional[float] = None
) -> Dict[str, Union[str, int]]:
    """Handle incoming HTTP request."""
    ...
```

### Type Checking
- Use **mypy** for static type checking
- Strict mode enabled in production code

```toml
[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

## Docstrings
Use **Google-style docstrings** for all public modules, classes, and functions.

```python
def cache_response(key: str, data: bytes, ttl: int = 3600) -> bool:
    """Cache HTTP response data.

    Args:
        key: Cache key (typically URL)
        data: Response body bytes
        ttl: Time-to-live in seconds (default: 3600)

    Returns:
        True if cached successfully, False otherwise.

    Raises:
        CacheError: If cache backend is unavailable.
    """
    ...
```

## Naming Conventions

### Variables and Functions
- **snake_case**: `handle_request`, `client_ip`, `is_blocked`
- **Descriptive names**: Avoid abbreviations unless obvious
- **Boolean variables**: Start with `is_`, `has_`, `should_`, `can_`

### Classes
- **PascalCase**: `ProxyServer`, `RequestHandler`, `CacheManager`
- **Exception classes**: End with `Error` (e.g., `ProxyError`, `FilterError`)

### Constants
- **UPPER_SNAKE_CASE**: `MAX_CONNECTIONS`, `DEFAULT_TIMEOUT`, `HTTP_PORT`

### Private Members
- **Single underscore prefix**: `_internal_method`, `_cache_store`
- **Double underscore**: Avoid unless name mangling is explicitly needed

## Project Structure

```
proxy/
├── src/
│   └── proxy/
│       ├── __init__.py
│       ├── server.py          # Main proxy server
│       ├── handlers/           # Request/response handlers
│       │   ├── __init__.py
│       │   ├── http.py
│       │   └── https.py
│       ├── filters/            # Content filtering
│       │   ├── __init__.py
│       │   ├── base.py
│       │   └── domain_block.py
│       ├── cache/              # Caching layer
│       │   ├── __init__.py
│       │   └── memory.py
│       ├── streaming/          # CLI integration & streaming
│       │   ├── __init__.py
│       │   └── process.py
│       ├── config.py           # Configuration management
│       ├── logging.py          # Logging setup
│       └── metrics.py          # Metrics collection
├── tests/
│   ├── unit/
│   └── integration/
├── config/
│   └── proxy.yaml
├── pyproject.toml
├── README.md
├── Dockerfile
└── docker-compose.yml
```

## Async/Await Patterns
- **Always use async/await** for I/O operations
- **Avoid blocking calls** in async context
- **Use `asyncio.create_task()`** for concurrent tasks
- **Proper exception handling** in async code

```python
async def proxy_request(request: Request) -> Response:
    """Proxy HTTP request to backend."""
    try:
        async with httpx.AsyncClient() as client:
            response = await client.request(
                method=request.method,
                url=request.url,
                headers=request.headers,
                content=request.body
            )
            return response
    except httpx.TimeoutException:
        raise ProxyTimeoutError("Backend timeout")
```

## Error Handling
- **Specific exceptions**: Catch specific exceptions, not bare `except:`
- **Custom exceptions**: Define domain-specific exception hierarchy
- **Logging**: Log exceptions with context before re-raising
- **Graceful degradation**: Return meaningful HTTP error codes

```python
class ProxyError(Exception):
    """Base exception for proxy errors."""

class BackendError(ProxyError):
    """Backend server error."""

class FilterError(ProxyError):
    """Content filtering error."""
```

## Testing Conventions
- **File naming**: `test_*.py` or `*_test.py`
- **Test function naming**: `test_<functionality>_<condition>_<expected>`
- **Fixtures**: Use pytest fixtures for setup/teardown
- **Async tests**: Mark with `@pytest.mark.asyncio`
- **Coverage target**: >80% line coverage

```python
@pytest.mark.asyncio
async def test_proxy_forwards_get_request_successfully():
    """Test that proxy correctly forwards GET request to backend."""
    ...
```

## Comments
- **Code should be self-documenting**: Minimize inline comments
- **Explain WHY, not WHAT**: Comments explain reasoning, not obvious code
- **TODO format**: `# TODO(username): Description of task`

## Security Best Practices
- **Input validation**: Validate all external inputs
- **No hardcoded secrets**: Use environment variables
- **Least privilege**: Run with minimal permissions
- **Dependencies**: Keep up-to-date, audit with `pip-audit` or `safety`
- **Bandit**: Security linting for common vulnerabilities

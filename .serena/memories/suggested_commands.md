# Suggested Commands

## Project Setup

### Initial Setup (uv - Recommended)
```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment
uv venv

# Activate virtual environment
source .venv/bin/activate  # macOS/Linux

# Install dependencies
uv pip install -e ".[dev]"
```

### Alternative Setup (poetry)
```bash
# Install poetry
curl -sSL https://install.python-poetry.org | python3 -

# Install dependencies
poetry install

# Activate virtual environment
poetry shell
```

### Alternative Setup (pip)
```bash
# Create virtual environment
python3 -m venv .venv

# Activate
source .venv/bin/activate

# Install dependencies
pip install -e ".[dev]"
```

## Development Workflow

### Code Quality (Run before committing)
```bash
# Format code with Black
black src/ tests/

# Sort imports
isort src/ tests/

# Lint with flake8
flake8 src/ tests/

# Type check with mypy
mypy src/

# Security check with bandit
bandit -r src/

# All quality checks in one command
black src/ tests/ && isort src/ tests/ && flake8 src/ tests/ && mypy src/
```

### Testing
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=proxy --cov-report=html --cov-report=term

# Run specific test file
pytest tests/unit/test_server.py

# Run tests matching pattern
pytest -k "test_proxy_forwards"

# Run with verbose output
pytest -v

# Run only async tests
pytest -m asyncio

# Run with live logging
pytest --log-cli-level=INFO
```

### Running the Proxy
```bash
# Development mode (with auto-reload)
python -m proxy.server --config config/proxy.yaml --reload

# Production mode
python -m proxy.server --config config/proxy.yaml

# With custom port
python -m proxy.server --port 8080

# With debug logging
python -m proxy.server --log-level DEBUG
```

## Docker Commands

### Build and Run
```bash
# Build Docker image
docker build -t custom-http-proxy:latest .

# Run container
docker run -p 8080:8080 -v $(pwd)/config:/app/config custom-http-proxy:latest

# Run with environment variables
docker run -p 8080:8080 \
  -e PROXY_PORT=8080 \
  -e LOG_LEVEL=INFO \
  custom-http-proxy:latest

# Run with docker-compose
docker-compose up

# Build and run
docker-compose up --build

# Run in background
docker-compose up -d

# View logs
docker-compose logs -f

# Stop containers
docker-compose down
```

### Docker Cleanup
```bash
# Remove stopped containers
docker-compose down

# Remove containers and volumes
docker-compose down -v

# Prune unused images
docker image prune
```

## Dependency Management

### uv
```bash
# Add dependency
uv pip install aiohttp

# Add dev dependency
uv pip install --dev pytest

# Update all dependencies
uv pip install --upgrade --all

# List installed packages
uv pip list
```

### poetry
```bash
# Add dependency
poetry add aiohttp

# Add dev dependency
poetry add --group dev pytest

# Update dependencies
poetry update

# Show dependency tree
poetry show --tree
```

### pip
```bash
# Install from requirements
pip install -r requirements.txt

# Freeze dependencies
pip freeze > requirements.txt

# Install development dependencies
pip install -r requirements-dev.txt
```

## Git Workflow

### Pre-commit Hooks
```bash
# Install pre-commit
pip install pre-commit

# Install hooks
pre-commit install

# Run hooks manually
pre-commit run --all-files
```

### Common Git Commands
```bash
# Create feature branch
git checkout -b feature/proxy-caching

# Commit changes (hooks will run automatically)
git commit -m "Add response caching support"

# Push branch
git push origin feature/proxy-caching
```

## Monitoring & Metrics

### View Metrics Endpoint
```bash
# Check Prometheus metrics
curl http://localhost:8080/metrics

# Pretty-print JSON metrics
curl http://localhost:8080/metrics | jq
```

### View Logs
```bash
# Tail application logs
tail -f logs/proxy.log

# Filter error logs
grep ERROR logs/proxy.log

# JSON log parsing
cat logs/proxy.log | jq 'select(.level=="ERROR")'
```

## System Utilities (macOS/Darwin)

### Process Management
```bash
# Find proxy process
ps aux | grep "proxy.server"

# Kill by port
lsof -ti:8080 | xargs kill

# Check listening ports
lsof -i -P | grep LISTEN | grep 8080
```

### Network Debugging
```bash
# Test proxy connection
curl -x http://localhost:8080 http://example.com

# Test with headers
curl -x http://localhost:8080 -H "X-Custom: test" http://example.com

# Test HTTPS tunneling
curl -x http://localhost:8080 https://example.com -v

# Performance testing
ab -n 1000 -c 10 -X localhost:8080 http://example.com/
```

### File Operations
```bash
# List directory contents
ls -la

# Find Python files
find . -name "*.py" -type f

# Search in files (macOS-compatible grep)
grep -r "pattern" src/

# Count lines of code
find src/ -name "*.py" -exec wc -l {} + | tail -1
```

## Task Completion Checklist

When a task is completed, run the following commands in order:

1. **Format and lint**:
   ```bash
   black src/ tests/ && isort src/ tests/ && flake8 src/ tests/ && mypy src/
   ```

2. **Run tests**:
   ```bash
   pytest --cov=proxy --cov-report=term
   ```

3. **Security check**:
   ```bash
   bandit -r src/
   ```

4. **Commit changes** (if all checks pass):
   ```bash
   git add .
   git commit -m "Descriptive commit message"
   ```

5. **Push** (if working on a branch):
   ```bash
   git push origin <branch-name>
   ```

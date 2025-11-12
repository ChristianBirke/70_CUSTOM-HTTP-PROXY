# Task Completion Protocol

## What to Do When a Task is Completed

Every time you complete a development task (feature, bugfix, refactor, etc.), follow this protocol to ensure code quality and consistency.

## Step-by-Step Checklist

### 1. Code Formatting
Format all modified code to ensure consistency.

```bash
# Format Python code with Black
black src/ tests/

# Sort imports with isort
isort src/ tests/
```

**Expected outcome**: All code is consistently formatted, no formatting errors.

---

### 2. Linting
Check for code quality issues and PEP 8 compliance.

```bash
# Lint with flake8
flake8 src/ tests/
```

**Expected outcome**: No linting errors. Address any warnings unless they're false positives.

**Common flake8 codes**:
- `E`: PEP 8 errors
- `W`: PEP 8 warnings
- `F`: PyFlakes errors (undefined names, unused imports)
- `C`: McCabe complexity

---

### 3. Type Checking
Verify type hints are correct and complete.

```bash
# Type check with mypy
mypy src/
```

**Expected outcome**: No type errors. All functions have proper type annotations.

---

### 4. Security Scanning
Check for common security vulnerabilities.

```bash
# Security check with bandit
bandit -r src/
```

**Expected outcome**: No high or medium severity issues.

**Common issues to watch for**:
- Hardcoded secrets
- SQL injection vulnerabilities
- Insecure random number generation
- Shell injection risks
- Insecure SSL/TLS usage

---

### 5. Run Tests
Ensure all tests pass and coverage is maintained.

```bash
# Run all tests with coverage
pytest --cov=proxy --cov-report=term --cov-report=html -v
```

**Expected outcome**:
- All tests pass (100% pass rate)
- Coverage ≥80% (target)
- No new failing tests introduced

**Coverage thresholds**:
- ✅ **Excellent**: ≥90%
- ✅ **Good**: 80-89%
- ⚠️ **Acceptable**: 70-79%
- ❌ **Poor**: <70%

---

### 6. Integration Tests (if applicable)
For features that interact with external systems or multiple components.

```bash
# Run integration tests
pytest tests/integration/ -v

# Or with specific markers
pytest -m integration
```

---

### 7. Documentation
Update relevant documentation.

**Check these files**:
- [ ] README.md - If user-facing changes
- [ ] CHANGELOG.md - Log the change (if using changelog)
- [ ] Docstrings - Ensure all new functions/classes are documented
- [ ] API documentation - If API endpoints changed

---

### 8. Git Commit
Commit your changes with a descriptive message.

```bash
# Stage changes
git add .

# Commit with descriptive message
git commit -m "feat: Add response caching with LRU eviction

- Implement in-memory cache using aiocache
- Add cache hit/miss metrics
- Configure TTL via environment variable
- Add unit tests for cache behavior
"
```

**Commit message format** (Conventional Commits):
```
<type>: <short summary>

<longer description (optional)>

<footer (optional)>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `docs`: Documentation changes
- `chore`: Maintenance tasks
- `perf`: Performance improvements
- `style`: Code style changes (formatting, etc.)

---

### 9. Pre-commit Hooks (Optional but Recommended)
If pre-commit is configured, it will run automatically on commit.

```bash
# Install pre-commit hooks (one-time setup)
pre-commit install

# Run manually (if not auto-running)
pre-commit run --all-files
```

---

### 10. Push Changes (if on a branch)
```bash
# Push to remote branch
git push origin <branch-name>

# Or push with force (if rebasing, use with caution)
git push origin <branch-name> --force-with-lease
```

---

## One-Liner for Quick Checks
Run all quality checks in a single command:

```bash
black src/ tests/ && isort src/ tests/ && flake8 src/ tests/ && mypy src/ && bandit -r src/ && pytest --cov=proxy --cov-report=term
```

If all pass: ✅ Ready to commit!

---

## When to Skip Steps

### Skip Type Checking (mypy)
- Working on test files only (sometimes acceptable)
- Quick prototyping (but don't commit without fixing)

### Skip Security Scanning (bandit)
- Pure test code changes
- Documentation-only changes

### Skip Tests
- **NEVER** skip tests on main/production code
- Only acceptable for documentation or config file updates

---

## CI/CD Integration
Ensure all these checks are also run in CI/CD pipeline to prevent regression.

Typical CI pipeline:
1. Install dependencies
2. Run linting (black, isort, flake8)
3. Run type checking (mypy)
4. Run security scan (bandit)
5. Run tests with coverage
6. Build Docker image
7. Run integration tests (if applicable)

---

## Emergency Hotfixes
For critical production issues:

1. Create hotfix branch: `git checkout -b hotfix/<issue>`
2. Make minimal changes to fix the issue
3. **Still run**: linting, type checking, tests
4. Fast-track review and merge
5. Follow up with comprehensive fix if needed

---

## Notes
- **Never commit broken code**: All checks must pass before committing
- **Fix warnings**: Don't ignore linter warnings; they often indicate real issues
- **Keep coverage high**: Aim for ≥80% coverage on all new code
- **Document complex logic**: If it's not obvious, add a comment or docstring
- **Security first**: Take bandit warnings seriously, especially for network-facing code

# Product Requirements Document - Proof of Concept
## Custom HTTP Proxy

**Version:** POC-1.0
**Date:** 2025-01-15
**Status:** Active
**Author:** Simplified from over-engineered v2.0

---

## Executive Summary

This PRD defines a **lean proof-of-concept** for a basic HTTP proxy that demonstrates core forwarding functionality. The goal is to prove the concept works before adding production features.

**POC Scope:** Minimal viable proxy that forwards HTTP/HTTPS requests safely.

---

## 1. Purpose and Scope

### 1.1 Purpose
Build a simple HTTP proxy to validate the core concept of request forwarding between clients and servers.

### 1.2 POC Scope

**In Scope:**
- HTTP/1.1 forwarding (GET, POST, PUT, DELETE, HEAD)
- HTTPS tunneling via CONNECT method
- Basic request/response passthrough
- Simple logging to stdout
- Basic error handling
- Minimal configuration (one YAML file)
- Docker container for portability

**Out of Scope (Future):**
- Caching
- Compression
- Content filtering
- Rate limiting
- CLI process integration
- Advanced security features
- Performance optimization
- Prometheus metrics
- Load balancing
- HTTP/2, HTTP/3
- Complex observability

### 1.3 Success Criteria

**The POC is successful if:**
1. ✅ HTTP requests forward correctly (test with curl)
2. ✅ HTTPS tunneling works (test with https://example.com)
3. ✅ Basic errors are handled (connection failures, timeouts)
4. ✅ Code is safe and follows best practices
5. ✅ Can run in Docker container
6. ✅ Simple to configure and test

---

## 2. Functional Requirements

### 2.1 HTTP Methods Support

**Priority:** P0 (Critical for POC)

| Method | Support | Notes |
|--------|---------|-------|
| GET | ✅ Full | Core functionality |
| POST | ✅ Full | With body forwarding |
| PUT | ✅ Full | With body forwarding |
| DELETE | ✅ Full | Standard implementation |
| HEAD | ✅ Full | No body returned |
| OPTIONS | ❌ Future | Not needed for POC |
| PATCH | ❌ Future | Not needed for POC |
| CONNECT | ✅ Full | HTTPS tunneling |

### 2.2 HTTPS Tunneling

**Priority:** P0 (Critical for POC)

**Requirements:**
- Support CONNECT method for TLS tunneling
- Establish bidirectional TCP connection
- No inspection of encrypted traffic (pass-through only)
- Handle connection errors gracefully

**Implementation:**
```
Client → Proxy: CONNECT example.com:443 HTTP/1.1
Proxy → Client: HTTP/1.1 200 Connection Established
[Bidirectional tunnel established]
Client ↔ Proxy ↔ Server: [Encrypted traffic pass-through]
```

### 2.3 Request Forwarding

**Simple forwarding logic:**
1. Receive request from client
2. Parse method, URL, headers
3. Forward to target server
4. Stream response back to client
5. Log request/response (basic info)

**Headers to preserve:**
- All client headers forwarded unchanged
- Add `X-Forwarded-For` with client IP
- Add `Via: 1.1 custom-proxy` for identification

### 2.4 Error Handling

**Basic error responses:**

| Error | HTTP Status | Response |
|-------|-------------|----------|
| Connection failed | 502 Bad Gateway | Plain text error |
| Timeout | 504 Gateway Timeout | Plain text error |
| Invalid request | 400 Bad Request | Plain text error |
| Internal error | 500 Internal Server Error | Plain text error |

### 2.5 Logging

**Simple stdout logging:**
- Request: `[timestamp] method url client_ip`
- Response: `[timestamp] status duration_ms`
- Errors: `[timestamp] ERROR: message`

**Format:**
```
2025-01-15T10:30:00Z GET http://example.com 192.168.1.100
2025-01-15T10:30:00Z 200 45ms
```

---

## 3. Non-Functional Requirements

### 3.1 Performance (POC Targets)

**Goal:** Prove it works, not optimize performance

- Handle 10 concurrent connections (not 1000+)
- Reasonable latency (<500ms acceptable for POC)
- No specific throughput target

### 3.2 Security (Basic Safety)

**Input validation:**
- URL length limit: 8192 bytes
- Header size limit: 16KB total
- Request body: forward as stream (no size limit in memory)

**Safe practices:**
- No code execution
- No file system access
- Validate URLs before forwarding
- Timeout all connections (30s default)

### 3.3 Reliability (POC Level)

- Handle connection failures gracefully
- Timeout requests that hang
- Log errors for debugging
- Exit cleanly on shutdown

---

## 4. System Architecture

### 4.1 Simple Architecture

```
┌──────────┐         ┌──────────────┐         ┌──────────┐
│  Client  │ ──────> │  HTTP Proxy  │ ──────> │  Server  │
└──────────┘         └──────────────┘         └──────────┘
                            │
                            v
                      [stdout logs]
```

### 4.2 Components

**Single Python script with:**
1. **Listener:** Accept connections on port 8080
2. **Request Parser:** Parse HTTP requests
3. **Forwarder:** Forward to target server
4. **Response Handler:** Stream response to client
5. **Logger:** Print to stdout

### 4.3 Technology Stack

**Language:** Python 3.8+ (3.11+ recommended)

**Core Libraries:**
- `aiohttp`: Async HTTP client/server
- `asyncio`: Event-driven I/O
- `pyyaml`: Configuration loading

**That's it.** No complex dependencies.

---

## 5. Configuration

### 5.1 Simple Configuration File

```yaml
# config.yaml
server:
  host: "0.0.0.0"
  port: 8080
  timeout: 30  # seconds

logging:
  level: "info"  # info, debug, error
```

### 5.2 Environment Variables (Optional)

```bash
PROXY_PORT=8080
PROXY_TIMEOUT=30
LOG_LEVEL=info
```

---

## 6. Implementation Plan

### 6.1 POC Tasks (5 tasks instead of 20)

**Task 1: Project Setup** (1-2 hours)
- Create project structure
- Set up `pyproject.toml` with minimal dependencies
- Create basic README
- Set up git repository

**Task 2: HTTP Forwarding** (2-4 hours)
- Implement basic HTTP server
- Parse incoming requests
- Forward to target server
- Stream response to client
- Test with GET requests

**Task 3: HTTPS Tunneling** (2-3 hours)
- Implement CONNECT method handler
- Establish bidirectional tunnel
- Test with https://example.com

**Task 4: Error Handling & Logging** (1-2 hours)
- Add timeout handling
- Add connection error handling
- Implement stdout logging
- Test error scenarios

**Task 5: Docker & Documentation** (1-2 hours)
- Create simple Dockerfile
- Write usage documentation
- Add testing examples
- Verify everything works

**Total Estimated Time:** 7-13 hours for complete POC

---

## 7. Testing Strategy

### 7.1 Manual Testing (POC Level)

**Test 1: Basic HTTP forwarding**
```bash
# Start proxy
python proxy.py

# Test with curl
curl -x http://localhost:8080 http://example.com
```

**Test 2: HTTPS tunneling**
```bash
curl -x http://localhost:8080 https://example.com
```

**Test 3: POST with body**
```bash
curl -x http://localhost:8080 -X POST -d '{"test":"data"}' http://httpbin.org/post
```

**Test 4: Error handling**
```bash
# Test timeout (should return 504)
curl -x http://localhost:8080 http://httpstat.us/200?sleep=35000

# Test connection failure (should return 502)
curl -x http://localhost:8080 http://localhost:9999
```

### 7.2 Automated Tests (Minimal)

**Simple pytest tests:**
- Test HTTP GET forwarding
- Test HTTPS CONNECT tunneling
- Test timeout handling
- Test connection failure handling

**Coverage target:** 60% (not 80%) - POC level only

---

## 8. Project Structure

```
custom-http-proxy/
├── README.md              # Usage instructions
├── PRD-POC.md            # This document
├── pyproject.toml         # Dependencies
├── config.yaml            # Configuration
├── src/
│   └── proxy.py           # Main proxy implementation (~200 lines)
├── tests/
│   └── test_proxy.py      # Basic tests
├── Dockerfile             # Container image
└── .gitignore            # Git ignore patterns
```

---

## 9. Development Commands

### 9.1 Setup

```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install aiohttp pyyaml pytest pytest-asyncio

# Or with uv
uv venv
source .venv/bin/activate
uv pip install aiohttp pyyaml pytest pytest-asyncio
```

### 9.2 Running

```bash
# Run proxy
python src/proxy.py

# Run with custom config
python src/proxy.py --config config.yaml

# Run in Docker
docker build -t http-proxy-poc .
docker run -p 8080:8080 http-proxy-poc
```

### 9.3 Testing

```bash
# Run tests
pytest -v

# Test manually with curl
curl -x http://localhost:8080 http://example.com
curl -x http://localhost:8080 https://google.com
```

---

## 10. Success Criteria (POC)

**The POC is complete when:**

- [x] Can forward HTTP GET requests
- [x] Can forward HTTP POST requests with body
- [x] HTTPS tunneling works (CONNECT method)
- [x] Handles connection errors gracefully
- [x] Handles timeouts gracefully
- [x] Logs requests/responses to stdout
- [x] Runs in Docker container
- [x] Has basic tests
- [x] Has clear documentation

**What success looks like:**
- Works for common use cases (browsing websites via proxy)
- Code is clean and understandable (~200 lines)
- Can demo in 5 minutes
- Ready to decide: continue development or pivot

---

## 11. What Comes After POC

**If POC is successful, consider adding:**
1. Basic caching (Phase 2)
2. Simple domain filtering (Phase 3)
3. Better logging (structured JSON) (Phase 3)
4. Performance testing and optimization (Phase 4)
5. Production features from original PRD (Phase 4+)

**If POC reveals issues:**
- Reassess approach
- Adjust architecture
- Pivot to different solution

---

## 12. Key Differences from v2.0 PRD

| Aspect | v2.0 PRD | POC PRD |
|--------|----------|---------|
| **Phases** | 4 phases, 12 weeks | 1 phase, 1-2 days |
| **Tasks** | 20 tasks | 5 tasks |
| **Features** | Caching, filtering, CLI integration, metrics | Basic forwarding only |
| **Performance** | 1000+ RPS, <50ms latency | 10 concurrent connections |
| **Security** | Rate limiting, ACLs, domain filtering | Basic input validation |
| **Lines of Code** | ~2000+ lines | ~200 lines |
| **Dependencies** | 10+ libraries | 3 libraries |
| **Time to Complete** | 12 weeks | 7-13 hours |

---

## 13. Risk Assessment (POC)

**Technical Risks:** LOW
- Simple implementation reduces complexity
- Well-known libraries (aiohttp)
- Limited scope reduces bugs

**Security Risks:** MEDIUM
- No authentication (by design for POC)
- No rate limiting (by design for POC)
- Mitigation: Document limitations, only use in safe environments

**Operational Risks:** LOW
- Docker container is portable
- Simple to deploy and test
- Easy to debug with stdout logging

---

**End of Document**

**Next Step:** Start with Task 1 (Project Setup) and build incrementally.

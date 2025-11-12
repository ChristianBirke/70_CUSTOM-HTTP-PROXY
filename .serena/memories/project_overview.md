# Project Overview: Custom HTTP Proxy

## Purpose
A custom HTTP/HTTPS proxy server that acts as an intermediary between clients (browsers/apps) and backend servers, providing:

- **Performance**: Caching, compression, persistent connections
- **Security**: TLS encryption, traffic filtering, access control, malicious content blocking
- **Streaming**: Real-time CLI process execution with HTTP chunked encoding
- **Flexibility**: Customizable routing/filtering rules, header manipulation

## Key Features

### 1. HTTP/HTTPS Protocol Support
- HTTP/1.1 fully implemented (keep-alive, chunked transfer encoding)
- HTTP/2/3 support (aspirational)
- CONNECT method for HTTPS tunneling
- All standard HTTP methods (GET, POST, PUT, DELETE, OPTIONS, etc.)

### 2. Security & Filtering
- TLS termination and re-encryption
- Malicious content inspection (WAF-lite)
- IP whitelist/blacklist
- Domain-based blocking
- Client IP anonymization
- Input validation (prevent injection attacks)

### 3. Caching & Performance
- Static content caching (images, CSS/JS)
- Response compression (GZIP/Brotli)
- Connection pooling to backends
- Minimal latency overhead (<50ms target)
- High throughput (1000+ RPS target)

### 4. CLI Integration & Streaming
- Execute external CLI tools (e.g., ffmpeg for video transcoding)
- Real-time streaming of stdout/stderr to HTTP clients
- HTTP chunked encoding for incremental data delivery
- Process lifecycle management (timeout, kill on disconnect)

### 5. Observability
- Comprehensive access logs (timestamp, client IP, URL, response code, latency)
- Metrics endpoint (RPS, latency, error rates, cache hit/miss)
- Prometheus integration support
- Audit trail for policy enforcement

### 6. Deployment Architecture
- Stateless design for horizontal scaling
- Containerized deployment (Docker/Kubernetes)
- Health-check endpoints
- Load balancer compatible
- 99.9% uptime target

## Use Cases
1. **Basic Forwarding**: Proxy HTTP/HTTPS requests to backends
2. **HTTPS Tunneling**: CONNECT method for encrypted traffic
3. **Domain Blocking**: Corporate content filtering
4. **Static Caching**: Reduce backend load and bandwidth
5. **CLI Streaming**: Real-time video transcoding, data processing
6. **Monitoring**: Operations metrics and access auditing

## Non-Goals (Out of Scope)
- WebSockets, gRPC, raw TCP (beyond HTTP CONNECT)
- Graphical UI/dashboard
- Persistent data storage (database)
- Advanced WAF features (deep packet inspection)
- Third-party auth integrations (OAuth, LDAP) in v1

## Platform
- **OS**: Linux (or Linux containers)
- **Language**: Python 3.8+
- **Deployment**: Containerized (Docker)
- **Orchestration**: Kubernetes-ready

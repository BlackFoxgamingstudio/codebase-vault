# Architecture: Sovereign Codebase Vault

## Overview

**Package ID:** `PKG-020`  
**Domain:** Static Analysis & Codebase Graphs  
**Microservice Port:** `8766`  
**n8n Webhook Path:** `codebase-vault-trigger`  
**GitHub:** [BlackFoxgamingstudio/codebase-vault](https://github.com/BlackFoxgamingstudio/codebase-vault)

Deep codebase intelligence engine: AST parsing, call-graph generation, dependency analysis, dead-code detection, and semantic code search via embeddings.

---

## System Architecture

```
                     ┌──────────────────────────────────┐
                     │       Sovereign Codebase Vault      │
                     │       Port: 8766            │
                     ├──────────────┬───────────────────┤
   n8n Webhook ────▶ │  REST API    │   Core Engine     │
   HTTP POST         │  /api/v1/*   │   Dispatcher      │
                     └──────┬───────┴────────┬──────────┘
                            │                │
              ┌─────────────▼────────────────▼─────────┐
              │          Component Layer                 │
              │  ASTParser       | CallGraphBuilde | DependencyAn  │
              └────────────────────────┬────────────────┘
                                       │
              ┌────────────────────────▼────────────────┐
              │      n8n Central Event Bus (:5678)       │
              └─────────────────────────────────────────┘
```

## Core Components

### `ASTParser`
Handles all astparser operations. Exposes async methods callable from the core dispatcher.

### `CallGraphBuilder`
Handles all callgraph operations. Exposes async methods callable from the core dispatcher.

### `DependencyAnalyzer`
Handles all dependencyanalyzer operations. Exposes async methods callable from the core dispatcher.

### `DeadCodeDetector`
Handles all deadcodedetector operations. Exposes async methods callable from the core dispatcher.

### `SemanticSearchEngine`
Handles all semanticsearch operations. Exposes async methods callable from the core dispatcher.

---

## API Contract

All interactions follow the SBB standard envelope:

```http
POST /api/v1/execute
Content-Type: application/json
X-SBB-API-Key: <api-key>

{
  "action": "<operation>",
  "payload": {},
  "trace_id": "optional-uuid"
}
```

**Success Response (HTTP 200):**
```json
{
  "status": "success",
  "data": {},
  "trace_id": "...",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

**Health Check:**
```http
GET /health
→ {"status": "healthy", "service": "sovereign-codebase-vault", "port": 8766}
```

## Integration Matrix

| System | Protocol | Direction | Purpose |
|--------|----------|-----------|---------|
| n8n Event Bus (:5678) | HTTP POST | Outbound | Event forwarding |
| n8n Webhook | HTTP POST | Inbound | Trigger execution |
| SBB Codebase Vault (:8766) | HTTP | Outbound | Code analysis |
| SBB Patterns Bible (:8794) | HTTP | Outbound | Standards validation |
| External APIs | HTTPS | Outbound | Domain-specific data |

## Deployment Architecture

```yaml
# docker-compose excerpt
sovereign-codebase-vault:
  image: sovereign-codebase-vault:latest
  ports: ["8766:8766"]
  healthcheck:
    test: curl -f http://localhost:8766/health
    interval: 30s
```

## Security Model

| Control | Implementation |
|---------|---------------|
| Authentication | `X-SBB-API-Key` header (env: `SBB_API_KEY`) |
| Rate Limiting | 100 req/min per client IP |
| Input Validation | Pydantic models (strict mode) |
| Container Security | Non-root user (`appuser:1001`) |
| Secrets | Environment variables only (never hardcoded) |
| TLS | Terminate at reverse proxy (nginx/caddy) |

## Tags
`static-analysis`, `ast`, `code-graph`, `embeddings`

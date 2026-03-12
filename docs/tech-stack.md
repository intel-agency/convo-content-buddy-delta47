# ConvoContentBuddy - Technology Stack

## Overview

ConvoContentBuddy is an autonomous, real-time semantic assistant designed to listen to technical conversations and proactively display relevant algorithmic problem solutions. This document outlines the complete technology stack.

---

## Runtime & Framework

| Category | Technology | Version | Purpose |
|----------|------------|---------|---------|
| **Runtime** | .NET SDK | 10.0.102 | Primary runtime environment |
| **Language** | C# | 14 | Primary development language |
| **Orchestration** | .NET Aspire | 10 | Microservices orchestration and container management |

### Framework Components

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Backend API** | ASP.NET Core 10 | REST API, SignalR Hub, Semantic Kernel integration |
| **Frontend** | Blazor WebAssembly 10 | Zero-interaction ambient UI dashboard |
| **Service Defaults** | Aspire.ServiceDefaults | Shared resilience, health checks, OpenTelemetry |

---

## AI & Machine Learning

| Category | Technology | Purpose |
|----------|------------|---------|
| **LLM Framework** | Microsoft.SemanticKernel | AI orchestration, prompt management, plugin system |
| **AI Abstraction** | Microsoft.Extensions.AI | Unified AI provider interface |
| **Primary LLM** | Gemini 2.5 Flash Preview | Intent analysis, problem verification, solution generation |
| **Embeddings** | Gemini text-embedding-004 | Vector embeddings for semantic search (1536 dimensions) |
| **Search Grounding** | Google Search Grounding (via Gemini) | Real-time solution retrieval |

---

## Data Storage

### Vector Database

| Property | Value |
|----------|-------|
| **Engine** | Qdrant |
| **Client** | Qdrant.Client (gRPC) |
| **Collection** | `leetcode_problems` |
| **Dimensions** | 1536 |
| **Similarity** | Cosine |
| **Purpose** | Semantic similarity search for problem matching |

### Relational/Graph Database

| Property | Value |
|----------|-------|
| **Engine** | PostgreSQL |
| **Extension** | pgvector |
| **ORM** | Npgsql.EntityFrameworkCore.PostgreSQL |
| **Schema** | Problems, ProblemEdges (adjacency list) |
| **Purpose** | Problem metadata, relationship graph traversal |

### Caching & State

| Property | Value |
|----------|-------|
| **Engine** | Redis |
| **Purpose** | SignalR backplane for multi-instance state sync |
| **Library** | Microsoft.AspNetCore.SignalR.StackExchangeRedis |

---

## Real-time Communication

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Protocol** | SignalR | WebSocket-based real-time updates |
| **Hub** | BuddyHub | Server-side push of problem detections |
| **Backplane** | Redis | Multi-instance message distribution |
| **Browser API** | webkitSpeechRecognition | Speech-to-text via JavaScript interop |

---

## Resilience & Reliability

| Category | Technology | Purpose |
|----------|------------|---------|
| **Resilience Library** | Polly | Retry, Circuit Breaker, Fallback policies |
| **Package** | Microsoft.Extensions.Http.Resilience | HTTP resilience pipeline integration |
| **Health Checks** | Microsoft.Extensions.Diagnostics.HealthChecks | Liveness/Readiness probes |
| **Redundancy** | Triple Modular Redundancy (TMR) | 3 API replicas for high availability |

### Failover Tiers

| Tier | Description | Behavior |
|------|-------------|----------|
| **Tier 1** | Primary | Gemini 2.5 Flash + Search Grounding |
| **Tier 2** | Fallback | Alternative model/region or secondary key |
| **Tier 3** | Safe Mode | Deterministic local vector match only |

---

## Observability

| Category | Technology | Purpose |
|----------|------------|---------|
| **Tracing** | OpenTelemetry (OTLP) | Distributed request tracing |
| **Metrics** | System.Diagnostics.Metrics | Performance counters |
| **Logging** | .NET ILogger | Structured logging |
| **Dashboard** | Aspire Dashboard | Real-time service monitoring |

---

## Frontend Styling

| Category | Technology | Purpose |
|----------|------------|---------|
| **CSS Framework** | Tailwind CSS | Utility-first styling |
| **Syntax Highlighting** | (TBD - likely Prism.js or Highlight.js) | Code snippet rendering |

---

## Containerization & Infrastructure

| Category | Technology | Purpose |
|----------|------------|---------|
| **Container Runtime** | Docker/Podman | Container execution |
| **Orchestration** | .NET Aspire AppHost | Service orchestration |
| **Deployment** | Azure Developer CLI (azd) | Infrastructure-as-code deployment |

### Container Services

| Service | Image | Purpose |
|---------|-------|---------|
| API.Brain | Custom (.NET 10) | Main API with Semantic Kernel |
| UI.Web | Custom (.NET 10 WASM) | Blazor frontend |
| DataSeeder | Custom (.NET 10) | LeetCode ingestion worker |
| Qdrant | qdrant/qdrant | Vector database |
| PostgreSQL | postgres:16 + pgvector | Relational/graph database |
| Redis | redis:7 | SignalR backplane |

---

## Development Tools

| Tool | Version | Purpose |
|------|---------|---------|
| **dotnet CLI** | 10.0.102 | Build, test, publish |
| **Bun** | 1.3.10 | Fast JS/TS runtime for frontend tooling |
| **uv** | 0.10.9 | Python package management (if needed) |
| **opencode CLI** | 1.2.24 | AI agent runtime |

---

## NuGet Packages

### Core Aspire Packages

```xml
<PackageReference Include="Aspire.Hosting.AppHost" Version="9.*" />
<PackageReference Include="Aspire.Hosting.Redis" Version="9.*" />
<PackageReference Include="Aspire.Hosting.PostgreSQL" Version="9.*" />
<PackageReference Include="Aspire.Hosting.Qdrant" Version="9.*" />
```

### Semantic Kernel & AI

```xml
<PackageReference Include="Microsoft.SemanticKernel" Version="1.*" />
<PackageReference Include="Microsoft.Extensions.AI" Version="9.*" />
<PackageReference Include="Microsoft.Extensions.AI.Google" Version="9.*" />
```

### Data Access

```xml
<PackageReference Include="Qdrant.Client" Version="1.*" />
<PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="10.*" />
<PackageReference Include="Microsoft.AspNetCore.SignalR.StackExchangeRedis" Version="10.*" />
```

### Resilience

```xml
<PackageReference Include="Microsoft.Extensions.Http.Resilience" Version="9.*" />
<PackageReference Include="Microsoft.Extensions.Diagnostics.HealthChecks" Version="10.*" />
```

---

## Configuration Requirements

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `GEMINI_API_KEY` | Google Gemini API authentication key |
| `CONNECTIONSTRINGS__QDRANT` | Qdrant connection string (managed by Aspire) |
| `CONNECTIONSTRINGS__POSTGRES` | PostgreSQL connection string (managed by Aspire) |
| `CONNECTIONSTRINGS__REDIS` | Redis connection string (managed by Aspire) |

### global.json

```json
{
  "sdk": {
    "version": "10.0.0",
    "rollForward": "latestFeature"
  }
}
```

---

## Performance Targets

| Metric | Target |
|--------|--------|
| Semantic vector matching | < 500ms |
| End-to-end processing (Transcript → UI Card) | < 2 seconds |
| Problem identification accuracy | ≥ 95% |
| System availability | 99.9% (with TMR) |

---

*Last updated: 2026-03-12*

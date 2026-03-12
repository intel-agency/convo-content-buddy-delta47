# ConvoContentBuddy - System Architecture

## Overview

ConvoContentBuddy is an autonomous, real-time semantic assistant that listens to technical conversations (e.g., coding interviews) and proactively displays relevant algorithmic problem solutions. The system is designed for **aerospace-grade resilience** with Triple Modular Redundancy (TMR) and an **ambient user experience** requiring zero user interaction.

---

## Architecture Layers

The system is organized into four distinct layers:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         AMBIENT UI LAYER                                 │
│                    (Blazor WebAssembly + Speech Interop)                 │
└─────────────────────────────────────────────────────────────────────────┘
                                    │ SignalR WebSocket
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     CONTEXT & INTENT ANALYSIS LAYER                      │
│                        ("The Brain" - ASP.NET Core API)                  │
│              Semantic Kernel + Gemini 2.5 Flash + Search Grounding       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │ gRPC / SQL
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      RESOURCE RETRIEVAL LAYER                            │
│            (Hybrid Chain: Qdrant Vector + PostgreSQL Graph)              │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    AUDIO INPUT & TRANSCRIPTION LAYER                     │
│                    (Browser Web Speech API)                              │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Core Services

### 1. ConvoContentBuddy.AppHost (Aspire Orchestrator)

**Responsibility:** Container orchestration and service discovery

- Manages lifecycle of all microservices
- Configures TMR with `withReplicas(3)` for API.Brain
- Wires connection strings and service discovery
- Generates Docker Compose manifests

```
AppHost
├── API.Brain (x3 replicas)
├── UI.Web (Blazor WASM)
├── DataSeeder (Background Worker)
├── Qdrant (Vector Store)
├── PostgreSQL (Graph Store)
└── Redis (SignalR Backplane)
```

### 2. ConvoContentBuddy.ServiceDefaults

**Responsibility:** Shared cross-cutting concerns

- OpenTelemetry configuration (OTLP exporter)
- Health check endpoints (`/health`, `/health/ready`)
- Polly resilience pipelines (Retry, Circuit Breaker)
- Standard HTTP client configuration

### 3. ConvoContentBuddy.API.Brain

**Responsibility:** Core intelligence and orchestration

| Component | Purpose |
|-----------|---------|
| **BuddyHub** | SignalR hub for real-time UI updates |
| **HybridRetrieverService** | Coordinates: Embed → Vector Search → Graph Fetch → LLM Verify |
| **VectorSearchProvider** | Qdrant gRPC similarity search |
| **GraphTraversalProvider** | PostgreSQL neighbor queries |
| **ModelFailoverManager** | 3-tier failover orchestration |
| **IntentAnalyzer** | Gemini-powered transcript analysis |

### 4. ConvoContentBuddy.UI.Web

**Responsibility:** Zero-interaction ambient dashboard

| Component | Purpose |
|-----------|---------|
| **SpeechInterop.js** | webkitSpeechRecognition wrapper |
| **BuddyHub Client** | SignalR connection to API.Brain |
| **TranscriptFeed** | Live transcription display |
| **ProblemCard** | Dynamic solution rendering |
| **AutonomousController** | Buffer management, debounced POSTs |

### 5. ConvoContentBuddy.DataSeeder

**Responsibility:** Knowledge base initialization

- Parses LeetCode problem data (JSON/Markdown)
- Generates embeddings via Gemini text-embedding-004
- Seeds Qdrant `leetcode_problems` collection
- Creates PostgreSQL graph edges for problem relationships

### 6. ConvoContentBuddy.Core

**Responsibility:** Shared domain models

- DTOs (Problem, Solution, TranscriptChunk)
- Event models (ProblemDetectedEvent, StatusUpdateEvent)
- Interfaces (IEmbeddingService, IVectorSearchProvider)
- Constants and configuration models

---

## Data Flow

### Primary Flow: Problem Detection

```
1. User speaks → webkitSpeechRecognition captures audio
2. SpeechInterop.js → Blazor via DotNetObjectReference
3. Blazor → POST /api/analyze (debounced, 100+ chars)
4. API.Brain → Gemini text-embedding-004 (vectorize transcript)
5. HybridRetriever → Qdrant (top-3 similar problems)
6. HybridRetriever → PostgreSQL (graph expansion for context)
7. HybridRetriever → Gemini 2.5 Flash (verify best match)
8. Gemini → Search Grounding (fetch solutions)
9. API.Brain → BuddyHub (push ProblemDetected event)
10. BuddyHub → SignalR → Blazor (render solution card)
```

### Failover Flow: Safe Mode

```
1. Gemini API rate limit or outage detected
2. ModelFailoverManager → Circuit Breaker opens
3. System enters Tier 3 (Safe Mode)
4. Skip LLM verification, return top vector match
5. UI displays "Low Confidence" badge
6. BuddyHub pushes StatusUpdate event
```

---

## High Availability Architecture

### Triple Modular Redundancy (TMR)

```
                    ┌─────────────────────┐
                    │   Redis Backplane   │
                    │  (Session State)    │
                    └─────────┬───────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  API.Brain #1 │    │  API.Brain #2 │    │  API.Brain #3 │
│   (Primary)   │    │  (Secondary)  │    │  (Tertiary)   │
└───────────────┘    └───────────────┘    └───────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                    ┌─────────▼───────────┐
                    │   Load Balancer     │
                    │   (Aspire/K8s)      │
                    └─────────┬───────────┘
                              │
                    ┌─────────▼───────────┐
                    │    Blazor Client    │
                    │   (SignalR Client)  │
                    └─────────────────────┘
```

**Behavior:**
- If 2 of 3 API instances fail, the remaining instance continues
- Redis backplane preserves SignalR connection state
- Client reconnects seamlessly without losing transcript

---

## N+2 Failover Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                      TIER 1: PRIMARY                            │
│                  Gemini 2.5 Flash + Search                      │
│              Full capabilities, highest quality                  │
└─────────────────────────────────────────────────────────────────┘
                              │ Rate Limit / Error
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      TIER 2: FALLBACK                           │
│              Alternative Model / Region / Key                    │
│           Reduced features, maintained quality                   │
└─────────────────────────────────────────────────────────────────┘
                              │ All Cloud Unavailable
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    TIER 3: SAFE MODE                            │
│              Local Vector Match Only (No LLM)                    │
│           Deterministic, low confidence, offline-capable         │
└─────────────────────────────────────────────────────────────────┘
```

---

## Database Schemas

### Qdrant Collection: `leetcode_problems`

```json
{
  "vectors": {
    "size": 1536,
    "distance": "Cosine"
  },
  "payload_schema": {
    "problem_id": "integer",
    "title": "text",
    "slug": "keyword",
    "difficulty": "keyword",
    "topics": "keyword[]"
  }
}
```

### PostgreSQL Tables

```sql
-- Problems table
CREATE TABLE problems (
    id SERIAL PRIMARY KEY,
    leetcode_id INTEGER UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    difficulty VARCHAR(20) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Problem relationships (adjacency list)
CREATE TABLE problem_edges (
    id SERIAL PRIMARY KEY,
    source_problem_id INTEGER REFERENCES problems(id),
    target_problem_id INTEGER REFERENCES problems(id),
    relationship_type VARCHAR(50), -- 'similar', 'prerequisite', 'follow_up'
    weight FLOAT DEFAULT 1.0
);

-- Vector index (pgvector)
CREATE INDEX ON problems USING ivfflat (embedding vector_cosine_ops);
```

---

## Key Interfaces

```csharp
// Core service interfaces
public interface IEmbeddingService
{
    Task<float[]> EmbedAsync(string text, CancellationToken ct = default);
}

public interface IVectorSearchProvider
{
    Task<IReadOnlyList<SearchResult>> SearchAsync(
        float[] queryVector, 
        int topK = 3, 
        CancellationToken ct = default);
}

public interface IGraphTraversalProvider
{
    Task<IReadOnlyList<RelatedProblem>> GetNeighborsAsync(
        int problemId, 
        CancellationToken ct = default);
}

public interface IHybridRetriever
{
    Task<ProblemMatch?> FindBestMatchAsync(
        string transcriptChunk, 
        CancellationToken ct = default);
}

public interface IModelFailoverManager
{
    Task<T> ExecuteWithFailoverAsync<T>(
        Func<CancellationToken, Task<T>> primary,
        Func<CancellationToken, Task<T>> fallback,
        Func<CancellationToken, Task<T>> safeMode,
        CancellationToken ct = default);
}
```

---

## Deployment Architecture

### Local Development (Aspire)

```
dotnet run --project ConvoContentBuddy.AppHost
    ↓
Aspire Dashboard (http://localhost:15000)
    ↓
All services containerized and networked
```

### Production (Azure via azd)

```
azd up
    ↓
Azure Container Apps / AKS
    ↓
Managed Redis, PostgreSQL, Qdrant (self-hosted or managed)
```

---

## Security Considerations

| Concern | Mitigation |
|---------|------------|
| API Key Storage | Azure Key Vault / GitHub Secrets |
| Transcript Privacy | No persistent storage of audio/text |
| CORS | Strict origin policy for Blazor client |
| Rate Limiting | Polly policies + API gateway throttling |

---

## Performance Characteristics

| Component | Latency Target | Notes |
|-----------|----------------|-------|
| Speech Recognition | Real-time | Browser-native |
| Embedding Generation | < 100ms | Gemini text-embedding-004 |
| Vector Search | < 200ms | Qdrant gRPC |
| Graph Traversal | < 100ms | PostgreSQL indexed |
| LLM Verification | < 500ms | Gemini 2.5 Flash |
| **Total E2E** | **< 2000ms** | Including network |

---

## Future Roadmap

1. **Multi-region deployment** with geo-replication
2. **"Glider Mode"** - Full offline with ONNX Runtime Web + IndexedDB
3. **DB Clustering** - Patroni for PostgreSQL, Qdrant Raft
4. **Custom model fine-tuning** for interview-specific terminology

---

*Architecture Document - Version 1.0*
*Last updated: 2026-03-12*

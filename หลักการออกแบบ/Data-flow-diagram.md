# Data Flow Diagram - LLM Chat System

## 🌐 Public Access Layer (Cloudflare)

```
┌─────────────────────────────────────────────────────────────────┐
│                        Cloudflare CDN                            │
│                  (https://pwa.sri-ketguide.com)                 │
│                  (https://api.sri-ketguide.com)                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ HTTPS
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Cloudflare Tunnels                             │
├─────────────────────────┬───────────────────────────────────────┤
│  Tunnel: PWA            │  Tunnel: API                          │
│  cf-tunnel-prod         │  cf-tunnel-api                        │
│  172.20.0.9             │  172.20.0.10                          │
└────────┬────────────────┴───────────────┬──────────────────────┘
         │                                │
         │                                │
```

## 🖥️ Application Layer

```
┌─────────────────────────────────────────────────────────────────┐
│                      PWA Production                              │
│                  vit-llm-chat-pwa-1                              │
│                  (Nginx + React Build)                          │
│                  Port: 80 → 5174 (host)                          │
│                  Container: vit_llm-chat-network                │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │ API Requests
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Development Frontend                          │
│                  vit-llm-chat-frontend-1                        │
│                  (Vite Dev Server)                              │
│                  Port: 5173                                     │
│                  Container: vit_llm-chat-network                │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │ HTTP Requests
                            ▼
```

## 🔧 Backend Services Layer

```
┌─────────────────────────────────────────────────────────────────┐
│                      API Server                                  │
│                  vit-llm-chat-api-1                             │
│                  (Node.js + Express)                             │
│                  Port: 3000                                     │
│                  Container: vit_llm-chat-network                │
├───────────────────────────┬─────────────────────────────────────┤
│ • HTTP REST API             │ • JWT Authentication              │
│ • Chat endpoints            │ • File Upload                     │
│ • User management           │ • SSE Streaming                  │
└─────────┬───────────────────┴──────────────────┬────────────────┘
          │                                       │
          │ Read/Write                          HTTP
          │                                      │
          ▼                                      ▼
```

## 👷 Background Worker Layer

```
┌─────────────────────────────────────────────────────────────────┐
│                      Background Worker                            │
│                  vit-llm-chat-worker-1                           │
│                  Container: vit_llm-chat-network                │
├───────────────────────────────────────────────────────────────────┤
│ • Process heavy tasks                                            │
│ • Queue management                                               │
│ • Async job processing                                           │
└───────────────────────────────────────────────────────────────────┘
```

## 🗄️ Data Storage Layer

```
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Database                           │
│                  vit-llm-chat-database-1                        │
│                  (PostgreSQL 15 + pgvector)                      │
│                  Port: 5432                                      │
│                  Container: vit_llm-chat-network                │
├───────────────────────────────────────────────────────────────────┤
│ Storage:                                                          │
│ • users (authentication)                                        │
│ • sessions (chat sessions)                                      │
│ • messages (chat history)                                       │
│ • embeddings (vector data with pgvector)                        │
└───────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         Redis Cache                              │
│                  vit-llm-chat-cache-1                           │
│                  Port: 6379                                     │
│                  Container: vit_llm-chat-network                │
├───────────────────────────────────────────────────────────────────┤
│ Cache:                                                            │
│ • Session data                                                   │
│ • API responses                                                  │
│ • Rate limiting counters                                         │
└───────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Embedder Service                          │
│                  vit-llm-chat-embedder-1                         │
│                  Port: 11400                                     │
│                  Container: vit_llm-chat-network                │
├───────────────────────────────────────────────────────────────────┤
│ Functions:                                                        │
│ • Text embedding generation                                      │
│ • Vector similarity search                                       │
│ • Semantic search indexing                                       │
└───────────────────────────────────────────────────────────────────┘
```

## 🔗 Network Flow

### User Request Flow (PWA):
```
User Browser
    │
    ├─► HTTPS Request → pwa.sri-ketguide.com
    │                      │
    │                      ├─► Cloudflare CDN
    │                      │       │
    │                      │       ├─► Cloudflare Tunnel (cf-tunnel-prod)
    │                      │       │       │
    │                      │       │       └─► vit-llm-chat-pwa-1:80
    │                      │       │                   │
    │                      │       │                   ├─► Serves static files (Nginx)
    │                      │       │                   │
    │                      │       │                   └─► API calls → api.sri-ketguide.com
    │
    └─► HTTPS Request → api.sri-ketguide.com
                            │
                            ├─► Cloudflare CDN
                            │       │
                            │       ├─► Cloudflare Tunnel (cf-tunnel-api)
                            │       │       │
                            │       │       └─► vit-llm-chat-api-1:3000
                            │       │                   │
                            │       │                   ├─► Reads from Database
                            │       │                   ├─► Reads from Cache
                            │       │                   ├─► Calls Embedder
                            │       │                   └─► SSE Streaming to Client
```

## 📊 Service Communication Matrix

| From Service | To Service | Protocol | Port | Purpose |
|--------------|-----------|----------|------|---------|
| PWA (Nginx) | API | HTTPS | 3000 | API requests |
| Frontend Dev | API | HTTP | 3000 | Dev API calls |
| API | Database | TCP | 5432 | SQL queries |
| API | Cache | TCP | 6379 | Redis operations |
| API | Embedder | HTTP | 11400 | Embedding generation |
| API | Worker | Queue | - | Background jobs |
| Tunnels | Services | HTTP | 5174, 3000 | Public access |

## 🔄 Data Flow Types

### 1. **User Authentication Flow**
```
PWA → API (/auth/login) → Database (verify credentials) 
   → Generate JWT → Redis (store session) → Return to PWA
```

### 2. **Chat Message Flow**
```
PWA → API (/chat) → Database (save message) 
   → Worker (process) → Embedder (generate embedding)
   → Database (store embedding) → Ollama (generate response)
   → API (SSE stream) → PWA (display)
```

### 3. **File Upload Flow**
```
PWA → API (/upload) → Multer (save file) 
   → Database (store metadata) → Embedder (process file)
   → Database (store embeddings) → Response to PWA
```

### 4. **History Retrieval Flow**
```
PWA → API (/chat/sessions) → Database (query sessions)
   → Cache (check cache) → Return data
   → Cache (update if needed) → Response to PWA
```

## 🌟 Key Characteristics

- **All services** run in `vit_llm-chat-network` (bridge network)
- **Public access** via Cloudflare Tunnels (secure HTTPS)
- **Local access** via localhost ports (5173, 3000, 5174)
- **Database** uses pgvector for semantic search
- **Cache** improves response times
- **Worker** handles heavy processing off API thread
- **Embedder** generates vector embeddings for AI search

---

**Updated**: October 28, 2025
**Environment**: Development + Production Tunnels

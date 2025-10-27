# Flowchart Diagram - LLM Chat System

## 🔐 1. User Authentication Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     User Authentication                           │
└─────────────────────────────────────────────────────────────────┘

Start
 │
 ├─► [Load App] ─────────────► https://pwa.sri-ketguide.com
 │       │
 │       ▼
 │   Check localStorage
 │   │
 │   ├─► Has accessToken? ──YES──► Validate Token
 │   │       │                           │
 │   │       │                           ├─► Valid? ──YES──► Show Chat Interface
 │   │       │                           │       │
 │   │       │                           │       └─NO─► Refresh Token
 │   │       │                           │               │
 │   │       │                           │               ├─► Success? ──YES──► Show Chat Interface
 │   │       │                           │               │
 │   │       │                           │               └─NO─► Show Login Page
 │   │       │                           │
 │   │       │                           └─NO─► Show Login Page
 │   │       │
 │   │       └─NO─► Show Login Page
 │   │
 │   └─► [Login Page]
 │         │
 │         ├─► User enters email & password
 │         │
 │         ├─► Submit form
 │         │
 │         ├─► POST https://api.sri-ketguide.com/auth/login
 │         │       │
 │         │       ▼
 │         │   Backend validates
 │         │   │
 │         │   ├─► Valid? ──YES──► Generate JWT tokens
 │         │   │       │                   │
 │         │   │       │                   ├─► accessToken (short-lived)
 │         │   │       │                   └─► refreshToken (long-lived)
 │         │   │       │                           │
 │         │   │       │                           ▼
 │         │   │       │                   Store in:
 │         │   │       │                   ├─► localStorage (browser)
 │         │   │       │                   ├─► Database (users + tokens)
 │         │   │       │                   └─► Redis (cache)
 │         │   │       │
 │         │   │       └─► Return tokens + user data
 │         │   │
 │         │   └─NO─► Return error message
 │         │
 │         ▼
 │   Save tokens to localStorage
 │
 └─► Redirect to Chat Interface

End ──► Chat Interface Loaded
```

---

## 💬 2. Chat Message Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      Chat Message Flow                           │
└─────────────────────────────────────────────────────────────────┘

User Types Message
 │
 ├─► [Send Message]
 │       │
 │       ▼
 │   Validate input
 │   │
 │   ├─► Empty? ──YES──► Show error
 │   │       │
 │   │       └─NO─► Continue
 │   │
 │   ├─► Add to UI immediately (optimistic update)
 │   │
 │   ├─► POST https://api.sri-ketguide.com/chat
 │   │       │
 │   │       ├─► Headers:
 │   │       │     Authorization: Bearer {accessToken}
 │   │       │     Content-Type: application/json
 │   │       │
 │   │       ├─► Body:
 │   │       │     { prompt: message, stream: true/false }
 │   │       │
 │   │       ▼
 │   │   [API Server validates]
 │   │       │
 │   │       ├─► Auth valid? ──NO─► Return 401
 │   │       │       │                   │
 │   │       │       │                   └─► Clear tokens
 │   │       │       │                       └─► Redirect to login
 │   │       │       │
 │   │       │       └─YES─► Continue
 │   │       │
 │   │       ├─► Rate limit check (per user.per_minute)
 │   │       │
 │   │       │   ├─► Exceeded? ──YES──► Return 429 (Too Many Requests)
 │   │       │   │       │
 │   │       │   │       └─NO─► Continue
 │   │       │
 │   │       ├─► Create/Get session
 │   │       │   │
 │   │       │   ├─► New chat? ──YES──► Create session in DB
 │   │       │   │       │
 │   │       │   │       └─NO─► Use existing session
 │   │       │
 │   │       ├─► Save user message to DB
 │   │       │   (chat_messages table)
 │   │       │
 │   │       ├─► Call Ollama API (LLM)
 │   │       │   │
 │   │       │   ├─► SSE Stream
 │   │       │   │   │
 │   │       │   │   ├─► Stream chunks to client
 │   │       │   │   │   │
 │   │       │   │   │   └─► Update UI in real-time
 │   │       │   │
 │   │       │   └─► Wait for complete response
 │   │       │
 │   │       ├─► Save assistant response to DB
 │   │       │   (chat_messages table)
 │   │       │
 │   │       └─► Update session.updated_at
 │   │
 │   └─► Return success
 │

End ──► Message displayed in chat
```

---

## 🔍 3. Business Search Flow (Semantic Search)

```
┌─────────────────────────────────────────────────────────────────┐
│                   Business Search Flow                           │
└─────────────────────────────────────────────────────────────────┘

User Initiates Search
 │
 ├─► [Search Query]
 │       │
 │       ├─► Text input: "โรงแรมในศรีสะเกษ"
 │       │
 │       ├─► GET https://api.sri-ketguide.com/search?q={query}
 │       │       │
 │       │       ▼
 │       │   [API Server]
 │       │       │
 │       │       ├─► 1. Generate Query Embedding
 │       │       │     │
 │       │       │     ├─► POST to Embedder Service
 │       │       │     │     │
 │       │       │     │     │   url: http://embedder:11400/embeddings
 │       │       │     │     │   body: { text: user_query }
 │       │       │     │     │
 │       │       │     │     └─► Return vector[1536]
 │       │       │
 │       │       ├─► 2. Vector Similarity Search
 │       │       │     │
 │       │       │     ├─► PostgreSQL + pgvector
 │       │       │     │     │
 │       │       │     │     │   SELECT id, name, embedding
 │       │       │     │     │   FROM businesses
 │       │       │     │     │   WHERE is_active = true
 │       │       │     │     │   ORDER BY embedding <=> ?
 │       │       │     │     │   LIMIT 10
 │       │       │     │
 │       │       │     └─► Return top N results
 │       │       │
 │       │       ├─► 3. Cache Results (Redis)
 │       │       │     │
 │       │       │     ├─► Key: search:{query_hash}
 │       │       │     ├─► Value: {results}
 │       │       │     └─► TTL: 1 hour
 │       │       │
 │       │       └─► 4. Return Results
 │       │
 │       ▼
 │   Display Results
 │       │
 │       ├─► Sort by relevance (cosine similarity)
 │       │
 │       ├─► Show business cards:
 │       │     - Name
 │       │     - Description
 │       │     - Contact info
 │       │     - Images
 │       │
 │       └─► Highlight matched features
 │

End ──► Results displayed
```

---

## 📤 4. File Upload Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                       File Upload Flow                           │
└─────────────────────────────────────────────────────────────────┘

User Selects File
 │
 ├─► [Choose File]
 │       │
 │       ├─► Image types: jpg, png, webp
 │       ├─► Max size: 5MB
 │       │
 │       ▼
 │   Validate file
 │   │
 │   ├─► Type valid? ──NO──► Show error: "Invalid file type"
 │   │       │
 │   │       └─YES─► Continue
 │   │
 │   ├─► Size valid? ──NO──► Show error: "File too large"
 │   │       │
 │   │       └─YES─► Continue
 │   │
 │   ├─► POST https://api.sri-ketguide.com/upload
 │   │       │
 │   │       ├─► Headers:
 │   │       │     Authorization: Bearer {accessToken}
 │   │       │     Content-Type: multipart/form-data
 │   │       │
 │   │       ├─► Body:
 │   │       │     file: {binary data}
 │   │       │     type: 'image/png'
 │   │       │
 │   │       ▼
 │   │   [API Server]
 │   │       │
 │   │       ├─► 1. Authenticate user
 │   │       │
 │   │       ├─► 2. Save file (Multer)
 │   │       │     │
 │   │       │     ├─► Destination: /uploads/{user_id}/
 │   │       │     ├─► Generate unique filename
 │   │       │     │     format: {timestamp}-{hash}.{ext}
 │   │       │     │
 │   │       │     └─► Save to filesystem
 │   │       │
 │   │       ├─► 3. Store metadata in DB
 │   │       │     │
 │   │       │     ├─► user_id
 │   │       │     ├─► filename
 │   │       │     ├─► file_url
 │   │       │     ├─► file_size
 │   │       │     └─► mime_type
 │   │       │
 │   │       ├─► 4. Generate embeddings (async)
 │   │       │     │
 │   │       │     ├─► Queue job to worker
 │   │       │     │     │
 │   │       │     │     ├─► Process image
 │   │       │     │     ├─► Generate vector
 │   │       │     │     └─► Store in DB
 │   │       │
 │   │       └─► 5. Return file URL
 │   │
 │   ▼
 └─► Display uploaded image in UI

End ──► File uploaded and ready
```

---

## 👑 5. Role-Based Access Control Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                  Role-Based Access Control                       │
└─────────────────────────────────────────────────────────────────┘

User Requests Resource
 │
 ├─► [Check Authentication]
 │       │
 │       ├─► Has valid token? ──NO──► Return 401 Unauthorized
 │       │       │                         │
 │       │       │                         └─► Redirect to login
 │       │       │
 │       │       └─YES─► Continue
 │       │
 │       ▼
 │   Extract user role from token
 │       │
 │       ├─► Decode JWT
 │       │     │
 │       │     └─► Get user.role
 │       │
 │       ▼
 ├─► [Check Role Permissions]
 │       │
 │       ├─► Admin
 │       │     ├─► Full access
 │       │     ├─► Manage users
 │       │     ├─► Manage businesses
 │       │     └─► View all data
 │       │
 │       ├─► Staff
 │       │     ├─► View users
 │       │     ├─► Manage businesses
 │       │     ├─► Moderate content
 │       │     └─► Limited admin access
 │       │
 │       ├─► Sponsor
 │       │     ├─► View own businesses
 │       │     ├─► Create businesses
 │       │     ├─► Upload images
 │       │     └─► Premium features
 │       │
 │       ├─► Member Pro
 │       │     ├─► Advanced chat features
 │       │     ├─► Higher rate limits
 │       │     └─► Priority support
 │       │
 │       ├─► Member
 │       │     ├─► Standard chat
 │       │     ├─► Basic search
 │       │     └─► Standard rate limits
 │       │
 │       └─► Guest
 │             ├─► Limited access
 │             ├─► Restricted features
 │             └─► Prompt for registration
 │
 ├─► [Check Rate Limits]
 │       │
 │       ├─► Get user.limit_per_minute
 │       │
 │       ├─► Check Redis counter
 │       │     │
 │       │     ├─► Key: rate:{user_id}:{minute}
 │       │     ├─► Current count vs limit
 │       │     │
 │       │     ├─► Exceeded? ──YES──► Return 429 Too Many Requests
 │       │     │       │                   │
 │       │     │       │                   └─► Show rate limit message
 │       │     │
 │       │     └─NO─► Increment counter
 │       │
 ├─► [Grant Access]
 │       │
 │       └─► Process request
 │

End ──► Resource served to user
```

---

## 📱 6. SSE (Server-Sent Events) Streaming Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    SSE Streaming Flow                            │
└─────────────────────────────────────────────────────────────────┘

Client Initiates Stream
 │
 ├─► [Open EventSource]
 │       │
 │       ├─► GET https://api.sri-ketguide.com/chat
 │       │       headers: { stream: true }
 │       │
 │       ▼
 │   Connection established
 │       │
 │       ▼
 ┌─── Backend Streaming Loop ───┐
 │                               │
 │   ┌───────────────────────┐  │
 │   │ Send "sessionId"      │  │
 │   │ data: {session_id}    │  │
 │   └───────────────────────┘  │
 │             │                │
 │             ▼                │
 │   ┌───────────────────────┐  │
 │   │ Call Ollama API       │  │
 │   │ (http://ollama:11434) │  │
 │   └───────────────────────┘  │
 │             │                │
 │             ▼                │
 │   ┌───────────────────────┐  │
 │   │ Receive chunks        │  │
 │   └───────────────────────┘  │
 │             │                │
 │             ▼                │
 │   ┌───────────────────────┐  │
 │   │ Stream each chunk:     │  │
 │   │ data: {"content":"..."}│  │
 │   │ data: {"content":"..."}│  │
 │   └───────────────────────┘  │
 │             │                │
 │             ▼                │
 │   Stream finished?           │
 │             │                │
 │             ├─NO─► Continue loop
 │             │                │
 │             └─YES─► Send "data: [DONE]"
 │                               │
 └───────────────────────────────┘
             │
             ▼
       Connection closed
             │
             ▼
┌─── Frontend Updates UI ───┐
│                            │
│   ┌────────────────────┐  │
│   │ EventListener on   │  │
│   │ 'message' event    │  │
│   └────────────────────┘  │
│            │              │
│            ▼              │
│   ┌────────────────────┐  │
│   │ Parse chunk        │  │
│   └────────────────────┘  │
│            │              │
│            ▼              │
│   ┌────────────────────┐  │
│   │ Append to chat UI  │  │
│   │ (real-time display)│  │
│   └────────────────────┘  │
│                            │
└────────────────────────────┘

End ──► Stream complete, full response displayed
```

---

## 🔄 7. Refresh Token Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      Refresh Token Flow                          │
└─────────────────────────────────────────────────────────────────┘

API Request Fails (401)
 │
 ├─► [Access Token Expired]
 │       │
 │       ▼
 │   Check localStorage
 │       │
 │       ├─► Has refreshToken? ──NO──► Redirect to login
 │       │       │
 │       │       └─YES─► Continue
 │       │
 │       ├─► POST https://api.sri-ketguide.com/auth/refresh
 │       │       │
 │       │       ├─► Body: { refreshToken }
 │       │       │
 │       │       ▼
 │       │   [API Server]
 │       │       │
 │       │       ├─► 1. Validate refreshToken
 │       │       │     │
 │       │       │     ├─► Valid? ──NO──► Return 401
 │       │       │     │       │             │
 │       │       │     │       │             └─► Clear tokens
 │       │       │     │       │                 └─► Redirect login
 │       │       │     │
 │       │       │     └─YES─► Continue
 │       │       │
 │       │       ├─► 2. Check DB for token
 │       │       │     │
 │       │       │     ├─► Exists? ──NO──► Return 401
 │       │       │     │
 │       │       │     └─YES─► Continue
 │       │       │
 │       │       ├─► 3. Generate new accessToken
 │       │       │     │
 │       │       │     └─► Short-lived (15 min)
 │       │       │
 │       │       └─► 4. Return new token
 │       │
 │       ▼
 │   Update localStorage
 │       │
 │       ├─► Save new accessToken
 │       │
 │       ├─► Retry original request
 │       │       │
 │       │       ▼
 │       │   Request succeeds with new token
 │       │
 │       └─► User continues using app
 │

End ──► Seamless re-authentication
```

---

## 🛠️ 8. Tool Usage Patterns

### Pattern 1: **PostgreSQL + pgvector (Semantic Search)**

```
User Query: "โรงแรมใกล้โรงพยาบาล"
       │
       ▼
   Query Text
       │
       ▼
   ┌────────────────────────┐
   │ Embedder Service       │
   │ (Generate vector)      │
   └────────────────────────┘
       │
       ▼
   Query Vector [0.1, 0.5, ...]
       │
       ▼
   ┌────────────────────────┐
   │ PostgreSQL Query       │
   │                       │
   │ SELECT *,             │
   │        embedding <=> ?│
   │ FROM businesses       │
   │ WHERE is_active=true  │
   │ ORDER BY distance     │
   │ LIMIT 10              │
   └────────────────────────┘
       │
       ▼
   Results (sorted by similarity)
       │
       ▼
   Display to user
```

### Pattern 2: **Redis Caching**

```
Request arrives
     │
     ├─► Check Redis cache
     │       │
     │       │ Key: cache:{resource}:{params}
     │       │
     │       ├─► Hit? ──YES──► Return cached data
     │       │       │              │
     │       │       │              └─► TTL: 1 hour
     │       │
     │       └─NO─► Continue to DB
     │
     ├─► Query Database
     │
     ├─► Store in Redis
     │
     └─► Return to client
```

### Pattern 3: **Worker Queue (Background Jobs)**

```
Heavy Operation Requested
     │
     ├─► API receives request
     │       │
     │       ├─► Validate request
     │       │
     │       ├─► Add job to queue
     │       │     │
     │       │     ├─► Job: process_image
     │       │     ├─► Job: generate_embedding
     │       │     └─► Job: send_email
     │       │
     │       └─► Return job_id to client
     │
     ▼
Worker picks up job
     │
     ├─► Process job
     │       │
     │       ├─► Success? ──YES──► Update status
     │       │       │                  │
     │       │       │                  └─► Notify client
     │       │
     │       └─NO─► Log error, retry
```

---

## 📊 Service Interaction Summary

```
┌─────────────────────────────────────────────────────────────────┐
│              Service Interaction Matrix                          │
└─────────────────────────────────────────────────────────────────┘

│ Service From     │ Service To      │ Protocol │ Purpose            │
├──────────────────┼─────────────────┼──────────┼────────────────────┤
│ Frontend/PWA     │ API Server      │ HTTPS    │ Chat, Search       │
│ Frontend/PWA     │ API Server      │ HTTPS    │ Auth, Upload       │
│ API Server       │ PostgreSQL      │ TCP      │ Data persistence   │
│ API Server       │ Redis           │ TCP      │ Caching, Sessions  │
│ API Server       │ Embedder        │ HTTP     │ Generate vectors   │
│ API Server       │ Worker          │ Queue    │ Background jobs    │
│ API Server       │ Ollama          │ HTTP     │ LLM generation     │
│ Worker           │ Embedder        │ HTTP     │ Process embeddings │
│ Worker           │ PostgreSQL      │ TCP      │ Store results      │
└──────────────────┴─────────────────┴──────────┴────────────────────┘
```

---

**Updated**: October 28, 2025
**Flow Types**: Authentication, Chat, Search, Upload, RBAC, SSE Streaming, Token Refresh
**Tools**: PostgreSQL, pgvector, Redis, Worker Queue, SSE

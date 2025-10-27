# Docker Services Documentation - LLM Chat Application

## 📋 ภาพรวมระบบ

ระบบ LLM Chat Application ประกอบด้วย 6 บริการหลักที่ทำงานร่วมกันเพื่อสร้างแอปพลิเคชันแชทที่ใช้ AI/LLM พร้อมระบบค้นหาข้อมูลแบบ Semantic Search

### 🏗️ สถาปัตยกรรมระบบ

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Server    │    │   Worker        │
│   (Vite Dev)    │◄──►│   (Express)     │◄──►│   (Background)  │
│   Port: 5173    │    │   Port: 3000    │    │   Port: -       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Database      │    │   Cache         │    │   Embedder      │
│   (PostgreSQL   │◄──►│   (Redis)        │◄──►│   (Mock Service)│
│   + pgvector)   │    │   Port: 6379     │    │   Port: 11400   │
│   Port: 5432    │    └─────────────────┘    └─────────────────┘
└─────────────────┘
```

---

## 🔧 บริการแต่ละตัว

### 1. 🚀 llm-chat-api (API Server)

**หน้าที่หลัก**: Backend API server ที่จัดการ HTTP requests และ business logic

**เทคโนโลยี**:
- **Runtime**: Node.js 20 Alpine
- **Framework**: Express.js
- **Database ORM**: Sequelize
- **Authentication**: JWT (jsonwebtoken)
- **File Upload**: Multer
- **Thai Text Processing**: wordcut

**การทำงาน**:
- รับ HTTP requests จาก frontend
- จัดการ authentication และ authorization
- ประมวลผล business logic
- เชื่อมต่อกับ database และ cache
- ส่งงานหนักไปให้ worker ประมวลผล

**Port**: `3000`
**Health Check**: ไม่มี (depends on other services)

**Environment Variables**:
```bash
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:pass@llm-chat-database:5432/db
REDIS_URL=redis://llm-chat-cache:6379
JWT_SECRET=${JWT_SECRET}
OLLAMA_EMBED_URL=http://llm-chat-embedder:11400/embeddings
```

**คำสั่งควบคุม**:
```bash
# ดู logs
docker logs vit-llm-chat-api-1

# เข้าไปใน container
docker exec -it vit-llm-chat-api-1 sh

# Restart service
docker-compose restart llm-chat-api

# ดู status
docker-compose ps llm-chat-api
```

---

### 2. 🎨 llm-chat-frontend (Frontend Development Server)

**หน้าที่หลัก**: Frontend development server สำหรับ UI/UX

**เทคโนโลยี**:
- **Build Tool**: Vite
- **Runtime**: Node.js 20 Alpine
- **UI Framework**: Bootstrap 5
- **Markdown**: marked + highlight.js
- **Alerts**: SweetAlert2
- **Thai Text**: wordcut
- **Security**: DOMPurify

**การทำงาน**:
- รัน Vite development server
- Hot reload สำหรับการพัฒนา
- Serve static files และ assets
- Proxy API requests ไปยัง backend

**Port**: `5173`
**Health Check**: ไม่มี

**Environment Variables**:
```bash
CHOKIDAR_USEPOLLING=true  # สำหรับ file watching ใน Docker
```

**คำสั่งควบคุม**:
```bash
# ดู logs
docker logs vit-llm-chat-frontend-1

# เข้าไปใน container
docker exec -it vit-llm-chat-frontend-1 sh

# Restart service
docker-compose restart llm-chat-frontend

# Build production
docker exec -it vit-llm-chat-frontend-1 npm run build
```

---

### 3. 🗄️ llm-chat-database (PostgreSQL Database)

**หน้าที่หลัก**: ฐานข้อมูลหลักสำหรับเก็บข้อมูลแอปพลิเคชัน

**เทคโนโลยี**:
- **Database**: PostgreSQL 15
- **Vector Extension**: pgvector
- **Data Persistence**: Docker Volume

**การทำงาน**:
- เก็บข้อมูลผู้ใช้, การแชท, และ metadata
- รองรับ vector similarity search ด้วย pgvector
- Auto-initialization จาก scripts ใน `db-init/`
- Health check เพื่อให้แน่ใจว่า database พร้อมใช้งาน

**Port**: `5432`
**Health Check**: `pg_isready` ทุก 5 วินาที

**Environment Variables**:
```bash
POSTGRES_DB=${POSTGRES_DB}
POSTGRES_USER=${POSTGRES_USER}
POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
```

**คำสั่งควบคุม**:
```bash
# เชื่อมต่อ database
docker exec -it vit-llm-chat-database-1 psql -U ${POSTGRES_USER} -d ${POSTGRES_DB}

# ดู logs
docker logs vit-llm-chat-database-1

# Backup database
docker exec vit-llm-chat-database-1 pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} > backup.sql

# Restore database
docker exec -i vit-llm-chat-database-1 psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} < backup.sql

# ดู database size
docker exec vit-llm-chat-database-1 psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} -c "SELECT pg_size_pretty(pg_database_size('${POSTGRES_DB}'));"
```

---

### 4. ⚡ llm-chat-cache (Redis Cache)

**หน้าที่หลัก**: Cache และ message queue สำหรับระบบ

**เทคโนโลยี**:
- **Cache**: Redis 7 Alpine
- **Message Queue**: BullMQ (ใช้ Redis เป็น backend)
- **Data Persistence**: Docker Volume

**การทำงาน**:
- Cache ข้อมูลที่ใช้บ่อยเพื่อเพิ่มประสิทธิภาพ
- Message queue สำหรับ background jobs
- Session storage
- Rate limiting

**Port**: `6379`
**Health Check**: `redis-cli ping` ทุก 5 วินาที

**คำสั่งควบคุม**:
```bash
# เชื่อมต่อ Redis CLI
docker exec -it vit-llm-chat-cache-1 redis-cli

# ดู logs
docker logs vit-llm-chat-cache-1

# ดู memory usage
docker exec vit-llm-chat-cache-1 redis-cli info memory

# Clear all cache
docker exec vit-llm-chat-cache-1 redis-cli flushall

# ดู keys
docker exec vit-llm-chat-cache-1 redis-cli keys "*"

# Monitor real-time commands
docker exec vit-llm-chat-cache-1 redis-cli monitor
```

---

### 5. 🔄 llm-chat-worker (Background Worker)

**หน้าที่หลัก**: ประมวลผลงานหนักในพื้นหลัง

**เทคโนโลยี**:
- **Runtime**: Node.js 20 Alpine
- **Job Queue**: BullMQ + Redis
- **Codebase**: ใช้โค้ดเดียวกับ API server

**การทำงาน**:
- ประมวลผล embedding generation
- ประมวลผลไฟล์ที่อัปโหลด
- ส่งอีเมล notifications
- Cleanup tasks
- Data processing jobs

**Port**: ไม่มี (internal only)
**Health Check**: ไม่มี

**Environment Variables**:
```bash
NODE_ENV=development
DATABASE_URL=postgresql://user:pass@llm-chat-database:5432/db
REDIS_URL=redis://llm-chat-cache:6379
JWT_SECRET=${JWT_SECRET}
OLLAMA_EMBED_URL=http://llm-chat-embedder:11400/embeddings
```

**คำสั่งควบคุม**:
```bash
# ดู logs
docker logs vit-llm-chat-worker-1

# เข้าไปใน container
docker exec -it vit-llm-chat-worker-1 sh

# Restart worker
docker-compose restart llm-chat-worker

# ดู job queue status
docker exec -it vit-llm-chat-worker-1 npm run enqueue:embeddings
```

---

### 6. 🧠 llm-chat-embedder (Vector Embedding Service)

**หน้าที่หลัก**: Mock service สำหรับสร้าง vector embeddings

**เทคโนโลยี**:
- **Runtime**: Node.js
- **Framework**: Express.js
- **Vector Dimension**: 1536 (OpenAI compatible)

**การทำงาน**:
- รับ text input และสร้าง vector embeddings
- Mock service ที่จำลอง OpenAI embedding API
- รองรับ semantic search functionality
- Health check endpoint

**Port**: `11400`
**Health Check**: HTTP GET `/health` ทุก 5 วินาที

**Environment Variables**:
```bash
VECTOR_DIM=1536
```

**คำสั่งควบคุม**:
```bash
# ดู logs
docker logs vit-llm-chat-embedder-1

# ทดสอบ health check
curl http://localhost:11400/health

# ทดสอบ embedding API
curl -X POST http://localhost:11400/embeddings \
  -H "Content-Type: application/json" \
  -d '{"input": "Hello world"}'

# เข้าไปใน container
docker exec -it vit-llm-chat-embedder-1 sh
```

---

## 🔄 การทำงานร่วมกันของระบบ

### 1. **User Request Flow**
```
User → Frontend (5173) → API (3000) → Database (5432)
                      ↓
                   Cache (6379)
                      ↓
                   Worker → Embedder (11400)
```

### 2. **Authentication Flow**
1. User login → API server
2. API server → Database (verify credentials)
3. API server → Cache (store session)
4. API server → Frontend (JWT token)

### 3. **Chat Message Flow**
1. User sends message → Frontend
2. Frontend → API server
3. API server → Database (store message)
4. API server → Worker (process for embeddings)
5. Worker → Embedder (generate vectors)
6. Worker → Database (store embeddings)
7. API server → Frontend (response)

### 4. **Search Flow**
1. User search query → Frontend
2. Frontend → API server
3. API server → Embedder (query vector)
4. API server → Database (vector similarity search)
5. API server → Frontend (search results)

---

## 🛠️ คำสั่งควบคุมระบบ

### การเริ่มต้นและหยุดระบบ
```bash
# เริ่มระบบทั้งหมด
docker-compose -f docker-compose.dev.yml up -d

# หยุดระบบทั้งหมด
docker-compose -f docker-compose.dev.yml down

# หยุดและลบ volumes
docker-compose -f docker-compose.dev.yml down -v

# หยุดและลบ orphan containers
docker-compose -f docker-compose.dev.yml down --remove-orphans

# Restart ระบบทั้งหมด
docker-compose -f docker-compose.dev.yml restart
```

### การจัดการ Services แต่ละตัว
```bash
# เริ่ม service เดียว
docker-compose -f docker-compose.dev.yml up -d llm-chat-api

# หยุด service เดียว
docker-compose -f docker-compose.dev.yml stop llm-chat-api

# Restart service เดียว
docker-compose -f docker-compose.dev.yml restart llm-chat-api

# ดู logs ของ service
docker-compose -f docker-compose.dev.yml logs -f llm-chat-api

# ดู logs ของทุก services
docker-compose -f docker-compose.dev.yml logs -f
```

### การตรวจสอบสถานะ
```bash
# ดูสถานะ services ทั้งหมด
docker-compose -f docker-compose.dev.yml ps

# ดู resource usage
docker stats

# ดู network connections
docker network inspect vit_default

# ดู volumes
docker volume ls
```

### การ Debug และ Troubleshooting
```bash
# เข้าไปใน container
docker exec -it vit-llm-chat-api-1 sh

# ดู logs แบบ real-time
docker logs -f vit-llm-chat-api-1

# ดู logs แบบ tail
docker logs --tail 100 vit-llm-chat-api-1

# ตรวจสอบ health status
docker inspect vit-llm-chat-database-1 | grep -A 10 Health

# ดู environment variables
docker exec vit-llm-chat-api-1 env

# ตรวจสอบ network connectivity
docker exec vit-llm-chat-api-1 ping llm-chat-database
```

---

## 🚨 Troubleshooting

### ปัญหาที่พบบ่อย

#### 1. **Port Already in Use**
```bash
# ตรวจสอบ port ที่ใช้งาน
netstat -tulpn | grep :3000

# หยุด process ที่ใช้ port
sudo kill -9 $(lsof -t -i:3000)

# หรือใช้ port อื่น
# แก้ไขใน docker-compose.dev.yml
```

#### 2. **Database Connection Failed**
```bash
# ตรวจสอบ database status
docker logs vit-llm-chat-database-1

# ตรวจสอบ network
docker network inspect vit_default

# ทดสอบ connection
docker exec vit-llm-chat-api-1 ping llm-chat-database
```

#### 3. **Cache Connection Failed**
```bash
# ตรวจสอบ Redis
docker logs vit-llm-chat-cache-1

# ทดสอบ Redis connection
docker exec vit-llm-chat-cache-1 redis-cli ping
```

#### 4. **Worker Not Processing Jobs**
```bash
# ตรวจสอบ worker logs
docker logs vit-llm-chat-worker-1

# ตรวจสอบ Redis queue
docker exec vit-llm-chat-cache-1 redis-cli llen bull:embeddings:waiting
```

#### 5. **Frontend Not Loading**
```bash
# ตรวจสอบ frontend logs
docker logs vit-llm-chat-frontend-1

# ตรวจสอบ port
curl http://localhost:5173

# Rebuild frontend
docker-compose restart llm-chat-frontend
```

### การแก้ไขปัญหาเฉพาะ

#### Database Issues
```bash
# Reset database
docker-compose down
docker volume rm vit_postgres_data
docker-compose up -d

# Run migrations
docker exec vit-llm-chat-api-1 npm run migrate
```

#### Cache Issues
```bash
# Clear Redis cache
docker exec vit-llm-chat-cache-1 redis-cli flushall

# Restart Redis
docker-compose restart llm-chat-cache
```

#### Build Issues
```bash
# Rebuild images
docker-compose build --no-cache

# Rebuild specific service
docker-compose build --no-cache llm-chat-api
```

---

## 📊 การ Monitor และ Performance

### การตรวจสอบ Performance
```bash
# ดู resource usage
docker stats

# ดู disk usage
docker system df

# ดู network traffic
docker network inspect vit_default

# ดู logs size
docker logs --details vit-llm-chat-api-1 | wc -l
```

### การ Backup และ Restore
```bash
# Backup database
docker exec vit-llm-chat-database-1 pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} > backup_$(date +%Y%m%d_%H%M%S).sql

# Backup Redis
docker exec vit-llm-chat-cache-1 redis-cli --rdb /data/backup.rdb
docker cp vit-llm-chat-cache-1:/data/backup.rdb ./redis_backup.rdb

# Backup volumes
docker run --rm -v vit_postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/postgres_backup.tar.gz /data
```

---

## 🔧 การพัฒนาและ Customization

### การเพิ่ม Service ใหม่
1. เพิ่ม service ใน `docker-compose.dev.yml`
2. กำหนด dependencies และ health checks
3. อัปเดต environment variables
4. ทดสอบการเชื่อมต่อ

### การเปลี่ยน Configuration
1. แก้ไข environment variables ใน `.env` file
2. Restart services ที่เกี่ยวข้อง
3. ตรวจสอบ logs เพื่อดูการเปลี่ยนแปลง

### การเพิ่ม Dependencies
1. แก้ไข `package.json` ใน service ที่เกี่ยวข้อง
2. Rebuild Docker image
3. Restart service

---

## 📝 สรุป

ระบบ LLM Chat Application นี้ใช้ microservices architecture ที่แยกหน้าที่ชัดเจน:

- **Frontend**: UI/UX และ user interaction
- **API**: Business logic และ request handling  
- **Database**: Data persistence และ vector search
- **Cache**: Performance optimization และ message queue
- **Worker**: Background processing
- **Embedder**: AI/ML functionality

การออกแบบนี้ทำให้ระบบมีความยืดหยุ่น, ขยายได้ง่าย, และบำรุงรักษาได้ดี โดยแต่ละ service สามารถ scale แยกกันได้ตามความต้องการ

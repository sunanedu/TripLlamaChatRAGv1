# Software Development Life Cycle (SDLC) - LLM Chat Application

## 📋 ภาพรวม SDLC

เอกสารนี้อธิบายขั้นตอนการพัฒนาแบบ SDLC สำหรับระบบ LLM Chat Application ที่ใช้ microservices architecture พร้อม RAG (Retrieval-Augmented Generation) functionality

### 🎯 เป้าหมายของระบบ
- สร้างแอปพลิเคชันแชทที่ใช้ AI/LLM
- รองรับการค้นหาข้อมูลแบบ Semantic Search
- ระบบจัดการผู้ใช้และธุรกิจ
- การอัปโหลดไฟล์และจัดการธุรกรรม

---

## 🔄 ขั้นตอน SDLC

### 1. 📊 Planning & Requirements Analysis

#### 1.1 การวิเคราะห์ความต้องการ (Requirements Analysis)

**Functional Requirements:**
- ระบบยืนยันตัวตน (Authentication/Authorization)
- การแชทกับ AI/LLM พร้อม RAG
- การจัดการข้อมูลธุรกิจ
- การอัปโหลดไฟล์ (สลิปการโอนเงิน)
- ระบบจัดการธุรกรรมและแพ็กเกจ
- การค้นหาข้อมูลแบบ Semantic Search

**Non-Functional Requirements:**
- Performance: รองรับผู้ใช้พร้อมกัน 100+ คน
- Scalability: สามารถขยายระบบได้
- Security: JWT authentication, file upload validation
- Availability: 99.9% uptime
- Maintainability: Code ที่อ่านง่ายและบำรุงรักษาได้

#### 1.2 การวางแผนโครงการ (Project Planning)

**Timeline:**
- Phase 1: Backend API Development (4-6 สัปดาห์)
- Phase 2: Frontend Development (3-4 สัปดาห์)
- Phase 3: Integration & Testing (2-3 สัปดาห์)
- Phase 4: Deployment & Monitoring (1-2 สัปดาห์)

**Resources:**
- Backend Developer: 1 คน
- Frontend Developer: 1 คน
- DevOps Engineer: 0.5 คน
- QA Tester: 0.5 คน

**Tools & Technologies:**
- Backend: Node.js, Express.js, PostgreSQL, Redis
- Frontend: HTML5, CSS3, JavaScript, Bootstrap
- DevOps: Docker, Docker Compose
- Testing: Jest, Postman
- Monitoring: Docker logs, Health checks

---

### 2. 🏗️ System Design & Architecture

#### 2.1 การออกแบบสถาปัตยกรรมระบบ (System Architecture)

**Microservices Architecture:**
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

#### 2.2 การออกแบบฐานข้อมูล (Database Design)

**Core Tables:**
- `users` - ข้อมูลผู้ใช้
- `businesses` - ข้อมูลธุรกิจ
- `packages` - แพ็กเกจบริการ
- `transactions` - ธุรกรรม
- `subscriptions` - การสมัครสมาชิก
- `chat_sessions` - เซสชันการแชท
- `messages` - ข้อความในแชท

**Vector Search:**
- ใช้ `pgvector` extension สำหรับ semantic search
- Embedding vectors เก็บในคอลัมน์ `embedding`

#### 2.3 การออกแบบ API (API Design)

**RESTful API Endpoints:**
- `/api/auth/*` - Authentication
- `/api/users/*` - User management
- `/api/businesses/*` - Business management
- `/api/transactions/*` - Transaction management
- `/api/chat/*` - Chat functionality
- `/api/upload/*` - File upload

**API Standards:**
- HTTP status codes
- JSON response format
- JWT authentication
- Rate limiting
- Error handling

---

### 3. 💻 Implementation Phase

#### 3.1 Backend Development

**Phase 3.1.1: Project Setup**
```bash
# สร้างโครงสร้างโปรเจค
mkdir llm-chat-backend
cd llm-chat-backend

# Initialize Node.js project
npm init -y

# Install dependencies
npm install express sequelize pg pg-hstore redis bullmq ioredis
npm install jsonwebtoken bcryptjs cors multer dotenv wordcut
npm install axios

# Install dev dependencies
npm install --save-dev nodemon sequelize-cli
```

**Phase 3.1.2: Database Setup**
```bash
# สร้างไฟล์ configuration
touch config/database.js
touch config/config.json

# สร้าง migrations
npx sequelize-cli init
npx sequelize-cli migration:generate --name create-users
npx sequelize-cli migration:generate --name create-businesses
npx sequelize-cli migration:generate --name create-packages
npx sequelize-cli migration:generate --name create-transactions
npx sequelize-cli migration:generate --name create-subscriptions
npx sequelize-cli migration:generate --name create-chat-sessions
npx sequelize-cli migration:generate --name create-messages
```

**Phase 3.1.3: Core Services Development**

**Authentication Service:**
```javascript
// src/services/authService.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

class AuthService {
  async register(userData) {
    // Hash password
    const hashedPassword = await bcrypt.hash(userData.password, 10);
    
    // Create user
    const user = await User.create({
      ...userData,
      password_hash: hashedPassword
    });
    
    // Generate tokens
    const tokens = this.generateTokens(user);
    
    return { user, tokens };
  }
  
  generateTokens(user) {
    const accessToken = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );
    
    const refreshToken = jwt.sign(
      { userId: user.id },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    return { accessToken, refreshToken };
  }
}
```

**Chat Service with RAG:**
```javascript
// src/services/chatService.js
const axios = require('axios');

class ChatService {
  async processMessage(prompt, sessionId) {
    // 1. Generate embedding for the prompt
    const embedding = await this.generateEmbedding(prompt);
    
    // 2. Search for relevant businesses
    const relevantBusinesses = await this.searchBusinesses(embedding);
    
    // 3. Generate context for LLM
    const context = this.buildContext(relevantBusinesses);
    
    // 4. Call LLM API
    const response = await this.callLLM(prompt, context);
    
    // 5. Save message to database
    await this.saveMessage(sessionId, prompt, response);
    
    return response;
  }
  
  async generateEmbedding(text) {
    const response = await axios.post(process.env.OLLAMA_EMBED_URL, {
      text: text
    });
    return response.data.embedding;
  }
  
  async searchBusinesses(embedding) {
    return await Business.findAll({
      where: {
        embedding: {
          [Op.ne]: null
        }
      },
      order: [
        [sequelize.literal(`embedding <-> '[${embedding.join(',')}]'`), 'ASC']
      ],
      limit: 5
    });
  }
}
```

**Worker Service:**
```javascript
// src/worker/worker.js
const { Worker } = require('bullmq');
const Redis = require('ioredis');

const redis = new Redis(process.env.REDIS_URL);

const worker = new Worker('embeddings', async (job) => {
  const { businessId } = job.data;
  
  // Get business data
  const business = await Business.findByPk(businessId);
  
  // Generate embedding
  const embedding = await generateEmbedding(business.description);
  
  // Update business with embedding
  await business.update({ embedding });
  
  console.log(`Generated embedding for business ${businessId}`);
}, {
  connection: redis
});

worker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
  console.error(`Job ${job.id} failed:`, err);
});
```

#### 3.2 Frontend Development

**Phase 3.2.1: Project Setup**
```bash
# สร้างโปรเจค frontend
mkdir llm-chat-frontend
cd llm-chat-frontend

# Initialize Vite project
npm create vite@latest . --template vanilla

# Install dependencies
npm install bootstrap jquery marked highlight.js sweetalert2 dompurify wordcut
```

**Phase 3.2.2: Core Components**

**Authentication Component:**
```javascript
// src/js/auth.js
class AuthManager {
  constructor() {
    this.apiBaseUrl = 'http://localhost:3000/api';
    this.token = localStorage.getItem('accessToken');
  }
  
  async login(email, password) {
    try {
      const response = await fetch(`${this.apiBaseUrl}/auth/login`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, password })
      });
      
      const data = await response.json();
      
      if (response.ok) {
        this.token = data.tokens.accessToken;
        localStorage.setItem('accessToken', this.token);
        localStorage.setItem('refreshToken', data.tokens.refreshToken);
        return data;
      } else {
        throw new Error(data.message);
      }
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    }
  }
  
  async makeAuthenticatedRequest(url, options = {}) {
    const headers = {
      'Authorization': `Bearer ${this.token}`,
      'Content-Type': 'application/json',
      ...options.headers
    };
    
    return fetch(url, { ...options, headers });
  }
}
```

**Chat Component:**
```javascript
// src/js/chat.js
class ChatManager {
  constructor(authManager) {
    this.authManager = authManager;
    this.currentSessionId = null;
    this.eventSource = null;
  }
  
  async sendMessage(prompt) {
    try {
      const response = await this.authManager.makeAuthenticatedRequest(
        `${this.authManager.apiBaseUrl}/chat/`,
        {
          method: 'POST',
          body: JSON.stringify({
            prompt: prompt,
            sessionId: this.currentSessionId
          })
        }
      );
      
      if (response.ok) {
        this.handleSSEResponse(response);
      } else {
        throw new Error('Failed to send message');
      }
    } catch (error) {
      console.error('Chat error:', error);
      throw error;
    }
  }
  
  handleSSEResponse(response) {
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    
    const readStream = async () => {
      while (true) {
        const { done, value } = await reader.read();
        
        if (done) break;
        
        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');
        
        for (const line of lines) {
          if (line.startsWith('data: ')) {
            const data = line.slice(6);
            this.handleSSEData(data);
          }
        }
      }
    };
    
    readStream();
  }
  
  handleSSEData(data) {
    try {
      const parsed = JSON.parse(data);
      
      if (parsed.type === 'session_created') {
        this.currentSessionId = parsed.sessionId;
      } else if (parsed.type === 'data') {
        this.appendMessage(parsed.content);
      } else if (parsed.type === 'end') {
        this.finishMessage();
      }
    } catch (error) {
      console.error('SSE parsing error:', error);
    }
  }
}
```

#### 3.3 Mock Embedder Service

**Phase 3.3.1: Development**
```javascript
// mock-embedder/index.js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const PORT = process.env.PORT || 11400;
const VECTOR_DIM = parseInt(process.env.VECTOR_DIM) || 1536;

app.use(bodyParser.json());

// Generate deterministic embedding
function generateEmbedding(text) {
  const hash = require('crypto').createHash('md5').update(text).digest('hex');
  const embedding = [];
  
  for (let i = 0; i < VECTOR_DIM; i++) {
    const seed = parseInt(hash.substr(i % hash.length, 2), 16);
    embedding.push((seed / 255) * 2 - 1); // Normalize to [-1, 1]
  }
  
  return embedding;
}

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ ok: true, VECTOR_DIM });
});

// Embedding endpoint
app.post('/embeddings', (req, res) => {
  const { input, text } = req.body;
  
  if (text) {
    // Single string format
    const embedding = generateEmbedding(text);
    res.json({ embedding });
  } else if (Array.isArray(input)) {
    // Array format (OpenAI-like)
    const data = input.map(text => ({
      embedding: generateEmbedding(text)
    }));
    res.json({ data });
  } else {
    res.status(400).json({ error: 'Invalid input format' });
  }
});

app.listen(PORT, () => {
  console.log(`Mock embedder running on port ${PORT}`);
});
```

---

### 4. 🧪 Testing Phase

#### 4.1 Unit Testing

**Backend Unit Tests:**
```javascript
// tests/services/authService.test.js
const AuthService = require('../src/services/authService');
const User = require('../src/models/User');

describe('AuthService', () => {
  beforeEach(() => {
    // Setup test database
  });
  
  afterEach(() => {
    // Cleanup
  });
  
  test('should register new user', async () => {
    const userData = {
      email: 'test@example.com',
      password: 'password123',
      first_name: 'Test',
      last_name: 'User'
    };
    
    const result = await AuthService.register(userData);
    
    expect(result.user.email).toBe(userData.email);
    expect(result.tokens.accessToken).toBeDefined();
    expect(result.tokens.refreshToken).toBeDefined();
  });
  
  test('should login existing user', async () => {
    const credentials = {
      email: 'test@example.com',
      password: 'password123'
    };
    
    const result = await AuthService.login(credentials);
    
    expect(result.user.email).toBe(credentials.email);
    expect(result.tokens.accessToken).toBeDefined();
  });
});
```

**Frontend Unit Tests:**
```javascript
// tests/js/auth.test.js
import { AuthManager } from '../src/js/auth.js';

describe('AuthManager', () => {
  let authManager;
  
  beforeEach(() => {
    authManager = new AuthManager();
  });
  
  test('should store token after login', async () => {
    // Mock fetch
    global.fetch = jest.fn(() =>
      Promise.resolve({
        ok: true,
        json: () => Promise.resolve({
          tokens: { accessToken: 'test-token' }
        })
      })
    );
    
    await authManager.login('test@example.com', 'password');
    
    expect(localStorage.getItem('accessToken')).toBe('test-token');
  });
});
```

#### 4.2 Integration Testing

**API Integration Tests:**
```javascript
// tests/integration/api.test.js
const request = require('supertest');
const app = require('../src/app');

describe('API Integration Tests', () => {
  let authToken;
  
  beforeAll(async () => {
    // Login and get token
    const response = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123'
      });
    
    authToken = response.body.tokens.accessToken;
  });
  
  test('should create business', async () => {
    const businessData = {
      name: 'Test Business',
      description: 'Test description',
      category: 'restaurant'
    };
    
    const response = await request(app)
      .post('/api/businesses/')
      .set('Authorization', `Bearer ${authToken}`)
      .send(businessData);
    
    expect(response.status).toBe(201);
    expect(response.body.name).toBe(businessData.name);
  });
  
  test('should send chat message', async () => {
    const messageData = {
      prompt: 'Hello, how are you?'
    };
    
    const response = await request(app)
      .post('/api/chat/')
      .set('Authorization', `Bearer ${authToken}`)
      .send(messageData);
    
    expect(response.status).toBe(200);
    expect(response.headers['content-type']).toContain('text/event-stream');
  });
});
```

#### 4.3 End-to-End Testing

**E2E Test Script:**
```javascript
// tests/e2e/chat-flow.test.js
const puppeteer = require('puppeteer');

describe('Chat Flow E2E', () => {
  let browser;
  let page;
  
  beforeAll(async () => {
    browser = await puppeteer.launch();
    page = await browser.newPage();
  });
  
  afterAll(async () => {
    await browser.close();
  });
  
  test('should complete chat flow', async () => {
    // Navigate to login page
    await page.goto('http://localhost:5173/login');
    
    // Login
    await page.type('#email', 'test@example.com');
    await page.type('#password', 'password123');
    await page.click('#login-btn');
    
    // Wait for redirect to chat
    await page.waitForSelector('#chat-container');
    
    // Send message
    await page.type('#message-input', 'Hello, how are you?');
    await page.click('#send-btn');
    
    // Wait for response
    await page.waitForSelector('.message-response');
    
    // Verify response exists
    const response = await page.$eval('.message-response', el => el.textContent);
    expect(response).toBeTruthy();
  });
});
```

---

### 5. 🚀 Deployment Phase

#### 5.1 Docker Configuration

**Dockerfile for Backend:**
```dockerfile
# Dockerfile.dev
FROM node:20-alpine

WORKDIR /usr/src/app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --silent

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start command
CMD ["npm", "run", "dev"]
```

**Docker Compose Configuration:**
```yaml
# docker-compose.dev.yml
services:
  llm-chat-api:
    build:
      context: ./llm-chat-backend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./llm-chat-backend:/usr/src/app
      - ./llm-chat-backend/node_modules:/usr/src/app/node_modules
    environment:
      - NODE_ENV=development
      - PORT=3000
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@llm-chat-database:5432/${POSTGRES_DB}
      - REDIS_URL=redis://llm-chat-cache:6379
      - JWT_SECRET=${JWT_SECRET}
      - OLLAMA_EMBED_URL=http://llm-chat-embedder:11400/embeddings
    depends_on:
      llm-chat-database:
        condition: service_healthy
      llm-chat-cache:
        condition: service_healthy
      llm-chat-embedder:
        condition: service_healthy
```

#### 5.2 Environment Configuration

**Environment Variables:**
```bash
# .env
# Database
POSTGRES_DB=llmchat_db
POSTGRES_USER=llmchat_user
POSTGRES_PASSWORD=strongpassword

# JWT
JWT_SECRET=replace_with_a_real_secret

# Server
SERVER_URL=http://localhost:3000

# External APIs
OLLAMA_API_URL=http://host.docker.internal:11434/api/generate
OLLAMA_EMBED_URL=http://llm-chat-embedder:11400/embeddings

# Redis
REDIS_URL=redis://llm-chat-cache:6379
```

#### 5.3 Deployment Commands

**Development Deployment:**
```bash
# Clone repository
git clone <repository-url>
cd llm-chat-application

# Setup environment
cp .env.example .env
# Edit .env with your values

# Start all services
docker-compose -f docker-compose.dev.yml up -d

# Run database migrations
docker-compose -f docker-compose.dev.yml exec llm-chat-api npx sequelize-cli db:migrate

# Seed initial data
docker-compose -f docker-compose.dev.yml exec llm-chat-api npm run seed:null-business

# Check service status
docker-compose -f docker-compose.dev.yml ps
```

**Production Deployment:**
```bash
# Build production images
docker-compose -f docker-compose.prod.yml build

# Deploy to production
docker-compose -f docker-compose.prod.yml up -d

# Setup SSL certificates
# Configure reverse proxy (nginx)
# Setup monitoring and logging
```

---

### 6. 🔧 Maintenance & Monitoring Phase

#### 6.1 Monitoring Setup

**Health Checks:**
```javascript
// src/middlewares/healthCheck.js
const healthCheck = (req, res) => {
  const healthcheck = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now(),
    checks: {
      database: 'unknown',
      redis: 'unknown',
      embedder: 'unknown'
    }
  };
  
  // Check database
  sequelize.authenticate()
    .then(() => healthcheck.checks.database = 'OK')
    .catch(() => healthcheck.checks.database = 'ERROR');
  
  // Check Redis
  redis.ping()
    .then(() => healthcheck.checks.redis = 'OK')
    .catch(() => healthcheck.checks.redis = 'ERROR');
  
  // Check embedder
  axios.get(process.env.OLLAMA_EMBED_URL.replace('/embeddings', '/health'))
    .then(() => healthcheck.checks.embedder = 'OK')
    .catch(() => healthcheck.checks.embedder = 'ERROR');
  
  res.status(200).json(healthcheck);
};
```

**Logging Configuration:**
```javascript
// src/utils/logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' }),
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});

module.exports = logger;
```

#### 6.2 Performance Monitoring

**Metrics Collection:**
```javascript
// src/middlewares/metrics.js
const prometheus = require('prom-client');

// Create metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Middleware to collect metrics
const metricsMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const labels = {
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode
    };
    
    httpRequestDuration.observe(labels, duration);
    httpRequestTotal.inc(labels);
  });
  
  next();
};
```

#### 6.3 Backup & Recovery

**Database Backup Script:**
```bash
#!/bin/bash
# backup-database.sh

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="llmchat_backup_${DATE}.sql"

# Create backup directory
mkdir -p $BACKUP_DIR

# Create database backup
docker exec vit-llm-chat-database-1 pg_dump \
  -U ${POSTGRES_USER} \
  -d ${POSTGRES_DB} \
  > ${BACKUP_DIR}/${BACKUP_FILE}

# Compress backup
gzip ${BACKUP_DIR}/${BACKUP_FILE}

# Remove old backups (keep last 7 days)
find ${BACKUP_DIR} -name "llmchat_backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: ${BACKUP_FILE}.gz"
```

**Redis Backup Script:**
```bash
#!/bin/bash
# backup-redis.sh

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="redis_backup_${DATE}.rdb"

# Create backup directory
mkdir -p $BACKUP_DIR

# Create Redis backup
docker exec vit-llm-chat-cache-1 redis-cli --rdb /data/${BACKUP_FILE}
docker cp vit-llm-chat-cache-1:/data/${BACKUP_FILE} ${BACKUP_DIR}/

# Compress backup
gzip ${BACKUP_DIR}/${BACKUP_FILE}

echo "Redis backup completed: ${BACKUP_FILE}.gz"
```

#### 6.4 Security Maintenance

**Security Checklist:**
- [ ] Regular dependency updates
- [ ] JWT secret rotation
- [ ] Database password rotation
- [ ] SSL certificate renewal
- [ ] Security vulnerability scanning
- [ ] Access log monitoring
- [ ] Rate limiting configuration
- [ ] File upload validation

**Automated Security Updates:**
```bash
#!/bin/bash
# security-update.sh

# Update dependencies
docker-compose -f docker-compose.dev.yml exec llm-chat-api npm audit fix

# Scan for vulnerabilities
docker-compose -f docker-compose.dev.yml exec llm-chat-api npm audit

# Update Docker images
docker-compose -f docker-compose.dev.yml pull
docker-compose -f docker-compose.dev.yml up -d

echo "Security updates completed"
```

---

### 7. 📈 Continuous Improvement

#### 7.1 Performance Optimization

**Database Optimization:**
```sql
-- Create indexes for better performance
CREATE INDEX idx_businesses_embedding ON businesses USING ivfflat (embedding vector_cosine_ops);
CREATE INDEX idx_messages_session_id ON messages(session_id);
CREATE INDEX idx_transactions_user_id ON transactions(user_id);
CREATE INDEX idx_subscriptions_user_id ON subscriptions(user_id);
```

**Caching Strategy:**
```javascript
// src/middlewares/cache.js
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

const cacheMiddleware = (duration = 600) => {
  return (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    const cached = cache.get(key);
    
    if (cached) {
      return res.json(cached);
    }
    
    res.sendResponse = res.json;
    res.json = (body) => {
      cache.set(key, body, duration);
      res.sendResponse(body);
    };
    
    next();
  };
};
```

#### 7.2 Feature Enhancement

**New Feature Development Process:**
1. **Requirement Analysis** - วิเคราะห์ความต้องการใหม่
2. **Design Review** - ออกแบบและ review
3. **Implementation** - พัฒนาฟีเจอร์
4. **Testing** - ทดสอบอย่างละเอียด
5. **Code Review** - Review โค้ด
6. **Deployment** - Deploy ไปยัง production
7. **Monitoring** - ติดตามผลลัพธ์

**Feature Flag System:**
```javascript
// src/utils/featureFlags.js
const featureFlags = {
  NEW_CHAT_UI: process.env.FEATURE_NEW_CHAT_UI === 'true',
  ADVANCED_SEARCH: process.env.FEATURE_ADVANCED_SEARCH === 'true',
  FILE_UPLOAD_V2: process.env.FEATURE_FILE_UPLOAD_V2 === 'true'
};

const isFeatureEnabled = (feature) => {
  return featureFlags[feature] || false;
};
```

---

### 8. 📚 Documentation & Knowledge Management

#### 8.1 Technical Documentation

**API Documentation:**
- Swagger/OpenAPI specification
- Postman collection
- Code examples
- Error handling guide

**Architecture Documentation:**
- System design documents
- Database schema
- Deployment guides
- Troubleshooting guides

#### 8.2 Knowledge Transfer

**Developer Onboarding:**
1. Environment setup guide
2. Code style guidelines
3. Testing procedures
4. Deployment process
5. Monitoring and alerting

**Code Review Guidelines:**
- Code quality standards
- Security best practices
- Performance considerations
- Documentation requirements

---

## 🎯 สรุป SDLC Process

### Development Workflow:
1. **Planning** → Requirements analysis และ project planning
2. **Design** → System architecture และ database design
3. **Implementation** → Backend, Frontend, และ Services development
4. **Testing** → Unit, Integration, และ E2E testing
5. **Deployment** → Docker containerization และ deployment
6. **Maintenance** → Monitoring, backup, และ security maintenance
7. **Improvement** → Performance optimization และ feature enhancement

### Key Success Factors:
- **Microservices Architecture** - แยกหน้าที่ชัดเจน
- **Containerization** - ใช้ Docker สำหรับ consistency
- **Automated Testing** - ทดสอบอัตโนมัติทุกขั้นตอน
- **Monitoring** - ติดตามระบบอย่างต่อเนื่อง
- **Documentation** - เอกสารที่ครบถ้วนและอัปเดต
- **Security** - ความปลอดภัยในทุกขั้นตอน
- **Scalability** - ออกแบบให้ขยายได้

ระบบ LLM Chat Application นี้ใช้ SDLC ที่ครอบคลุมทุกขั้นตอนตั้งแต่การวางแผนจนถึงการบำรุงรักษา เพื่อให้ได้ระบบที่มีคุณภาพสูง มีความเสถียร และสามารถขยายได้ตามความต้องการในอนาคต

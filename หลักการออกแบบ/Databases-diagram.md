# Database Diagram - LLM Chat System

## 📊 Database Architecture Overview

ระบบใช้ **PostgreSQL 15 + pgvector extension** สำหรับจัดการข้อมูลและการค้นหาแบบ Semantic Search ด้วย embeddings

```
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Database                           │
│                  (Port: 5432)                                    │
│                  vit-llm-chat-database-1                        │
├───────────────────────────────────────────────────────────────────┤
│  Database: llmchat_db                                           │
│  User: llmchat_user                                             │
│  Extensions: pgvector (vector similarity search)                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📋 Database Tables

### 1. 👤 users
**วัตถุประสงค์**: จัดการข้อมูลผู้ใช้ และ authentication

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: users                                                   │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  UUID        │  PK         │  User ID       │
│  email           │  STRING      │  UNIQUE     │  Email address │
│  password_hash   │  STRING      │  NOT NULL   │  Bcrypt hash   │
│  role            │  STRING      │  CHECK      │  User role     │
│                  │              │             │  (admin/staff  │
│                  │              │             │  /sponsor/     │
│                  │              │             │  member_pro/   │
│                  │              │             │  member/guest)  │
│  first_name      │  STRING      │             │  First name    │
│  last_name       │  STRING      │             │  Last name     │
│  limit_per_minute│  INTEGER     │  DEF:60     │  API rate limit│
│  is_protected    │  BOOLEAN     │  DEF:false │  Protected user│
│  is_suspended    │  BOOLEAN     │  DEF:false │  Suspended user│
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- จัดเก็บข้อมูล authentication (JWT-based)
- จัดการ role-based access control (RBAC)
- กำหนด rate limiting ต่อ user
- ป้องกันการลบ user สำคัญ (is_protected = true)

**Relationships**:
- One-to-One: `tokens` (refresh tokens)
- One-to-Many: `businesses`, `chat_sessions`, `subscriptions`, `transactions`

---

### 2. 🔑 tokens
**วัตถุประสงค์**: จัดเก็บ refresh tokens สำหรับ JWT authentication

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: tokens                                                  │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  user_id         │  UUID        │  PK, FK     │  Reference     │
│                  │              │             │  to users.id    │
│  refresh_token   │  STRING      │  NOT NULL   │  JWT token     │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- จัดเก็บ refresh tokens สำหรับ token rotation
- ใช้สำหรับ refresh JWT access tokens
- 1 user มีได้ 1 refresh token (One-to-One)

**Relationships**:
- Belongs-to: `users` (user_id)

---

### 3. 💬 chat_sessions
**วัตถุประสงค์**: จัดเก็บ chat sessions ของผู้ใช้แต่ละคน

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: chat_sessions                                           │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  UUID        │  PK         │  Session ID    │
│  user_id         │  UUID        │  FK         │  Reference     │
│                  │              │             │  to users.id    │
│  title           │  STRING      │             │  Session title │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- จัดการ chat sessions (แต่ละการสนทนา)
- เก็บ title ของ chat เพื่อแสดงใน history
- อัพเดท `updated_at` เมื่อมีการส่งข้อความใหม่

**Relationships**:
- Belongs-to: `users` (user_id)
- One-to-Many: `chat_messages` (has many messages)

---

### 4. 💬 chat_messages
**วัตถุประสงค์**: จัดเก็บข้อความใน chat sessions

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: chat_messages                                           │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Message ID    │
│  session_id      │  UUID        │  FK         │  Reference     │
│                  │              │             │  to chat_      │
│                  │              │             │  sessions.id   │
│  role            │  STRING      │             │  'user' or     │
│                  │              │             │  'assistant'   │
│  content         │  TEXT        │             │  Message text │
│  created_at      │  TIMESTAMP   │             │  Created time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- เก็บข้อความที่ user ส่ง (role = 'user')
- เก็บคำตอบที่ LLM/AI สร้าง (role = 'assistant')
- ไม่มี `updated_at` เพราะ message ไม่มีการแก้ไข
- เรียงลำดับตาม `created_at` เพื่อแสดงประวัติการสนทนา

**Relationships**:
- Belongs-to: `chat_sessions` (session_id)
- Messages ใน session เดียวถูกจัดเก็บตามลำดับเวลา

---

### 5. 🏢 businesses
**วัตถุประสงค์**: จัดเก็บข้อมูลสถานประกอบการ (Business listings)

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: businesses                                              │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Business ID   │
│  user_id         │  UUID        │  FK         │  Reference         │
│                  │              │             │  to users.id    │
│  name            │  STRING      │  NOT NULL   │  Business name  │
│  description     │  TEXT        │             │  Description   │
│  business_type   │  STRING      │             │  Type (โรงแรม   │
│                  │              │             │  ์,ร้านอาหาร)    │
│  package_id      │  INTEGER     │             │  Package ref    │
│  contact_email   │  STRING      │             │  Contact email │
│  phone_number    │  STRING      │             │  Phone number  │
│  website_url     │  TEXT        │             │  Website URL   │
│  embedding       │  VECTOR(1536)│             │  pgvector      │
│                  │              │             │  embedding     │
│  is_active       │  BOOLEAN     │  DEF:true   │  Active status │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- เก็บข้อมูลสถานประกอบการ (โรงแรม ร้านอาหาร)
- ใช้ `embedding` (pgvector) สำหรับ semantic search
- ค้นหาด้วย cosine similarity จาก embeddings
- `is_active` ใช้กรองสถานที่ที่เปิดให้บริการ

**Relationships**:
- Belongs-to: `users` (user_id - owner)
- One-to-Many: `business_images` (มีรูปภาพหลายรูป)
- One-to-Many: `business_packages` (มีหลาย packages)

---

### 6. 📦 packages
**วัตถุประสงค์**: จัดเก็บข้อมูลแพ็คเกจสำหรับ business subscription

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: packages                                                │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Package ID    │
│  name            │  STRING      │             │  Package name  │
│  price           │  DECIMAL(10) │             │  Package price │
│  rate_limit_per  │  INTEGER     │             │  API rate      │
│  _min            │              │             │  limit/min     │
│  is_active       │  BOOLEAN     │  DEF:true   │  Active status │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- เก็บข้อมูลแพ็คเกจ subscription สำหรับธุรกิจ
- กำหนด rate limiting สำหรับแต่ละแพ็คเกจ
- `is_active` ใช้เปิด/ปิดการขายแพ็คเกจ

**Relationships**:
- One-to-Many: `subscriptions`, `transactions`, `businesses`

---

### 7. 🔗 business_packages
**วัตถุประสงค์**: Join table ระหว่าง businesses และ packages

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: business_packages                                       │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  business_id     │  INTEGER     │  PK, FK     │  Business ref   │
│  package_id      │  INTEGER     │  PK, FK     │  Package ref    │
│  started_at      │  TIMESTAMP   │             │  Start date     │
│  ended_at        │  TIMESTAMP   │             │  End date       │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- Many-to-Many relationship ระหว่าง businesses และ packages
- เก็บวันที่เริ่มต้นและสิ้นสุดของ subscription

**Relationships**:
- Belongs-to: `businesses` (business_id)
- Belongs-to: `packages` (package_id)

---

### 8. 📸 business_images
**วัตถุประสงค: เก็บรูปภาพของธุรกิจ

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: business_images                                        │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Image ID      │
│  business_id     │  INTEGER     │  FK         │  Business ref   │
│  image_url       │  STRING      │             │  Image URL     │
│  caption         │  STRING      │             │  Caption       │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- เก็บรูปภาพของธุรกิจ
- รองรับรูปภาพได้หลายรูปต่อธุรกิจ
- เก็บ caption สำหรับแต่ละรูปภาพ

**Relationships**:
- Belongs-to: `businesses` (business_id)

---

### 9. 💳 subscriptions
**วัตถุประสงค์**: เก็บข้อมูล subscription ของผู้ใช้

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: subscriptions                                           │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Subscription ID│
│  user_id         │  UUID        │  FK         │  User ref      │
│  package_id      │  INTEGER     │  FK         │  Package ref    │
│  status          │  STRING      │             │  Status        │
│  started_at      │  TIMESTAMP   │             │  Start date    │
│  ended_at        │  TIMESTAMP   │             │  End date      │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time    │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- จัดการ subscription ของ users
- เก็บสถานะและระยะเวลาของ subscription
- ใช้ตรวจสอบสิทธิ์การใช้งาน features

**Relationships**:
- Belongs-to: `users` (user_id)
- Belongs-to: `packages` (package_id)

---

### 10. 💰 transactions
**วัตถุประสงค์**: เก็บข้อมูลการทำธุรกรรม

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: transactions                                           │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Transaction ID│
│  user_id         │  UUID        │  FK         │  User ref      │
│  package_id      │  INTEGER     │  FK         │  Package ref    │
│  promotion_id    │  INTEGER     │  FK         │  Promotion ref  │
│  amount          │  DECIMAL(10) │             │  Amount paid   │
│  status          │  STRING      │             │  Status        │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- เก็บประวัติการทำธุรกรรม
- รองรับ promotion codes
- ใช้สำหรับ payment tracking

**Relationships**:
- Belongs-to: `users` (user_id)
- Belongs-to: `packages` (package_id)
- Belongs-to: `promotions` (promotion_id)

---

### 11. 🎁 promotions
**วัตถุประสงค์**: เก็บข้อมูล promotion codes

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: promotions                                              │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Promotion ID  │
│  code            │  STRING      │  UNIQUE     │  Promo code    │
│  discount        │  DECIMAL     │             │  Discount %    │
│  is_active       │  BOOLEAN     │             │  Active status │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- จัดเก็บ promotion codes
- ใช้ลดราคาในการซื้อแพ็คเกจ

**Relationships**:
- One-to-Many: `transactions`

---

### 12. 🧾 payment_slips
**วัตถุประสงค์**: เก็บหลักฐานการชำระเงิน (slip images)

```
┌─────────────────────────────────────────────────────────────────┐
│  Table: payment_slips                                          │
├──────────────────┬──────────────┬─────────────┬────────────────┤
│  Column          │  Type        │  Constraints│  Description   │
├──────────────────┼──────────────┼─────────────┼────────────────┤
│  id              │  INTEGER     │  PK, AI     │  Slip ID       │
│  transaction_id  │  INTEGER     │  FK         │  Transaction   │
│  user_id         │  UUID        │  FK         │  User ref      │
│  image_url       │  STRING      │             │  Slip image URL│
│  status          │  STRING      │             │  Status        │
│  created_at      │  TIMESTAMP   │             │  Created time  │
│  updated_at      │  TIMESTAMP   │             │  Updated time  │
└──────────────────┴──────────────┴─────────────┴────────────────┘
```

**การใช้งาน**:
- เก็บไฟล์รูปภาพของสลิปการชำระเงิน
- ใช้สำหรับการยืนยันการชำระเงินจาก admin

**Relationships**:
- Belongs-to: `transactions` (transaction_id)
- Belongs-to: `users` (user_id)

---

## 🔗 Entity Relationships (ER)

```
┌───────────────────────────────────────────────────────────────────┐
│                    Database Relationships                         │
└───────────────────────────────────────────────────────────────────┘

User (1) ───────────────► (many) ChatSession
User (1) ───────────────► (many) Business
User (1) ───────────────► (1) Token
User (1) ───────────────► (many) Subscription
User (1) ───────────────► (many) Transaction
User (1) ───────────────► (many) PaymentSlip

ChatSession (1) ───────────────► (many) ChatMessage

Business (1) ───────────────► (many) BusinessImage
Business (1) ───────────────► (many) BusinessPackage

Package (1) ───────────────► (many) Subscription
Package (1) ───────────────► (many) Transaction
Package (1) ───────────────► (many) BusinessPackage

Promotion (1) ───────────────► (many) Transaction

Transaction (1) ───────────────► (many) PaymentSlip
```

---

## 🎯 Database Design Principles

### 1. **Normalization**
- ตารางแยกตามหน้าที่ชัดเจน (users, sessions, messages)
- หลีกเลี่ยงข้อมูลซ้ำซ้อน
- ใช้ foreign keys เพื่อ integrity

### 2. **Indexing**
- **Primary Keys**: ทั้ง UUID และ INTEGER (auto-increment)
- **Foreign Keys**: แบบ cascading deletes สำหรับ relationships ที่เหมาะสม
- **Unique Constraints**: email (users), code (promotions)

### 3. **Vector Search (pgvector)**
- **Field**: `embedding` ใน `businesses` table
- **Type**: `vector(1536)` (1536 dimensions)
- **Use Case**: Semantic search สำหรับค้นหาธุรกิจตามคำอธิบาย
- **Query**: Cosine similarity (`<=>` operator)

### 4. **Timestamps**
- **Standard**: `created_at`, `updated_at`
- **Exceptions**: `chat_messages` ไม่มี `updated_at` (ข้อมูล immutable)

### 5. **Soft Deletes**
- **Pattern**: `is_active`, `is_suspended`, `is_protected` flags
- **Purpose**: เก็บประวัติข้อมูล แทนการลบถาวร

### 6. **UUID vs INTEGER**
- **UUID**: users, sessions (distributed systems)
- **INTEGER**: Businesses, transactions (performance)

---

## 🔍 Query Patterns

### 1. **Authentication Flow**
```sql
-- Find user by email
SELECT * FROM users WHERE email = 'user@example.com';

-- Get refresh token
SELECT refresh_token FROM tokens WHERE user_id = ?
```

### 2. **Chat History**
```sql
-- Get all sessions for user
SELECT * FROM chat_sessions WHERE user_id = ? ORDER BY updated_at DESC;

-- Get messages in session
SELECT * FROM chat_messages WHERE session_id = ? ORDER BY created_at ASC;
```

### 3. **Semantic Search**
```sql
-- Find businesses by embedding similarity
SELECT id, name, embedding
FROM businesses
WHERE embedding <=> ?::vector
ORDER BY embedding <=> ?::vector
LIMIT 10;
```

---

**Updated**: October 28, 2025
**Database Version**: PostgreSQL 15 + pgvector

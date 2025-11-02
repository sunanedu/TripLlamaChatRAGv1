# Begin-user-use-flow-techstark-step-by-step-to-end.md

## 1. ผู้ใช้เข้าใช้งาน (Frontend/App)

- **UI/Entry Point**
  - SPA หน้า `Chat` ของระบบ (React/JS/Vue หรือ HTML + JS) เช่น `https://frontend.sri-ketguide.com/#/chat`
  - ผู้ใช้กรอกอีเมล guest@example.com (หรือเข้าสู่ระบบอัตโนมัติถ้าเป็น guest)

- **Login/Authentication Workflow**
  - กด “เข้าสู่ระบบ” (POST `/api/auth/login`)
    - โปรโตคอล: HTTP, Content-Type: `application/json`
    - Request payload:
      ```json
      {
          "email": "guest@example.com",
          "password": "1234"
      }
      ```
    - ระบบ Auth API (Node.js/Express, Controller: `auth.controller.js`) ตรวจสอบ username/password
    - ถ้าสำเร็จ ตอบกลับ:
      ```json
      {
        "tokens": {
          "accessToken": "JWT-TOKEN...",  // ใช้สำหรับ auth ถัดไป
          "refreshToken": "..."           // สำหรับต่อ session
        },
        "user": {
          "id": "xx...",
          "role": "guest",
          ...
        }
      }
      ```
  - Frontend เก็บ accessToken (เช่น memory/LocalStorage)

---

## 2. “สร้างคำถาม” ที่กล่องแชท

- ผู้ใช้พิมพ์ เช่น “โรงแรม B2 Sisaket Boutique & Budget Hotel มีห้องพักประเภทใดบ้าง และแต่ละประเภทมีราคาเท่าไหร่?” แล้วกดส่ง
- Frontend สร้าง “chat payload” และเรียก [Chat Completion API]

### Request:
- **API Endpoint:** `POST /api/chat`
- **Headers:**  
  - `Authorization: Bearer {accessToken}`
  - `Content-Type: application/json`
- **Body/Payload Example:**
  ```json
  {
    "prompt": "โรงแรม B2 Sisaket Boutique & Budget Hotel มีห้องพักประเภทใดบ้าง และแต่ละประเภทมีราคาเท่าไหร่?",
    "sessionId": null,
    "responseMode": "quick" // หรือ detailed/comparison
  }
  ```
---

## 3. Backend Service Stack ทำงาน (llm-chat-backend)

1. **รับ Request**
    - Node.js/Express รับ POST `/api/chat` (Controller: `chat.controller.js`)
    - ดึง JWT, ถอดรหัส, ระบุ user/session

2. **Log + สร้าง Conversation Session**
    - Log คำถาม/สร้าง `ChatSession` ใหม่ ใน PostgreSQL (Table: `chat_sessions`)
    - เก็บประวัติ/คำถามลง Table `chat_messages` (user→AI) พร้อม timestamp

3. **RAG Retrieval (Embeddings + Keyword)**
    - [RAG] เรียก service ฝั่ง backend เพื่อดึง context เกี่ยวกับ “โรงแรม B2...”: 
        - **Vector Search:** 
            - เรียกค้นตาราง/vector index (pgvector) ใน `businesses`/`business_vectors`/`business_descriptions`
            - ใช้ embedding model (เช่น OpenAI/ollama-embedder/MockEmbedder) → Vector API รูปแบบ HTTP/REST (Request “embed text” ได้เวกเตอร์/array กลับ)
        - **Keyword Filter:** 
            - ค้นในช่อง “name”, “description” ด้วย wordcut/LIKE (Node.js + wordcut/thai-cut-slim)
        - รวมผล context ที่แมตช์มากสุดในหมู่โรงแรม/คำที่ถาม
        - ใส่ลง context string (คัด db field ที่จำเป็น เช่น ชื่อ, ประเภท, หัวข้อ, ราคา, เบอร์โทร, ที่อยู่, สิ่งอำนวยความสะดวก)

4. **Prompt Engineering/Prompt Template**
    - ประกอบ System Prompt (เช่นจาก `getSystemPromptForMode`)
      - รวม basePrompt, mode, + instruction พิเศษ (เช่น bullet list, ห้ามแต่ง, ใช้ key info, รูปแบบตัวอย่าง)
    - ใส่ CONVERSATION_HISTORY, CONTEXT, QUESTION, HOW_TO_RESPOND สลับ Section ดังนี้:

      ```
      SYSTEM PROMPT: ...[บทบาท, กฎ, แนวพิเศษ ฯลฯ]...
      HISTORY: ...[สนทนาก่อนหน้า (ลำดับข้อความ user<->ai)]...
      CONTEXT FROM DATABASE: ...[raw context, field หลัก, รายการ, รูปแบบ list หรือแยกบรรทัด]...
      คำถาม: ...prompt...
      คำตอบของคุณ: [**ให้ LLM ตอบต่อไปตรงนี้**]
      ```

5. **เรียก LLM Service**
    - Cloud LLM (OpenAI, Huggingface, LoRA/Alpaca) หรือ Local LLM
    - ผ่าน SDK HTTP หรือ REST API เช่น POST `/generate`
    - ส่ง prompt (หลักพัน token), context, history แบบ String
    - รับ Event Stream (Server-Sent Events, SSE)/response callback
    - Service ฝั่ง LLM มีชื่อว่า `llm-chat-worker` (ดึงคิว, สร้างคำตอบ, ส่งคืนทันทีหรือแบบ stream line-by-line)

---

## 4. Database Access/Logging

- LLM ไม่เข้าถึง DB โดยตรง แต่ผ่าน Intermediate services (Controller/RAG Layer)
- POSTGRES: ตารางหลัก
  - `businesses` (ข้อมูลโรงแรม, ที่อยู่, contact, ราคา, จุดเด่น, ฯลฯ)
  - `business_vectors`, `embeddings`
  - `chat_sessions`, `chat_messages` (log session, history)
- บางครั้งมี Redis (cache vector search, session, token, rate-limit)

---

## 5. ส่งข้อมูลกลับ (Streaming/Chunk)

- “worker/LLM” ส่งข้อความทีละ chunk, frontend รับผ่าน SSE (เช่น `event: data`, `data: {...}`)
- API `/api/chat` ตอบกลับเป็น HTTP chunked transfer/SSE stream ให้ frontend แสดงข้อความแบบทีละคำ/บรรทัด

### รูปแบบที่หน้าบราวเซอร์รับ
- front รับ “event stream” หรือ message buffer
  - event: message
  - data: { content: ... }
- เมื่อ LLM ตอบครบ ส่ง “end” event
- Frontend เติมข้อความลงกล่องสนทนาแบบ real-time (JS/React)

---

## 6. สรุปชื่อบริการ/เครื่อง/โปรโตคอล/Data Shape สำคัญ

- **Frontend:** SPA/React+Bootstrap
- **APIs:**
  - `/api/auth/login` (POST JSON, JWT)
  - `/api/chat` (POST JSON, Bearer JWT, Response: SSE/HTTP stream)
- **Services:** Express Controller `auth.controller.js`, `chat.controller.js`, LLM Worker, Retrieval, pgvector, Redis
- **Database:** PostgreSQL + pgvector + keyword search
- **Intermediate APIs/Microservices:** Embeddings, RAG Search, LLM Service (OpenAI/ollama/worker)
- **Data flow:** JSON (REST/HTTP, SSE), Vector array, text
- **Security:** JWT Auth, Access control @ controller, session/refresh tokens
- **Realtime:** Server-sent events (SSE), chunk, stream
- **Logging:** DB logging (chat_sessions, chat_messages), access log

---


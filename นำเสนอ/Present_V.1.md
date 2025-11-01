# LLM Chat RAG Platform — Presentation Blueprint (Version 1)

> **Goal**: arm a third-party presenter with a complete storyline that convinces stakeholders why the system matters *and* how it works under the hood.

---

## 1. Presentation Flow (สำหรับผู้บรรยาย)

1. **Warm-up (2 นาที)** — เกริ่นปัญหา/ความต้องการของผู้ใช้ธุรกิจท้องถิ่นที่ต้องการตอบลูกค้ารวดเร็วและตรงประเด็น
2. **Vision & Concept (3 นาที)** — อธิบายว่าแพลตฟอร์มนี้แก้ปัญหาอะไร และมอบประสบการณ์แบบใดให้ผู้ใช้ทั่วไป
3. **Demo Narrative (5 นาที)** — เล่า journey ตั้งแต่การล็อกอิน จนถึงการได้คำตอบ พร้อมชี้ให้เห็นเทคโนโลยีที่ทำงานอยู่เบื้องหลัง
4. **Technical Deep Dive (7 นาที)** — เจาะสถาปัตยกรรม, data flow, pipeline, การฝังข้อมูล (embedding), gateway ไปยัง LINE/Facebook/Telegram
5. **Development Story (3 นาที)** — ลำดับขั้นตอนการพัฒนา, major milestones, team rhythm
6. **Challenges & Lessons (3 นาที)** — ปัญหาหลัก, วิธีแก้, สิ่งที่เรียนรู้
7. **Future & Business Expansion (3 นาที)** — โอกาสในอนาคต, โมเดลรายได้, การต่อยอด ecosystem
8. **Closing (2 นาที)** — Recap “สิ่งนี้คืออะไร”, Impact, Call-to-action

> เวลารวม ~28 นาที (ปรับได้ตามบริบท)

---

## 2. Dual-Layer Narrative (General vs Technical)

| Audience | Key Messages | Supporting Visual/Artifact |
| --- | --- | --- |
| **Business / Non-technical** | • ระบบช่วยธุรกิจตอบลูกค้า 24/7<br>• เนื้อหามาจากข้อมูลจริงของธุรกิจ (ลดข้อมูลผิดๆ)<br>• เชื่อม LINE / Web / PWA ในคลิกเดียว | • Slides โชว์ flow ง่ายๆ<br>• Screenshot หน้า Admin & Chat<br>• Storyboard journey ลูกค้า |
| **Technical** | • Hybrid architecture: RAG + Messaging Gateway + Multi-tenant admin<br>• Deployment via Docker Compose + Cloudflare Tunnel<br>• Embedding job orchestration (BullMQ + Worker) | • System diagram (Data flow, API, Worker)<br>• Table Tech Stack<br>• Sequence diagram login → answer |

---

## 3. Vision & Concept

- **Mission**: “ทำให้ข้อมูลธุรกิจท้องถิ่นถูก ‘เข้าใจ’ และ ‘สื่อสาร’ ได้เสมือนมีผู้ช่วยส่วนตัวที่ตอบได้ทั้งภาษาไทยและภาษาอังกฤษ ตลอดเวลา”
- **Key Outcomes**
  - ลูกค้าได้รับคำตอบที่ตรงกับข้อมูลจริง (นำมาจากฐานข้อมูลธุรกิจ + เอกสารที่อัพโหลด)
  - เจ้าของธุรกิจบริหารข้อมูล/สลิปชำระเงิน/แพ็กเกจได้จากหน้า Admin เดียว
  - ทีมบริการเชื่อมต่อแชทผ่าน LINE Official หรือ Messaging Gateway ได้ทันที

---

## 4. แนวคิดสถาปัตยกรรม & Tech Stack

### 4.1 ภาพรวมสถาปัตยกรรม

```
User (Web / PWA / LINE)
        │ (HTTPS / Webhook)
        ▼
Cloudflare Tunnel ➜ Messaging Gateway ➜ LLM Chat API
                                     │
                                     ├─> RAG Pipeline (Postgres + pgvector)
                                     └─> Background Worker (BullMQ, Redis)

Admin Portal / Sponsor Portal ─┬─> APIs (Auth, Business, Payment)
                               └─> Embedding Dashboard & Jobs
```

### 4.2 ชุดเทคโนโลยีสำคัญ

- **Frontend SPA**: Vite + Vanilla JS + Bootstrap (Web) / PWA build for mobile-like experience
- **Backend**: Node.js (Express 5), Sequelize ORM, PostgreSQL + pgvector, Redis, BullMQ queue
- **LLM & RAG**: Embedding service (Ollama/Custom) + Vector search + Context builder
- **Messaging Gateway**: Express + LINE Bot SDK + Facebook/Telegram hooks (พร้อมขยาย)
- **Deployment**: Docker Compose (dev / prod profiles) + Cloudflare Tunnel (secure ingress)
- **Observability**: `/health` endpoints, job dashboard, event logs (`events.log`), Admin embedding tab

---

## 5. ลำดับการพัฒนา (จากศูนย์ถึงใช้งานจริง)

1. **Sprint 0 — Foundations**
   - นิยามข้อมูลธุรกิจ, โครงสร้าง Postgres, seed data ตัวอย่าง
   - สร้างโครง Express API + JWT Auth + User roles
2. **Sprint 1 — RAG Core**
   - เชื่อม Embedding service (Ollama), สร้าง BullMQ queue + Worker สำหรับฝังข้อมูลธุรกิจ
   - พัฒนา Chat controller: รับ prompt → จัดข้อความ → เรียก LLM → ส่งกลับแบบ Markdown
3. **Sprint 2 — Admin & Sponsor Portals**
   - CRUD ธุรกิจ, แพ็กเกจ, สลิปชำระเงิน, สถานะ embedding + dashboard
   - เพิ่ม Sponsor flow (เจ้าของธุรกิจอัปเดตข้อมูล, รูปภาพ, ตรวจสถานะ)
4. **Sprint 3 — Messaging Gateway & LINE Integration**
   - สร้าง service ใหม่ + routing inbound/outbound messages + Session mapping
   - Validate signatures (LINE), event logger (`logs/events.log`), real-time tail
5. **Sprint 4 — Deployment & Cloudflare**
   - แตก compose เป็น dev/lan/prod, เชื่อม Cloudflare Tunnel, health checks
   - สร้างเอกสาร How-to-Deploy / HowtoUse / Function-Line-Setup
6. **Sprint 5+ — Enhancements**
   - Embedding dashboard, auto re-embed, job cleanup, UI/UX fine-tune, CLI scripts

---

## 6. ระบบทำงานอย่างไร (Journey: Login → Ask → Answer)

### 6.1 มุมมองผู้ใช้ทั่วไป

1. เปิดเว็บ `https://frontend.sri-ketguide.com`, ล็อกอิน (หรือเพิ่มเพื่อน LINE แล้วทัก)
2. ระบบจำผู้ใช้, โหลดเมนูลัด (suggestions), แสดง prompt ที่นิยม
3. ผู้ใช้พิมพ์คำถาม เช่น “เมนู signature ของ Hap Cafe คืออะไร?”
4. ภายในไม่ถึงวินาที ได้คำตอบภาษาไทยพร้อมรายละเอียด (ราคา, เวลาเปิด, รูปภาพ)
5. ถ้าถามต่อ (follow-up) ระบบจำบริบทการสนทนาก่อนหน้า

### 6.2 มุมมองเทคนิค (Sequence)

1. **Auth**: JWT verified → role-based routes → load business/profile context
2. **Prompt Processing**: Chat controller เรียก `prepareChatRequest` → bundle history 10 รายการล่าสุด + client hints
3. **Retrieval**: Worker ดึง vector embedding ของธุรกิจ (pgvector) + keyword fallback
4. **LLM Call**: ส่ง system prompt + context + user prompt ไปยัง LLM (ผ่าน HTTP streaming)
5. **Response Cleaning**: จัด Markdown, แทรก URL ภาพแบบ `![alt](url)`
6. **Streaming**: ส่งกลับผ่าน SSE (web) หรือ push ผ่าน LINE SDK
7. **Logging**: เก็บคำถาม/คำตอบใน Postgres, append event ไป `events.log` (กรณี LINE)

> **เทคนิค**: ใช้ multi-hop retrieval (vector + business tier), queue debounce 5 วินาทีเพื่อหลีกเลี่ยง re-embed ซ้ำ, job `Clean failed` จาก admin เพื่อควบคุมคิว

---

## 7. ตัวอย่างสถานการณ์ (Real-life Scenario)

> **ผู้ใช้**: นักท่องเที่ยวเพิ่มเพื่อน LINE ของ “Boonsiri Boutique Hotel”

1. เพิ่มเพื่อน → ระบบ (LINE webhook) บันทึก userId, สร้าง session ID
2. ผู้ใช้ถาม “มีห้อง Deluxe ว่างไหมพรุ่งนี้?”
3. Gateway ส่งข้อความไป API พร้อม metadata (replyToken, platformMeta)
4. API ดึงข้อมูลห้องพักจากฐาน + คิวฝังข้อมูล (ล่าสุดมีรูปภาพ 6 รูป)
5. LLM สร้างคำตอบ: รายละเอียดห้อง, ราคา, เงื่อนไข (non-smoking), ลิงก์จอง → ส่งกลับ LINE SDK
6. ผู้ใช้ได้รับคำตอบภายใน 2 วิ พร้อมปุ่ม “โทรหาเรา” (rich menu)
7. Admin เห็นประวัติคำถามใน Dashboard → สามารถ re-embed ข้อมูลห้องพักที่เพิ่งอัปเดต

**ผลลัพธ์**: ลูกค้าได้ข้อมูลแม่นยำ, ลดภาระ call center, เจ้าของรู้ว่าสิ่งใดถูกถามบ่อยที่สุด → ปรับ content ได้ทันที

---

## 8. ประเด็นเชิงเทคนิคที่น่าพูดถึง

- **RAG Accuracy**: ใช้ pgvector + cosine similarity, enrich context โดย dynamic system prompts (Persona + Business tiers)
- **Queue Management**: BullMQ + Redis, job debounce ป้องกัน spam, Admin สามารถ clean failed jobs
- **Gateway Extensibility**: ออกแบบให้รองรับ LINE, Facebook Messenger, Telegram ผ่าน unified normalized payload
- **Security**: API key สำหรับ internal routes, JWT per user, Cloudflare ตรวจสอบ traffic, `/health` แยก API/Worker
- **Observability**: Embedding dashboard, Events log, `/api/embeddings/stats`, `/api/embeddings/clean`

---

## 9. ปัญหาและสิ่งที่เรียนรู้

| ประเด็น | คำอธิบาย | วิธีแก้ / บทเรียน |
| --- | --- | --- |
| **Embedding ล้มเหลว (DB auth)** | Worker ใช้ ENV ผิด ทำให้ queue ค้างสถานะ failed | Fix DB credential, ลด keep failed = 50, เพิ่มปุ่ม Clear failed ใน Admin |
| **Cloudflare 502 / Mapping สลับ service** | ช่วงเปลี่ยน routing dev → prod สับสน container alias | เพิ่มคู่มือ + ชื่อ service ตรงกับ compose, ทำ bridge container เฉพาะจุด |
| **UI ความสอดคล้อง** | Users สับสนเมื่อ logout แล้วโดนพาไปหน้า PWA | ปรับ router ให้ redirect hash `#/login` โดยตรง, อัปเดตคู่มือ deploy |
| **LINE Signature / Dev Testing** | ทดสอบ webhook ลำบากเพราะ signature | เพิ่ม option `LINE_SKIP_SIGNATURE` สำหรับ dev, log raw payload |

---

## 10. Roadmap & Expansion

- **Automation**: Auto re-embed เมื่อธุรกิจอัปเดตรายละเอียด, auto sync สลิปการชำระเงินกับ CRM
- **Analytics**: Dashboard คำถามยอดนิยม, conversion funnel จาก chat → booking
- **Multi-language**: รองรับภาษาอังกฤษ/จีน/ญี่ปุ่น ด้วย prompt templates เฉพาะตลาด
- **Smart Campaign**: ส่งข้อเสนอพิเศษผ่าน LINE / Messenger จาก insight ที่ได้
- **Partner API**: เปิด API ให้ third-party ดึงข้อมูล business curated responses

---

## 11. มูลค่าทางธุรกิจ & การต่อยอด

- **Subscription Model**: แพ็กเกจ Premium/Standard/Free (จัดลำดับกา รตอบคำถาม + สิทธิในการฝังข้อมูล)
- **Managed Service**: เสนอให้หน่วยงานท้องถิ่น/สมาคมผู้ประกอบการใช้ platform พร้อมทีมบริการ
- **Marketplace**: เชื่อมผู้ให้บริการคอนเทนต์หรือ production houses สำหรับสร้างสื่อ (รูป, วิดีโอ) ที่ฝังเข้า RAG ได้ทันที
- **Data Monetization**: วิเคราะห์แนวโน้มคำถาม → เสนอข้อมูลให้เมือง/หน่วยงานท่องเที่ยวเพื่อวางแผนกิจกรรม

ตัวอย่าง: ร้านอาหารที่ใช้ระบบสามารถ upsell เมนู signature ผ่านข้อความอัตโนมัติ เมื่อมีลูกค้าถามถึง “เมนูแนะนำ” และยังเก็บข้อมูลมุลค่าต่อคำถาม เพื่อคำนวณ ROI ได้ real-time

---

## 12. สรุปสุดท้าย (One-liner + Deep impact)

> **“นี่คือผู้ช่วยอัจฉริยะสำหรับธุรกิจท้องถิ่น ที่ยืนอยู่ทุกช่องทางตอบลูกค้าได้ทันที โดยใช้ข้อมูลจริงจากระบบหลังบ้านเดียวกัน”**

- สำหรับผู้ฟังทั่วไป: “นึกถึงการมีพนักงานบริการที่ไม่หลับไม่นอน ช่วยปิดการขายให้ตลอด 24 ชั่วโมง”
- สำหรับทีมเทคนิค: “เป็นแพลตฟอร์ม RAG-driven, queue-aware, multi-channel ที่ deploy ได้จริงผ่าน Docker + Cloudflare”

> จุดขาย: ทำให้ “ข้อมูล” ของธุรกิจกลายเป็น “บทสนทนา” ที่ขายสินค้า/บริการให้เจ้าของได้ทันที

---

## 13. Checklist ก่อนนำเสนอ

- [ ] เตรียมเดโม (เว็บ + LINE) ให้พร้อม มีข้อมูลธุรกิจตัวอย่าง
- [ ] เปิด Cloudflare tunnels ที่จำเป็น (`cf-tunnel-prod`, `cf-tunnel-api`, `cf-tunnel-frontend`, `cf-tunnel-pwa`)
- [ ] ล็อกอิน Admin (`admin@example.com / 1234`) → ตรวจ embedding dashboard ว่าคิวว่าง
- [ ] เปิดไฟล์นี้ระหว่างซ้อมเพื่อนึก key message ครบทุกหัวข้อ
- [ ] กำหนดเวลาพูดและ Q&A (เผื่อ 5 นาทีตอนท้าย)

---

**Version**: 1.0 (ปรับปรุงล่าสุดโดย AI Assistant — 2025-11-01)



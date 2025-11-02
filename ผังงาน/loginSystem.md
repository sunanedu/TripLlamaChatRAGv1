# 🔐 Login System Flow Diagram

แผนภาพแสดงการทำงานของระบบ Login ทั้งหมดตั้งแต่ผู้ใช้เปิดหน้า login จนถึงการ redirect ไปยังหน้าแอปพลิเคชัน

## 📊 Deployment Diagram (Mermaid)

```mermaid
graph TB
    subgraph "Client Layer"
        U[👤 User Browser<br/>frontend.sri-ketguide.com]
        LF[📝 Login Form<br/>login.html<br/>Email + Password]
        FA[⚙️ Frontend Auth Handler<br/>auth.js<br/>localStorage Management]
        AC[🌐 API Call<br/>api.js<br/>POST /api/auth/login]
    end

    subgraph "Network Layer"
        CT[☁️ Cloudflare Tunnel<br/>HTTPS → HTTP<br/>api.sri-ketguide.com]
    end

    subgraph "Backend Layer"
        ER[🚀 Express Route<br/>auth.routes.js<br/>POST /api/auth/login]
        AUTH[🎯 Auth Controller<br/>auth.controller.js<br/>Validate + Authenticate]
        TS[🎫 Token Service<br/>token.service.js<br/>JWT Generation]
        BC[🔒 Bcrypt<br/>Password Hashing<br/>bcrypt.compare]
    end

    subgraph "Database Layer"
        DB[(🗄️ PostgreSQL<br/>llm-chat-database:5432<br/>users + tokens tables)]
    end

    subgraph "Token Storage"
        LS[💾 localStorage<br/>Access Token<br/>Refresh Token<br/>User Role]
    end

    subgraph "Success Flow"
        SR[✅ Success Response<br/>User Info + Tokens]
        RD[➡️ Redirect<br/>#/chat or #/admin]
    end

    subgraph "Error Handling"
        ERR1[❌ Invalid Credentials<br/>401 Unauthorized]
        ERR2[❌ Account Suspended<br/>403 Forbidden]
    end

    subgraph "Protected Routes"
        AM[🛡️ Auth Middleware<br/>auth.middleware.js<br/>JWT Verification]
        PR[🔒 Protected Route<br/>POST /api/chat<br/>Requires Auth]
    end

    %% Main Login Flow
    U -->|1. Open login page| LF
    LF -->|2. Submit form| FA
    FA -->|3. Prepare API call| AC
    AC -->|4. POST /api/auth/login<br/>Headers + Body| CT
    CT -->|5. Proxy request| ER
    ER -->|6. Route to controller| AUTH

    %% Authentication Flow
    AUTH -->|7. Query user by email| DB
    DB -->|8. Return user data| AUTH
    AUTH -->|9. Compare password| BC
    BC -->|10. Password match| AUTH
    AUTH -->|11. Generate tokens| TS
    TS -->|12. Save refresh token| DB
    AUTH -->|13. Return success| SR

    %% Success Response
    SR -->|14. Receive tokens| FA
    FA -->|15. Save to localStorage| LS
    LS -->|16. Redirect to app| RD

    %% Error Handling
    AUTH -.->|Invalid email/password| ERR1
    AUTH -.->|Account suspended| ERR2
    ERR1 -.->|Show error message| LF
    ERR2 -.->|Show error message| LF

    %% Protected Routes Flow
    AC -->|For protected routes| AM
    AM -->|Verify JWT token| PR
    AM -.->|Invalid token 401/403| ERR1

    %% Styling
    classDef client fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef network fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef backend fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef database fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef storage fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef success fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef error fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef protected fill:#e0f2f1,stroke:#004d40,stroke-width:2px

    class U,LF,FA,AC client
    class CT network
    class ER,AUTH,TS,BC backend
    class DB database
    class LS storage
    class SR,RD success
    class ERR1,ERR2 error
    class AM,PR protected
```

## 📋 **ลำดับการทำงาน (Flow Sequence)**

### **Main Login Flow (1-16):**

1. **User opens login page** → User เปิด `frontend.sri-ketguide.com/#/login`
2. **Login form displayed** → แสดงฟอร์ม login (email + password)
3. **User submits form** → User กดปุ่ม "เข้าสู่ระบบ"
4. **Frontend auth handler** → `auth.js` เตรียมข้อมูล
5. **API call prepared** → `api.js` สร้าง `POST /api/auth/login` request
6. **Request sent** → ส่ง request พร้อม `{email, password}` body
7. **Cloudflare Tunnel** → Proxy HTTPS → HTTP ภายใน
8. **Express route** → `auth.routes.js` รับ request → ส่งไป `authController.login`
9. **Auth controller** → ตรวจสอบข้อมูล:
   - Validate input (email, password)
   - Query database: `User.findOne({ where: { email } })`
10. **Database query** → ค้นหา user จาก `users` table
11. **Password verification** → ใช้ `bcrypt.compare(password, password_hash)`
12. **Check suspended** → ตรวจสอบ `user.is_suspended === false`
13. **Generate tokens** → `tokenService.generateTokens({ userId, role })`
   - Access Token: 1 hour expiry
   - Refresh Token: 7 days expiry
14. **Save refresh token** → บันทึก `refresh_token` ใน `tokens` table
15. **Success response** → ส่งกลับ `{ message, user, tokens }`
16. **Save to localStorage** → Frontend เก็บ tokens ใน localStorage:
   - `llm_access_token`
   - `llm_refresh_token`
   - `llm_user_role`
17. **Redirect** → เปลี่ยน route ไปยัง `#/chat` หรือ `#/admin` (ตาม role)

### **Error Handling:**

**Invalid Credentials (401):**
- User ไม่พบ หรือ password ไม่ตรง
- Response: `{ message: 'Invalid email or password' }`
- Frontend แสดง error message

**Account Suspended (403):**
- `user.is_suspended === true`
- Response: `{ message: 'Your account has been suspended' }`
- Frontend แสดง error message

### **Protected Routes Flow:**

เมื่อผู้ใช้เรียก API endpoint ที่ต้องการ authentication:

1. **Request with token** → Frontend ส่ง `Authorization: Bearer <token>` header
2. **Auth middleware** → `auth.middleware.js` ตรวจสอบ:
   - Extract token from header
   - Verify JWT: `jwt.verify(token, JWT_SECRET)`
   - Attach user to `req.user = { userId, role }`
3. **Protected route** → Controller ใช้ `req.user` เพื่อ authorization
4. **Invalid token** → Return 401/403 และ redirect ไปหน้า login

## 🔧 **Components Details**

### **Frontend Components:**
- **login.html**: Login form template
- **auth.js**: Token management functions
- **api.js**: HTTP client สำหรับ API calls
- **localStorage**: Client-side token storage

### **Backend Components:**
- **auth.routes.js**: Express routes (`/register`, `/login`, `/refresh`)
- **auth.controller.js**: Business logic สำหรับ authentication
- **auth.middleware.js**: JWT verification middleware
- **token.service.js**: JWT token generation และ management

### **Database Tables:**
- **users**: `email`, `password_hash`, `role`, `is_suspended`
- **tokens**: `user_id`, `refresh_token`, `created_at`

### **Security Features:**
- ✅ Password hashing ด้วย bcrypt
- ✅ JWT tokens สำหรับ stateless authentication
- ✅ Refresh tokens สำหรับ token renewal
- ✅ Token verification สำหรับ protected routes
- ✅ Account suspension check

---

**หมายเหตุ**: ไฟล์ `loginSystem.drawio` สามารถเปิดได้ด้วย draw.io application สำหรับแก้ไขแผนภาพแบบ visual editor


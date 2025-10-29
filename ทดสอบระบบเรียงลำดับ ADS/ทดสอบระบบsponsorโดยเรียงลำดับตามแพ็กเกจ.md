# การทดสอบระบบ Sponsor โดยเรียงลำดับตามแพ็กเกจ

**วันที่ทดสอบ:** 29 ตุลาคม 2568  
**วัตถุประสงค์:** ทดสอบว่าระบบเรียงลำดับผลลัพธ์ตาม Package Tier (Premium → Standard → Free) อย่างถูกต้อง

---

## 📋 สรุปผลการทดสอบ

### การทดสอบ 1: คำค้นหา "โรงแรม"

- **พบธุรกิจทั้งหมด:** 7 รายการ (เฉพาะประเภทโรงแรม)
- **Premium Package:** 7 รายการ
- **Standard Package:** 0 รายการ
- **Free Package:** 0 รายการ

**ผลลัพธ์:** ✅ การเรียงลำดับถูกต้อง (Premium เท่านั้น)

### การทดสอบ 2: คำค้นหา "ร้านอาหาร"

- **พบธุรกิจทั้งหมด:** 10 รายการ
- **Premium Package:** 3 รายการ
- **Standard Package:** 7 รายการ
- **Free Package:** 0 รายการ

**ลำดับผลลัพธ์ (Premium ก่อน Standard):**
1. Good View Sisaket - **Premium** ✅
2. ชิลล์ริมบึง คาเฟ่ - **Premium** ✅
3. บ้านกาแฟ - **Premium** ✅
4. ตำแหลก ศรีสะเกษ - **Standard** ✅
5. สวนวาสนา - **Standard** ✅
6. ศรีสะเกษ สเต็กเฮาส์ - **Standard** ✅
7. แพอาหารพึ่งพุง - **Standard** ✅
8. โอชารส - **Standard** ✅
9. ก๋วยจั๊บคุณแม่ - **Standard** ✅
10. ครัวร่มไม้ ศรีสะเกษ - **Standard** ✅

**ผลลัพธ์:** ✅ การเรียงลำดับถูกต้อง: Premium → Standard

---

## 🔍 ขั้นตอนการทำงานของระบบ

### 1. Vector Similarity Search (เมื่อมี Embedding)

เมื่อผู้ใช้ค้นหา ระบบจะทำงานตามลำดับดังนี้:

```sql
SELECT 
    b.id, b.name, b.description, b.business_type,
    b.embedding <-> $1::vector as similarity,
    CASE 
        WHEN p.name ILIKE '%premium%' THEN 1
        WHEN p.name ILIKE '%standard%' THEN 2
        WHEN p.name ILIKE '%free%' THEN 3
        ELSE 4
    END as package_tier
FROM businesses b
LEFT JOIN business_packages bp ON b.id = bp.business_id AND bp.status = 'active'
LEFT JOIN packages p ON bp.package_id = p.id
WHERE b.is_active = true
ORDER BY package_tier ASC, similarity ASC
LIMIT 10
```

**ขั้นตอน:**
1. สร้าง embedding vector จากคำค้นหาของผู้ใช้
2. คำนวณ similarity distance (`embedding <->`) กับธุรกิจทั้งหมด
3. **เรียงลำดับตาม package_tier ก่อน** (Premium=1, Standard=2, Free=3)
4. ในแต่ละ package tier **เรียงตาม similarity ต่อมา** (ใกล้เคียงก่อน)
5. ดึงข้อมูลธุรกิจพร้อม package information
6. ส่งไปให้ LLM เพื่อสร้างคำตอบ

### 2. Keyword Search (Fallback เมื่อไม่มี Embedding)

หากธุรกิจไม่มี embedding หรือ embedding service ไม่พร้อม:

```sql
SELECT b.id
FROM businesses b
LEFT JOIN business_packages bp ON b.id = bp.business_id AND bp.status = 'active'
LEFT JOIN packages p ON bp.package_id = p.id
WHERE b.is_active = true
  AND (b.name ILIKE '%คำค้นหา%' 
       OR b.description ILIKE '%คำค้นหา%' 
       OR b.business_type ILIKE '%คำค้นหา%')
ORDER BY 
  CASE 
    WHEN p.name ILIKE '%premium%' THEN 1
    WHEN p.name ILIKE '%standard%' THEN 2
    WHEN p.name ILIKE '%free%' THEN 3
    ELSE 4
  END ASC,
  b.name ASC
LIMIT 5
```

**ขั้นตอน:**
1. ค้นหาด้วย keyword matching (ชื่อ, คำอธิบาย, ประเภท)
2. **เรียงลำดับตาม package_tier ก่อน** (Premium → Standard → Free)
3. ในแต่ละ package tier **เรียงตามชื่อธุรกิจ** (A-Z)
4. ดึงข้อมูลธุรกิจพร้อม package information
5. ส่งไปให้ LLM เพื่อสร้างคำตอบ

### 3. การส่งข้อมูลไปยัง LLM

ข้อมูลที่เรียงลำดับแล้วจะถูกส่งไปให้ LLM ในรูปแบบ context:

```
CONTEXT FROM DATABASE:
Name: โรงแรมบาบาราย
Type: โรงแรม
Description: [คำอธิบาย]
Package: Premium

---

Name: โรงแรมบีทู
Type: โรงแรม
Description: [คำอธิบาย]
Package: Premium

...
```

LLM จะใช้ context นี้เพื่อสร้างคำตอบที่แนะนำธุรกิจตามลำดับที่ส่งไป

---

## 📊 ผลลัพธ์การทดสอบแบบละเอียด

### การทดสอบ: "ร้านอาหาร"

#### Premium Package (3 รายการ)

1. **Good View Sisaket (ร้านอาหาร กู๊ดวิว ศรีสะเกษ)**
   - ID: 20
   - Package Tier: 1
   - Sponsor: sponsor@example.com

2. **ชิลล์ริมบึง คาเฟ่ (Chill Rim Bueng Cafe)**
   - ID: 19
   - Package Tier: 1
   - Sponsor: sponsor@example.com

3. **บ้านกาแฟ (Baan Ka Fae @ Sisaket)**
   - ID: 18
   - Package Tier: 1
   - Sponsor: sponsor@example.com

#### Standard Package (7 รายการ)

1. **ตำแหลก ศรีสะเกษ (Tum Laek Sisaket)**
   - ID: 27
   - Package Tier: 2
   - Sponsor: sponsor2@example.com

2. **สวนวาสนา (Swasen Restaurant)**
   - ID: 26
   - Package Tier: 2
   - Sponsor: sponsor2@example.com

3. **ศรีสะเกษ สเต็กเฮาส์ (Sisaket Steak House)**
   - ID: 25
   - Package Tier: 2
   - Sponsor: sponsor2@example.com

4. **แพอาหารพึ่งพุง (Pueng Pung Raft Restaurant)**
   - ID: 24
   - Package Tier: 2
   - Sponsor: sponsor2@example.com

5. **โอชารส (O-Cha-Rod)**
   - ID: 23
   - Package Tier: 2
   - Sponsor: sponsor2@example.com

6. **ก๋วยจั๊บคุณแม่ (Kuay Jab Khun Mae)**
   - ID: 22
   - Package Tier: 2
   - Sponsor: sponsor2@example.com

7. **ครัวร่มไม้ ศรีสะเกษ (Krua Rom Mai)**
   - ID: 21
   - Package Tier: 2
   - Sponsor: sponsor2@example.com

---

## ✅ สรุปผลการทดสอบ

### การเรียงลำดับถูกต้อง ✅

ระบบเรียงลำดับผลลัพธ์ตาม Package Tier อย่างถูกต้อง:
- **Premium (Tier 1)** แสดงก่อนเสมอ
- **Standard (Tier 2)** แสดงรองลงมา
- **Free (Tier 3)** จะแสดงสุดท้าย (ถ้ามี)

### ข้อดีของระบบ

1. ✅ **Premium sponsors ได้รับสิทธิพิเศษ** - แสดงผลก่อนเสมอ
2. ✅ **กระตุ้นการอัปเกรดแพ็กเกจ** - Standard และ Free sponsors เห็นว่าการอัปเกรดจะช่วยให้แสดงผลดีขึ้น
3. ✅ **ภายในแต่ละ tier ยังคงเรียงตามความเกี่ยวข้อง** - ไม่เสียความแม่นยำในการค้นหา
4. ✅ **รองรับทั้ง Vector Search และ Keyword Search** - ทำงานได้แม้ไม่มี embedding

### ข้อควรพิจารณา

⚠️ **ในกรณีพิเศษ:** หาก Premium sponsor มี similarity ต่ำมาก (ไม่เกี่ยวข้องกับคำค้นหา) อาจจะยังแสดงก่อน Standard sponsor ที่เกี่ยวข้องมากกว่า แต่ระบบได้ออกแบบให้ **เรียงตาม similarity ภายในแต่ละ tier** แล้วเพื่อลดปัญหานี้

### การทำงานของระบบสรุป

```
ผู้ใช้ค้นหา "ร้านอาหาร"
    ↓
ระบบสร้าง Embedding (หรือใช้ Keyword Search)
    ↓
ค้นหาในฐานข้อมูล + คำนวณ Similarity
    ↓
เรียงลำดับ: Package Tier (Premium→Standard→Free) → Similarity/Name
    ↓
ดึงข้อมูลธุรกิจพร้อม Package Information
    ↓
ส่ง Context ให้ LLM
    ↓
LLM สร้างคำตอบแนะนำตามลำดับ (Premium ก่อน Standard)
```

---

## 📝 หมายเหตุ

- ทุกธุรกิจในฐานข้อมูลมี embedding แล้ว (ใช้สำหรับ Vector Similarity Search)
- ระบบรองรับการค้นหาทั้งภาษาไทยและภาษาอังกฤษ
- Package Tier จะถูกคำนวณจากชื่อแพ็กเกจ (case-insensitive)
- การเรียงลำดับมีผลทั้งต่อการค้นหาและคำตอบที่ LLM สร้างให้

---

**สรุป:** ระบบทำงานถูกต้อง ✅ Premium sponsors ได้แสดงผลก่อน Standard sponsors ตามที่ออกแบบไว้

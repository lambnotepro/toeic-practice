# TOEIC Practice — เว็บแอปฝึกทำข้อสอบ TOEIC

- **Live:** https://lambnotepro.github.io/toeic-practice/
- **Repo:** https://github.com/lambnotepro/toeic-practice

> เอกสารนี้คือสรุปสถานะระบบ + วิธีเพิ่มข้อสอบ ให้ทุกคน/ทุกแชตเข้าใจตรงกัน

---

## สถานะระบบ (อัปเดต 2026-08-28)
- แอปไฟล์เดียว `index.html` ธีม neon dark โฮสต์บน GitHub Pages
- **ต่อ Firebase Firestore แล้ว ✅** — อ่าน/เขียนข้อสอบจากคลาวด์จริง
- ตอนนี้มีข้อสอบตัวอย่าง **4 ข้อในคลาวด์** (Part 5×2, Part 6, Part 7)
- ถ้าต่อ Firestore ไม่ได้ (ออฟไลน์/SDK โหลดไม่ขึ้น) แอปจะ fallback ใช้ตัวอย่างในไฟล์ + แสดงสถานะให้รู้

## Firebase
| รายการ | ค่า |
|---|---|
| Project ID | `toeic-practice-41159` |
| Collection | `toeic_questions` |
| Region | asia-southeast1 (Singapore) — เปลี่ยนไม่ได้ |
| Rules | **Test mode** (เปิดอ่าน/เขียนถึง 27 ก.ย. 2026) |
| Plan | Spark (ฟรี) |

Firebase config อยู่ใน `index.html` (Section 2) — เป็น web config ที่เปิดเผยได้ ปลอดภัยที่จะฝังใน client

## โครงสร้างข้อสอบ 1 ข้อ (Firestore document)
```js
{
  part: 5,                 // 1-7
  passage: "",             // บทความ (สำหรับ Part 6/7) เว้นว่างได้
  question: "...",         // โจทย์คำถาม
  choices: ["A","B","C","D"],
  answer: 0,               // index ของข้อที่ถูก เริ่มนับที่ 0
  explainCorrect: "...",   // เหตุผลว่าทำไมข้อนี้ถูก
  explainWrong: { "1": "...", "2": "..." },  // เหตุผลข้อผิด (key = index ของตัวเลือก)
  difficulty: "medium",    // easy | medium | hard
  tags: ["preposition"]    // แท็ก ใช้วิเคราะห์จุดอ่อน
}
```
> `id` ใช้ document id ของ Firestore อัตโนมัติ **ไม่ต้องใส่ field id เอง**

## วิธีเพิ่มข้อสอบ (3 ทาง — ทุกทางบันทึกลง Firestore จริง)
1. **แท็บ "จัดการข้อสอบ" → เพิ่มข้อสอบ** — กรอกฟอร์มทีละข้อ กดบันทึก → เขียนขึ้น Firestore ทันที
2. **แท็บ "จัดการข้อสอบ" → นำเข้าแบบกลุ่ม (Import)** — วาง JSON array หลายข้อพร้อมกัน กดนำเข้า → เขียนแบบ batch
3. **Firebase Console โดยตรง** — console.firebase.google.com → project `toeic-practice` → Firestore → collection `toeic_questions` → Add document (ตาม schema ด้านบน)

## สถาปัตยกรรมโค้ด (index.html)
- **Section 1** — ข้อสอบตัวอย่าง (ใช้ตอน Firestore ว่าง/ต่อไม่ติด)
- **Section 2** — จุดเชื่อม data source ทั้งหมดอยู่ที่เดียว:
  - `firebaseConfig`, init Firebase
  - `loadQuestions()` — ฟังก์ชันเดียวที่ดึงข้อมูล: อ่าน Firestore ก่อน ถ้าว่าง/พัง fallback ตัวอย่างในเครื่อง
  - `dbAddQuestion()` / `dbAddMany()` — เขียนขึ้น Firestore (ทีละข้อ / batch)
  - `reloadPool()` — โหลดใหม่จากคลาวด์แล้ว refresh UI
- ไม่ใช้ localStorage/sessionStorage — state อยู่ใน JS variable ล้วน

## ⚠️ สิ่งที่ต้องทำก่อนใช้งานจริง (สำคัญ)
1. **Test mode หมดอายุ 27 ก.ย. 2026** — ตอนนี้ใครก็อ่าน/เขียน Firestore ได้ หลังวันนั้นจะถูกล็อกทั้งหมด
   ต้องเข้าไปตั้ง Security Rules เอง เช่น (อ่านได้ทุกคน แต่ห้ามเขียนจากฝั่ง client):
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /toeic_questions/{doc} {
         allow read: if true;
         allow write: if false;   // เพิ่มข้อสอบผ่าน Console เท่านั้น (หรือทำ Auth)
       }
     }
   }
   ```
2. ถ้าตั้ง `write: false` ปุ่มเพิ่ม/Import ในแอปจะเขียนไม่ได้ → ต้องเพิ่มข้อสอบผ่าน Firebase Console
   ถ้าอยากเพิ่มผ่านแอปได้อย่างปลอดภัย → ทำระบบ **Firebase Auth (admin login)** แล้วตั้ง rules ให้ write เฉพาะ admin

## วิธีอัปเดตเว็บ
แก้ `index.html` → push ขึ้น branch `main` → GitHub Pages redeploy อัตโนมัติภายใน ~1 นาที
(เปิดเว็บด้วย `?v=xxx` ต่อท้าย URL เพื่อ bypass cache ตอนเช็คเวอร์ชันใหม่)

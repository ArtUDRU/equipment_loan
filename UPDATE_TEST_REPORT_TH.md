# รายงานการตรวจชุดอัปเดต Admin Members

## สิ่งที่ตรวจผ่าน

- `flutter analyze`: No issues found
- Flutter unit/widget tests: **24 รายการผ่าน** (รวมของเดิมและฟอร์ม/สถานะใหม่)
- Edge Function handler tests: **16 รายการผ่าน** ด้วย backend จำลอง
- Flutter Web release build: ผ่านโดยใช้ URL/key จำลองสำหรับตรวจการ compile ไม่ใช่การเชื่อมบัญชีจริง
- Deno type check: ผ่านทั้ง index.ts สำหรับ CLI และ single-file สำหรับ Dashboard
- PostgreSQL 16 regression: **36 กรณีผ่าน** ใช้ Auth stub เฉพาะฐานข้อมูล local รวมสิทธิ์ audit และการรัน migration 004 ซ้ำ

ทดสอบสำคัญ: สมาชิก/แอดมินถูกระงับเรียกคำสั่งสิทธิ์สูงไม่ได้, token ไม่ถูกต้องถูกปฏิเสธ, client ส่ง role=admin ไม่ได้สิทธิ์เพิ่ม, ผู้สมัครรอการยืนยันแสดงจากสถานะ Auth, ฟอร์มซ่อนรหัสผ่านและตรวจข้อมูล, รับรองเฉพาะ UID ที่เลือก, มีเหตุผล/actor ใน audit, ไม่มีรหัสผ่านใน response/audit, การสร้างสมาชิกไม่เปลี่ยน profile แอดมินในชุดทดสอบหน้าจอ

## ข้อจำกัด

- ยังไม่ได้ Deploy Edge Function หรือ Run migration บน Supabase ของผู้ใช้
- ยังไม่ได้ทดสอบ Admin API ด้วยบัญชีจริง, session จริง, CORS/Realtime ผ่าน Supabase hosted หรือระบบอีเมลจริง
- ยังไม่ได้ Build Android APK/AAB หรือลองโค้ดอัปเดตบน Emulator ของผู้ใช้
- unit tests ไม่แทน end-to-end test: ต้องทำตามรายการทดสอบใน UPDATE_ADMIN_MEMBERS_TH.md หลังติดตั้ง
- Auth API และ audit table อยู่คนละ transaction หาก audit ล้มเหลวหลังเปลี่ยนบัญชี ระบบแจ้ง success-with-warning เพื่อไม่ให้สร้างซ้ำ
- ไม่บังคับเปลี่ยนรหัสผ่านครั้งแรก ยังต้องจัดการส่งรหัสเริ่มต้นอย่างปลอดภัย

ชุดอัปเดตไม่เก็บ key แอดมินใน Flutter ไม่มี config.json และไม่ทับไฟล์ Android/Gradle

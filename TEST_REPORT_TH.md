# ผลการตรวจโปรเจกต์

## สภาพแวดล้อมทดสอบ

- Linux
- Flutter 3.47.6 (stable)
- Dart 3.13.5
- PostgreSQL 16.14 บนฐานข้อมูลชั่วคราวที่เป็น UTF-8

## ผลที่ผ่าน

| การตรวจ | ผล |
|---|---|
| Dart formatter | ผ่าน |
| `flutter analyze` | No issues found |
| Flutter unit/widget tests | 16 tests passed |
| SQL syntax + PL/pgSQL parser | ผ่าน |
| PostgreSQL regression checks | 30 checks passed |
| Flutter Web release build (JavaScript) | ผ่าน |
| Web Wasm dry run | ผ่าน (ไม่ใช่การทดสอบรัน Wasm จริง) |
| Android runner และ INTERNET permission | มีครบในโค้ด |

ชุดทดสอบ Flutter ครอบคลุมการแปลสถานะ การเปลี่ยนสถานะตามสิทธิ์ จำนวน วันที่ เกินกำหนด หน้าตั้งค่า การค้นหาอุปกรณ์ หน้าจอมือถือ/เดสก์ท็อป และการเปิดฟอร์มแอดมินบนหน้าจอเล็ก

ชุดทดสอบฐานข้อมูลใช้ PostgreSQL จริง และจำลอง Supabase `auth.uid()` เพื่อตรวจสิทธิ์ ครอบคลุม RLS, role escalation, การจอง/คืนจำนวน, สถานะผิดลำดับ, การแก้จำนวน และการอนุมัติจากสอง connection พร้อมกันในกรณีเหลืออุปกรณ์หนึ่งชิ้น (สำเร็จเพียงคำขอเดียว)

Web release build ใช้ URL/key จำลองที่ไม่ใช่บัญชีจริง เพื่อตรวจการ compile ไม่ใช่เว็บพร้อมใช้งานหรือการเชื่อมฐานข้อมูลจริง Build มีข้อความเตือน font CupertinoIcons ที่ไม่ได้ใช้โดยหน้าจอแอป แต่สร้างเว็บสำเร็จ; หากเพิ่ม Cupertino UI ในอนาคตให้เพิ่ม package cupertino_icons

## สิ่งที่ยังไม่ได้ทดสอบ/ทำ

- ยังไม่ได้เชื่อมโปรเจกต์ Supabase ของผู้ใช้
- ยังไม่ได้ทดสอบ Supabase Auth อีเมลยืนยัน / SMTP / PostgREST / Realtime กับบริการจริง
- ยังไม่ได้ Build APK/AAB หรือลองบน Android emulator/มือถือจริง (เครื่องทดสอบไม่มี Android SDK)
- ยังไม่ได้ production signing, deploy hosting, custom domain หรือเผยแพร่ Google Play
- ยังไม่ได้ทดสอบภาระงานจำนวนมากหรือข้อมูลเกินขีดจำกัด 1,000 รายการ

ต้องตั้งค่า Supabase และตรวจตามรายการ E2E ใน README_TH.md ก่อนเปิดใช้งานจริง

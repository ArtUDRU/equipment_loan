# แอปยืม–คืนอุปกรณ์ (Flutter + Supabase)

โปรเจกต์เริ่มต้นสำหรับ Android และ Web เขียน Dart เปิดใน VS Code ได้ ใช้ฐานข้อมูล PostgreSQL บน Supabase ไม่ใช่แอปที่เชื่อมกับ Notion

## ฟังก์ชันที่มี

- สมัครสมาชิก / เข้าสู่ระบบ / ออกจากระบบ และรองรับการยืนยันอีเมล
- แยกสิทธิ์สมาชิกและแอดมินทั้งหน้าจอและฐานข้อมูล
- แอดมินเพิ่ม/แก้ไข/ปิดใช้งานอุปกรณ์ และระบุปกติ/ชำรุด/ซ่อมบำรุง
- สมาชิกค้นหาอุปกรณ์ ดูจำนวนพร้อมยืม และดูชื่อผู้ยืมปัจจุบัน
- ส่งคำขอยืม → แอดมินอนุมัติ/ปฏิเสธ → ส่งมอบ → แจ้งคืน → แอดมินยืนยันรับคืน
- แอดมินแก้ไขจำนวน วันที่ และหมายเหตุของรายการที่ยังไม่สิ้นสุด พร้อมประวัติการแก้ไข
- แอดมินแก้ชื่อ เบอร์โทร สิทธิ์ และระงับสมาชิก
- แสดงเกินกำหนดโดยคำนวณตามวันที่ Asia/Bangkok โดยไม่ต้องมีงาน cron
- หน้าจอปรับตามขนาด: แถบเมนูล่างบนมือถือ / NavigationRail บนจอใหญ่
- Realtime refresh และปุ่มรีเฟรชสำรอง

รายการอุปกรณ์เป็นแบบ **จำนวนต่อประเภท** เช่น โน้ตบุ๊ก 5 ชิ้น ยังไม่แยก serial number ของแต่ละเครื่อง

## 1. สิ่งที่ต้องติดตั้งบนเครื่องคุณ

1. Flutter stable ที่ใช้ Dart >= 3.12 / Flutter >= 3.44 (https://docs.flutter.dev/get-started/install)
2. VS Code และ extensions Flutter / Dart
3. Chrome สำหรับ Web
4. Android Studio พร้อม Android SDK และ Emulator หรือมือถือที่เปิด USB debugging
5. บัญชี Supabase (https://supabase.com)

ตรวจเครื่อง:

```sh
flutter doctor
flutter doctor --android-licenses
```

## 2. เปิดและเตรียมโปรเจกต์

แตกไฟล์ ZIP แล้วเปิดโฟลเดอร์ `equipment_loan` ใน VS Code

โปรเจกต์มีโฟลเดอร์ Android/Web ที่สร้างด้วย Flutter SDK หากไม่มีหรือคุณต้องการสร้างใหม่ ใช้:

**Windows PowerShell:**

```powershell
powershell -ExecutionPolicy Bypass -File tools/bootstrap.ps1
```

**macOS / Linux:**

```sh
bash tools/bootstrap.sh
```

สคริปต์สร้าง official Android/Web runners หากยังไม่มี และเพิ่ม INTERNET permission ใน Android release build โดยไม่ทับโค้ดแอป

จากนั้น:

```sh
flutter pub get
```

ก่อนเผยแพร่จริง เปลี่ยน applicationId / namespace / bundle identity จาก `com.example.equipment_loan` เป็นของคุณ และตั้งค่า signing

## 3. สร้างฐานข้อมูล Supabase

1. สร้าง **โปรเจกต์ใหม่** บน Supabase
2. เปิด **SQL Editor**
3. คัดลอกเนื้อหาไฟล์ `supabase/001_schema.sql` แล้ว Run **ครั้งเดียว**
4. Run `supabase/002_seed.sql` ถ้าต้องการอุปกรณ์ตัวอย่าง
5. Run `supabase/003_realtime.sql` เพื่อเปิด realtime

อย่ารัน schema ซ้ำกับฐานข้อมูลเดิม: ไฟล์นี้เป็น initial migration ไม่ใช่สคริปต์ upgrade

ระบบเก็บข้อมูลใน:

- `profiles`: ชื่อ เบอร์โทร สิทธิ์ สถานะบัญชี
- `equipment`: รายการและจำนวนอุปกรณ์ทั้งหมด
- `borrow_records`: คำขอและรายการยืม–คืน
- `borrow_status_history`: ประวัติสถานะและการแก้ไขรายการ

รหัสผ่านเก็บและจัดการโดย **Supabase Auth** ไม่อยู่ใน `profiles`

## 4. ตั้งค่าการเชื่อมต่อ

คัดลอก `config.example.json` เป็น `config.json` และกรอก:

```json
{
  "SUPABASE_URL": "https://your-project.supabase.co",
  "SUPABASE_ANON_KEY": "your-publishable-or-anon-key"
}
```

ดู URL และ Publishable key / legacy anon key ในหน้าตั้งค่า API ของ Supabase (ชื่อเมนูอาจต่างตาม UI)

**ห้ามใส่ service_role / secret key ใน Flutter** — แอปเว็บและ APK สามารถถูกตรวจดู key ได้ Publishable/anon key เป็น key สำหรับฝั่ง client การป้องกันข้อมูลอยู่ที่ RLS และ RPC ที่ตรวจสิทธิ์

`config.json` ถูกละเว้นด้วย `.gitignore` อย่าเก็บค่าลับลง Git และอย่าใส่รหัสผ่านฐานข้อมูลในแอป

## 5. รันบน Web

```sh
flutter run -d chrome --web-port=3000 --dart-define-from-file=config.json
```

ใน Supabase → Authentication → URL Configuration:

- ตั้ง Site URL เป็น `http://localhost:3000` ขณะพัฒนาเว็บ
- เพิ่ม Redirect URL ที่เกี่ยวข้องกับเว็บทดสอบ เช่น `http://localhost:3000/**`
- เมื่อ deploy แล้วเปลี่ยน Site URL และ allowlist เป็นโดเมนจริง

หากเปิด Confirm email: สมัครแล้วเปิดลิงก์ยืนยันในอีเมล จากนั้นกลับมาเข้าสู่ระบบในเว็บหรือ Android แอปนี้ยังไม่ตั้ง native deep link ดังนั้นบน Android ให้ยืนยันผ่านหน้าเว็บก่อนแล้วกลับมาเข้าสู่ระบบด้วยรหัสผ่าน

บริการอีเมลเริ่มต้นของ Supabase อาจจำกัดผู้รับและอัตราส่ง ให้ตั้ง custom SMTP ตามคำแนะนำ Supabase ก่อนเปิดรับสมาชิกจริง

สามารถทดสอบโดยปิด Confirm email เฉพาะ **โปรเจกต์พัฒนา** แต่ไม่ควรลดมาตรการความปลอดภัยของระบบใช้งานจริงเพียงเพื่อให้ทดสอบผ่าน

## 6. สร้างแอดมินคนแรก

1. สมัครสมาชิกผ่านแอป
2. ยืนยันอีเมล ถ้าเปิดใช้
3. เข้า SQL Editor แล้วรันโดยแทนอีเมลให้ตรงกับบัญชีที่สมัคร:

```sql
update public.profiles
set role = 'admin'
where id = (select id from auth.users where email = 'YOUR_ADMIN_EMAIL');
```

4. กดรีเฟรช หรือออกและเข้าสู่ระบบใหม่ จะเห็นเมนูแอดมิน

สมาชิกสมัครใหม่จะเป็น `member` เสมอ แม้ส่ง metadata role=admin มาเองก็ไม่ได้สิทธิ์ ไม่มีบัญชี/รหัสผ่านแอดมินเริ่มต้นฝังในโปรเจกต์

## 7. รันบน Android

เปิด Emulator หรือเสียบมือถือ แล้วดู device id:

```sh
flutter devices
flutter run -d YOUR_ANDROID_DEVICE_ID --dart-define-from-file=config.json
```

Supabase ใช้ HTTPS Android ต้องมีอินเทอร์เน็ตและ `android.permission.INTERNET` ซึ่ง bootstrap เตรียมให้

## 8. Workflow และจำนวนคงเหลือ

```text
pending → approved → borrowed → return_requested → returned
    ├── rejected
    └── cancelled
approved → cancelled (แอดมิน)
borrowed → returned (แอดมินรับคืนโดยตรงได้)
```

- `pending`: ยังไม่จองจำนวน อาจมีหลายคำขอมากกว่าคงเหลือได้
- `approved`: จองจำนวนทันที แม้วันยืมเป็นวันในอนาคต
- `borrowed` / `return_requested`: จำนวนยังถูกใช้งาน
- `returned` / `rejected` / `cancelled`: ไม่จองจำนวน
- `overdue` เป็น **สถานะแสดงผล** ของ borrowed/return_requested ที่กำหนดคืนผ่านแล้ว ไม่ใช่ enum ในฐานข้อมูล
- ปฏิเสธต้องมีเหตุผล
- สมาชิกยกเลิกได้เฉพาะ pending
- สมาชิกแจ้งคืนเองได้ แต่ **เพิ่มคงเหลือได้เมื่อแอดมินยืนยันรับคืนเท่านั้น**
- แอดมินแก้ไขรายการที่ยังไม่สิ้นสุดได้ โดยไม่เปลี่ยนผู้ยืมหรือชนิดอุปกรณ์ ถ้าต้องการเปลี่ยน ให้ยกเลิกรายการที่ยังไม่ส่งมอบแล้วสร้างคำขอใหม่
- ปิดใช้งานอุปกรณ์แทนการลบถาวร เพื่อรักษาประวัติ; การปิดใช้งาน/ชำรุดจะไม่อนุญาตให้อนุมัติหรือส่งมอบเพิ่ม แต่แอดมินรับคืนรายการเดิมได้

จำนวนพร้อมยืมคำนวณจาก `total_quantity - sum(approved, borrowed, return_requested)` ไม่แก้ตัวเลขคงเหลือด้วยมือ ใช้ transaction และ row lock เมื่ออนุมัติ แก้จำนวน ยกเลิก หรือคืน เพื่อป้องกัน double allocation

## 9. สิทธิ์และความเป็นส่วนตัว

- สมาชิกอ่านได้เฉพาะ profile ของตน และรายการยืม/ประวัติของตน
- แอดมินที่ active อ่านสมาชิกและรายการทั้งหมดได้
- ไม่มีสิทธิ์เขียนตารางจาก client ตรง ๆ การเขียนผ่าน checked RPC เท่านั้น
- role และ is_active ตรวจจากฐานข้อมูล ไม่เชื่อค่าที่ส่งจากแอป
- RPC `current_borrowers` เผยเฉพาะชื่อ จำนวน กำหนดคืน ของผู้ที่รับอุปกรณ์แล้ว ไม่เผยอีเมล/เบอร์โทรหรือประวัติยืมทั้งหมด
- หน้าสมัครแจ้งสมาชิกว่าชื่อที่แสดงจะเห็นโดยสมาชิกคนอื่นระหว่างยืมอุปกรณ์ หากเป็นองค์กรให้จัดทำนโยบายความเป็นส่วนตัวและกำหนดฐานการประมวลผลที่เหมาะสมก่อนเปิดใช้งาน
- บัญชีถูกระงับจะใช้งานและส่งคำขอไม่ได้ แอดมินยังดำเนินการรับคืนแทนได้
- ป้องกันแอดมินระงับ/ลดสิทธิ์ตัวเอง และต้องมีแอดมิน active อย่างน้อยหนึ่งบัญชีผ่าน RPC

## 10. ตรวจและ Build

```sh
flutter analyze
flutter test
flutter build web --release --dart-define-from-file=config.json
flutter build apk --release --dart-define-from-file=config.json
# สำหรับ Google Play หลังตั้ง signing:
flutter build appbundle --release --dart-define-from-file=config.json
```

- เว็บอยู่ใน `build/web` ใช้ HTTPS hosting เช่น Firebase Hosting / Cloudflare Pages / static hosting
- GoRouter ใช้ hash URL เป็นค่าเริ่มต้น จึงรีเฟรชหน้าเว็บได้โดยไม่ต้องตั้ง rewrite แบบ path URL
- APK อยู่ใน `build/app/outputs/flutter-apk/app-release.apk`
- ยังไม่ได้ตั้ง production Android signing, custom domain หรือ deploy ให้บัญชีของคุณ

## 11. ทดสอบกับ Supabase จริงก่อนใช้งาน

ใช้บัญชี 3 บัญชี: Admin, Member A, Member B และอุปกรณ์ 1 ชิ้น:

1. สมัคร/ยืนยันอีเมล/เข้าสู่ระบบครบทุกบัญชี
2. Member ไม่เห็นเมนูสมาชิก และเปิด /members แล้วถูกพากลับหน้าอุปกรณ์
3. A ส่งคำขอ: pending, จำนวนว่างยัง 1
4. B ส่งคำขอ: pending
5. Admin อนุมัติ A: approved, จำนวนว่าง 0
6. พยายามอนุมัติ B: ต้องได้ข้อผิดพลาดจำนวนไม่พอ
7. ลองอนุมัติทั้งสองจากสองหน้าต่างพร้อมกัน: ต้องสำเร็จเพียงหนึ่งรายการ
8. ส่งมอบ A: borrowed, B เห็นชื่อ A แต่ไม่เห็นเบอร์โทร/รายการส่วนตัวของ A
9. A แจ้งคืน: return_requested, จำนวนว่างยัง 0
10. Admin รับคืน: returned, จำนวนว่างกลับ 1
11. ลองกดยืนยันซ้ำ: ต้องไม่เพิ่มคงเหลือเกิน total
12. สมาชิกยกเลิก pending ได้ และยกเลิก approved เองไม่ได้
13. ปฏิเสธโดยไม่มีเหตุผลต้องไม่ได้
14. ลด total ให้น้อยกว่าจำนวนที่ยืม/จองอยู่ต้องไม่ได้
15. เปลี่ยนกำหนดคืนเป็นเมื่อวานบน borrowed: ต้องแสดงเกินกำหนด
16. ระงับสมาชิก: สมาชิกส่งคำขอใหม่ไม่ได้ แม้ยังมี auth session
17. ยิง API ตรงเพื่อแก้ profile.role / insert borrow_records: ต้องถูกปฏิเสธ
18. สมาชิกเรียก admin RPC: ต้องถูกปฏิเสธ
19. แก้รายการแล้วเปิดประวัติ: ต้องเห็นค่าเดิมและใหม่
20. ทดสอบเว็บรีเฟรชหน้า, Android กลับจาก background, ภาษาไทย และจอมือถือขนาดเล็ก

Flutter unit/widget tests ใน ZIP ตรวจ logic หน้าตั้งค่า และหน้าสมาชิก/แอดมินที่ความกว้างมือถือและเดสก์ท็อป ไม่แทนการทดสอบจริงกับ Supabase หรือ Android

## โครงสร้าง

```text
lib/
  main.dart
  core/       # providers, auth/session, router, shared widgets
  models/     # equipment/profile/loan + transition helpers
  services/   # Supabase repository
  screens/    # auth, responsive shell, equipment, loans, members
supabase/     # schema + RPC/RLS, sample equipment, realtime
android/      # official generated Android runner
web/          # official generated Web runner
.vscode/      # debug launch configurations
config.example.json
test/         # unit / widget tests
tools/        # bootstrap scripts
```

## ขอบเขตเวอร์ชันเริ่มต้น

ยังไม่มีระบบอัปโหลดรูป, QR/barcode, push notification, reset password UI, export Excel, serial tracking, การจองช่วงเวลาที่ไม่ชนกัน หรือ pagination เต็มรูปแบบ รายการยืมและสมาชิกแสดงสูงสุด 1,000 รายการจากฐานข้อมูล หากมีมากกว่านี้ต้องเพิ่ม server-side search/pagination ก่อนใช้งานในขนาดใหญ่

ไม่สามารถแก้ email/password สมาชิกจากหน้าจัดการสมาชิก ให้ใช้ Supabase Auth flow ที่เหมาะสม การแก้สิทธิ์และการระงับในหน้านี้เป็นสิทธิ์ของระบบยืมคืน ไม่ใช่การลบบัญชี Auth

## ทดสอบฐานข้อมูลอัตโนมัติ (สำหรับนักพัฒนา — ไม่จำเป็นตอนติดตั้งแอป)

มี `tools/test_database.py` สำหรับ PostgreSQL 16 บนเครื่องคุณ ใช้ auth schema จำลอง ไม่แทนการทดสอบ Supabase Auth/PostgREST/Realtime จริง สคริปต์อนุญาตเฉพาะฐานข้อมูลใหม่ที่ว่างบน localhost และปฏิเสธฐานข้อมูลที่มีตารางระบบอยู่แล้ว

```sh
python -m pip install -r requirements-test.txt
createdb -h 127.0.0.1 -T template0 -E UTF8 equipment_loan_test
# เปลี่ยน USER / password / port ให้ตรงกับ PostgreSQL บนเครื่องทดสอบเท่านั้น
export TEST_DATABASE_DSN="host=127.0.0.1 port=5432 dbname=equipment_loan_test user=USER"
python tools/test_database.py
```

ผู้ใช้ฐานข้อมูลทดสอบต้องมีสิทธิ์สร้าง role/schema/publication ต้องสร้างฐานข้อมูลว่างใหม่ก่อนรันซ้ำ ห้ามชี้ไปฐานข้อมูลจริง รายการตรวจรวม role escalation, RLS, สถานะคืน, การแก้จำนวน และการอนุมัติพร้อมกัน

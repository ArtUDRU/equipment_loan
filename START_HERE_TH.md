# เริ่มใช้งาน

1. ติดตั้ง Flutter stable (Flutter >=3.44, Dart >=3.12), VS Code, ส่วนเสริม Dart/Flutter; สำหรับ Android ให้ติดตั้ง Android Studio/SDK ด้วย
2. แตก ZIP และเปิดโฟลเดอร์ equipment_loan ใน VS Code
3. สร้างโปรเจกต์ Supabase ใหม่ แล้ว Run SQL ตามคู่มือ README_TH.md
4. คัดลอก config.example.json เป็น config.json และใส่ URL + publishable/anon key ของโปรเจกต์
5. เปิด Terminal ในโฟลเดอร์นี้:

```sh
flutter pub get
flutter run -d chrome --web-port=3000 --dart-define-from-file=config.json
```

6. สมัครบัญชีของคุณ ยืนยันอีเมลถ้าเปิดใช้ แล้วกำหนดแอดมินคนแรกด้วย SQL ใน README_TH.md
7. ทดสอบบน Android:

```sh
flutter devices
flutter run -d YOUR_ANDROID_DEVICE_ID --dart-define-from-file=config.json
```

อย่าใส่ service_role / secret key หรือรหัสผ่านฐานข้อมูลใน config.json

ไฟล์โค้ดและฐานข้อมูลพร้อมให้คุณตั้งค่าและรัน ยังไม่มีฐานข้อมูล/บัญชี Supabase ที่เชื่อมไว้ และยังไม่ได้เผยแพร่เว็บหรือสร้าง APK สำหรับบัญชีคุณ

คู่มือฉบับเต็ม: README_TH.md
ผลตรวจและขอบเขตการทดสอบ: TEST_REPORT_TH.md

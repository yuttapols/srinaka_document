# Srinaka — Requirement Specification

> ธุรกิจ: ร้านสปา/นวด เน้นเว็บสวยงาม ให้ลูกค้าดูบริการ/สินค้า+ราคา, จองนัด, จ่ายเงิน, ยกเลิก/ขอคืนเงิน,
> และให้คะแนนได้ผ่านเว็บ

## ⚠️ ยังไม่ตัดสินใจ / เป็น assumption ที่ต้อง confirm ก่อนเริ่มเขียนโค้ดจริง

- **Payment**: Phase 1 ใช้ **แนบสลิปโอนเงิน** (reuse pattern `FileStorageService`+Cloudinary จาก Share Money)
  แทน payment gateway จริง เพราะเร็วกว่า/ต้นทุนต่ำกว่าตอนเริ่ม — เปลี่ยนไปใช้ payment gateway จริง
  (Omise/2C2P ฯลฯ) ทีหลังได้ถ้าปริมาณจองเยอะจนตรวจสลิปมือไม่ทัน
- **ข้อมูลพนักงานเพิ่มเติม** (nickname, รูป, bio, เลขบัตรประชาชน, ที่อยู่, เบอร์ติดต่อฉุกเฉิน, วันเริ่มงาน,
  ความถนัด/specialty) — เคยเสนอแยกตาราง `employee_profiles`+`employee_specialties` ไว้ แต่**ยังไม่ confirm
  ว่าต้องการฟิลด์ไหนบ้าง** ห้ามถือว่าตัดสินใจแล้ว รอถามให้ชัดอีกทีก่อนใส่ลง `03-database-design.md`
- **นโยบาย refund/cancel ที่ชัดเจน** — ยืนยันแล้วว่า**ต้องมี** feature ยกเลิก+คืนเงิน (ดู FR-7) แต่รายละเอียด
  ตัวเลข (คืนเต็ม/บางส่วน/ไม่คืนถ้ายกเลิกใกล้เวลานัดเกินไปกี่ชั่วโมง) ยังไม่กำหนด
- Employee เห็นตารางนัดของตัวเองอย่างเดียว หรือเห็นของทุกคน — สมมติไว้ว่า **เห็นแค่ของตัวเอง**
- การจัดการตาราง/กะทำงานของ Employee (availability) — **Phase 1 ไม่ทำ**
- **รูปแบบใบเสร็จ walk-in sale** — สมมติไว้ว่าใช้ browser print ฝั่ง frontend (เร็วสุด) แทน backend generate PDF — ยังไม่ confirm

## เลื่อนออกไปก่อน (ยังไม่ทำตอนนี้ — user บอกว่า "อาจจะยังไม่ต้องทำ")

- **ระบบทิป** (tip) — เคยออกแบบไว้เป็น FR แล้ว ตอนนี้พักไว้ก่อน ไม่อยู่ใน roadmap ปัจจุบัน
- **ระบบแจ้งปัญหา** (issue report จากลูกค้า) — เคยออกแบบไว้เป็น FR แล้ว ตอนนี้พักไว้ก่อนเช่นกัน

(อย่าสับสนกับ **Phase 5 "Issue"** ใน `09-implementation-roadmap.md` — อันนั้นคือ phase หาบั๊ก/ช่องโหว่ของ
ทีมเอง หลัง Phase 1-4 เสร็จ คนละเรื่องกับ feature แจ้งปัญหาของลูกค้าที่พักไว้นี้)

## 1. ภาพรวมระบบ (Overview)

ระบบเว็บร้านสปา/นวด ให้ลูกค้าดูสินค้า/บริการ+ราคา, สมัครสมาชิก (กรอกเอง/Google/LINE) + ยืนยันตัวตนก่อนถึงจะ
จองนัดได้, จ่ายเงินผ่านการแนบสลิป, ยกเลิก/ขอคืนเงินได้, ให้คะแนนร้าน/พนักงาน ฝั่งร้านมี Supervisor จัดการสินค้า/บริการ/
โปรโมชั่น และดู report ภาพรวม Employee ให้บริการจริงตามที่ถูกมอบหมาย **และขายบริการหน้าร้านให้ลูกค้า walk-in
เก็บเงินสดเองได้ (POS-lite)** ส่วน Admin ดูแลระบบ/บัญชีผู้ใช้เท่านั้น ไม่ยุ่งข้อมูลธุรกิจ

## 2. Actor / Role

| Role | คำอธิบาย | login |
|---|---|---|
| **Admin** | ดูแลระบบ/infra, จัดการบัญชี Supervisor/Employee, nav-menu permission — ไม่เห็น/ไม่ยุ่งข้อมูลธุรกิจ | ต้อง login |
| **Supervisor** | เจ้าของ/ผู้ดูแลธุรกิจ จัดการสินค้า/บริการ/โปรโมชั่น, มอบหมาย booking ให้ Employee, ดู report ภาพรวม, ตรวจสลิป, อนุมัติ refund | ต้อง login |
| **Employee** | พนักงานให้บริการ ดูตารางนัดของตัวเอง, **ขายบริการ/รับชำระเงินสดหน้าร้าน (walk-in) ได้เอง**, ถูกลูกค้าให้คะแนน | ต้อง login |
| **Customer** | ลูกค้า ดูสินค้า/บริการ+ราคาได้โดยไม่ login แต่**ต้องสมัครสมาชิก+ยืนยันตัวตนผ่าน Google/LINE ก่อนถึงจะจองได้** (ไม่มี guest booking) | ดู public content ไม่ต้อง login, จองต้อง login+verify |

## 3. ศัพท์เฉพาะ (Glossary)

| คำ | ความหมาย |
|---|---|
| บริการ (service) / สินค้า (product) | รายการที่ร้านขาย — บริการสปา/นวด (มีระยะเวลา) หรือสินค้าขายเลย (เช่น น้ำมันนวด) |
| โปรโมชั่น (promotion) | ส่วนลด/แพ็กเกจพิเศษที่ Supervisor สร้าง ผูกกับบริการ/สินค้า |
| การจอง (booking) | คำขอนัดของ Customer ที่ verify ตัวตนแล้ว ระบุบริการ+วันเวลาที่ต้องการ |
| สถานะการจอง | `PENDING_PAYMENT` → `CONFIRMED` (หลังสลิปผ่านการตรวจ) → `COMPLETED` / `CANCELLED` |
| สถานะการคืนเงิน | `NONE` → `REQUESTED` → `APPROVED`/`REJECTED` → `REFUNDED` |
| Walk-in sale | รายการขายบริการหน้าร้านที่ Employee/Supervisor สร้างเองให้ลูกค้าที่เดินเข้าร้านมาเลย (ไม่ได้จองออนไลน์ล่วงหน้า) — ใช้ `bookings`/`payments` ตารางเดียวกับ booking ออนไลน์ แค่ `channel=WALK_IN` |

## 4. Functional Requirements

### FR-1 Authentication & Session
- FR-1.1 Login ด้วย username/password คืน JWT access+refresh token, role, name (Admin/Supervisor/Employee/Customer)
- FR-1.2 Logout (revoke refresh token)
- FR-1.3 บัญชีที่ถูกปิดใช้งานห้าม login
- FR-1.4 บันทึก login/logout log — Admin ดูย้อนหลังได้
- FR-1.5 รหัสผ่านเก็บด้วย BCrypt
- FR-1.6 **Public endpoint** (ดูสินค้า/บริการ+ราคา) ไม่ต้องผ่าน JWT filter เลย ตั้งแต่ level ของ Security config
- FR-1.7 Customer สมัครสมาชิกได้ 3 ทาง — **กรอกเอง** (username/password), **Google**, **LINE** — ทุกทางบังคับ
  กรอก username, password, ชื่อ, เบอร์โทร เหมือนกันหมด (ให้ login ด้วย username/password ได้เสมอไม่ว่าสมัคร
  ทางไหนมา — เบอร์โทรเป็นข้อมูลติดต่อธรรมดา ไม่มีการยืนยันว่าใช้งานได้จริงอีกต่อไป ไม่ใช้ SMS OTP)
- FR-1.8 บัญชีที่สมัครผ่าน **Google หรือ LINE ถือว่า verified ทันที** (`verified_at` = เวลาสมัคร) เพราะ OAuth
  คือการยืนยันตัวตนอยู่แล้ว
- FR-1.9 บัญชีที่สมัคร**กรอกเอง**ยังไม่ verified — ต้อง login เข้าระบบก่อน แล้วไป**ลิงก์บัญชี Google หรือ LINE
  เพิ่ม** (อย่างใดอย่างหนึ่งพอ) ผ่านหน้า profile ระบบจะผูก `google_user_id`/`line_user_id` เข้ากับ user เดิม
  (ไม่ merge บัญชีใหม่ ไม่สร้าง user ซ้ำ) แล้ว set `verified_at` ทันที

### FR-2 Profile
- FR-2.1 ทุก role เปลี่ยนรหัสผ่านได้ (ต้องกรอกรหัสเดิมให้ถูก)
- FR-2.2 แก้ไขชื่อ, เบอร์โทร ได้อิสระ — การแก้เบอร์โทรไม่กระทบสถานะ `verified_at` เลย (verify มาจากการลิงก์
  Google/LINE ไม่ใช่จากตัวเบอร์โทรอีกต่อไป)

### FR-3 User Management
- FR-3.1 Admin สร้างบัญชี Supervisor
- FR-3.2 Supervisor สร้างบัญชี Employee ที่อยู่ในร้านตัวเอง
- FR-3.3 username ไม่ซ้ำกันทั้งระบบ
- FR-3.4 **Admin ไม่มีสิทธิ์เข้าถึง product/service catalog, promotion, booking, payment, refund, report, rating เด็ดขาด**

### FR-4 Product & Service Catalog
- FR-4.1 Supervisor สร้าง/แก้ไข/ลบ สินค้าและบริการ (ชื่อ, คำอธิบาย, ราคา, ระยะเวลา (ถ้าเป็นบริการ), รูปภาพ)
- FR-4.2 รูปภาพเก็บผ่าน Cloudinary (`FileStorageService`, public delivery)
- FR-4.3 Public (ไม่ login) ดูรายการสินค้า/บริการ+ราคา+รูปได้ทั้งหมด
- FR-4.4 Employee ดูรายการได้ (read-only)

### FR-5 Promotion
- FR-5.1 Supervisor สร้าง/แก้ไข/ลบโปรโมชั่น ผูกกับสินค้า/บริการ (ส่วนลด/แพ็กเกจ)
- FR-5.2 Public เห็นโปรโมชั่นที่ active อยู่

### FR-6 Booking
- FR-6.1 Customer ที่ login+verified แล้วเท่านั้น (ดู FR-1.8/FR-1.9) เลือกบริการ+วันเวลา → สร้าง booking
  สถานะ `PENDING_PAYMENT`
- FR-6.2 Customer แนบสลิปโอนเงินตอนจอง (ดู FR-7)
- FR-6.3 Supervisor/Employee ดู booking ที่เข้ามา, มอบหมาย Employee ผู้ให้บริการ (manual — ไม่มี auto-assign ตาม availability ใน Phase 1)
- FR-6.4 Employee เห็น booking ที่ตัวเองถูกมอบหมายเท่านั้น, อัปเดตสถานะเป็น `COMPLETED` หลังให้บริการเสร็จ
- FR-6.5 ไม่มีการเช็ค conflict กับตารางเวลา Employee ใน Phase 1

### FR-7 Payment, Cancel & Refund (แนบสลิป — Phase 1)
- FR-7.1 Customer อัปโหลดสลิปโอนเงินผูกกับ booking (validate content-type จาก header จริง + ขนาดไฟล์ ตาม pattern เดียวกับ Share Money)
- FR-7.2 Supervisor/Employee ตรวจสลิป → กด confirm ให้ booking เปลี่ยนเป็น `CONFIRMED`, หรือ reject ถ้าสลิปผิด/ปลอม
- FR-7.3 เก็บ payment record ผูกกับ booking (จำนวนเงิน, สถานะตรวจสอบ, ผู้ตรวจ, เวลาที่ตรวจ)
- FR-7.4 **Customer ขอยกเลิก booking ก่อนถึงเวลานัดได้** → สร้างคำขอคืนเงิน (สถานะ `REQUESTED`)
- FR-7.5 **Supervisor อนุมัติ/ปฏิเสธคำขอคืนเงิน** — ถ้าอนุมัติ (`APPROVED`) โอนเงินคืนเองนอกระบบ (bank transfer) แล้วกด mark เป็น `REFUNDED` ในระบบ (ไม่มี automated refund ผ่าน gateway เพราะ payment เป็นแบบแนบสลิป ไม่ใช่ gateway จริง)
- FR-7.6 booking ที่ refund สำเร็จ → สถานะเปลี่ยนเป็น `CANCELLED`
- FR-7.7 (Phase หลัง, ยังไม่ทำ) เปลี่ยนไปใช้ payment gateway จริงแทน/เสริมการแนบสลิป — ถ้าทำจริงค่อยเปลี่ยน refund เป็น automated ผ่าน gateway API

**Walk-in sale (POS-lite) — ขายบริการหน้าร้านแบบไม่ต้องจองออนไลน์ล่วงหน้า**
- FR-7.8 Employee/Supervisor สร้าง booking แบบ walk-in ได้เอง (`channel=WALK_IN`) — เลือกบริการ+พนักงานผู้ให้บริการ (คือตัวเอง หรือเลือกคนอื่นก็ได้) ทันที ไม่ต้องมี Customer login เลย, กรอกชื่อลูกค้า walk-in ได้ (optional ไม่บังคับ)
- FR-7.9 รับชำระเป็น**เงินสด** (`method=CASH`) แทนแนบสลิป — ไม่ต้องอัปโหลดไฟล์
- FR-7.10 Employee **คีย์ส่วนลด**ได้ตรง ๆ ตอนขาย (`discount_amount`) แยกจากระบบโปรโมชั่น (FR-5) — เก็บ audit ว่าลดเท่าไหร่ ใครเป็นคนขาย (`created_by`)
- FR-7.11 Walk-in booking ข้ามสถานะ `PENDING_PAYMENT` ไปเป็น `CONFIRMED` ทันทีหลังรับเงินสด (ไม่ต้องรอตรวจสลิปเหมือน booking ออนไลน์)
- FR-7.12 พิมพ์ใบเสร็จ (receipt) ได้หลัง walk-in sale สำเร็จ — แนะนำใช้ browser print (`window.print()` + CSS print stylesheet) ฝั่ง frontend เพราะเร็ว/ไม่ต้องพึ่ง PDF library ฝั่ง backend (assumption, ยังไม่ confirm)
- FR-7.13 **ไม่มีระบบตัด stock/นับสินค้าคงเหลือ** — ยืนยันแล้วว่าไม่ต้องทำ เพราะร้านขายบริการ ไม่ใช่สินค้าที่ต้องนับจำนวนคงคลัง (ถึงจะมี `services.type=PRODUCT` ในระบบก็ไม่ตัด stock อัตโนมัติ)

### FR-8 Rating
- FR-8.1 Customer ให้คะแนนร้าน (rating ภาพรวม) หลังใช้บริการ
- FR-8.2 Customer ให้คะแนน Employee ที่ให้บริการตัวเอง
- FR-8.3 Supervisor เห็นคะแนนเฉลี่ย (public เห็นได้ไหมยังไม่กำหนด — เช่นแสดงหน้าร้านเพื่อสร้างความน่าเชื่อถือ)

### FR-9 Report (Supervisor)
- FR-9.1 Report ภาพรวม: booking, รายได้, คำขอคืนเงิน, คะแนนเฉลี่ย — filter ตามช่วงเวลา

### FR-10 Employee schedule (deferred — ไม่ทำ Phase 1)
- FR-10.1 (Phase หลัง) Employee ตั้ง available time ของตัวเอง, Supervisor เห็น availability ตอน assign booking
- FR-10.2 (Phase หลัง) ระบบเช็ค conflict อัตโนมัติตอนสร้าง booking ใหม่

### FR-11 ข้อมูลติดต่อร้าน (Shop Contact Info)
- FR-11.1 Supervisor แก้ไขข้อมูลติดต่อร้านได้ (ที่อยู่, เบอร์โทร, อีเมล, ลิงก์แผนที่/พิกัด, เวลาเปิด-ปิด,
  ลิงก์ social เช่น Line/Facebook/Instagram) — ทุกฟิลด์ไม่บังคับกรอกครบ
- FR-11.2 Public ดูข้อมูลติดต่อร้านได้โดยไม่ต้อง login (หน้า "ติดต่อเรา")
- FR-11.3 เป็นข้อมูลแถวเดียว (single-shop, ไม่ใช่ list ของหลายสาขา) — Supervisor แก้ไขได้อย่างเดียว ไม่มี
  create/delete แยก

## 5. Non-Functional Requirements (ย่อ)

- ลอก pattern security/RBAC/response wrapper/file-storage จาก Share Money มาตรง ๆ (ดู `docs/05-tech-stack.md`)
- หน้า public (browse สินค้า/บริการ) ต้องโหลดเร็ว, เน้น UI/UX สวยงามเป็นพิเศษ
- สลิป ต้อง validate content-type จากไฟล์จริง (ไม่เชื่อ extension) + จำกัดขนาด เหมือนกติกาเดิมของ Share Money

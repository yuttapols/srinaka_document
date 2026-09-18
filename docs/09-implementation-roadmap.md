# Implementation Roadmap — Frontend & Backend (5 Phases)

แบ่งงาน 5 Phase ยึด requirement ใน `01-requirements.md` (FR-1 ถึง FR-10) — Phase 1-4 คือ feature ตามลำดับ,
Phase 5 เป็น phase หาบั๊ก/ช่องโหว่/ช่องว่างของทีมเอง ไม่ใช่ feature ใหม่

สถานะ task: `TODO` → `IN PROGRESS` → `DONE` → `ACCEPTED`

## Dependency และลำดับส่งมอบ

| ลำดับ | Backend | Frontend | ผลลัพธ์ที่ตรวจรับได้ | สถานะ |
|---|---|---|---|---|
| 1 | Foundation, Auth (ทุก role), Profile | App shell, Auth, Design system | ทุก role login ได้ | ✅ DONE |
| 2 | Product/Service catalog, Promotion, Booking, Customer register (กรอกเอง/LINE) | หน้า public สินค้า/บริการ/โปรโมชั่น, flow จองนัด, สมัครสมาชิก+ลิงก์บัญชี | ลูกค้าสมัคร+verify+จองนัดได้ (ยังไม่จ่ายเงิน) | ✅ DONE (รันจริงผ่านแล้ว 2026-09-18) |
| 3a | Payment (แนบสลิป/เงินสด), Walk-in/Cashier, Cancel/Refund | หน้าจ่ายเงิน(แนบสลิป), หน้าแคชเชียร์ walk-in+ใบเสร็จ, ยกเลิก/ขอคืนเงิน | ครบ flow ตั้งแต่จองถึงจบบริการ+ยกเลิก/รีฟันด์, ขาย walk-in ได้ | **กำลังทำ (priority ตอนนี้)** |
| 3b | Rating | ให้คะแนน | รีวิวร้าน/พนักงานได้ | Backlog — รอหลัง 3a |
| 4 | Report ภาพรวม, Employee schedule, Admin ops | Report dashboard, Employee schedule UI, Admin UI, polish | พร้อม deploy จริง, UAT ทั้งระบบ |
| 5 | หาบั๊ก/ช่องโหว่/ช่องว่างทั้งระบบ แล้วแก้/เติมเต็ม | เช่นเดียวกันฝั่ง FE | ระบบผ่านการรีวิวรอบสุดท้ายก่อน launch จริง |

## Backend — 5 Phases

### BE Phase 1 — Foundation, Security, User, Profile ✅ DONE

- [x] Spring Boot project, profiles, PostgreSQL (Neon) + Flyway baseline
- [x] schema: users, refresh_tokens, login_logs, menu_items, menu_permissions
- [x] login/refresh/logout/me/change-password ด้วย BCrypt/JWT (FR-1, FR-2)
- [x] แยก public vs authenticated endpoint ตั้งแต่ Security config
- [x] RBAC (Admin/Supervisor/Employee/Customer) + ownership policy, error response มาตรฐาน
- [x] Nav-menu API ตาม role
- [x] Docker/Render/Neon deploy setup (scaffold พร้อม — ยังไม่ได้ deploy ขึ้น Render จริงตามที่ตกลงว่ารอครบทุก Phase)

Acceptance: ทุก role login ได้ตามสิทธิ์, deploy ขึ้น Render+Neon ได้ — Customer self-register (กรอกเอง/
LINE) เป็นงาน Phase 2 (ดู FR-1.7–FR-1.9), ไม่ใช่ Phase 1

### BE Phase 2 — Product/Service Catalog, Promotion, Booking ✅ DONE (รันจริงผ่านแล้ว)

- [x] schema: products/services, promotions, bookings — เพิ่ม `line_user_id`/`verified_at` ใน `users`
      (migration ใหม่ต่อจาก Phase 1)
- [x] Customer สมัครสมาชิก 2 ทาง (กรอกเอง/LINE OAuth) — FR-1.7, FR-1.8
- [x] ลิงก์บัญชี LINE จากหน้า profile (สำหรับคนที่สมัครแบบกรอกเอง) — FR-1.9
- [x] CRUD สินค้า/บริการ/โปรโมชั่น (Supervisor เท่านั้น) — FR-4, FR-5
- [x] Public API ดูสินค้า/บริการ/โปรโมชั่น (ไม่ต้อง auth)
- [x] สร้าง booking (**ต้อง login+verified แล้วเท่านั้น** — บังคับที่ service layer ไม่ใช่แค่ frontend) — FR-6
- [x] Supervisor/Employee ดู booking, manual assign Employee
- [x] Employee ดู booking ของตัวเอง, อัปเดตสถานะ `COMPLETED`
- [x] Shop contact info: schema `shop_info` (แถวเดียว), Supervisor แก้ไขได้, Public ดูได้ไม่ต้อง auth — FR-11

Acceptance: ลูกค้าที่ verify แล้วจองได้, ลูกค้าที่ยังไม่ verify ถูกกันไม่ให้จอง, Supervisor assign พนักงานได้,
หน้า "ติดต่อเรา" ดึงข้อมูลจริงได้ — **ครบตามเกณฑ์แล้ว, รัน+ทดสอบบน DB จริงผ่านแล้ว (2026-09-18)**

**ยังไม่ทำในรอบนี้** (ตั้งใจ, ไม่ใช่ของหาย): รูปสินค้า/บริการผ่าน Cloudinary (FR-4.2) — ยัง Supervisor
อัปโหลดรูปผ่าน API ไม่ได้ รอ `FileStorageService` ที่จะทำพร้อม Phase ที่ต้องใช้ไฟล์จริงจัง

### BE Phase 3 — แบ่งเป็น 2 ช่วงตามที่คุยกัน (2026-09-18): **ทำก่อน = จอง+แคชเชียร์**, **backlog = รีวิว**

> โปรโมชั่น (FR-5) ทำ CRUD ครบใน Phase 2 แล้ว ไม่มีงานเพิ่มในส่วนนี้ตอนนี้ — ไม่ต้องแตะอีกจนกว่าจะมี requirement ใหม่

#### 3a — Payment (Slip/Cash) + Walk-in/Cashier — **ทำตอนนี้**

- [ ] อัปโหลดสลิปผูกกับ booking ผ่าน `FileStorageService`/Cloudinary (validate content-type จริง+ขนาด) — FR-7.1
- [ ] Supervisor/Employee ตรวจสลิป → confirm/reject booking (`PENDING_PAYMENT` → `CONFIRMED`) — FR-7.2, FR-7.3
- [ ] **Walk-in sale**: Employee/Supervisor สร้าง booking `channel=WALK_IN` เอง (ไม่ต้องมี Customer), เลือกบริการ+พนักงาน, คีย์ส่วนลด (`discount_amount`) — FR-7.8, FR-7.10
- [ ] **Cash payment**: `payments.method=CASH`, ข้าม `PENDING_PAYMENT` ไป `CONFIRMED` ทันที — FR-7.9, FR-7.11
- [ ] Receipt data endpoint (join booking+payment+service ให้ frontend เอาไป render/print) — FR-7.12
- [ ] Customer ขอยกเลิก booking → สร้างคำขอคืนเงิน (`REQUESTED`) — FR-7.4
- [ ] Supervisor อนุมัติ/ปฏิเสธคำขอคืนเงิน, mark `REFUNDED` หลังโอนคืนเองนอกระบบ — FR-7.5, FR-7.6

Acceptance: จองนัด→แนบสลิป→ร้านตรวจ confirm→ให้บริการ ครบ flow จริง, ยกเลิก+ขอคืนเงินทำได้ครบ flow, Employee
ขาย walk-in รับเงินสด+พิมพ์ใบเสร็จได้จากหน้าเว็บ

#### 3b — Rating — **Backlog (พักไว้ก่อนตามที่สั่ง)**

- [ ] Rating: ให้คะแนนร้าน+พนักงาน — FR-8

ไม่ต้องเริ่มจนกว่าจะได้รับแจ้งให้ทำต่อ — เก็บไว้ท้าย Phase 3 เหมือนเดิมในเอกสาร แต่จะ**ทำหลัง 3a เสร็จ**เท่านั้น

### BE Phase 4 — Report, Employee Schedule, Admin, Operations

- [ ] Report API ภาพรวม (booking, รายได้, คำขอคืนเงิน, คะแนนเฉลี่ย) — FR-9
- [ ] Employee availability/schedule (deferred จาก Phase 1) — FR-10
- [ ] Admin: user management เต็มรูปแบบ, nav-menu permission, login log
- [ ] Backup/restore runbook, hardening, production deploy checklist

Acceptance: ระบบพร้อม deploy จริง, Report ครบ, Employee schedule ทำงาน, ครบ UAT

### BE Phase 5 — Issue: หาปัญหาแล้วเติมเต็ม

- [ ] รีวิว flow ทั้งหมดตั้งแต่ Phase 1-4 หา edge case/บั๊ก/validation ที่ขาด
- [ ] Security review (auth bypass, ownership check ครบทุก endpoint, file upload validation)
- [ ] Load/performance check เบื้องต้น (Render free tier cold start, Neon connection limit)
- [ ] อุดช่องว่างที่เจอ + เขียน/อัปเดตเอกสารตามจริง

Acceptance: ไม่มีบั๊ก/ช่องโหว่ที่รู้อยู่แล้วค้างก่อน launch จริง

## Frontend — 5 Phases

### FE Phase 1 — Foundation, Design System, Auth, Profile

- [ ] Scaffold จาก `share_money_frontend` (copy `core/`, `shared/`, config ตาม `05-tech-stack.md` หัวข้อ
      "Frontend Reuse Plan") + เตรียม config Prerender (SSG) ไว้ (ยังไม่เปิดใช้จริงจนถึง FE Phase 2), routing,
      environment config
- [ ] Design system: retheme shared component เดิม (`app-button`/`app-input`/`app-card`/`app-dialog` ฯลฯ) +
      `ThemeService` ให้เป็น `white`/`green` ตาม brand token (Emerald/Gold) — ดู `05-tech-stack.md` หัวข้อ
      "Brand Assets / Design Tokens"
- [ ] Brand asset: favicon, logo (header/sidebar/login/dark mode) copy จาก Logo Pack เข้า `src/assets/brand/`
      ตามกติกาการใช้งานที่กำหนดไว้
- [ ] i18n: reuse `@ngx-translate` + `assets/i18n/th.json`/`en.json` เดิม เขียนคำแปลใหม่ตาม business สปา
- [ ] App shell (Admin/Supervisor/Employee dashboard) + public layout แยกต่างหาก (ปรับจาก `core/layout/
      app-shell/` เดิมที่มี shell เดียว)
- [ ] Login/logout/refresh/change-password + guards (`auth.guard`/`role.guard` reuse) + interceptor (reuse)
- [ ] เตรียม guard `verifiedGuard` ใหม่ไว้ (ยังไม่เปิดใช้จริงจนกว่าจะมีหน้าจองใน FE Phase 2)
- [ ] หน้า Profile แก้ชื่อ/เบอร์โทร (reuse โครงเดิม ปรับฟิลด์)

Acceptance: ทุก role login เข้าเมนูตาม role ได้ — สมัครสมาชิก Customer (กรอกเอง/LINE) + ลิงก์บัญชีเป็นงาน
FE Phase 2 (ดู `01-requirements.md` FR-1.7–FR-1.9), ไม่ใช่ Phase 1

### FE Phase 2 — Public Catalog, Promotion, Booking Flow

- [ ] หน้า public แสดงสินค้า/บริการ/โปรโมชั่น+ราคา+รูป (ไม่ต้อง login) — shared component เดิม + GSAP/
      ScrollTrigger (respect `prefers-reduced-motion`)
- [ ] เปิดใช้ Prerender (SSG) จริงสำหรับ route public (หน้าแรก/catalog/รายละเอียดบริการ/โปรโมชั่น)
- [ ] สมัครสมาชิก Customer 2 ทาง — กรอกเอง / ปุ่ม "เข้าสู่ระบบด้วย LINE"
      (ส่วนที่ share_money_frontend ไม่มี) — FR-1.7, FR-1.8
- [ ] หน้า profile: ปุ่มลิงก์บัญชี LINE (สำหรับคนที่สมัครแบบกรอกเอง ให้ verify ทีหลังได้) — FR-1.9
- [ ] เปิดใช้ `verifiedGuard` จริง — Flow จองนัด (บังคับ login+verified ก่อนเข้าถึงหน้าจอง) — Reactive Forms
- [ ] Supervisor: จัดการสินค้า/บริการ/โปรโมชั่น (table/form จาก shared component เดิม), หน้าดู booking + assign
      พนักงาน
- [ ] Employee: หน้าดู booking ของตัวเอง, mark completed
- [ ] หน้า public "ติดต่อเรา" (ที่อยู่/เบอร์/แผนที่/เวลาเปิด-ปิด/social), Supervisor: หน้าแก้ไขข้อมูลติดต่อร้าน

Acceptance: ลูกค้าจองนัดได้จริงหลัง verify ตัวตน, ฝั่งร้าน assign งานได้

### FE Phase 3 — แบ่งเป็น 2 ช่วงเหมือน BE (2026-09-18)

#### 3a — Payment (Slip/Cash) + Walk-in/Cashier — **ทำตอนนี้**

- [ ] หน้าอัปโหลดสลิปตอนจอง, สถานะรอตรวจ/confirmed
- [ ] Supervisor/Employee: หน้าตรวจสลิป confirm/reject
- [ ] **Employee: หน้าขาย walk-in (แคชเชียร์)** — เลือกบริการ, กรอกชื่อลูกค้า (optional), คีย์ส่วนลด, รับเงินสด
- [ ] **หน้าใบเสร็จ** — แสดงผล + ปุ่ม print (browser `window.print()` + CSS print stylesheet)
- [ ] หน้าขอยกเลิก booking + ขอคืนเงิน (Customer)
- [ ] หน้าอนุมัติ/ปฏิเสธคำขอคืนเงิน (Supervisor)

Acceptance: ครบ flow จองจริง ตั้งแต่แนบสลิปถึงจบบริการ, ยกเลิก/รีฟันด์ทำได้จากหน้าเว็บ, Employee ขาย walk-in
รับเงินสด+พิมพ์ใบเสร็จได้จริงจากหน้าเว็บ

#### 3b — Rating — **Backlog (พักไว้ก่อนตามที่สั่ง)**

- [ ] หน้าให้คะแนนร้าน+พนักงาน

ทำหลัง 3a เสร็จเท่านั้น

### FE Phase 4 — Report, Employee Schedule, Admin, Release

- [ ] Report dashboard (Supervisor)
- [ ] หน้า Employee schedule/availability
- [ ] หน้า Admin: user management, nav-menu permission
- [ ] Error pages, responsive/accessibility review, production build, deploy (Cloudflare Pages)

Acceptance: ระบบพร้อม deploy จริง, ทุก flow ผ่าน UAT

### FE Phase 5 — Issue: หาปัญหาแล้วเติมเต็ม

- [ ] รีวิว UI/UX ทุกหน้าซ้ำ หา edge case (error state, empty state, loading state ที่ขาด)
- [ ] Cross-browser/responsive check รอบสุดท้าย
- [ ] อุดช่องว่างที่เจอ

Acceptance: ไม่มีบั๊ก UI/UX ที่รู้อยู่แล้วค้างก่อน launch จริง

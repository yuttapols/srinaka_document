# Screen Design — Srinaka (Frontend Screen Inventory & Content Spec)

> เอกสารนี้เป็น **screen list + content spec ระดับเอกสาร** (ไม่ใช่ wireframe/mockup ภาพจริง) — ระบุว่า
> แต่ละหน้าจอ (screen) มีอะไรบ้าง, role ไหนเข้าถึงได้, เนื้อหา/component หลักคืออะไร, ผูกกับ FR ข้อไหนใน
> `01-requirements.md`, และอยู่ Phase ไหนตาม `09-implementation-roadmap.md` — เมื่อเริ่ม implement Phase
> ไหนจริง ค่อยทำ wireframe/mockup ละเอียด (Figma หรือ artifact) จากเอกสารนี้อีกที

## 0. หลักการออกแบบ (ยึดจาก tech stack + requirement)

- Angular standalone components + shared component library (reuse จาก `share_money_frontend`) + Tailwind
  (ดู `05-tech-stack.md`) — เน้น **หน้า public สวยงาม** เป็น
  จุดขายหลัก ตาม NFR ใน `01-requirements.md`
- **2 shell แยกกันชัดเจน** (อย่าสับสนกับ nav-menu permission ที่คุมแค่ dashboard):
  1. **Public/Marketing shell** — Header (โลโก้, เมนู: หน้าแรก/บริการ&สินค้า/โปรโมชั่น/เข้าสู่ระบบ/สมัครสมาชิก)
     + Footer — ใช้กับทุกหน้าที่ Customer เข้าถึงได้โดยไม่ต้อง login และหน้า Customer หลัง login (บัญชี/การจอง
     ของฉัน) เพราะ Customer ไม่ใช่ผู้ดูแลระบบ ไม่ควรเห็น sidebar แบบ dashboard
  2. **Dashboard shell** — Sidebar (คุมด้วย nav-menu permission ของ Admin/Supervisor) + Topbar (user menu:
     profile/logout) — ใช้กับ Admin/Supervisor/Employee เท่านั้น
- ทุกหน้าที่มี list/table ต้องมี 3 state ที่ต้องออกแบบไว้ตั้งแต่ต้น: **loading / empty / error** (ไม่ใช่แค่
  happy path) — QA เช็คซ้ำใน `08-qa-test-plan.md` ข้อ 4
- Route guard: `authGuard` (ต้อง login), `roleGuard` (ต้องตรง role), และ guard เพิ่มเฉพาะ Customer
  ("verifiedGuard" — ต้อง `verified_at` ไม่เป็น null ก่อนเข้าหน้าจอง — ได้ค่านี้จากสมัคร/ลิงก์บัญชีผ่าน
  Google หรือ LINE ไม่ใช่ SMS OTP) — ฝั่ง frontend กันไว้ก่อน UX ดี แต่ **backend ต้องบังคับซ้ำที่ service
  layer เสมอ** (ตามที่ระบุใน FR-6.1/roadmap BE Phase 2)

## 1. Screen Inventory ตาม Phase

ตารางทุก Phase ใช้คอลัมน์: **Screen** | **Role/Access** | **เนื้อหา/component หลัก** | **FR อ้างอิง** | **หมายเหตุ**

### Phase 1 — Foundation (Auth, Profile, Admin เริ่มต้น)

| Screen | Role/Access | เนื้อหา/component หลัก | FR | หมายเหตุ |
|---|---|---|---|---|
| Login | Public (ยังไม่ login) | ฟอร์ม username/password, ลิงก์ไปสมัครสมาชิก (เฉพาะ Customer) | FR-1.1 | Admin/Supervisor/Employee ไม่มีลิงก์สมัคร (สร้างบัญชีจากฝั่งแอดมินเท่านั้น) — สมัครสมาชิก/ลิงก์บัญชี Google-LINE ย้ายไปอยู่ Phase 2 แล้ว (ดูตารางด้านล่าง) |
| Dashboard home (per role, placeholder) | Admin/Supervisor/Employee | หน้าว่างหลัง login ก่อนมี business data จริง (Phase 2+ ค่อยเติม widget) | — | เป็น landing หลัง login ของ dashboard shell |
| Profile | ทุก role (shared component) | แก้ชื่อ, เบอร์โทร (แก้เบอร์โทรได้อิสระ ไม่กระทบ `verified_at`) | FR-2.2 | |
| เปลี่ยนรหัสผ่าน | ทุก role (shared component) | กรอกรหัสเดิม + รหัสใหม่ + ยืนยัน | FR-2.1 | อาจฝังในหน้า Profile เป็น tab เดียวกันก็ได้ |
| Admin: จัดการบัญชี Supervisor | Admin เท่านั้น | list + form สร้าง/ปิดใช้งานบัญชี Supervisor | FR-3.1 | |
| Admin: Nav-menu permission | Admin เท่านั้น | จัดการ `menu_items`/`menu_permissions` — คุม sidebar เท่านั้น | — | ⚠️ อย่าสับสนกับเมนูสินค้า/บริการ (ดู README หัวข้อเตือน) |
| Admin: Login log | Admin เท่านั้น | ตาราง login/logout log, filter ตาม user/วันที่ | FR-1.4 | |
| Error/403 เบื้องต้น | ทุก role | หน้า role ผิด/ไม่มีสิทธิ์เข้า route | — | ทำ error page เต็มชุดจริงใน Phase 4 |

### Phase 2 — Public Catalog, Promotion, Booking

| Screen | Role/Access | เนื้อหา/component หลัก | FR | หมายเหตุ |
|---|---|---|---|---|
| หน้าแรก (Home) | Public | Hero, บริการ/สินค้าแนะนำ, โปรโมชั่นเด่น | FR-4.3, FR-5.2 | เป็นจุดขาย UI/UX หลักตาม NFR — ควรลง visual design ละเอียดตอน implement จริง |
| รายการบริการ/สินค้า (Catalog listing) | Public | grid/list การ์ดสินค้า-บริการ, filter ตามหมวด, ราคา | FR-4.3 | |
| รายละเอียดบริการ/สินค้า (Detail) | Public | ชื่อ, คำอธิบาย, ราคา, ระยะเวลา (ถ้าเป็นบริการ), รูปภาพ, ปุ่ม "จองเลย" | FR-4.1, FR-4.3 | ปุ่มจอง: ถ้ายังไม่ login → redirect ไปหน้าสมัคร/login ก่อน, ถ้า login แล้วแต่ยังไม่ verified → redirect ไปหน้าลิงก์บัญชี Google/LINE (ไม่มี guest booking) |
| รายการโปรโมชั่น | Public | list โปรโมชั่นที่ active, ผูกกับบริการ/สินค้า | FR-5.2 | |
| สมัครสมาชิก (Register) | Public, เฉพาะ flow Customer | ปุ่ม 3 ทาง — กรอกเอง (ฟอร์ม username/password/ชื่อ/เบอร์โทร), "เข้าสู่ระบบด้วย Google", "เข้าสู่ระบบด้วย LINE" — Google/LINE ก็ยังต้องมีฟอร์มกรอกเบอร์โทร+password เพิ่มหลัง OAuth callback | FR-1.7, FR-1.8 | ไม่มีหน้ากรอก OTP อีกต่อไป — สมัครผ่าน Google/LINE ถือว่า verified ทันที |
| ลิงก์บัญชี Google/LINE (จากหน้า Profile) | Customer ที่ login อยู่แล้ว (สมัครแบบกรอกเอง, ยังไม่ verified) | ปุ่ม "เชื่อมต่อ Google"/"เชื่อมต่อ LINE" ในหน้า Profile, badge "ยืนยันตัวตนแล้ว/ยังไม่ยืนยัน" | FR-1.9 | ทำครั้งเดียวพอ ไม่ต้องทำซ้ำทุกครั้งที่ login |
| Supervisor: จัดการบริการ/สินค้า | Supervisor เท่านั้น | list + form CRUD (ชื่อ, คำอธิบาย, ราคา, ระยะเวลา, รูป) | FR-4.1 | Employee เห็น list เดียวกันแบบ read-only (FR-4.4) — คอมโพเนนต์เดียวกัน ต่าง permission |
| Supervisor: จัดการโปรโมชั่น | Supervisor เท่านั้น | list + form CRUD ผูกกับบริการ/สินค้า | FR-5.1 | |
| Booking flow (จองนัด) | Customer (login+verified เท่านั้น) | wizard: เลือกบริการ → เลือกวันเวลา → ตรวจสอบ/ยืนยัน | FR-6.1 | multi-step ในหน้าเดียวหรือแยกหน้าได้ — แนะนำ wizard component เดียวกันเพื่อลดการทำซ้ำ |
| การจองของฉัน (My Bookings) | Customer | list booking พร้อมสถานะ (`PENDING_PAYMENT`/`CONFIRMED`/`COMPLETED`/`CANCELLED`) | FR-6.1 | |
| รายละเอียดการจอง (Booking detail) | Customer | ข้อมูล booking, สถานะ, ปุ่มไปหน้าแนบสลิป/ยกเลิก (ผูกกับ Phase 3) | FR-6.1 | |
| Supervisor/Employee: รายการ booking + มอบหมายพนักงาน | Supervisor เท่านั้น (assign) | list booking เข้ามาใหม่, เลือก Employee ผู้ให้บริการ (manual) | FR-6.3 | list ต้องกัน N+1 (join customer/service/employee) — ดู CLAUDE.md |
| Employee: booking ของฉัน | Employee เท่านั้น | list เฉพาะที่ตัวเองถูกมอบหมาย, ปุ่ม mark `COMPLETED` | FR-6.4 | |
| ติดต่อเรา (Contact Us) | Public | ที่อยู่, เบอร์โทร, แผนที่, เวลาเปิด-ปิด, ลิงก์ social | FR-11.2 | |
| Supervisor: แก้ไขข้อมูลติดต่อร้าน | Supervisor เท่านั้น | ฟอร์มแก้ที่อยู่/เบอร์/แผนที่/เวลาเปิด-ปิด/social (แถวเดียว ไม่มี list) | FR-11.1 | |

### Phase 3 — Payment (Slip/Cash), Walk-in Sale, Cancel/Refund, Rating

| Screen | Role/Access | เนื้อหา/component หลัก | FR | หมายเหตุ |
|---|---|---|---|---|
| แนบสลิปโอนเงิน | Customer | upload สลิป (drag&drop/เลือกไฟล์), preview, ปุ่มส่ง | FR-6.2, FR-7.1 | validate content-type จาก header จริง + ขนาดไฟล์ ฝั่ง frontend เป็น UX เสริม, backend ต้อง validate ซ้ำเสมอ |
| Supervisor/Employee: ตรวจสลิป | Supervisor, Employee | รูปสลิป + รายละเอียด booking, ปุ่ม confirm/reject | FR-7.2, FR-7.3 | |
| ขอยกเลิก/ขอคืนเงิน | Customer | จากหน้า booking detail: ปุ่มยกเลิก → ฟอร์มเหตุผล → ยืนยัน | FR-7.4 | ⚠️ ข้อความ/เงื่อนไขคืนเงิน (คืนเต็ม/บางส่วน) ต้องรอ confirm นโยบายก่อนใส่ copy จริง |
| Supervisor: อนุมัติ/ปฏิเสธคำขอคืนเงิน | Supervisor เท่านั้น | list คำขอ `REQUESTED`, ปุ่ม approve/reject, mark `REFUNDED` หลังโอนคืนนอกระบบ | FR-7.5, FR-7.6 | |
| ให้คะแนน (Rating) | Customer | หลัง booking `COMPLETED`: ให้คะแนนร้าน + ให้คะแนนพนักงานที่ให้บริการ | FR-8.1, FR-8.2 | |
| Employee/Supervisor: ขาย walk-in (POS-lite) | Employee, Supervisor | เลือกบริการ, เลือกพนักงานผู้ให้บริการ, ชื่อลูกค้า (optional), คีย์ส่วนลด, รับเงินสด | FR-7.8–7.11 | ไม่มี Customer login เกี่ยวข้องเลย — หน้าเดียว จบ flow ในตัว (เลือกของ→คิดเงิน→รับเงินสด) |
| ใบเสร็จ (Receipt) | Employee, Supervisor (หลัง walk-in sale) | สรุปรายการ+ราคา+ส่วนลด+ผู้ขาย, ปุ่มพิมพ์ (`window.print()` + CSS print stylesheet) | FR-7.12 | ⚠️ รูปแบบยังไม่ confirm (browser print vs backend PDF) — ดู assumption ใน `01-requirements.md` |

### Phase 4 — Report, Employee Schedule, Admin, Release

| Screen | Role/Access | เนื้อหา/component หลัก | FR | หมายเหตุ |
|---|---|---|---|---|
| Report dashboard | Supervisor เท่านั้น | booking, รายได้, คำขอคืนเงิน, คะแนนเฉลี่ย — filter ช่วงเวลา, กราฟสรุป | FR-9.1 | ใช้ dataviz pattern เดียวกับที่กำหนดไว้สำหรับกราฟ/สถิติ |
| Employee: ตาราง/กะทำงานของฉัน (Availability) | Employee เท่านั้น | ตั้งช่วงเวลาที่พร้อมให้บริการ | FR-10.1 | Deferred จาก Phase 1 — เพิ่งเริ่มทำ Phase นี้ |
| Supervisor: ดู availability ตอน assign | Supervisor เท่านั้น | ผนวกเข้าหน้า assign booking เดิม (Phase 2) แสดง availability ของ Employee ประกอบการเลือก | FR-10.1 | ยังไม่มี auto-conflict check (FR-10.2 เป็น phase ถัดไปอีก) |
| Admin: จัดการบัญชีผู้ใช้เต็มรูปแบบ | Admin เท่านั้น | list ทุก role, เปิด/ปิดใช้งานบัญชี | FR-3.1–3.3 | ขยายจาก Phase 1 ที่ทำแค่ Supervisor |
| Error pages เต็มชุด | ทุก role | 403 / 404 / 500 พร้อม UI ที่ตรงกับ design system | — | |
| Responsive/accessibility review | ทุกหน้า | ไม่ใช่หน้าใหม่ — รีวิว breakpoint มือถือ/แท็บเล็ต ของหน้าที่มีอยู่ทั้งหมด | — | รายละเอียดเช็คลิสต์อยู่ใน `08-qa-test-plan.md` |

### Phase 5 — Issue (ไม่ใช่ feature ใหม่)

ไม่มีหน้าจอใหม่ — เป็นรอบรีวิว UI/UX ของหน้าจอทั้งหมดข้างบนซ้ำ หา edge case ที่ขาด (error state, empty state,
loading state, responsive) ก่อน launch จริง — รายละเอียด task ดู `08-qa-test-plan.md`

## 2. หน้าที่ตั้งใจ "ไม่ใส่" ในเอกสารนี้ (เพื่อไม่เดา requirement เอง)

- หน้า "ลืมรหัสผ่าน" (forgot password) — **ไม่มีใน FR ปัจจุบัน** (FR-2.1 มีแค่เปลี่ยนรหัสผ่านตอน login อยู่แล้ว)
  ถ้าต้องการ flow นี้ ต้อง confirm กับ user ก่อนเพิ่มเป็น FR ใหม่ แล้วค่อยเพิ่มหน้าจอ
- หน้า "เกี่ยวกับเรา" (About) — ไม่มีใน FR ปัจจุบัน ไม่ใส่จนกว่าจะ confirm ว่าต้องการ (หน้า "ติดต่อเรา" มี FR-11
  แล้ว ไม่ถือเป็น open item อีกต่อไป — เพิ่มเป็น screen ใน Phase 2 ตามตารางด้านบน)
- ระบบทิป, ระบบแจ้งปัญหา — พักไว้ตาม `01-requirements.md` หัวข้อ "เลื่อนออกไปก่อน"

## 3. Open items ที่กระทบหน้าจอโดยตรง (ต้อง confirm ก่อน implement Phase ที่เกี่ยวข้อง)

| Open item | กระทบหน้าจอ |
|---|---|
| ข้อมูลพนักงานเพิ่มเติม (nickname/รูป/bio/specialty) | Employee profile, หน้าเลือกพนักงานตอนจอง/assign, หน้ารายละเอียดบริการ (ถ้าจะโชว์พนักงานแนะนำ) |
| ตัวเลขนโยบาย refund/cancel | หน้าขอยกเลิก/คืนเงิน (ข้อความ, validation ก่อนอนุมัติ) |
| รูปแบบใบเสร็จ walk-in | หน้าใบเสร็จ |
| Public เห็นคะแนนเฉลี่ยไหม | หน้ารายการ/รายละเอียดบริการ (จะมี rating badge หรือไม่) |

อ้างอิงรายการ assumption เต็มที่ `01-requirements.md` หัวข้อบนสุด — **ห้ามฟันธง copy ลงหน้าจอจริงก่อน
confirm กับ user**

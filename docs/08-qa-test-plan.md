# QA Test Plan — Srinaka

> เอกสารงานสำหรับ QA — test case/checklist ที่ใช้ตรวจรับแต่ละ Phase ก่อนเปลี่ยนสถานะ task จาก `DONE` เป็น
> `ACCEPTED` ใน `09-implementation-roadmap.md` อ้างอิง FR-1 ถึง FR-10 (`01-requirements.md`) และหน้าจอใน
> `04-screen-design.md`

## 0. ขอบเขต (Scope)

- **อยู่ในขอบเขต**: functional testing ตาม FR ทุกข้อ, frontend tech-specific checklist (Prerender/GSAP/
  รูปภาพ/brand — หัวข้อ 4), cross-cutting security/performance checklist (หัวข้อ 5), regression เมื่อ open
  item ถูก confirm (หัวข้อ 6)
- **ไม่อยู่ในขอบเขต**: ระบบทิป/แจ้งปัญหา (พักไว้), payment gateway จริง (ยังไม่ทำ Phase ปัจจุบัน),
  auto-conflict check ตารางพนักงาน (FR-10.2, phase ถัดไปอีก)
- เอกสารนี้คือ test ที่ทำ**ระหว่าง**แต่ละ Phase 1-4 ไม่ใช่แค่ตอนจบ — ความสัมพันธ์กับ Phase 5 ดูหัวข้อ 7

## 1. Test Environment & ข้อมูลทดสอบที่ต้องเตรียมก่อนเริ่ม

- Seed data ขั้นต่ำต่อ SIT environment: 1 Admin, 1 Supervisor, 2 Employee, 2 Customer (1 คน verify แล้ว, 1 คน
  ยังไม่ verify) เพื่อทดสอบทั้ง positive/negative case
- ⚠️ ต้องมีบัญชี Google/LINE ทดสอบ (test account) สำหรับ QA ใช้ผ่าน OAuth consent จริงตอน SIT — ไม่มี
  "test mode" แบบ SMS OTP อีกต่อไป เพราะเปลี่ยนไปใช้ Google/LINE login แทนแล้ว (ดู FR-1.7–FR-1.9)
- ต้องมีไฟล์ทดสอบสำหรับ upload: รูปภาพถูกต้อง (jpg/png ขนาดปกติ), ไฟล์ที่สวม extension ผิด (เช่น .exe
  เปลี่ยนนามสกุลเป็น .jpg), ไฟล์เกินขนาดที่กำหนด — ใช้ทดสอบ FileValidator ทุก Phase ที่มี upload (สลิป/รูป
  บริการ)

## 2. Test Case ตาม Phase

รูปแบบ Test ID: `TC-P{phase}-{ลำดับ}`

### Phase 1 — Foundation

| ID | Role | Scenario | ผลที่คาดหวัง | FR |
|---|---|---|---|---|
| TC-P1-01 | ทุก role | Login ด้วย username/password ถูกต้อง | ได้ JWT access+refresh, role/name ถูกต้อง | FR-1.1 |
| TC-P1-02 | ทุก role | Login ผิดรหัสผ่านซ้ำหลายครั้ง | โดน lockout ตาม `LoginAttemptService` pattern | FR-1.1 |
| TC-P1-03 | Admin | บัญชีถูกปิดใช้งาน (`active=false`) พยายาม login | ปฏิเสธ พร้อม error message ชัดเจน | FR-1.3 |
| TC-P1-04 | Admin | ดู login/logout log ย้อนหลัง | เห็น log ครบ, filter ได้ | FR-1.4 |
| TC-P1-05 | ทุก role | Logout | refresh token ถูก revoke — เอา token เก่ามาเรียก API ต้องไม่ผ่าน | FR-1.2 |
| TC-P1-06 | — (no auth) | เรียก public endpoint โดยไม่ส่ง JWT header | ผ่านได้ปกติ ไม่ 401 | FR-1.6 |
| TC-P1-10 | ทุก role | เปลี่ยนรหัสผ่าน กรอกรหัสเดิมผิด | ปฏิเสธ | FR-2.1 |
| TC-P1-11 | Customer | แก้เบอร์โทรใหม่ | บันทึกสำเร็จทันที ไม่กระทบ `verified_at` เลย (ไม่ใช้ SMS OTP แล้ว) | FR-2.2 |
| TC-P1-12 | Admin | เรียก API ข้อมูลธุรกิจ (เช่น booking/สินค้า) ด้วยบัญชี Admin | ต้องถูกปฏิเสธ (Admin ไม่มีสิทธิ์) — ทดสอบตรงที่ service layer ไม่ใช่แค่ UI ซ่อนปุ่ม | FR-3.4 |
| TC-P1-13 | ทุก role | ดู nav-menu | เห็นเฉพาะเมนูที่ตรงสิทธิ์ role ตัวเอง | — |

### Phase 2 — Catalog, Promotion, Booking

| ID | Role | Scenario | ผลที่คาดหวัง | FR |
|---|---|---|---|---|
| TC-P2-01 | — (no auth) | ดู catalog สินค้า/บริการ | เห็นครบ, โหลดเร็ว, ไม่ต้อง login | FR-4.3 |
| TC-P2-02 | Supervisor | CRUD สินค้า/บริการ | สร้าง/แก้ไข/ลบสำเร็จ | FR-4.1 |
| TC-P2-03 | Employee | เปิดหน้าจัดการสินค้า/บริการ | เห็น list ได้ (read-only) แต่แก้ไข/ลบไม่ได้ | FR-4.4 |
| TC-P2-04 | — (no auth) | ดูโปรโมชั่น active | เห็นเฉพาะที่ active, ไม่เห็นที่หมดอายุ/ยังไม่เริ่ม | FR-5.2 |
| TC-P2-05 | Customer (verified) | จองนัดผ่าน booking flow | สร้าง booking สถานะ `PENDING_PAYMENT` สำเร็จ | FR-6.1 |
| TC-P2-06 | Customer (**ยังไม่** verify) | พยายามจองนัด | ถูกกันตั้งแต่ backend (เรียก API ตรง ไม่ผ่าน UI) ไม่ใช่แค่ frontend ซ่อนปุ่ม | FR-6.1, roadmap BE note |
| TC-P2-07 | Supervisor | มอบหมาย Employee ให้ booking | บันทึกสำเร็จ, booking ปรากฏในหน้า Employee ที่ถูก assign | FR-6.3 |
| TC-P2-08 | Employee | ดู booking ของตัวเอง | เห็นเฉพาะที่ตัวเองถูก assign เท่านั้น ไม่เห็นของ Employee คนอื่น | FR-6.4 |
| TC-P2-09 | Employee | mark booking เป็น `COMPLETED` | อัปเดตสถานะสำเร็จ | FR-6.4 |
| TC-P2-10 | Supervisor/Employee | เปิดหน้า booking list ที่ join customer/service/employee | ตรวจ SQL log ต้องไม่มี N+1 (ใช้ `JOIN FETCH`/`@EntityGraph`) | CLAUDE.md — performance |
| TC-P2-11 | Customer | สมัครสมาชิกแบบกรอกเอง (username/password/ชื่อ/เบอร์โทร) | บัญชีสร้างสำเร็จ, role=CUSTOMER, `verified_at` เป็น null | FR-1.7 |
| TC-P2-12 | Customer | สมัครสมาชิกด้วย username ซ้ำ | ปฏิเสธ พร้อม error ชัดเจน | FR-3.3 |
| TC-P2-13 | Customer | สมัครสมาชิกผ่าน Google | บัญชีสร้างสำเร็จ, `google_user_id` ถูกบันทึก, `verified_at` ถูกตั้งค่าทันที | FR-1.7, FR-1.8 |
| TC-P2-14 | Customer | สมัครสมาชิกผ่าน LINE | บัญชีสร้างสำเร็จ, `line_user_id` ถูกบันทึก, `verified_at` ถูกตั้งค่าทันที | FR-1.7, FR-1.8 |
| TC-P2-15 | Customer (สมัครแบบกรอกเอง, ยังไม่ verified) | login แล้วไปลิงก์บัญชี Google หรือ LINE จากหน้า Profile | ผูก `google_user_id`/`line_user_id` เข้ากับ user เดิม, `verified_at` ถูกตั้งค่าทันที, ไม่มี user ซ้ำเกิดขึ้น | FR-1.9 |
| TC-P2-16 | Public | ดูหน้า "ติดต่อเรา" | เห็นข้อมูลร้านที่ Supervisor ตั้งไว้, ไม่ต้อง login | FR-11.2 |
| TC-P2-17 | Supervisor | แก้ไขข้อมูลติดต่อร้าน | บันทึกสำเร็จ, หน้า public เห็นข้อมูลใหม่ทันที | FR-11.1 |

### Phase 3 — Payment (Slip/Cash), Walk-in, Cancel/Refund, Rating

| ID | Role | Scenario | ผลที่คาดหวัง | FR |
|---|---|---|---|---|
| TC-P3-01 | Customer | อัปโหลดสลิปไฟล์รูปถูกต้อง | อัปโหลดสำเร็จ ผูกกับ booking | FR-7.1 |
| TC-P3-02 | Customer | อัปโหลดไฟล์สวม extension ผิด (เนื้อไฟล์ไม่ใช่รูปจริง) | ถูกปฏิเสธ — ยืนยันว่า validate จาก content-type header จริง ไม่ใช่แค่นามสกุลไฟล์ | FR-7.1, CLAUDE.md |
| TC-P3-03 | Customer | อัปโหลดไฟล์เกินขนาดที่กำหนด | ถูกปฏิเสธ | FR-7.1 |
| TC-P3-04 | Supervisor/Employee | ตรวจสลิป → confirm | booking เปลี่ยนเป็น `CONFIRMED`, payment record ถูกต้อง (จำนวนเงิน/ผู้ตรวจ/เวลา) | FR-7.2, FR-7.3 |
| TC-P3-05 | Supervisor/Employee | ตรวจสลิป → reject | booking ไม่เปลี่ยนเป็น `CONFIRMED`, มี state ให้ลูกค้าเห็นว่าถูก reject | FR-7.2 |
| TC-P3-06 | Customer | ขอยกเลิก booking ก่อนถึงเวลานัด | สร้างคำขอคืนเงินสถานะ `REQUESTED` | FR-7.4 |
| TC-P3-07 | Supervisor | อนุมัติคำขอคืนเงิน + mark `REFUNDED` | booking เปลี่ยนเป็น `CANCELLED` | FR-7.5, FR-7.6 |
| TC-P3-08 | Supervisor | ปฏิเสธคำขอคืนเงิน | booking คงสถานะเดิม ลูกค้าเห็นผลปฏิเสธ | FR-7.5 |
| TC-P3-09 | Employee/Supervisor | สร้าง walk-in booking (ไม่มี Customer login) | สร้างสำเร็จ `channel=WALK_IN`, กรอกชื่อลูกค้าแบบ optional ได้ | FR-7.8 |
| TC-P3-10 | Employee | รับชำระ walk-in ด้วยเงินสด | ข้าม `PENDING_PAYMENT` ไป `CONFIRMED` ทันที, `payments.method=CASH` | FR-7.9, FR-7.11 |
| TC-P3-11 | Employee | คีย์ส่วนลดตอนขาย walk-in | บันทึก `discount_amount` ถูกต้อง + audit `created_by` เป็นคนขายจริง | FR-7.10 |
| TC-P3-12 | Employee | พิมพ์ใบเสร็จหลัง walk-in sale | ข้อมูลในใบเสร็จตรงกับรายการที่ขาย (บริการ, ราคา, ส่วนลด, ผู้ขาย) | FR-7.12 |
| TC-P3-13 | — (regression) | ทำ walk-in sale ที่ `services.type=PRODUCT` | ยืนยันว่า**ไม่มี**การตัด stock ใด ๆ (พฤติกรรมตั้งใจ ไม่ใช่บั๊ก) | FR-7.13 |
| TC-P3-14 | Customer | ให้คะแนนก่อน booking ถึงสถานะ `COMPLETED` | ต้องถูกกัน ให้คะแนนได้เฉพาะหลัง `COMPLETED` เท่านั้น | FR-8.1 |
| TC-P3-15 | Customer | ให้คะแนนร้าน + ให้คะแนน Employee ที่ให้บริการ | บันทึกแยกกันถูกต้อง ผูกกับ booking/employee ที่ถูกต้อง | FR-8.1, FR-8.2 |

### Phase 4 — Report, Employee Schedule, Admin, Release

| ID | Role | Scenario | ผลที่คาดหวัง | FR |
|---|---|---|---|---|
| TC-P4-01 | Supervisor | Report filter ตามช่วงเวลา | ตัวเลข booking/รายได้/คำขอคืนเงิน/คะแนนเฉลี่ย ตรงกับข้อมูลจริงในช่วงที่เลือก | FR-9.1 |
| TC-P4-02 | Employee | ตั้งช่วงเวลาที่พร้อมให้บริการ (availability) | บันทึกสำเร็จ | FR-10.1 |
| TC-P4-03 | Supervisor | ดู availability ของ Employee ตอน assign booking | แสดงถูกต้องตามที่ Employee ตั้งไว้ | FR-10.1 |
| TC-P4-04 | Admin | จัดการบัญชีผู้ใช้ทุก role (สร้าง/ปิดใช้งาน) | ทำงานถูกต้องตามสิทธิ์ Admin, ยังคงไม่เห็นข้อมูลธุรกิจ | FR-3.1–3.4 |
| TC-P4-05 | ทุก role | เข้า route ที่ไม่มีสิทธิ์ / route ไม่มีอยู่จริง / server error | เห็นหน้า 403/404/500 ที่ถูกต้อง ไม่ใช่หน้าขาวหรือ error ดิบ | — |
| TC-P4-06 | ทุก role | เปิดทุกหน้าจอบนมือถือ/แท็บเล็ต | layout ไม่พัง, ใช้งานได้ครบ | NFR |
| TC-P4-07 | — | Production build + deploy smoke test | ขึ้น Render+Neon+Cloudflare Pages ได้จริง, `PORT` env var ถูกอ่านถูกต้อง | CLAUDE.md — บทเรียน Share Money |

## 3. State ที่ต้องเช็คทุกหน้าที่มี list/table (ไม่ใช่แค่ happy path)

สำหรับทุกหน้าใน `04-screen-design.md` ที่มี list/table (catalog, booking list, report, log ฯลฯ) ต้องเช็ค 3
state นี้เพิ่มจาก test case ปกติ:

- **Loading** — แสดง skeleton/spinner ระหว่างรอข้อมูล ไม่ใช่หน้าว่างเปล่า
- **Empty** — ไม่มีข้อมูล (เช่น Customer ที่ยังไม่เคยจอง) แสดงข้อความที่เข้าใจง่าย ไม่ใช่ตารางว่าง/error
- **Error** — API fail (เช่น 500/timeout) แสดง error state ที่มีทางออก (ปุ่มลองใหม่) ไม่ใช่หน้าขาว/crash

## 4. Frontend Tech-specific Checklist (ตาม `05-tech-stack.md`)

เช็คเพิ่มเฉพาะจุดที่เกี่ยวกับ stack ที่เพิ่งตัดสินใจ (Prerender, GSAP, รูปภาพ, brand asset) — เริ่มมีผลตั้งแต่
FE Phase 2 ที่เริ่มมีหน้า public จริง:

- [ ] **Prerender (SSG)**: ปิด JavaScript ใน browser แล้วเปิดหน้า public (หน้าแรก/catalog/รายละเอียดบริการ/
      โปรโมชั่น) ต้องยังเห็นเนื้อหาจริงใน HTML (ไม่ใช่หน้าว่าง) — เช็ค SEO meta tag (title/description)ต่อ
      หน้าด้วยว่า render มาถูกต้องตั้งแต่ HTML แรก
- [ ] **หน้าที่ login** (booking/dashboard) ต้องยังเป็น CSR ปกติ ไม่ถูกดึงเข้า prerender โดยไม่ตั้งใจ (ข้อมูล
      เฉพาะ user ต้อง fresh ทุกครั้ง ไม่ใช่ HTML ที่ build ไว้ล่วงหน้า)
- [ ] **GSAP/ScrollTrigger**: เปิด OS setting "ลดการเคลื่อนไหว" (`prefers-reduced-motion: reduce`) แล้วเช็คว่า
      animation ถูกปิด/ลดลงตาม ไม่บังคับ user ที่ sensitive กับ motion
- [ ] **GSAP performance**: หน้า public ที่มี animation เยอะ (โดยเฉพาะหน้าแรก) scroll ลื่นบนมือถือจริง ไม่ใช่
      แค่ desktop, ไม่ค้าง/กระตุก
- [ ] **รูปภาพ**: เช็คว่า Cloudinary ส่ง AVIF/WebP ตาม browser ที่รองรับจริง และ fallback เป็น JPEG ได้ถูกต้อง
      บน browser ที่ไม่รองรับ (บังคับผ่าน dev tool)
- [ ] **Brand consistency**: โลโก้แต่ละจุด (favicon/sidebar/header/login/dark mode) ใช้ไฟล์ variant ถูกต้อง
      ตามตารางใน `05-tech-stack.md` (symbol เท่านั้นที่ favicon/sidebar, full logo ที่ header/login), เว้น
      พื้นที่รอบโลโก้อย่างน้อย 10% ของความสูง
- [ ] **Theme ขาว/เขียว**: สลับ `ThemeService` (`white`/`green`) แล้วสีเปลี่ยนถูกต้องครบทุกหน้า (dashboard +
      storefront ใช้ token ชุดเดียวกัน ไม่เพี้ยนสีกันระหว่าง 2 shell), ค่าที่เลือกไว้ต้องจำข้าม session
      (`localStorage`) ไม่ใช่สีฟ้า default เดิมของ Share Money หลุดมาให้เห็น
- [ ] **i18n ไทย/อังกฤษ**: สลับภาษาแล้ว UI เปลี่ยนครบทุกหน้า ไม่มี translation key ตกหล่น (ขึ้นเป็น key ดิบ
      แทนคำแปล) — เช็คทั้งหน้า public และ dashboard
- [ ] **Lighthouse** (หรือเทียบเท่า) หน้า public: Performance/SEO/Accessibility อยู่ในเกณฑ์ยอมรับได้ —
      โดยเฉพาะหลังใส่ GSAP + รูปภาพเยอะ อาจกระทบ Performance score ต้องเช็คซ้ำ

## 5. Cross-cutting Security & Performance Checklist (เช็คซ้ำทุก Phase ก่อนปิด)

- [ ] Ownership/role check อยู่ที่ **service layer จริง** ไม่ใช่พึ่ง `@PreAuthorize` อย่างเดียว — ทดสอบโดยยิง
      API ตรงด้วย Postman/curl ข้าม role (เช่น Employee เรียก API approve refund ของ Supervisor)
- [ ] List API ที่ join หลายตาราง (booking+customer+service+employee ฯลฯ) ไม่มี N+1 — เปิด SQL log นับจำนวน
      query ต่อ 1 request
- [ ] Public vs Authenticated endpoint แยกถูกต้องตาม Security config — เรียก endpoint ที่ควร auth โดยไม่ส่ง
      JWT ต้องได้ 401/403 เสมอ ไม่มี endpoint หลุดเป็น public โดยไม่ตั้งใจ
- [ ] Upload (สลิป, รูปบริการ) validate content-type จาก **header ไฟล์จริง** (ลองปลอม extension) + จำกัดขนาด
      ไฟล์ ทุกจุดที่มี upload
- [ ] JWT expiry + refresh flow ทำงานถูกต้อง, token ถูก revoke จริงหลัง logout
- [ ] Login lockout ทำงานตามจำนวนครั้งที่กำหนด และปลดล็อกถูกต้องตามเวลา
- [ ] Secret (Cloudinary key, JWT secret ฯลฯ) ไม่ hardcode ใน `application.yml` — grep source ก่อน deploy
      จริงทุกครั้ง (บทเรียนจาก Share Money)

## 6. Regression Checklist เมื่อ Open Item ถูก confirm/เปลี่ยนแปลง

รายการ assumption เต็มอยู่ใน `01-requirements.md` หัวข้อบนสุด — เมื่อข้อไหนถูก confirm/เปลี่ยน ต้อง retest
ตามนี้:

| Open item เปลี่ยน | ต้อง retest |
|---|---|
| นโยบาย refund/cancel มีตัวเลขชัดเจน | TC-P3-06 ถึง TC-P3-08 (เงื่อนไข/จำนวนเงินคืนต้องตรงนโยบายใหม่) |
| แนบสลิป → payment gateway จริง | TC-P3-01 ถึง TC-P3-08 ทั้งหมด (payment+refund flow เปลี่ยนจาก manual เป็น automated) |
| รูปแบบใบเสร็จ (browser print → backend PDF) | TC-P3-12 |

## 7. Defect Severity (ใช้ระบุความรุนแรงตอน log bug)

| ระดับ | คำอธิบาย |
|---|---|
| Critical | ข้อมูลผิด/รั่วไหล, security hole (เช่น ownership check หลุด), ระบบใช้งานไม่ได้ทั้ง flow |
| High | feature หลักใช้งานไม่ได้ แต่มี workaround หรือกระทบเฉพาะ role/สถานการณ์เดียว |
| Medium | UX ผิดปกติ/ข้อมูลแสดงคลาดเคลื่อนเล็กน้อย ไม่กระทบ business logic |
| Low | UI/wording ไม่สมบูรณ์ ไม่กระทบการใช้งาน |

## 8. ความสัมพันธ์กับ "Phase 5 — Issue" ใน roadmap

เอกสารนี้ (test case หัวข้อ 2 + checklist หัวข้อ 3-5) คือชุดที่ QA รันระหว่างแต่ละ Phase 1-4 เพื่อตัดสิน
`DONE` → `ACCEPTED` ส่วน **Phase 5** ใน `09-implementation-roadmap.md` คือรอบสุดท้ายที่ทีม dev+QA รัน
test case **ทั้งหมด**ข้างบนซ้ำอีกครั้งแบบ full regression พร้อม exploratory testing เพิ่มเติม (หาสิ่งที่ไม่มี
test case รองรับ) ก่อน launch จริง — ไม่ใช่ QA process ชุดใหม่ แต่เป็นการทำหัวข้อ 2-5 ซ้ำให้ครบทั้งระบบ

## 9. Phase Sign-off

Task จะ mark เป็น `ACCEPTED` ได้ก็ต่อเมื่อ QA ผ่านทุก test case ในขอบเขต Phase นั้น (หัวข้อ 2) **และ**
checklist ที่เกี่ยวข้อง (หัวข้อ 4 Frontend tech-specific, หัวข้อ 5 cross-cutting security/performance) ครบ —
ไม่ใช่แค่ acceptance criteria แบบสรุปสั้นที่อยู่ใน `09-implementation-roadmap.md`

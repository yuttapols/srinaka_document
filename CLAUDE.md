# Srinaka — Documentation Repo

เอกสาร design/planning สำหรับโปรเจคร้านสปา/นวด (ชื่อโปรเจค "Srinaka") ระบบใหม่ ยังไม่มี repo backend/frontend
จริง — repo นี้มีแค่เอกสารวางแผนล่วงหน้าก่อนเริ่มเขียนโค้ด

## บริบท

โปรเจคนี้ลอก**โครงสร้างกลาง** (ไม่ใช่ business logic) มาจากโปรเจคก่อนหน้า **Share Money**
(`D:\GIT\BACK-END\share_money_backend`, เอกสารต้นทางที่ `D:\GIT\DOCUMENT\share_money_document\docs`) — เพราะ
โครงสร้าง auth/RBAC/response pattern/deploy พิสูจน์แล้วว่าใช้งานได้จริง (deploy ขึ้น Render+Neon+Cloudflare
Pages สำเร็จแล้ว) เอามาเป็นฐานแทนที่จะออกแบบใหม่ทั้งหมด

**ธุรกิจเปลี่ยนไปแล้วระหว่างคุยกัน** (ลำดับ: ลอกโครงสร้าง Share Money → คิดจะทำร้านโรตี → เปลี่ยนเป็นร้าน
สปา/นวด) — ตอนนี้ธุรกิจคือ **ร้านสปา/นวด เน้นเว็บสวยงาม** ให้ลูกค้าดูสินค้า/บริการ+ราคา+โปรโมชั่น, สมัคร
สมาชิก 2 ทาง (กรอกเอง/LINE) + ยืนยันตัวตนผ่าน LINE OAuth (**ตัดสินใจแล้ว** ไม่ใช้ SMS OTP อีกต่อไป —
เคยพิจารณา Google login ด้วยแต่ตัดออกเพราะติดขั้นตอน billing verification ของ Google Cloud — ดู
FR-1.7–FR-1.9), จองนัด+จ่ายเงิน
(แนบสลิป)+ยกเลิก/ขอคืนเงิน, ให้คะแนน, **และ Employee ขายบริการหน้าร้านให้ลูกค้า walk-in เอง (POS-lite: รับ
เงินสด, คีย์ส่วนลด, พิมพ์ใบเสร็จ — ไม่มีระบบตัด stock)** (module เต็มดู FR-1 ถึง FR-10 ใน `01-requirements.md`)
— **ไม่มี guest booking แบบออนไลน์** (ต้อง login+verify ตัวตนก่อนจอง) แต่ walk-in หน้าร้านไม่ต้องมี Customer
account เลย **ระบบทิป/แจ้งปัญหาถูกพักไว้ก่อน** ("อาจจะยังไม่ต้องทำ") ไม่อยู่ใน roadmap ปัจจุบัน

## สถานะปัจจุบัน

**ออกแบบครบ 5 Phase แล้ว** (requirement เต็มอยู่ใน `01-requirements.md`, roadmap ใน
`09-implementation-roadmap.md`) — **Backend Phase 1 (Foundation: auth ทุก role, RBAC, profile, nav-menu,
deploy setup) ทำเสร็จแล้ว** ที่ `D:\GIT\BACK-END\srinaka_backend` (ดูสถานะจริงจาก git log ของ repo นั้น ไม่ใช่
จากไฟล์นี้) — Customer สมัครสมาชิก+verify ตัวตน (LINE) เป็นงาน **Phase 2** ไม่ใช่ Phase 1 **Phase 5
ไม่ใช่ feature ใหม่** เป็น phase หาบั๊ก/ช่องโหว่/ช่องว่างทั้งระบบของทีมเองก่อน launch จริง (ต่างจาก feature
"แจ้งปัญหา" ของลูกค้าที่ถูกพักไว้ — คนละเรื่องกัน อย่าสับสน)

`01-requirements.md` มี assumption/จุดที่ยังไม่ confirm หลายจุด (payment gateway ถ้าจะทำต่อจาก slip ทีหลัง,
ตัวเลขนโยบาย refund ที่ชัดเจน ฯลฯ) — **ห้ามถือว่าเป็นข้อสรุปจริง** ต้องเช็คกับ user อีกทีก่อน implement Phase
ที่เกี่ยวข้อง (เรื่อง SMS OTP vs LINE login **ตัดสินใจแล้ว** ไม่ใช่ open item อีกต่อไป — ดู FR-1.7)

**อย่าเดา business requirement เองแล้วเขียนเป็นเอกสารทันที** — รอบก่อนหน้าเคยเขียน requirement จาก quick-poll
คำตอบสั้น ๆ ไปก่อน แล้ว user บอกว่ายังไม่ได้เล่า business จริงเลย ต้องรื้อเขียนใหม่ทั้งหมด — ให้ฟัง user เล่า
เองให้ครบก่อน ค่อย formalize เป็นเอกสาร

## เอกสารในนี้

- [README.md](README.md) — index + สรุประบบ
- [docs/01-requirements.md](docs/01-requirements.md) — requirement เต็ม, FR ทุกหมวด, NFR, จุดที่ยังไม่ตัดสินใจ
- [docs/02-roles-and-structure.md](docs/02-roles-and-structure.md) — 4 actor (Admin/Supervisor/Employee/
  Customer), สิทธิ์ระดับกว้าง ๆ, phase plan, จุดที่ยังไม่ตัดสินใจ
- [docs/05-tech-stack.md](docs/05-tech-stack.md) — tech stack เต็ม (Angular + Spring Boot), โครงสร้าง
  package แนะนำ
- [docs/03-database-design.md](docs/03-database-design.md) — ER diagram, ตาราง/คอลัมน์/index ครบทุก Phase,
  คำแนะนำ implement ให้ไวไม่เสียของ (แบ่ง Flyway ตาม Phase จริง ไม่สร้างทุกตารางทีเดียว)
- [docs/04-screen-design.md](docs/04-screen-design.md) — screen inventory ฝั่ง Frontend ทุก Phase (1-4):
  หน้าจอ, role ที่เข้าถึงได้, เนื้อหาหลัก, ผูกกับ FR, open item ที่กระทบหน้าจอ (ยังไม่ใช่ wireframe จริง)
- [docs/08-qa-test-plan.md](docs/08-qa-test-plan.md) — QA test case/checklist ทุก Phase (1-4),
  cross-cutting security/performance checklist, defect severity, เกณฑ์ sign-off `DONE`→`ACCEPTED`
- [docs/09-implementation-roadmap.md](docs/09-implementation-roadmap.md) — แผนงาน FE/BE ฝั่งละ 5 Phase

## Role (สรุปย่อ — รายละเอียดดู docs/02-roles-and-structure.md)

| Role | บทบาท | login |
|---|---|---|
| Admin | ดูแลระบบ/infra เท่านั้น ไม่ยุ่งข้อมูลธุรกิจ (เหมือน pattern ที่ Share Money กัน ADMIN ออกจากข้อมูลหนี้) | ต้อง login |
| Supervisor | เจ้าของ/ผู้ดูแลธุรกิจ จัดการสินค้า/บริการ/โปรโมชั่น, ตรวจสลิป, อนุมัติ refund, ดู report | ต้อง login |
| Employee | พนักงานร้าน ถูกให้คะแนน | ต้อง login |
| Customer | ลูกค้า | browse ได้โดยไม่ login — **แต่ต้องสมัคร (กรอกเอง/LINE) + verify ตัวตนผ่าน LINE ก่อนถึงจะจองได้ ไม่มี guest booking** |

## ⚠️ ระวังคำว่า "เมนู" ชนกัน 2 ความหมาย

- **Nav menu** (`menu_items`/`menu_permissions` ลอกจาก Share Money) = คุม sidebar ของ Admin/Supervisor
  dashboard เท่านั้น
- **เมนูสินค้า/บริการสปา** (product/service catalog ที่ลูกค้าดู) = business domain ใหม่ทั้งหมด คนละเรื่องกับ
  nav menu เด็ดขาด

## บทเรียนจาก Share Money ที่ต้องกันไว้ตั้งแต่แรก

- **ห้าม hardcode secret ใน `application.yml`** — Share Money เคย hardcode Cloudinary key จริงแล้ว push ขึ้น
  GitHub มาแล้ว (leak เข้า git history) ต้องใช้ env var ล้วนตั้งแต่ commit แรก
- **Render free tier ต้องอ่าน `PORT` env var** สำหรับ `server.port` (Render inject เอง ไม่ใช่ fixed 8080)
- **Postgres `IDENTITY` column ห้าม hardcode id ใน seed migration** — ถ้า id นั้นมีโอกาสถูก auto-generate
  ไปแล้วจาก flow อื่น (เช่น สร้างผ่าน UI/API) จะชนกัน ให้ insert แบบไม่ระบุ id แล้ว subquery หา id กลับมาใช้แทน
- **ระบบนี้ต้องรองรับ public/guest endpoint ตั้งแต่ต้น** (Customer ไม่ login ได้) — Share Money ออกแบบไว้ว่า
  ทุก endpoint ต้อง auth หมด เอา pattern นั้นมาตรง ๆ ไม่ได้ ต้องแยก public vs authenticated ตั้งแต่ Security
  config

## Code Style & Quality (บังคับใช้ทุกครั้งที่เขียนโค้ดจริง)

- **Format โค้ดให้เรียบร้อยเสมอ** ก่อน commit — ตาม convention เดียวกับที่ลอกมาจาก Share Money (ตั้งชื่อสื่อ
  ความหมาย, 1 class ทำหน้าที่เดียว, ไม่เขียน comment เกินจำเป็น ยกเว้นอธิบาย constraint/เหตุผลที่ไม่ obvious)
- **Security และ performance คือสิ่งที่ต้องคำนึงถึงเป็นหลักทุกครั้ง** ไม่ใช่แค่ "ทำให้ทำงานได้" — ก่อน merge
  โค้ดทุกจุดต้องเช็ค:
  - ownership/role check ที่ service layer เสมอ (ไม่พึ่ง `@PreAuthorize` อย่างเดียว — ตาม pattern Share Money)
  - query ที่ join entity หลายตัว (list booking พร้อม customer/service/employee ฯลฯ) ต้องกัน N+1 ด้วย
    `JOIN FETCH`/`@EntityGraph` เสมอ ตามที่ระบุไว้ใน `03-database-design.md` หัวข้อ 3
  - ไฟล์ที่ลูกค้า/พนักงานอัปโหลด (สลิป, รูปบริการ) validate content-type จาก header จริง + ขนาดไฟล์เสมอ
  - public endpoint (browse สินค้า/บริการ) กับ authenticated endpoint ต้องแยกชัดตั้งแต่ Security config —
    ห้ามพลาดเผลอเปิด endpoint ที่ควร auth ให้เป็น public
- **ห้ามเขียนโค้ดซ้ำซ้อนเด็ดขาด** — ก่อนเขียน logic ใหม่ ให้เช็คก่อนว่ามีของกลาง/เคยเขียนไว้แล้วหรือยัง
  (`common/` ที่ copy มาจาก Share Money, หรือ module อื่นที่เขียนไปแล้วในโปรเจคนี้เอง) ถ้ามีอยู่แล้ว **ต้องแจ้ง
  ผู้ใช้ว่าเจอของซ้ำ แล้วไปเรียกใช้ตัวกลางแทนการ copy-paste หรือเขียนใหม่** — ถ้า logic ไหนถูกใช้ซ้ำเกิน 1
  module ให้ย้ายเข้า `common/` ทันที ตาม pattern เดิมของ Share Money (ดูตาราง "Shared/Common Code" ใน
  `share_money_backend/CLAUDE.md` เป็นตัวอย่าง)

## เมื่อจะเริ่มเขียนโค้ดจริง

Repo แยกต่างหาก (backend/frontend) **ถูก `git init` ไว้แล้ว** (ยังไม่มีโค้ดจริง แค่เตรียม repo เปล่าไว้):
- Backend: `D:\GIT\BACK-END\srinaka_backend`
- Frontend: `D:\GIT\FRONT-END\srinaka_frontend` — มี `CLAUDE.md` เขียนไว้ล่วงหน้าแล้ว (สรุปจาก docs ชุดนี้)

ตัดสินใจแล้วว่าทั้ง backend และ frontend **copy โครงสร้างจากโปรเจค Share Money มาเป็นฐาน ดีกว่าสร้างใหม่
ทั้งหมด** (ประหยัด token กว่ามาก เพราะ auth/RBAC/error-handling/Cloudinary/i18n/theme/deploy เป็นโค้ดที่ผ่าน
การทดสอบจริงมาแล้ว ทั้ง build/deploy สำเร็จ และเจอ-แก้บั๊กมาแล้วหลายจุด — เขียนใหม่มีแต่เสี่ยงเจอปัญหาเดิมซ้ำโดย
เปล่าประโยชน์) — Backend copy จาก `D:\GIT\BACK-END\share_money_backend`, **Frontend copy จาก
`D:\GIT\FRONT-END\share_money_frontend`** (repo Angular ที่ scaffold จริงแล้ว มี guards/interceptors/i18n
(`@ngx-translate`)/theme service/shared component library ของตัวเองพร้อมใช้ — แผน reuse เต็มอยู่ใน
`docs/05-tech-stack.md` หัวข้อ "Frontend Reuse Plan" และสรุปไว้แล้วใน `srinaka_frontend/CLAUDE.md`)

### Backend

#### ✅ Copy มาได้ตรง ๆ (แค่ rename package `com.sharemoney` → `com.srinaka`)

- `pom.xml` (แก้ groupId/artifactId/name), `Dockerfile`, `.dockerignore`, `render.yaml`, `docker-compose.yml`
  (แก้ชื่อ service/db เป็น srinaka), `application.yml`/`application-dev.yml`/`application-prod.yml`
- `src/main/java/com/sharemoney/common/` ทั้งโมดูล (`audit`, `config`, `domain`, `entity`, `error`,
  `response`, `security`, `storage`) — `ApiResponse`, `ErrorCode`, `GlobalExceptionHandler`, `BaseEntity`,
  `SecurityUtils`, `AuditLogService`, `FileStorageService`+`CloudinaryFileStorageService`, `FileValidator`,
  `JwtSecretGuard`
- `src/main/java/com/sharemoney/auth/` ทั้งโมดูล (login/refresh/logout/JWT, `LoginAttemptService`) — **แต่ต้อง
  แก้เพิ่ม** ให้รองรับ public/guest endpoint (ดูหัวข้อ "บทเรียน" ด้านล่าง)
- `src/main/java/com/sharemoney/profile/` ทั้งโมดูล — ตรงกับ FR-2 (Profile) พอดีอยู่แล้ว
- `src/main/java/com/sharemoney/menu/` ทั้งโมดูล — nav-menu permission system
- `src/main/java/com/sharemoney/admin/` **เฉพาะส่วน login log** (`LoginLogController`,
  `LoginLogQueryService`, `LoginLogResponse`) — ไม่เอา installment choices/migration
- Flyway migration ของ `users`/`refresh_tokens`/`login_logs`/`menu_items`/`menu_permissions` เป็นจุดเริ่ม
  (ต้องแก้ schema `users` ตามหัวข้อถัดไป ไม่ copy ตรง ๆ ทั้งหมด)

#### ❌ ไม่เอา (ลบทิ้ง — เป็น business logic เฉพาะของ Share Money)

`debt/`, `slip/`, `document/`, `report/`, `migration/` (`LegacyDataMigrationTool`) ทั้งโมดูล,
`admin/controller/InstallmentChoiceController` + ของที่เกี่ยวข้อง, `scripts/migrate-legacy-data.sh`

#### ⚠️ Copy มาแล้วต้องแก้ ไม่ใช่ copy ตรง ๆ

- **`user/` module** — `User` entity เดิมมี `creditor_id` (ผูก debtor→creditor) ไม่เกี่ยวกับ Srinaka เลย ต้อง
  ตัดออก, เปลี่ยน role enum เป็น `ADMIN`/`SUPERVISOR`/`EMPLOYEE`/`CUSTOMER`, เพิ่ม `phone`/`google_user_id`/
  `line_user_id`/`verified_at` ตาม `03-database-design.md` ตาราง `users` (ไม่มี `phone_verified_at`/OTP แล้ว)
- **Security config** (`auth/security/SecurityConfig.java` หรือเทียบเท่า) — ของเดิมบังคับ login ทุก endpoint
  ห้าม copy pattern นี้ตรง ๆ ต้องแยก public (`GET` สินค้า/บริการ/โปรโมชั่น) ออกจาก authenticated endpoint
  ตั้งแต่ต้น
- ทุกไฟล์ที่ copy มา — เปลี่ยน `package com.sharemoney.*` → `package com.srinaka.*` และ import ที่เกี่ยวข้อง
  ทั้งหมด (มองหาด้วย find-and-replace ทั้ง repo หลัง copy)

### Frontend

แผน reuse เต็ม (✅ copy ตรง / ❌ ไม่เอา / ⚠️ ต้องแก้) อยู่ที่ `docs/05-tech-stack.md` หัวข้อ "Frontend Reuse
Plan" — สรุปไว้แล้วอีกรอบใน `D:\GIT\FRONT-END\srinaka_frontend\CLAUDE.md` (เขียนไว้ล่วงหน้าตั้งแต่ยัง
ไม่ scaffold) สรุปสั้นที่สุด:

- ✅ copy ตรง: `core/guards/{auth,role}.guard.ts`, `core/interceptors/*`, `core/services/theme.service.ts`
  (relabel เป็น `white`/`green`), `shared/components/*` (design system เดิม), `assets/i18n/{th,en}.json`
  (mechanism), config files
- ❌ ไม่เอา: `features/{debts,debtors,bank-accounts,documents}/` และ model/service ที่ผูก debt/migration/
  bank-account เดิม
- ⚠️ ต้องแก้: `guest.guard.ts` (ต้องรองรับ public browse), `user.model.ts` (role enum + phone field ใหม่),
  `auth.service.ts` (เพิ่ม flow สมัคร 2 ทาง กรอกเอง/LINE + ลิงก์บัญชีทีหลัง — ไม่ใช่ OTP),
  `app-shell/` (แยก public shell กับ dashboard shell), `features/slips/`
  (เอา pattern อัปโหลด/ตรวจสลิปมาอ้างอิง แต่ business logic เขียนใหม่ผูกกับ booking/payment)
- **ไม่ใช้ PrimeNG/daisyUI/Bootstrap** — เคยพิจารณาแล้วแต่ยกเลิก เพราะ `share_money_frontend` มี shared
  component library ของตัวเองอยู่แล้ว (`app-button`/`app-input`/`app-card`/`app-dialog` ฯลฯ) การเพิ่ม UI
  library ใหม่ซ้อนเป็นการเขียนซ้ำซ้อนโดยไม่จำเป็น

### หลัง scaffold เสร็จ

เขียน `CLAUDE.md` ของ repo backend/frontend ใหม่อีกรอบให้ตรงโครงสร้างจริงหลัง scaffold (ตอนนี้เขียนไว้
ล่วงหน้าจาก docs ชุดนี้เท่านั้น) โดยสรุปเนื้อหาจากไฟล์นี้ + เอกสารใน `docs/` ให้ครบ (เหมือนที่
`share_money_backend/CLAUDE.md` และ `share_money_frontend/CLAUDE.md` สรุปจาก `share_money_document/docs`)
และอัปเดตไฟล์นี้ให้ชี้กลับไปที่ repo backend/frontend จริงด้วย

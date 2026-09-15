# Tech Stack — Srinaka (Angular + Spring Boot)

เหมือนกับ tech stack ของ Share Money แทบทั้งหมด (โครงสร้างกลาง พิสูจน์แล้วว่าใช้งานได้จริง) ต่างกันแค่ชื่อ
package/โมดูลที่ผูกกับ business domain ใหม่

## 1. Frontend — Angular

> **ตัดสินใจแล้ว: reuse โครงสร้างจาก `share_money_frontend` เต็มที่** เหมือนที่ backend reuse จาก
> `share_money_backend` — **ไม่ใช้ PrimeNG/daisyUI** (แผนเดิมที่เคยคุยไว้ถูกยกเลิก เพราะ share_money_frontend
> มี custom shared component library ของตัวเองอยู่แล้ว การเพิ่ม component library ใหม่ซ้อนเป็นการเขียนซ้ำซ้อน
> โดยไม่จำเป็น) ดูแผน reuse เต็มที่หัวข้อ 1.1

| หมวด | เลือกใช้ | หมายเหตุ |
|---|---|---|
| Framework | Angular (เวอร์ชันล่าสุด — `share_money_frontend` อยู่ที่ 22.1.x อยู่แล้ว) | Standalone Components |
| ภาษา | TypeScript (strict mode) | |
| State/Reactive | RxJS + Angular Signals | |
| UI Component Library | **Custom shared component library เดิมจาก `share_money_frontend`** (`app-button`, `app-input`, `app-card`, `app-dialog`, `skeleton`, `empty-state`, `error-state`, `error-page`, `breadcrumb`) — reuse + extend | ต่อ component ใหม่ที่ยังไม่มี (เช่น hero section, promo card, booking wizard step) เข้าไปใน `shared/components/` ชุดเดียวกัน ไม่สร้าง design system คู่ขนาน |
| CSS | Tailwind CSS + SCSS | เหมือนเดิมจาก `share_money_frontend` — retheme สีเป็น Emerald/Gold (ดูหัวข้อ 1.2) |
| Layout | CSS Grid/Flexbox | responsive มือถือ-desktop |
| Theme (ขาว/เขียว) | `ThemeService` เดิม (signal + `localStorage` + toggle CSS class) — **relabel** จาก `light`/`dark` เป็น `white`/`green` ตาม brand แทนโหมด accessibility | ดูหัวข้อ 1.2 |
| i18n (ไทย/อังกฤษ) | `@ngx-translate/core` เดิม + `assets/i18n/th.json`, `en.json` — reuse mechanism, เขียน key/คำแปลใหม่ตาม business สปา | ต้องแปลครบทุกหน้าจอ ไม่ปล่อย key ตกหล่น (เช็คใน QA) |
| Animation พื้นฐาน | CSS transition/animation | ปุ่ม, การ์ด, hover effect เล็ก ๆ (มีบางส่วนอยู่แล้วใน shared component เดิม) |
| Animation เสริม | GSAP + ScrollTrigger (**เพิ่มใหม่** — share_money_frontend ไม่มี) | ภาพ/ข้อความเคลื่อนไหวตามการเลื่อนหน้า — **เฉพาะหน้า public** เท่านั้น, ต้อง respect `prefers-reduced-motion` |
| Rendering | Angular Prerender (SSG) เฉพาะ route public (**เพิ่มใหม่**) | หน้า public (หน้าแรก/catalog/รายละเอียดบริการ/โปรโมชั่น) มี HTML พร้อมก่อน JS ทำงาน เพื่อ SEO + first paint เร็ว — หน้าที่ต้อง login (booking/dashboard) ยังเป็น CSR ปกติ **ไม่ทำ full SSR** เพื่อให้ยัง deploy เป็น static files ขึ้น Cloudflare Pages ได้เหมือนเดิม (ไม่ต้องมี Node server) |
| Forms | Angular Reactive Forms + custom Validators (reuse `shared/utils/validators.util.ts`) | ฟอร์มจอง (เลือกบริการ/วันเวลา/ข้อมูลลูกค้า), ฟอร์มหลังบ้านทั้งหมด — ทุก input ต้องมี maxlength (default 50) |
| รูปภาพ | Cloudinary auto-format (`f_auto`/`q_auto`) → WebP/AVIF อัตโนมัติตาม browser | ผ่าน `FileStorageService` เดิมที่ลอกจาก Share Money ฝั่ง backend — reuse `shared/utils/image-file.util.ts` ฝั่ง frontend สำหรับ validate ก่อนอัปโหลด |
| Routing | Angular Router + Route Guards (`auth.guard`, `role.guard` reuse ตรง ๆ + `verifiedGuard` **เพิ่มใหม่**) | `verifiedGuard` เฉพาะ Customer — กันเข้าหน้าจองก่อนสมัคร/ลิงก์บัญชีผ่าน LINE (ไม่ใช่ SMS OTP แล้ว); `guest.guard` เดิมอาจต้องปรับเพราะระบบนี้มีหน้า public browse โดยไม่ login ได้ (ต่างจาก Share Money ที่บังคับ login ทุก route) |
| HTTP | HttpClient + Interceptor เดิม (`auth.interceptor`, `error.interceptor`, `loading.interceptor`, `api-response.interceptor`) reuse ตรง ๆ | |
| Build/Tooling | Angular CLI, ESLint, Prettier (reuse config เดิม) | |
| Deployment | Build (พร้อม prerender route public) → static files → Cloudflare Pages | ตาม pattern เดิมที่ใช้กับ Share Money frontend |

โครงสร้างโฟลเดอร์ (ยึดจาก `share_money_frontend` จริง):
```
frontend/src/app/
  core/
    guards/         (auth.guard, role.guard — reuse ตรง ๆ, + verifiedGuard ใหม่)
    interceptors/    (auth/error/loading/api-response — reuse ตรง ๆ)
    layout/          (app-shell — ปรับให้แยก public shell กับ dashboard shell)
    models/          (ตัด debt/migration/bank-account model ทิ้ง, เพิ่ม model ธุรกิจสปาใหม่)
    services/        (theme.service reuse+relabel, token-storage/menu.service reuse — ตัด debt/slip/report-api เดิมทิ้ง)
  shared/
    components/      (app-button/app-input/app-card/app-dialog/skeleton/empty-state/error-state — reuse+extend)
    services/        (sweet-alert.service reuse)
    utils/           (validators.util reuse, image-file.util reuse — ตัด excel/document-file util ทิ้งถ้าไม่ใช้)
  features/
    auth/            (reuse โครงเดิม ปรับ flow ให้มี register 2 ทาง (กรอกเอง/LINE) + link account)
    public/          (ใหม่ทั้งหมด — catalog/booking/promotion, ใช้ GSAP+prerender)
    admin/, profile/ (reuse โครงเดิม)
    (ตัด debtors/debts/bank-accounts/documents/slips/reports เดิมทิ้ง — เป็น business logic เฉพาะ Share Money
    ยกเว้น "slips" ที่ concept ใกล้เคียง FR-7 แนบสลิป — ดูหัวข้อ 1.1 ว่าเอา pattern ไหนมาต่อยอดได้)
src/assets/
  i18n/              (th.json/en.json — reuse mechanism, เขียนคำแปลใหม่)
  brand/             (logo/favicon จาก Logo Pack — ดูหัวข้อ 1.2)
```

## 1.1 Frontend Reuse Plan (จาก `share_money_frontend`)

เหมือนกับที่ `CLAUDE.md` หลักกำหนดไว้สำหรับ backend — นี่คือแผน reuse ฝั่ง frontend แบบเดียวกัน (จริงจนกว่า
frontend repo จะ scaffold จริง แล้วอัปเดต `CLAUDE.md` ของ repo นั้นให้ตรงของจริงอีกที)

**✅ Copy มาได้ตรง ๆ (แค่ rename/retheme):**
- `core/guards/auth.guard.ts`, `core/guards/role.guard.ts`
- `core/interceptors/` ทั้งหมด (`auth`, `error`, `loading`, `api-response` — `mock-api` เอาไว้ก็ได้เผื่อ dev
  offline)
- `core/services/theme.service.ts` — relabel `Theme = 'light'|'dark'` → `'white'|'green'`, เปลี่ยนสีที่ผูกไว้
- `core/services/token-storage.service.ts`, `core/services/menu.service.ts`
- `shared/components/` ทั้งหมด (`app-button`, `app-input`, `app-card`, `app-dialog`, `skeleton`,
  `empty-state`, `error-state`, `error-page`, `breadcrumb`, `feature-placeholder`)
- `shared/services/sweet-alert.service.ts`
- `shared/utils/validators.util.ts`, `shared/utils/image-file.util.ts`
- `assets/i18n/th.json`/`en.json` (โครง mechanism — เนื้อหาคำแปลเขียนใหม่ทั้งหมดตาม business สปา)
- `.editorconfig`, `.prettierrc`, `tailwind.config.js`, ESLint config

**❌ ไม่เอา (ลบทิ้ง — business logic เฉพาะ Share Money):**
- `features/debts/`, `features/debtors/`, `features/bank-accounts/`, `features/documents/` ทั้งโมดูล
- `core/models/debt.model.ts`, `core/models/migration.model.ts`, `core/models/phase-three.model.ts`
- `core/services/debt-api.service.ts`, `bank-account-api.service.ts`, `document-api.service.ts`,
  `import-api.service.ts`, `report-api.service.ts` (Report เขียนใหม่ตาม FR-9 ของ Srinaka)
- `shared/utils/excel-file.util.ts`, `document-file.util.ts` (เว้นแต่จะมี requirement export excel จริงทีหลัง)

**⚠️ Copy มาแล้วต้องแก้ ไม่ใช่ copy ตรง ๆ:**
- `core/guards/guest.guard.ts` — ของเดิมออกแบบมาสำหรับระบบที่บังคับ login ทุก route ต้องปรับให้เข้ากับ Srinaka
  ที่มีหน้า public browse ได้โดยไม่ login (ไม่มี guest booking แต่มี guest browse) เพิ่ม `verifiedGuard` ใหม่
- `core/models/user.model.ts` — ตัด field ที่ผูก debt/creditor ทิ้ง, เปลี่ยน role enum เป็น
  `ADMIN`/`SUPERVISOR`/`EMPLOYEE`/`CUSTOMER`, เพิ่ม `phone`/`verifiedAt`/`hasLinkedLine`
- `core/services/auth.service.ts` — เพิ่ม flow สมัครสมาชิก 2 ทาง (กรอกเอง/LINE OAuth) + ลิงก์บัญชี
  LINE จากหน้า profile ที่ Share Money ไม่มี
- `core/layout/app-shell/` — ต้องแยกเป็น 2 shell (public/marketing shell vs dashboard shell) ตามที่ระบุใน
  `04-screen-design.md` หัวข้อ 0 ไม่ใช่ shell เดียวแบบเดิม
- `features/slips/` — **concept ใกล้เคียง FR-7 มาก** (Share Money ก็มีอัปโหลด+ตรวจสลิปเหมือนกัน) ดู
  component/flow เดิมเป็นแนวทางได้ แต่ business logic ผูกกับ debt เดิม ต้องเขียนใหม่ผูกกับ `booking`/`payment`
  ของ Srinaka แทน — **ห้าม copy ตรง ๆ** แค่เอา pattern การอัปโหลด/preview/review รูปมาอ้างอิง
- `features/admin/`, `features/profile/`, `features/auth/` — โครงสร้าง component reuse ได้ แต่เนื้อหา
  ฟิลด์/validation ต้องปรับตาม `01-requirements.md`
- ทุกไฟล์ที่ copy มา — เปลี่ยนชื่อ/คอมเมนต์/asset ที่อ้างอิง "Share Money" → "Srinaka"/"ศรีนาคาออร่า" ทั้งหมด

## 1.2 Brand Assets / Design Tokens

แหล่งที่มา: `D:\GIT\DOCUMENT\Sri_Naga_Aura_Logo_Pack` (Logo Pack แบรนด์ "ศรีนาคาออร่า — Beauty Spa & Sleep
Salon") — เก็บไว้นอก repo เอกสารชุดนี้ (ไฟล์ svg/png ต้นฉบับขนาดใหญ่หลาย MB) ตอน scaffold frontend repo จริง
ค่อย copy เฉพาะไฟล์ที่ใช้จริงเข้า `src/assets/brand/`

**สี (design token):**

| Token | Hex | ใช้ทำอะไร |
|---|---|---|
| Emerald | `#003D2E` | สีหลัก (พื้นหลัง/ปุ่มหลัก) |
| Emerald Accent | `#006B4F` | สี accent/hover |
| Gold | `#D9A526` | สี highlight/CTA/โลโก้ |
| Light | `#FFF3C4` | พื้นหลังอ่อน/ข้อความบนพื้นเข้ม |

ผูกเป็น Tailwind theme token (`tailwind.config`) + CSS custom properties ที่ `ThemeService` toggle
(`theme-white`/`theme-green`) ชุดเดียวกัน เพื่อให้ dashboard กับ storefront ใช้สีตรงกันไม่เพี้ยน — ธีม
**ขาว** ใช้ Light/White เป็นพื้นหลัก + Gold เป็น accent, ธีม **เขียว** ใช้ Emerald เป็นพื้นหลัก + Gold เป็น
accent (ทั้งคู่ยังต้องผ่านเกณฑ์ contrast ที่อ่านง่าย ไม่ใช่แค่สลับสีตรงตัว)

**กติกาใช้โลโก้** (ตาม `README.txt` ใน Logo Pack — ต้องทำตามตอน implement จริง, มีไฟล์ให้ครบทุกกรณีแล้ว):

| กรณีใช้งาน | ใช้ไฟล์จากโฟลเดอร์ |
|---|---|
| Favicon | `01_Favicon/` (มีครบ 16/32/180/512px + `.ico`) |
| Sidebar (dashboard) | `03_Sidebar/` — ใช้ symbol เท่านั้น ไม่ใช้ full logo |
| Header หน้า public | `02_Landing_Header/` — ใช้ full logo |
| หน้า Login/Register | `05_Login_Register/` — ใช้ full logo |
| Dashboard พื้นหลังมืด (ถ้ามี dark mode) | `04_Admin_Dark/` |
| รูป default profile (ยังไม่อัปโหลดรูปเอง) | `06_Profile_Picture/` |

กฎทั่วไป: **เว้นพื้นที่รอบโลโก้อย่างน้อย 10% ของความสูงเสมอ**, favicon/sidebar ใช้ symbol เท่านั้น ห้ามใช้ full
logo ในที่แคบ

## 2. Backend — Spring Boot

| หมวด | เลือกใช้ | หมายเหตุ |
|---|---|---|
| Framework | Spring Boot 3.3.x | |
| ภาษา/JDK | Java 21 (LTS) | |
| Build tool | Maven | |
| Web layer | Spring Web (REST Controller) | |
| Security | Spring Security 6 + JWT (access + refresh token) | ต้องรองรับ public endpoint (ไม่ผ่าน JWT filter) คู่กับ endpoint ที่ต้อง auth — ตั้งแต่ Phase 1 |
| Data Access | Spring Data JPA + Hibernate | |
| Database | PostgreSQL (Neon สำหรับ SIT/prod ตาม pattern เดิม) | |
| Migration | Flyway | |
| Validation | Jakarta Bean Validation | |
| Mapping DTO↔Entity | MapStruct | |
| File Storage | Cloudinary ผ่าน `FileStorageService` abstraction (ลอกจาก Share Money ตรง ๆ) | ใช้ตอนมีรูปบริการ/รูปร้าน |
| API Docs | springdoc-openapi (Swagger UI) | |
| Logging | SLF4J + Logback, structured log สำหรับ audit | |
| Monitoring | Spring Boot Actuator (health, metrics) | |
| Deployment | Render (backend) + Neon (DB), Docker build จาก `Dockerfile` เดียวกับ pattern Share Money | |

โครงสร้างแพ็กเกจ (แนะนำ, Phase 1 เฉพาะ Admin):
```
backend/src/main/java/com/srinaka/
  auth/           (JWT filter, AuthController, TokenService)
  user/           (User entity, UserRole enum: ADMIN/SUPERVISOR/EMPLOYEE/CUSTOMER)
  menu/           (nav menu_items/menu_permissions — สิทธิ์ sidebar เท่านั้น ไม่ใช่เมนูบริการสปา)
  admin/          (SettingsController, LoginLogController)
  common/         (response wrapper, error handler, security config, audit, file storage abstraction)
```
Domain ของธุรกิจสปา (บริการ, ราคา, การจอง) ยังไม่เพิ่ม module — รอสรุป requirement

## 3. Cross-cutting

| หัวข้อ | รายละเอียด |
|---|---|
| Auth token | JWT access token อายุสั้น + refresh token เก็บใน DB (revoke ได้) — เหมือน Share Money |
| Password hashing | BCrypt |
| CORS | เปิดเฉพาะ origin ของ frontend ที่ deploy จริง |
| Public vs Auth endpoint | ต้องแยกชัดตั้งแต่ Security config — endpoint public (Customer ไม่ login) ไม่ผ่าน JWT filter เลย ไม่ใช่แค่ optional-auth |
| Environment config | `application-{dev,prod}.yml`, secrets ผ่าน env var เท่านั้น (ห้าม hardcode ใน yml — บทเรียนจาก Share Money ที่เคย leak Cloudinary key) |
| Containerization | Docker: `Dockerfile` เดียว, deploy ผ่าน Render + Neon (ไม่ต้องมี local docker-compose DB ก็ได้ถ้าจะ dev ตรงกับ SIT) |

## 4. อ้างอิง

โครงสร้าง backend ลอกมาจาก `share_money_backend`, โครงสร้าง frontend ลอกมาจาก `share_money_frontend` (ดู
`CLAUDE.md` ของแต่ละ repo ประกอบ) — ถ้ามีจุดไหนใน Share Money ที่ทำผิดพลาดหรือแก้ทีหลัง (เช่น เรื่อง secret
hardcode, Render free-tier PORT env var, menu_items identity-column collision ตอน seed migration) ให้เอา
บทเรียนพวกนี้มาป้องกันไว้ตั้งแต่แรกในโปรเจคนี้

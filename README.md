# Srinaka — Documentation

> ชื่อแบรนด์จริง: **ศรีนาคาออร่า (Sri Naga Aura) — Beauty Spa & Sleep Salon** ("Srinaka" เป็นชื่อโปรเจค/
> codename ที่ใช้ในเอกสาร+ชื่อ package (`com.srinaka`) เท่านั้น) — Logo Pack อยู่ที่
> `D:\GIT\DOCUMENT\Sri_Naga_Aura_Logo_Pack` (สี Emerald `#003D2E`/`#006B4F`, Gold `#D9A526`, Light `#FFF3C4`)
> ดูรายละเอียดการใช้งานที่ `docs/05-tech-stack.md` หัวข้อ "Brand Assets / Design Tokens"

เอกสารชุดนี้ออกแบบระบบใหม่สำหรับร้านสปา/นวด ด้วย **Angular (frontend) + Spring Boot (backend)**
โดยนำโครงสร้าง (auth, RBAC, response pattern, deploy setup) มาจากโปรเจค Share Money ที่ทำไว้ก่อนหน้า
(`share_money_backend`) มาใช้ซ้ำ ส่วน business logic ของร้านสปา (สินค้า/บริการ, โปรโมชั่น, การจอง, การจ่ายเงิน,
ยกเลิก/คืนเงิน, ให้คะแนน) ออกแบบไว้ครบใน `01-requirements.md`

## สถานะ

**Backend Phase 1 (Foundation: auth ทุก role, RBAC, profile, nav-menu, deploy setup) ทำเสร็จแล้ว** — ดูสถานะ
จริงจาก git log ของ `srinaka_backend` ไม่ใช่จากไฟล์นี้ ส่วนที่เหลือ (Phase 2-5) ออกแบบไว้ครบแล้วตามลำดับใน
`09-implementation-roadmap.md` — **Phase 5 ไม่ใช่ feature ใหม่** เป็น phase หาบั๊ก/ช่องโหว่/ช่องว่างของทีมเอง
ก่อน launch จริง

**Customer สมัครสมาชิก+ยืนยันตัวตน** (เดิมวางแผนใช้ SMS OTP) **เปลี่ยนเป็น LINE OAuth login แล้ว**
(ตัดสินใจแล้ว ไม่ใช่ assumption อีกต่อไป — เคยพิจารณา Google login ด้วยแต่ตัดออกเพราะติดขั้นตอน billing
verification ของ Google Cloud) — สมัครได้ 2 ทาง (กรอกเอง/LINE), สมัครผ่าน LINE ถือว่า
verified ทันที, สมัครกรอกเองต้องลิงก์บัญชี LINE เพิ่มทีหลังถึงจะจองได้ — เป็นงานของ **Phase 2** ไม่ใช่
Phase 1 ดูรายละเอียดที่ `01-requirements.md` FR-1.7–FR-1.9

**ระบบทิปและระบบแจ้งปัญหาของลูกค้า** ถูกพักไว้ก่อน (ไม่อยู่ใน roadmap ปัจจุบัน) ส่วน **ยกเลิก/ขอคืนเงิน**
ยืนยันแล้วว่าต้องทำ (อยู่ใน Phase 3)

มี assumption/จุดที่ยังไม่ตัดสินใจอยู่หลายจุด (payment gateway ถ้าจะทำต่อจาก slip, ตัวเลขนโยบาย refund
ที่ชัดเจน ฯลฯ) — ดูหัวข้อ "ยังไม่ตัดสินใจ" ใน `01-requirements.md` ก่อนเริ่ม implement Phase ที่เกี่ยวข้อง

## เอกสาร

| ไฟล์ | เนื้อหา |
|---|---|
| [docs/01-requirements.md](docs/01-requirements.md) | Requirement เต็ม: actor, FR-1 ถึง FR-10 (auth, profile, user mgmt, catalog, promotion, booking, payment/slip+cancel/refund, rating, report, employee schedule), NFR, จุดที่ยังไม่ตัดสินใจ, feature ที่พักไว้ก่อน |
| [docs/02-roles-and-structure.md](docs/02-roles-and-structure.md) | Actor/role ทั้งหมด, ขอบเขตสิทธิ์คร่าว ๆ, จุดที่ต่างจาก Share Money (guest browse แต่จองต้อง verify) |
| [docs/03-database-design.md](docs/03-database-design.md) | ER diagram + ตาราง/คอลัมน์/index ครบทุก Phase (1-4), คำแนะนำวิธี implement ให้ไวไม่เสียของ |
| [docs/04-screen-design.md](docs/04-screen-design.md) | Screen inventory ฝั่ง Frontend ทุก Phase (1-4) — หน้าจอ, role ที่เข้าถึงได้, เนื้อหาหลัก, ผูกกับ FR, open item ที่กระทบหน้าจอ |
| [docs/05-tech-stack.md](docs/05-tech-stack.md) | Tech stack ฝั่ง Angular และ Spring Boot (โครงสร้างเดียวกับ Share Money) |
| [docs/08-qa-test-plan.md](docs/08-qa-test-plan.md) | QA test case/checklist ทุก Phase (1-4), cross-cutting security/performance checklist, defect severity, เกณฑ์ sign-off |
| [docs/09-implementation-roadmap.md](docs/09-implementation-roadmap.md) | แผนงาน Frontend และ Backend ฝั่งละ 5 Phase, task checklist และเกณฑ์ส่งมอบ |

เอกสารอื่น (API spec, validation rules แบบละเอียด) ยังไม่เขียน — รอเริ่ม Phase ที่เกี่ยวข้องจริงแล้วค่อยลง
รายละเอียด

## สรุประบบโดยย่อ

ระบบเว็บร้านสปา/นวด เน้นหน้าเว็บสวยงามให้ลูกค้าดูสินค้า/บริการ+ราคา+โปรโมชั่น, สมัครสมาชิก (กรอกเอง/LINE)
+ยืนยันตัวตนแล้วจองนัด+จ่ายเงิน(แนบสลิป)+ยกเลิก/ขอคืนเงิน+ให้คะแนนร้าน/พนักงานผ่านเว็บได้ ฝั่ง Employee ยังขาย
บริการหน้าร้านให้ลูกค้า walk-in เก็บเงินสด คีย์ส่วนลด พิมพ์ใบเสร็จได้เอง (POS-lite, ไม่มีระบบตัด stock) มี 4
actor: **Admin** (ดูแลระบบ ไม่ยุ่งธุรกิจ), **Supervisor** (เจ้าของ/ผู้ดูแลธุรกิจ), **Employee** (พนักงาน),
**Customer** (browse ได้โดยไม่ login, จองต้อง login+verify ตัวตน)

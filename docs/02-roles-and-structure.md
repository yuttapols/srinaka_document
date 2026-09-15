# Roles & Structure — Srinaka

เอกสารนี้พูดถึงโครงสร้าง actor/สิทธิ์ระดับกว้าง ๆ — รายละเอียด business logic เต็มดูที่ `01-requirements.md`

## Actor

| Role | คำอธิบาย | ต้อง login ไหม |
|---|---|---|
| **Admin** | ดูแลระบบ/infra เท่านั้น ไม่เห็น/ไม่ยุ่งข้อมูลธุรกิจ (สินค้า, บริการ, ราคา, booking, ยอดขาย) — เหมือน pattern ที่ใช้ใน Share Money ที่กัน ADMIN ออกจากข้อมูลหนี้ | ต้อง login |
| **Supervisor** | เจ้าของ/ผู้ดูแลธุรกิจ จัดการสินค้า/บริการ/โปรโมชั่น, มอบหมายงาน, ดู report ภาพรวม | ต้อง login |
| **Employee** | พนักงานร้าน (หมอนวด/staff) ใช้งานหน้างาน ถูกให้คะแนน | ต้อง login |
| **Customer** | ลูกค้า | ดู public content (สินค้า/บริการ/โปรโมชั่น) ได้โดยไม่ login — แต่**ต้องสมัครสมาชิก+ยืนยันตัวตนผ่าน LINE ก่อนถึงจะจองนัดได้** (ไม่มี guest booking) |

จุดที่ต่างจาก Share Money เดิมชัดเจน: ระบบเดิมบังคับ login ทุก role ไม่มีแนวคิด guest/public access เลย —
ระบบนี้ต้องรองรับ endpoint แบบ public (ไม่ต้องมี JWT) คู่กับ endpoint แบบต้อง auth ในระบบเดียวกัน และมีชั้น
"login แล้วแต่ยัง verify ไม่ผ่าน" เพิ่มมาอีกชั้นสำหรับ Customer (login ได้ แต่ยังจองไม่ได้จนกว่าจะสมัครผ่าน
LINE โดยตรง หรือลิงก์บัญชี LINE เพิ่มถ้าสมัครแบบกรอกเอง — ดู `01-requirements.md` FR-1.8/FR-1.9)

## Phase การทำงาน

ดูรายละเอียดเต็มที่ `09-implementation-roadmap.md` — สรุปย่อ:

1. **Phase 1** — Foundation: auth/JWT ทุก role, profile, RBAC, nav-menu permission, deploy setup (Customer
   สมัคร+verify ผ่าน LINE เป็นงาน Phase 2 — ดู `01-requirements.md` FR-1.7)
2. **Phase 2** — Product/Service catalog, โปรโมชั่น, ระบบจองนัด
3. **Phase 3** — Payment (แนบสลิป), ยกเลิก/ขอคืนเงิน, ให้คะแนน
4. **Phase 4** — Report ภาพรวม, Employee schedule, Admin ops, release
5. **Phase 5** — Issue: รีวิวหาบั๊ก/ช่องโหว่/ช่องว่างทั้งระบบแล้วอุด ก่อน launch จริง (ไม่ใช่ feature ใหม่)

**ระบบทิปและระบบแจ้งปัญหาของลูกค้า** เคยออกแบบไว้เป็น FR แล้วแต่ตอนนี้ถูกพักไว้ก่อน (user บอกว่า "อาจจะยังไม่
ต้องทำ") ไม่ได้อยู่ใน roadmap ด้านบน — ดูรายละเอียดที่ `01-requirements.md` หัวข้อ "เลื่อนออกไปก่อน"

## สิ่งที่ลอกจาก Share Money ได้ตรง ๆ (โครงสร้างกลาง ไม่ผูก business)

- Auth: JWT (access + refresh), `BCryptPasswordEncoder`, login lockout, `JwtSecretGuard`
- RBAC: `@PreAuthorize` + ownership-check pattern ที่ service layer
- Response pattern: `ApiResponse<T>` wrapper, `BusinessException` + `ErrorCode` enum, `GlobalExceptionHandler`
- `common/`: `AuditLogService`, `FileStorageService` (Cloudinary abstraction — ใช้ได้ทั้งรูปสินค้า/บริการ และสลิปการจ่ายเงิน), `BaseEntity`
- Nav-menu permission system (`menu_items`/`menu_permissions`) — ใช้คุม sidebar ฝั่ง Admin/Supervisor
  dashboard เท่านั้น **ไม่ใช่** เมนูสินค้า/บริการที่ลูกค้าดู (คนละเรื่องกัน ดูหัวข้อถัดไป)
- Deploy: `Dockerfile`, `render.yaml`, docker-compose pattern

## ⚠️ ระวังคำว่า "เมนู" ชนกัน 2 ความหมาย

- **Nav menu** (`menu_items`/`menu_permissions` จาก Share Money) — คุมว่า role ไหนเห็นเมนูไหนใน sidebar ของ
  Admin/Supervisor dashboard เท่านั้น
- **เมนูสินค้า/บริการสปา** (product/service catalog ที่ลูกค้าดู) — เป็น **business domain ใหม่ทั้งหมด**
  ไม่เกี่ยวกับ nav menu เดิมเลย ดูรายละเอียดที่ `01-requirements.md` FR-4

## ยังไม่ตัดสินใจ (ดูรายการเต็มที่ `01-requirements.md` หัวข้อบนสุด)

- Payment gateway จริง (ถ้าจะทำต่อจาก slip), ตัวเลขนโยบาย refund/cancel ที่ชัดเจน
- Supervisor กับ Employee ต่างสิทธิ์กันตรงไหนเพิ่มเติมนอกจาก FR ที่ระบุไว้แล้ว

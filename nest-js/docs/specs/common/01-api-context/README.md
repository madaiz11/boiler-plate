# API Specification & Context Architecture

**Project:** MVP Game Backoffice API  
**Stack:** NestJS, Prisma, PostgreSQL, Redis

---

## 1. Context Management (CLS Concept)

ระบบจะใช้ Continuation-Local Storage (CLS) ผ่าน `nestjs-cls` เพื่อหลีกเลี่ยงการทำ Prop Drilling (การส่งพารามิเตอร์ต่อๆ กันไปในทุกฟังก์ชัน)

Context จะถูกแบ่งออกเป็น 2 ระดับ:

### 1.1 RequestContext (Middleware Level)

ทำงานตั้งแต่ Request วิ่งเข้ามาถึงเซิร์ฟเวอร์ โดยจะเก็บข้อมูลพื้นฐานของ HTTP Request

- **x-request-id (Trace ID):** ดึงจาก Header `x-request-id` หากไม่มี ระบบจะสร้าง UUID V4 ขึ้นมาใหม่
- **IP Address:** ไอพีต้นทางที่ยิง Request
- **User-Agent:** ข้อมูลเบราว์เซอร์หรืออุปกรณ์

### 1.2 SessionContext (Guard Level)

ทำงานในชั้น Authentication Guard หลังจากแกะ JWT Token สำเร็จ โดยใช้ **Cache-Aside Pattern** ผ่าน Redis เพื่อหลีกเลี่ยงการยิง DB ทุก Request

- **User ID:** รหัสของผู้ใช้งาน (Admin) ที่ยืนยันตัวตนแล้ว
- **Role/Permissions:** สิทธิ์การใช้งาน — โหลดจาก Redis Cache ก่อน ถ้า miss ค่อยดึงจาก Database แล้ว write กลับเข้า Cache

#### Cache Strategy

| Item | ค่า |
| :--- | :--- |
| **Cache Key** | `session:admin:{adminId}` |
| **TTL** | 300 วินาที (5 นาที) |
| **Cached Fields** | `adminId`, `role`, `permissions[]` |
| **ห้าม Cache** | `hashedPassword`, `jwtToken`, PII (email, phone) — เป็น `StrictDataType` ตาม `101-sensitive-data.mdc` |

#### Cache Invalidation

ต้อง `DEL session:admin:{adminId}` ทันทีเมื่อเกิดเหตุการณ์ต่อไปนี้:

- เปลี่ยน role / permissions ของ Admin
- Reset password
- Ban / Suspend account

*เมื่อนำข้อมูลเข้า CLS Store แล้ว ชั้น Use Case และ Repository จะสามารถเรียกใช้ค่าเหล่านี้ได้ทันทีผ่าน `cls.get()`*

---

## 2. API Specification

**Feature:** Deduct Pang Coin (ตัดเงินผู้เล่น)

- **Method:** `POST`
- **Endpoint:** `/api/v1/admin/players/{playerId}/coins/deduct`
- **Description:** ตัดเหรียญ Pang Coin ของผู้เล่นตามจำนวนที่ระบุ (สำหรับ Admin)

### 2.1 Request Headers

| Key | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `Authorization` | String | Yes | Bearer Token ของ Admin |
| `x-request-id` | String | No | Trace ID สำหรับติดตาม Log |
| `User-Agent` | String | Yes | ข้อมูลอุปกรณ์ที่เรียก API |

### 2.2 Path Parameters

| Key | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `playerId` | String (UUID) | Yes | ไอดีของผู้เล่นเป้าหมาย |

### 2.3 Request Body (application/json)

```json
{
  "amount": 500,
  "reason": "buy_item_001"
}
```

### 2.4 Responses

**Success - 200 OK**

```json
{
  "success": true,
  "status": 200,
  "message": "success",
  "traceId": "req-1234-abcd",
  "timestamp": "2026-05-10T10:50:41.000Z",
  "body": {
    "id": "player-uuid-1234",
    "name": "Marut",
    "pangCoin": 1500
  }
}
```

**Error - 400 Bad Request (Insufficient Funds)**

```json
{
  "success": false,
  "status": 400,
  "message": "Insufficient Pang Coins for this transaction.",
  "error": "PLAYER_1002",
  "traceId": "req-1234-abcd",
  "timestamp": "2026-05-10T10:50:41.000Z"
}
```

---

## 3. Sequence Diagram

Flow การทำงานตั้งแต่ Client ยิง Request จนถึงการคืนค่าด้วย CommonResponse

```plantuml
@startuml
!theme plain
skinparam sequenceArrowThickness 2
skinparam roundcorner 10
skinparam maxmessagesize 150

autonumber

actor "Admin (Client)" as adminClient

box "NestJS Backoffice API" #fdfdfd
    participant "Context Middleware" as contextMiddleware #dae8fc
    participant "Auth Guard" as authGuard #fff2cc
    participant "CLS Store\n(AsyncLocalStorage)" as clsStore #e1d5e7
    participant "AdminPlayer\nController" as adminPlayerController #d5e8d4
    participant "DeductPangCoin\nUseCase" as deductPangCoinUseCase #d5e8d4
    database "Redis\n(Session Cache)" as redisCache #ffe6cc
    database "PostgreSQL\n(Prisma)" as postgresDb #f8cecc
end box

adminClient -> contextMiddleware : POST /players/{id}/coins/deduct\nHeaders: [x-request-id, Token]
activate contextMiddleware

opt Missing x-request-id header
    contextMiddleware -> contextMiddleware : Generate UUID v4 trace id
end

contextMiddleware -> contextMiddleware : Extract x-request-id, IP, User-Agent

note over contextMiddleware
  Persist RequestContext into CLS
  (trace id, IP, User-Agent)
end note

contextMiddleware -> clsStore : set(requestContext)
activate clsStore
clsStore --> contextMiddleware : Ack
deactivate clsStore

contextMiddleware -> authGuard : Forward Request
activate authGuard

note over authGuard
  JWT verification
  Cache-Aside: Redis → DB fallback
end note

authGuard -> authGuard : Verify Token & Extract AdminId

authGuard -> redisCache : GET session:admin:{adminId}
activate redisCache

alt Cache HIT
    redisCache --> authGuard : Cached SessionContext\n(adminId, role, permissions)
    deactivate redisCache
else Cache MISS
    deactivate redisCache
    authGuard -> postgresDb : findAdminUserById(adminId)
    activate postgresDb
    postgresDb --> authGuard : Admin Data
    deactivate postgresDb
    authGuard -> redisCache : SET session:admin:{adminId} EX 300\n(adminId, role, permissions only — no StrictDataType fields)
    activate redisCache
    redisCache --> authGuard : Ack
    deactivate redisCache
end

authGuard -> clsStore : set(sessionContext)
activate clsStore

note right of clsStore
  SessionContext: adminId,
  role/permissions
end note

clsStore --> authGuard : Ack
deactivate clsStore

authGuard -> adminPlayerController : Route to Endpoint
deactivate authGuard
activate adminPlayerController

adminPlayerController -> deductPangCoinUseCase : execute(playerId, amount)
activate deductPangCoinUseCase

note over deductPangCoinUseCase, clsStore
  Read trace + admin session
  via cls.get(...)
end note

deductPangCoinUseCase -> clsStore : get('x-request-id'), get('session.adminId')
activate clsStore
clsStore --> deductPangCoinUseCase : traceId, adminId
deactivate clsStore

deductPangCoinUseCase -> postgresDb : findById(playerId)
activate postgresDb
postgresDb --> deductPangCoinUseCase : PlayerEntity
deactivate postgresDb

alt Sufficient Coins
    deductPangCoinUseCase -> deductPangCoinUseCase : player.deductCoins(amount)
    deductPangCoinUseCase -> postgresDb : save(player)
    activate postgresDb
    postgresDb --> deductPangCoinUseCase : Success
    deactivate postgresDb
    deductPangCoinUseCase --> adminPlayerController : CommonResponse<Success>
else Insufficient Coins
    deductPangCoinUseCase --> adminPlayerController : CommonResponse<ErrorResponse>
end

adminPlayerController --> contextMiddleware : Return Result
deactivate deductPangCoinUseCase
deactivate adminPlayerController

contextMiddleware --> adminClient : JSON Response (HTTP 200/400)
deactivate contextMiddleware

@enduml
```

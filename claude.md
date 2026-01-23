# Claude.md — Project Context (DECO Demo: Company Site + Catalog/Cart + Order + Invoice-ready)

> Single source of truth for DECO demo project context.
> This document is optimized for AI-assisted development:
> - Domain + ERD mapping
> - API contracts summary
> - Implementation constraints & conventions

---

## 0) Project Overview

### Project Name
**DECO Demo (Company Website + Catalog/Cart + Order + Admin System)**

### Why we build this
This is a **portfolio-grade** demo built like a real service:
- ERD → API spec → FE/BE collaboration → deployment
- Keep a real deployed artifact online (demo legacy)
- Simulate export jewelry business workflow

### Business Context
Industry: Precious metal / Jewelry export  
Current pain: post-order admin work is manual (invoice in Excel + printing).  
This project builds a demo system where:
- Users browse catalog and place orders (without payment integration in MVP demo version)
- Admin manages users/products/orders
- System stores data reliably and can be extended to invoice issuing later

---

## 1) Scope Definition

### MVP In Scope ✅
- Public landing page
- Product catalog
- Cart
- Order creation + order status tracking
- Authentication & authorization
- Admin portal:
  - user approval/role management
  - product CRUD
  - order management
  - qna management

### Out of Scope ❌
- Payment integration (real PG gateway)  
  > Note: `payments` table + APIs exist for "bank transfer/manual payment proof" demo,
  > but we do NOT integrate with real online payment gateway in MVP.

---

## 2) Tech Stack

### Frontend
- Language: JavaScript
- Framework: React (Vite)
- Deployment: Cloudflare Pages

### Backend
- Language: **Java**
- Framework: Spring Boot
- Security: Spring Security
- API: REST JSON

### Database
- **PostgreSQL**
- Migration: Flyway (recommended)

> IMPORTANT:
> Current ERD DDL is written in MySQL style.
> Implementation must convert to PostgreSQL schema migration scripts
> (types, auto increment, boolean defaults, timestamps).

---

## 3) Architecture

### Components
1) **Public Web (React)**
   - landing / catalog / cart / my page
2) **Admin Web (React)**
   - admin dashboard
3) **API Server (Spring Boot)**
   - auth, domain logic, admin endpoints
4) **PostgreSQL DB**
   - users/products/cart/orders/payments/qna etc

---

## 4) Domain Model / ERD (As Designed)

This section reflects the team-designed ERD tables:

### 4.1 users
Represents both **Personal users** and **Corporate buyers**.

Key fields:
- `user_type`: PERSONAL / CORPORATE
- `role`: USER / ADMIN
- `status`: PENDING / VERIFIED / WITHDRAWN
- `email`, `password`, `salt`
- personal profile: `last_name`, `first_name`
- corporate profile: `company_name`, `business_number`
- `shipping_address` saved after first order
- soft delete: `deleted_at`

Notes / implementation:
- Password hashing should NOT use sha256 + salt in modern systems.
  - Replace with BCrypt (Spring Security encoder).
  - Keep column names for compatibility but implementation should store BCrypt hash in `password`.
  - `salt` can be deprecated (store empty or remove in migration if agreed).

---

### 4.2 log_auth
Admin audit trail for authorization / approval actions.

Fields:
- `admin_id`, `user_id`
- `auth_type` (가입승인, 권한회수 등)
- `created_at`, `updated_at`

---

### 4.3 products
Product catalog.

Fields:
- `name`, `category`, `carat`
- `price`, `stock`
- `status` (판매중/품절/숨김 등)
- `desc`
- `created_at`, `updated_at`

---

### 4.4 product_images
Product image URLs.

Fields:
- `product_id`
- `image_url`
- `is_thumbnail`

---

### 4.5 carts
Cart per user.

Fields:
- `user_id`
- `status` boolean: cart has items or not (as designed)
- `created_at`, `updated_at`

Implementation note:
- Better naming would be `has_items` or `is_active`.
- But we keep this as designed.

---

### 4.6 cart_products
Items inside cart.

Fields:
- `product_id`
- `cart_id`
- `quantity`
- `unit_price`
- `total_price`

Implementation note:
- `currency DECIMAL(10,2)` is likely incorrect type for currency.
  - Should be VARCHAR(10) (USD, KRW, JPY).
  - For MVP: treat this column as optional/deprecated and use `orders.currency_snapshot`.

---

### 4.7 orders
Order placed by user.

Fields:
- `order_number`
- `order_price`
- `currency_snapshot` default USD
- `status`: CREATED/PENDING_PAYMENT/PAID/SHIPPED/DELIVERED/CANCELLED/REFUNDED
- `address`
- `tracking_number`
- timestamps: `ordered_at`, `created_at`, `updated_at`
- `cancelled_at` (should be DATETIME ideally)

---

### 4.8 order_products
Order line items.

Fields:
- `order_id`
- `product_id`
- `quantity`
- `unit_price`
- `total_price`

---

### 4.9 payments
Manual payment/bank transfer proof entity.

Fields:
- `order_id`, `user_id`
- `method` default BANK_TRANSFER
- `status` PENDING/PAID/FAILED/CANCELLED/REFUNDED
- sender/bank/swift/amount
- receipt_url (upload proof)
- paid_at
- created_at

Notes:
- We keep payments in ERD and API, but **no real PG integration**.

---

### 4.10 qna
User Q&A.

Fields:
- `title`, `content`
- `status` OPEN (답변대기/답변완료)
- `answer`, `answered_at`

---

### 4.11 currency
Currency list / snapshot.

Fields:
- `currency`
- `created_at` (periodic snapshot)

---

### 4.12 gold_rate
Gold/silver rate snapshots.

Fields:
- `gold`, `silver`
- `created_at`

---

## 5) Key Business Rules

### User approval flow
- user registers → status=PENDING
- admin approves → status=VERIFIED and recorded in `log_auth`

### Products
- if `status` == 숨김: not shown in public catalog
- stock cannot go below zero

### Cart → Order
- cart_products are turned into order_products at checkout
- order_price = sum(order_products.total_price)

### Payment (manual)
- payments entry can be created/updated by user/admin
- status transitions must be validated

---

## 6) API Endpoints (As Designed)

> Below is the raw endpoint catalog normalized into grouped format.
> (팀 설계 API 그대로 유지하되, 중복은 의미 기준으로 정리)

---

### 6.1 Auth
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `POST /api/auth/refresh`
- `POST /api/auth/password` (password reset/change)

Email verification:
- `POST /api/auth/email/verification`
- `POST /api/auth/email/verify`

---

### 6.2 User (Self)
- `GET /api/users/me`
- `PUT /api/users/me`
- `PATCH /api/users/me` *(if used; normalize to PUT recommended)*

---

### 6.3 Admin — Users
- `GET /api/admin/users`
- `GET /api/admin/users/{userId}`
- `PUT /api/admin/users/{userId}`
- `DELETE /api/admin/users/{userId}` *(soft delete recommended)*

Auth log (optional exposure):
- (optional) `GET /api/admin/auth/logs`

---

### 6.4 Products
Public:
- `GET /api/products`
- `GET /api/products/{productID}`

Admin:
- `POST /api/admin/products`
- `PUT /api/admin/products/{productID}`
- `DELETE /api/admin/products/{productID}`

(duplicate entries from design merged)
- `/api/admin/products/{productID}`
- `/api/admin/products`

---

### 6.5 Cart
As designed:
- `GET /api/cart/{userID}`
- `POST /api/cart/{userID}`
- `DELETE /api/cart/{userID}`

Implementation notes:
- This design implies 1 cart per user.
- In REST purity, cart should be `/api/users/me/cart`, but we keep design.

---

### 6.6 Orders
User:
- `POST /api/orders` (create order from cart)
- `GET /api/orders` (list own orders)
- `GET /api/orders/{orderID}`
- `POST /api/orders/{orderID}/cancel`

Admin:
- `GET /api/admin/orders`
- `GET /api/admin/orders/{orderID}`
- `PUT /api/admin/orders/{orderID}` *(if status update/fulfillment)*

(duplicate design endpoints merged):
- `/api/admin/orders/`
- `/api/admin/orders/{orderID}`

---

### 6.7 QnA
User:
- `GET /api/qna`
- `POST /api/qna`
- `GET /api/qna/{qnaID}`
- `PUT /api/qna/{qnaID}` *(or PATCH based on policy)*

Admin:
- `GET /api/admin/qna`
- `PUT /api/admin/qna/{qnaID}` *(answer/update status)*

---

### 6.8 Payments
User:
- `POST /api/payments` *(register manual payment proof)*

Admin:
- (as designed appears mixed with admin orders/products duplicates)
- recommend:
  - `GET /api/admin/payments`
  - `PUT /api/admin/payments/{paymentId}`

But keep minimum:
- `/api/payments`

---

## 7) Implementation Conventions

### HTTP method conventions
- GET: 조회
- POST: 생성
- PUT: 전체 수정
- PATCH: 부분 수정
- DELETE: 삭제 (soft delete preferred)

### Status codes
- 200 OK
- 201 CREATED
- 400 validation error
- 401 unauthorized
- 403 forbidden
- 404 not found
- 409 conflict (stock/order collisions)

### Soft delete
- `users.deleted_at` used for withdrawal
- for other tables, use `status` or `deleted_at` based on need

---

## 8) Environment Variables

Backend:
- `APP_ENV=dev|staging|prod`
- `SERVER_PORT=8080`
- `DB_URL=jdbc:postgresql://...`
- `DB_USER=...`
- `DB_PASSWORD=...`
- `JWT_SECRET=...`
- `JWT_ACCESS_TTL=...`
- `JWT_REFRESH_TTL=...`

Frontend:
- `VITE_API_BASE_URL=...`

---

## 9) Local Development

Backend:
- `./gradlew bootRun`

Frontend:
- `npm install`
- `npm run dev`

DB:
- docker compose with postgres (recommended)
- flyway migration auto-run

---

## 10) Deployment

Static:
- Cloudflare Pages

Dynamic:
- AWS Free Tier OR Oracle Free Tier

CI/CD checklist:
- build + test
- deploy to staging
- manual promote to prod-demo

---

## 11) Out of Scope Reminder
- Payment gateway integration is excluded from MVP.
- Cart/Catalog/Order are included.

---

End of Claude.md

# AF-EtMart: API Specification

> Status: **draft contract**. No endpoint exists yet.

## Conventions

- Base path `/api/v1`; JSON with `camelCase` fields; uploads use `multipart/form-data`.
- Authentication: `Authorization: Bearer <accessToken>`.
- Money: integer santim (`125000` = ETB 1,250.00). Times: ISO 8601 UTC. IDs: numbers.
- Lists: `?page=1&limit=20` (maximum 100) with `meta: { page, limit, total }`.
- Success: `{ "data": ..., "meta": ... }`.
- Records owned by someone else return 404.

| Status | Meaning |
|---|---|
| 200 / 201 / 204 | OK / created / no content |
| 400 | Malformed request |
| 401 / 403 | Not authenticated / not allowed |
| 404 | Not found or not yours |
| 409 | Conflict (stock, price change, state, duplicate) |
| 422 | Validation failed |
| 429 | Rate limited |
| 500 | Unexpected error |

```json
{ "error": { "code": "INSUFFICIENT_STOCK", "message": "Some items are no longer available.",
  "details": [{ "productId": 41, "available": 1 }], "requestId": "req_8f2c" } }
```

## Endpoints

Access: P public, U any signed-in user, C customer or seller (as buyer), S approved active seller, A admin.

### Auth and account
| Method | Path | Access | Responsibility |
|---|---|---|---|
| POST | /auth/register | P | Create customer account |
| POST | /auth/login | P | Issue tokens |
| POST | /auth/refresh | cookie | Rotate tokens |
| POST | /auth/logout | cookie | Revoke current token family |
| GET | /me | U | Current user |
| PATCH | /me | U | Update profile |
| PUT | /me/password | U | Change password, revoke sessions |
| GET, POST | /me/addresses | U | List, add addresses |
| PATCH, DELETE | /me/addresses/:id | U | Update, remove own address |

### Catalog
| Method | Path | Access | Responsibility |
|---|---|---|---|
| GET | /categories | P | Active category tree |
| GET | /products | P | Search: `q, categoryId, sellerId, minPrice, maxPrice, sort` |
| GET | /products/:slug | P | Product with images and seller details |
| GET | /sellers/:slug | P | Public store profile |

### Cart and orders
| Method | Path | Access | Responsibility |
|---|---|---|---|
| GET | /cart | C | Cart with current prices and availability |
| POST | /cart/items | C | Add item |
| PATCH, DELETE | /cart/items/:id | C | Change or remove |
| POST | /checkout/preview | C | Totals per seller for review; changes nothing |
| POST | /orders | C | Place order (requires `Idempotency-Key`) |
| GET | /orders | C | Own order history |
| GET | /orders/:orderNumber | C | Own order detail and history |
| POST | /orders/:orderNumber/seller-orders/:id/cancel | C | Cancel while pending |
| POST | /orders/:orderNumber/seller-orders/:id/return-requests | C | Request return after delivery |

### Seller
| Method | Path | Access | Responsibility |
|---|---|---|---|
| POST | /seller-applications | C | Apply (customers only) |
| GET | /seller-applications/mine | C | Own application status |
| GET, PATCH | /seller/profile | S | Own profile; legal fields need admin review |
| GET, POST | /seller/products | S | List, create |
| GET, PATCH, DELETE | /seller/products/:id | S | Read, update, archive |
| POST | /seller/products/:id/images | S | Upload image |
| DELETE | /seller/products/:id/images/:imageId | S | Remove image |
| POST | /seller/products/:id/stock-adjustments | S | Change stock by a delta |
| GET | /seller/orders | S | Own seller orders |
| GET | /seller/orders/:id | S | Items and delivery details |
| PATCH | /seller/orders/:id/status | S | Allowed transitions only |
| POST | /seller/orders/:id/cash-collection | S | Record cash after delivery |
| GET | /seller/sales/summary | S | Totals for a date range |

### Administrator
| Method | Path | Access | Responsibility |
|---|---|---|---|
| GET | /admin/seller-applications | A | Review queue |
| GET | /admin/seller-applications/:id | A | Detail and documents |
| POST | /admin/seller-applications/:id/approve | A | Approve, create profile |
| POST | /admin/seller-applications/:id/reject | A | Reject with reason |
| GET | /admin/users | A | Search users |
| PATCH | /admin/users/:id/status | A | Suspend or restore |
| GET | /admin/sellers | A | List sellers |
| PATCH | /admin/sellers/:id/status | A | Suspend or restore |
| GET | /admin/products | A | All products |
| PATCH | /admin/products/:id/status | A | Moderate with reason |
| GET, POST | /admin/categories | A | List, create |
| PATCH | /admin/categories/:id | A | Update or deactivate |
| GET | /admin/orders | A | Monitor orders |
| GET | /admin/orders/:orderNumber | A | Detail |
| PATCH | /admin/seller-orders/:id/status | A | Intervene with reason |
| GET | /admin/return-requests | A | Return queue |
| PATCH | /admin/return-requests/:id | A | Accept, reject, record refund |
| GET | /admin/dashboard | A | Basic counts |
| GET | /admin/audit-logs | A | Audit trail |
| GET | /health | P | Liveness and database check |

**Deferred until decided:** `POST /payments/initialize` (C), `POST /payments/webhooks/:provider` (provider signature), review endpoints, password reset and verification.

## Examples

### Register
`POST /api/v1/auth/register`
```json
{ "fullName": "Hana Bekele", "email": "hana@example.com", "phone": "+251900000000", "password": "a long example passphrase" }
```
`201`
```json
{ "data": { "id": 12, "fullName": "Hana Bekele", "email": "hana@example.com", "role": "customer" } }
```
A `role` field is rejected. The duplicate-account message must not reveal more than necessary.

### Login
`POST /api/v1/auth/login`
```json
{ "identifier": "hana@example.com", "password": "a long example passphrase" }
```
`200`, plus the refresh cookie
```json
{ "data": { "accessToken": "<jwt>", "expiresIn": 900, "user": { "id": 12, "fullName": "Hana Bekele", "role": "customer" } } }
```

### Search products
`GET /api/v1/products?q=coffee&sort=price_asc&page=1&limit=20`
```json
{ "data": [ { "id": 41, "name": "Roasted Coffee 500 g", "slug": "roasted-coffee-500g-41",
    "priceSantim": 45000, "inStock": true, "thumbnailUrl": "/media/p41-thumb.webp",
    "seller": { "id": 3, "storeName": "Example Coffee", "slug": "example-coffee" } } ],
  "meta": { "page": 1, "limit": 20, "total": 1 } }
```

### Create product
`POST /api/v1/seller/products`
```json
{ "name": "Roasted Coffee 500 g", "categoryId": 4, "description": "Whole beans.", "priceSantim": 45000, "stockQuantity": 20, "status": "draft" }
```
`201`
```json
{ "data": { "id": 41, "slug": "roasted-coffee-500g-41", "priceSantim": 45000, "stockQuantity": 20, "status": "draft" } }
```
The seller comes from the token, never the body.

### Add to cart
`POST /api/v1/cart/items`
```json
{ "productId": 41, "quantity": 2 }
```
`201`
```json
{ "data": { "items": [ { "id": 7, "productId": 41, "name": "Roasted Coffee 500 g", "quantity": 2, "unitPriceSantim": 45000, "lineTotalSantim": 90000 } ], "itemsTotalSantim": 90000 } }
```

### Preview checkout
`POST /api/v1/checkout/preview`
```json
{ "addressId": 5, "paymentMethod": "cod" }
```
`200` (delivery fee is an example only)
```json
{ "data": { "sellerOrders": [ { "sellerId": 3, "subtotalSantim": 90000, "deliveryFeeSantim": 10000, "totalSantim": 100000 } ],
  "itemsTotalSantim": 90000, "deliveryTotalSantim": 10000, "totalSantim": 100000 } }
```

### Place order
`POST /api/v1/orders` with header `Idempotency-Key: 2f1c0c9e-6a0b-4c1e-9d55-1f3f0f6a2b11`
```json
{ "addressId": 5, "paymentMethod": "cod", "expectedTotalSantim": 100000 }
```
`201`
```json
{ "data": { "orderNumber": "AF-7K3M9Q2X", "status": "pending", "totalSantim": 100000, "paymentMethod": "cod",
  "sellerOrders": [ { "id": 88, "sellerId": 3, "status": "pending", "totalSantim": 100000 } ] } }
```
A repeat with the same key and body returns the same order (`200`). Errors: `409 INSUFFICIENT_STOCK`, `409 TOTAL_CHANGED`, `409 IDEMPOTENCY_KEY_REUSED`, `422`.

### Update seller order
`PATCH /api/v1/seller/orders/88/status`
```json
{ "status": "confirmed" }
```
`200`
```json
{ "data": { "id": 88, "status": "confirmed", "updatedAt": "2027-01-01T09:00:00Z" } }
```
An invalid transition returns `409 INVALID_TRANSITION`. `rejected` requires `reason`.

### Approve seller
`POST /api/v1/admin/seller-applications/9/approve`
```json
{ "note": "Documents checked." }
```
`200`
```json
{ "data": { "applicationId": 9, "status": "approved", "sellerId": 3 } }
```
A previously reviewed application returns `409`.

## Rate limits *(proposed starting values)*

| Route | Limit |
|---|---|
| Login | 5 per 15 minutes per account and IP |
| Register | 5 per hour per IP |
| Order placement | 10 per hour per user |
| Other | 100 per minute per IP |

Mobile users may share IP addresses, so monitor and tune these.

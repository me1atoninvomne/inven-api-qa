# Test Cases — Store Inventory API

Legend: **P** = priority (H / M / L), **Type** = Pos (positive), Neg (negative), Bnd (boundary), State,
**Auto** = automated in the Postman collection (`postman/store-inventory-api.postman_collection.json`).

Every automated request additionally checks: response time < 1000 ms and `Content-Type: application/json`
(except `204 No Content`).

## 01 Products — positive

| ID    | Title | Steps / request | Expected result | P | Type | Auto |
|-------|-------|-----------------|-----------------|---|------|------|
| TC-01 | List products | `GET /api/products` | 200; JSON array; every item has `id, sku, name, quantity, lowStock` | H | Pos | ✅ |
| TC-02 | Create product | `POST /api/products` with valid body, SKU in lower case | 201; `Location` header ends with `/api/products/{id}`; SKU returned in upper case; `lowStock=false` | H | Pos | ✅ |
| TC-03 | Get product by id | `GET /api/products/{id}` of the created product | 200; same id, quantity 20, price 1.5 | H | Pos | ✅ |
| TC-04 | Update product | `PUT /api/products/{id}`: price 3.49, quantity 12, minStock 15 | 200; fields updated; `lowStock=true` (12 ≤ 15) | H | Pos | ✅ |
| TC-05 | Low-stock report | `GET /api/products/low-stock` | 200; contains the updated product; every item has `quantity ≤ minStock` | M | Pos | ✅ |
| TC-06 | Sell items | `POST /api/products/{id}/sell` `{"quantity": 2}` | 200; quantity 12 → 10 | H | State | ✅ |
| TC-07 | Filter by category | `GET /api/products?category=<category in lower case>` | 200; exactly 1 product (filter is case-insensitive) | M | Pos | ✅ |

## 02 Products — negative

| ID    | Title | Steps / request | Expected result | P | Type | Auto |
|-------|-------|-----------------|-----------------|---|------|------|
| TC-08 | Duplicate SKU | `POST /api/products` with an existing SKU | 409; standard error body | H | Neg | ✅ |
| TC-09 | Invalid fields | `POST` with `sku=""`, `price=-1`, `quantity=-5` | 400; `message="Validation failed"`; exactly 3 field errors: sku, price, quantity | H | Neg | ✅ |
| TC-10 | Price precision | `POST` with `price=1.999` | 400; error refers to `price` | M | Bnd | ✅ |
| TC-11 | Stock limit on create | `POST` with `quantity=1000001` | 400; error refers to `quantity` | M | Bnd | ✅ |
| TC-12 | Malformed JSON | `POST` with body `{"sku": "BROKEN", "price": }` | 400; `message="Malformed JSON request body"` | M | Neg | ✅ |
| TC-13 | Overselling | `POST /sell` `{"quantity": 11}` when stock is 10 | 409; message contains "Not enough stock"; stock unchanged | H | Bnd | ✅ |
| TC-14 | Sell zero | `POST /sell` `{"quantity": 0}` | 400 | M | Bnd | ✅ |
| TC-15 | Unknown id | `GET /api/products/999999` | 404; standard error body | M | Neg | ✅ |
| TC-16 | Non-numeric id | `GET /api/products/abc` | 400 (not 500) | M | Neg | ✅ |

## 03 Deliveries

| ID    | Title | Steps / request | Expected result | P | Type | Auto |
|-------|-------|-----------------|-----------------|---|------|------|
| TC-17 | Delivery with discrepancy | `POST /api/deliveries`: invoiced 10, received 8 | 201; `status=DISCREPANCY`; `discrepancyCount=1`; `difference=-2` | H | Pos | ✅ |
| TC-18 | Stock after delivery | `GET` product | quantity 10 + 8 = 18 (received, not invoiced, quantity is added) | H | State | ✅ |
| TC-19 | Get delivery by id | `GET /api/deliveries/{id}` | 200; invoice number matches | M | Pos | ✅ |
| TC-20 | Duplicate invoice | repeat `POST` with the same invoice number | 409 | H | Neg | ✅ |
| TC-21 | Unknown SKU | delivery with one valid and one unknown SKU | 404 | H | Neg | ✅ |
| TC-22 | Atomicity | `GET` product after TC-21 | quantity still 18 — the valid line was **not** applied | H | State | ✅ |
| TC-23 | Duplicate SKU lines | the same SKU twice in one delivery | 409 | M | Neg | ✅ |
| TC-24 | Empty delivery | `items: []` | 400 | M | Neg | ✅ |
| TC-25 | Matching delivery | invoiced 2, received 2 | 201; `status=OK`; `discrepancyCount=0` | H | Pos | ✅ |
| TC-26 | List deliveries | `GET /api/deliveries` | 200; contains both successful test invoices | L | Pos | ✅ |

## 04 Cleanup

| ID    | Title | Steps / request | Expected result | P | Type | Auto |
|-------|-------|-----------------|-----------------|---|------|------|
| TC-27 | Delete product | `DELETE /api/products/{id}` | 204, empty body | M | Pos | ✅ |
| TC-28 | Deleted product | `GET /api/products/{id}` | 404 | M | Neg | ✅ |

## Manual test cases

| ID    | Title | Steps | Expected result | P | Type | Auto |
|-------|-------|-------|-----------------|---|------|------|
| TC-29 | Sell exactly all stock | create product with quantity 5 → sell 5 | 200; quantity 0; `lowStock=true` | M | Bnd | ❌ |
| TC-30 | Stock limit via delivery | create product with quantity 999 999 → deliver 2 items | 409 "Stock limit exceeded"; quantity stays 999 999 | M | Bnd | ❌ |
| TC-31 | Data reset on restart | create a product → restart the API → list products | only the 5 demo products are present (in-memory DB) | L | State | ❌ |
| TC-32 | SKU case in delivery | deliver `"sku": "milk-1l"` | 201; line stored as `MILK-1L`; stock of MILK-1L increased | L | Pos | ❌ |

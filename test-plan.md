# Test Plan — Store Inventory API

| Item            | Value                                                         |
|-----------------|---------------------------------------------------------------|
| Product         | [store-inventory-api](https://github.com/me1atoninvomne/store-inventory-api) v1.0.0          |
| Document owner  | Denis Avlastimov                                              |
| Test types      | Functional API testing (manual + automated with Postman)      |

## 1. Objective

Verify that the Store Inventory API correctly manages products, sales and incoming deliveries,
rejects invalid input with clear error messages and never leaves stock in an inconsistent state.

## 2. Scope

**In scope**

- Product CRUD: create, read, list, filter by category, update, delete
- Sales: stock decrease, overselling protection
- Low-stock report
- Deliveries: receiving against an invoice, discrepancy detection, duplicate invoice protection,
  atomicity (an invalid delivery must not change any stock)
- Input validation and the standard error format (`timestamp`, `status`, `error`, `message`, `details`)
- HTTP status codes: 200, 201, 204, 400, 404, 409

**Out of scope**

- Performance / load testing (only a basic response-time check < 1000 ms)
- Security testing (the API has no authentication in v1.0.0)
- UI (the product is a backend API only)

## 3. Approach

| Technique                    | Where it is used                                                   |
|------------------------------|--------------------------------------------------------------------|
| Equivalence partitioning     | valid vs. invalid quantities, prices, SKUs                          |
| Boundary value analysis      | quantity 0 / 1, stock limit 1 000 000 / 1 000 001, price 2 vs 3 decimals, selling exactly all stock |
| Negative testing             | malformed JSON, non-numeric IDs, unknown IDs and SKUs, duplicates   |
| State-based testing          | stock value checked after sale, delivery and rejected delivery      |
| Exploratory testing          | time-boxed sessions, notes added to bug reports if issues are found |

Automated checks are implemented as a Postman collection and run from the command line with **Newman**.
Each run generates a unique `runId`, so the collection can be executed repeatedly without restarting the API.

## 4. Test environment

- API started locally: `./gradlew bootRun` → `http://localhost:8080`
- H2 in-memory database with 5 demo products (reset on every restart)
- Postman (desktop) or Newman (CLI), environment file `postman/local.postman_environment.json`

## 5. Entry and exit criteria

**Entry:** the API builds, `./gradlew test` passes, the application starts and `GET /api/products` returns 200.

**Exit:**
- all High-priority test cases executed
- 100 % of High-priority cases passed, or every failure has a bug report
- no open Critical/High bugs

## 6. Risks

| Risk                                                        | Mitigation                                         |
|-------------------------------------------------------------|----------------------------------------------------|
| In-memory DB resets on restart → test data disappears       | Collection creates its own data (no dependency on demo data except TC-01) |
| Tests depend on execution order (create → sell → delete)    | Folders are numbered; run the whole collection in order |
| Concurrent sales of the same product are not covered        | Documented as a known gap for a future iteration    |

## 7. Deliverables

- `test-cases.md` — test case specification (TC-01 … TC-32)
- `postman/` — automated collection + environment
- `bug-reports/` — bug report template and any reported defects
- `reports/` — Newman run reports (generated, see `reports/README.md`)

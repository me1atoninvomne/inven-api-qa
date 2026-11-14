# Inventory API — QA

Test documentation and automated API tests for
[store-inventory-api](https://github.com/me1atoninvomne/store-inventory-api) — a Spring Boot REST API
for managing stock, sales and supplier deliveries.

## What is inside

| File / folder | Content |
|---------------|---------|
| [`test-plan.md`](test-plan.md) | Scope, approach (equivalence partitioning, boundary values, negative and state-based testing), environment, entry/exit criteria, risks |
| [`test-cases.md`](test-cases.md) | 32 test cases (28 automated, 4 manual) with priority and type |
| [`postman/`](postman) | Postman collection with **28 requests and ~110 assertions** + local environment |
| [`bug-reports/`](bug-reports) | Bug report template (and reported defects) |
| [`reports/`](reports) | How to generate Newman HTML / JUnit reports |

## Coverage at a glance

- Product CRUD, category filter, low-stock report
- Sales: stock decrease, overselling protection, boundary values (0, exact stock, stock + 1)
- Deliveries: discrepancy detection (invoiced vs. received), duplicate invoices, unknown and duplicated SKUs
- **Atomicity**: a rejected delivery must not change any stock (TC-21 → TC-22)
- Validation: blank fields, negative numbers, price precision, stock limit, malformed JSON, wrong ID type
- Contract: standard error body (`timestamp, status, error, message, details`), status codes, `Location` header,
  `Content-Type`, response time

## How to run

1. Start the API (see the API repository): `./gradlew bootRun`
2. **Postman:** import `postman/store-inventory-api.postman_collection.json` and
   `postman/local.postman_environment.json`, select the environment, run the collection.
3. **Command line (Newman):**

   ```bash
   npm install -g newman
   newman run postman/store-inventory-api.postman_collection.json -e postman/local.postman_environment.json
   ```

The collection creates its own test data with a unique `runId`, so it can be run many times
without restarting the API. Requests depend on each other (create → sell → deliver → delete),
so run the folders in order.

## Tools

Postman, Newman, JSON test scripts (Chai assertions), Markdown documentation.

---



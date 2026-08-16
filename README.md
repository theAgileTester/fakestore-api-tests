# FakeStore API Test Suite

Automated API test suite for `https://fakestoreapi.com`, built with Postman and Newman, covering Products, Carts, Users, and Auth.

## Test Strategy

27 test cases covering positive, negative, and boundary scenarios across Products, Carts, Users, and Auth — including CSV-driven boundary testing, two full chained CRUD lifecycles (Carts and Users, including a PUT/update step), and pre-request scripts for collision-free repeatable test data.

| Category | Coverage |
|---|---|
| Positive | Valid IDs, valid category filters, valid login, valid creation across all resources |
| Negative | Out-of-range IDs, invalid category, wrong password, missing required fields |
| Boundary | Lower/upper ID limits on Products, Carts, and Users; CSV-driven boundary sweep on Products |
| Chaining | Full create → read → update → delete lifecycles on Carts and Users |
| Data-driven | CSV-fed boundary test across the full valid product ID range |

## Key Findings

**1. No 404 behavior.** All negative test cases (out-of-range or non-existent IDs) return 200 instead of 404, consistently across Products, Carts, and Users. The API returns an empty string, `null`, or an empty array depending on the resource, meaning consumers must inspect the response body rather than relying on status codes to detect "not found" conditions.

**2. JSON syntax is validated, but required fields are not — confirmed across all three writable resources.** Malformed JSON is rejected with 400. A syntactically valid but incomplete object is still accepted with 201 on Products, Users, AND Carts — none of the three perform field-level validation.

**3. No write persistence.** POST, PUT, and DELETE requests return success-looking responses, but the underlying data is never saved. Confirmed via two independent chained sequences (Carts and Users) — a resource created via POST could not be retrieved via a follow-up GET, which instead returned `null` or an empty body.

**4. Status codes alone are insufficient for validation.** Because every request returns 200 regardless of resource existence (Finding 1), boundary and existence tests must inspect the response body, not just the status code — a real pitfall when testing this API without deeper investigation.

**5. Users' create endpoint returns a minimal response, unlike Products.** `POST /products` echoes back the full submitted object. `POST /users` returns only a generated `{"id": ...}`, omitting all submitted fields — an inconsistency worth knowing before building automation against this API.

**6. GitHub Actions CI runs are blocked by Cloudflare bot protection.** This suite passes consistently (93/93 assertions) when run locally via Postman or Newman. When executed via GitHub Actions, requests are blocked by Cloudflare's bot-protection challenge — the API returns an HTML "Just a moment..." interstitial page (HTTP 403) instead of real JSON responses, since FakeStoreAPI applies stricter anti-bot measures to traffic from shared cloud CI IP ranges than to individual/residential traffic. This is a known constraint of testing free public APIs from CI infrastructure, not a defect in the test suite itself — the same 27 test cases pass reliably when run locally.

## How to Run

### Postman
1. Import `Fakestore Api Test Suite.postman_collection.json`
2. Import `FakestoreApi.postman_environment.json`
3. Run via Collection Runner — set **Delay to 500ms** to avoid request-timing race conditions
4. For the CSV-driven boundary test (TC_21), attach `product_boundary.csv` in the Runner's Data field

### Newman (command line)
```bash
npm install -g newman newman-reporter-html
newman run "Fakestore Api Test Suite.postman_collection.json" -e "FakestoreApi.postman_environment.json" --delay-request 500 -r html
```

### CI/CD
Runs automatically on every push via GitHub Actions (`.github/workflows/api-tests.yml`). See Finding 6 above regarding expected CI behavior against this particular API.

## Tech Stack
Postman · Newman · GitHub Actions · JavaScript (Chai assertions)

## Author
Built as part of a self-directed API testing practice project, applying skills from a 35-day Postman/Newman/CI-CD learning program.

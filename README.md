## Test Strategy
27 test cases covering positive, negative, and boundary scenarios across Products, Carts, Users, and Auth — including CSV-driven boundary testing, two full chained CRUD lifecycles (Carts and Users, including a PUT/update step), and pre-request scripts for collision-free repeatable test data.

## Key Findings

**1. No 404 behavior.** All negative test cases (out-of-range or non-existent IDs) return 200 instead of 404, consistently across Products, Carts, and Users. The API returns an empty string, `null`, or an empty array depending on the resource.

**2. JSON syntax is validated, but required fields are not — confirmed across all three writable resources.** Malformed JSON is rejected with 400. A syntactically valid but incomplete object is accepted with 201 on Products, Users, AND Carts — none of the three perform field-level validation.

**3. No write persistence.** POST, PUT, and DELETE requests return success responses, but the underlying data is never saved. Confirmed via two independent chained sequences (Carts and Users) — a resource created via POST could not be retrieved via a follow-up GET.

**4. Status codes alone are insufficient for validation.** Because every request returns 200 regardless of resource existence, boundary and existence tests must inspect the response body, not just the status code.

**5. Users' create endpoint returns a minimal response, unlike Products.** `POST /products` echoes back the full submitted object; `POST /users` returns only a generated `{"id": ...}`, omitting all submitted fields — an inconsistency worth knowing before building automation against this API.

**6. GitHub Actions CI runs are blocked by Cloudflare bot protection.** This suite passes 100% (93/93 assertions) when run locally via Postman or Newman. When executed via GitHub Actions, requests are blocked by a Cloudflare "Just a moment..." challenge page (HTTP 403), since FakeStoreAPI applies stricter anti-bot measures to shared cloud CI IP ranges than to individual traffic. This is a known constraint of testing free public APIs from CI infrastructure, not a defect in the test suite.

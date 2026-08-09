# FakeStore API Test Suite

Automated API test suite for `https://fakestoreapi.com`, built with Postman and Newman, covering Products, Carts, Users, and Auth.

## Test Strategy
20 test cases covering positive, negative, and boundary scenarios across all 4 resource areas, including a full chained create → read → delete sequence on Carts.

## Key Findings

**1. No 404 behavior.** All negative test cases (out-of-range or non-existent IDs) return 200 instead of 404, consistently across Products, Carts, and Users. The API returns an empty string, `null`, or an empty array depending on the resource, meaning consumers must inspect the response body rather than relying on status codes to detect "not found" conditions.

**2. JSON syntax is validated, but required fields are not.** Malformed JSON is rejected with 400. However, a syntactically valid but incomplete object (missing required fields) is still accepted with 201 — the API validates structure, not completeness.

**3. No write persistence.** POST and DELETE requests return success responses, but the underlying data is never saved. A cart created via POST could not be retrieved via a follow-up GET.

**4. Status codes alone are insufficient for validation.** Because every request returns 200 regardless of resource existence, boundary and existence tests must inspect the response body, not just the status code — a real pitfall when testing this API.

## How to Run

### Postman
Import the collection and environment files, then run via Collection Runner (set Delay to 500ms).

### Newman
\`\`\`bash
npm install -g newman newman-reporter-html
newman run "Fakestore Api Test Suite.postman_collection.json" -e "FakestoreApi.postman_environment.json" --delay-request 500 -r html
\`\`\`

### CI/CD
Runs automatically on every push via GitHub Actions.

## Tech Stack
Postman · Newman · GitHub Actions · JavaScript (Chai assertions)

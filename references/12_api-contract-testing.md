# 12 — API Contract Testing: Postman · Newman · OpenAPI

---

## What API Contract Testing Covers

| Layer | What to test |
|-------|-------------|
| **Schema** | Response structure matches the defined contract |
| **Status codes** | Correct HTTP code per scenario |
| **Data types** | Fields are the right type (string, number, boolean, null) |
| **Required fields** | Non-nullable fields always present |
| **Auth enforcement** | Protected routes return 401/403 without valid token |
| **Error format** | Error responses follow consistent schema |
| **Edge cases** | Empty arrays, null values, max-length strings |

---

## Postman Collection Structure

Organize by resource, not by HTTP method:

```
[Project] API Tests
├── 🔐 Auth
│   ├── POST /auth/login — valid → 200 + token
│   ├── POST /auth/login — invalid → 401
│   ├── POST /auth/login — brute force → 429
│   ├── POST /auth/logout → 200, token invalidated
│   └── POST /auth/refresh — expired → 401
├── 👤 Users
│   ├── GET /users/me — authenticated → 200
│   ├── GET /users/me — no token → 401
│   ├── PATCH /users/me — valid update → 200
│   └── GET /users/:id — another user → 403
├── 📦 [Resource]
│   ├── GET /[resource] — paginated → 200
│   ├── POST /[resource] — valid → 201
│   ├── POST /[resource] — invalid → 400 with field errors
│   ├── PUT /[resource]/:id — unauthorized → 403
│   └── DELETE /[resource]/:id — not found → 404
└── 🏥 Health
    └── GET /health → 200
```

---

## Postman Test Scripts

### Response Schema Validation

```javascript
// Tests tab in Postman

pm.test('Status 200', () => {
  pm.response.to.have.status(200);
});

pm.test('Response time < 500ms', () => {
  pm.expect(pm.response.responseTime).to.be.below(500);
});

pm.test('Schema is valid', () => {
  const schema = {
    type: 'object',
    required: ['id', 'email', 'createdAt'],
    properties: {
      id:        { type: 'string' },
      email:     { type: 'string', format: 'email' },
      name:      { type: 'string' },
      createdAt: { type: 'string', format: 'date-time' },
      // Sensitive fields must NOT be present
    },
    additionalProperties: false,  // strict — no unexpected fields
  };

  pm.response.to.have.jsonSchema(schema);
});

pm.test('No sensitive fields leaked', () => {
  const body = pm.response.json();
  pm.expect(body).to.not.have.property('password');
  pm.expect(body).to.not.have.property('passwordHash');
  pm.expect(body).to.not.have.property('apiKey');
});
```

### Auth Token Extraction (save for subsequent requests)

```javascript
// In POST /auth/login Tests tab:
const body = pm.response.json();

pm.test('Token exists in response', () => {
  pm.expect(body.accessToken).to.be.a('string');
  pm.expect(body.accessToken.split('.')).to.have.lengthOf(3); // valid JWT structure
});

pm.collectionVariables.set('ACCESS_TOKEN', body.accessToken);
pm.collectionVariables.set('REFRESH_TOKEN', body.refreshToken);
```

### Error Response Validation

```javascript
// For any 4xx/5xx test:
pm.test('Error response has correct structure', () => {
  const body = pm.response.json();
  pm.expect(body).to.have.property('error');
  pm.expect(body.error).to.be.a('string');
  // Must NOT expose internals
  pm.expect(body).to.not.have.property('stack');
  pm.expect(body).to.not.have.property('sql');
  pm.expect(JSON.stringify(body)).to.not.include('prisma');
  pm.expect(JSON.stringify(body)).to.not.include('supabase');
});
```

### Pagination Contract

```javascript
pm.test('Pagination structure valid', () => {
  const body = pm.response.json();
  pm.expect(body).to.have.property('data').that.is.an('array');
  pm.expect(body).to.have.property('pagination');
  pm.expect(body.pagination).to.have.keys(['cursor', 'hasMore', 'total']);
  pm.expect(body.pagination.hasMore).to.be.a('boolean');
});

pm.test('Results count respects limit param', () => {
  const body = pm.response.json();
  const limit = parseInt(pm.request.url.query.get('limit') || '20');
  pm.expect(body.data.length).to.be.at.most(limit);
});
```

---

## Environments

Create one environment per deployment target:

```
[Project] — Local
  BASE_URL      = http://localhost:3000
  TEST_EMAIL    = test@example.com
  TEST_PASSWORD = TestPassword123!

[Project] — Staging
  BASE_URL      = https://staging.yourapp.com
  TEST_EMAIL    = {{$processEnv.STAGING_TEST_EMAIL}}
  TEST_PASSWORD = {{$processEnv.STAGING_TEST_PASSWORD}}

[Project] — Production (read-only checks only)
  BASE_URL      = https://yourapp.com
  # No write operations in production collection
```

---

## Newman — CI Execution

Newman runs Postman collections from the command line.

### Install

```bash
npm install --save-dev newman newman-reporter-htmlextra
```

### Run Collection

```bash
# Export collection from Postman → Save as collection.json
newman run collection.json \
  --environment staging.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/api-report.html \
  --bail                    # stop on first failure
```

### GitHub Actions Integration

```yaml
# .github/workflows/api-tests.yml
name: API Contract Tests

on:
  pull_request:
    branches: [main, dev]
  schedule:
    - cron: '0 6 * * *'    # Daily smoke test at 6am

jobs:
  api-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm install -g newman newman-reporter-htmlextra

      - name: Run API contract tests
        run: |
          newman run tests/api/collection.json \
            --env-var "BASE_URL=${{ secrets.STAGING_URL }}" \
            --env-var "TEST_EMAIL=${{ secrets.TEST_EMAIL }}" \
            --env-var "TEST_PASSWORD=${{ secrets.TEST_PASSWORD }}" \
            --reporters cli,htmlextra \
            --reporter-htmlextra-export newman-report.html

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: api-test-report
          path: newman-report.html
```

---

## OpenAPI Contract Validation

If your API has an OpenAPI/Swagger spec, validate responses against it automatically.

### Setup with `openapi-fetch` + Zod

```typescript
// Auto-generate types from OpenAPI spec
// npx openapi-typescript openapi.yaml -o src/types/api.ts

import createClient from 'openapi-fetch';
import type { paths } from './types/api';

const client = createClient<paths>({ baseUrl: process.env.BASE_URL });

// TypeScript will error at compile time if response doesn't match spec
const { data, error } = await client.GET('/users/{id}', {
  params: { path: { id: '123' } },
});
```

### Validate Spec Coverage (Spectral)

```bash
npm install --save-dev @stoplight/spectral-cli

# Create .spectral.yaml
echo "extends: ['spectral:oas']" > .spectral.yaml

# Run lint on your OpenAPI spec
npx spectral lint openapi.yaml
```

Common Spectral rules enforced:
- All paths have `operationId`
- All responses have schemas
- No `any` type schemas
- All parameters described
- Error responses (4xx/5xx) defined

---

## API Test Checklist

### Per Endpoint
- [ ] 200 with valid input + valid auth
- [ ] 400 with invalid input (each required field missing separately)
- [ ] 400 with invalid data types
- [ ] 401 with no Authorization header
- [ ] 401 with expired token
- [ ] 403 with valid token but wrong role/ownership
- [ ] 404 for non-existent resource ID
- [ ] 429 when rate limit exceeded
- [ ] 500 does NOT expose stack trace or internal details
- [ ] Response schema matches contract (no missing/extra fields)
- [ ] Response time < 500ms (staging, simple queries)

### Security Checks per Endpoint
- [ ] IDOR — resource owned by another user → 403
- [ ] Admin endpoint with user token → 403
- [ ] SQL/NoSQL injection in body params → no 500 with DB details
- [ ] Oversized payload → 413 or 400 (not hang/crash)
- [ ] Malformed JSON → 400 (not 500)

---

## API Contract Test Output Format

```markdown
## API Contract Test Report — [Service / Version]
**Tool:** Newman / Postman  
**Reviewer:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  
**Environment:** [Staging / Production]  
**Base URL:** [URL]

### Summary
| Metric | Value |
|--------|-------|
| Total requests | XX |
| Passed | XX |
| Failed | XX |
| Avg response time | XXXms |
| Slowest endpoint | [endpoint] — XXXms |

### Failures

#### ❌ [METHOD] [/endpoint] — [Test name]
- **Expected:** [status/schema/value]
- **Actual:** [what was returned]
- **Request:** [curl equivalent]
- **Response:** [relevant excerpt]
- **Severity:** S[X]
- **Linked bug:** BUG-[XXX]

### Schema Violations
| Endpoint | Field | Expected Type | Actual |
|----------|-------|--------------|--------|
| GET /users/me | `createdAt` | `date-time` string | number (timestamp) |

### Performance
| Endpoint | Avg | P95 | Status |
|----------|-----|-----|--------|
| POST /auth/login | 145ms | 320ms | ✅ |
| GET /orders | 89ms | 180ms | ✅ |

### Verdict
🟢 CONTRACT VALID — all schemas match / 🔴 CONTRACT BROKEN — [X] violations
```

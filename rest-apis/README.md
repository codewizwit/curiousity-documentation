# REST APIs

_Designing interfaces that make sense._

## What's a REST API? (The Restaurant Menu Metaphor)

Imagine ordering food at a restaurant:

- **Menu** (API documentation) — Shows what's available
- **Order** (HTTP request) — You tell the waiter what you want
- **Kitchen** (server) — Prepares your order
- **Dish delivered** (HTTP response) — You receive what you ordered

**A REST API is the menu and ordering system for software.** It defines how clients (web apps, mobile apps) can request data or actions from a server.

**REST** = Representational State Transfer (don't worry, the acronym doesn't help). What matters: **resources**, **HTTP methods**, and **statelessness**.

---

## Core Principles

### 1. Resources & URIs

Everything is a **resource** (user, post, order) identified by a **URI** (Uniform Resource Identifier).

```
https://api.example.com/users          # Collection of users
https://api.example.com/users/123      # Specific user
https://api.example.com/users/123/posts # User's posts
```

**Think nouns, not verbs:**
- ✅ `/users` (noun)
- ❌ `/getUsers` (verb)

---

### 2. HTTP Methods (Verbs)

Use HTTP methods to describe actions:

| Method | Action | Example |
|--------|--------|---------|
| **GET** | Retrieve data | `GET /users/123` |
| **POST** | Create new resource | `POST /users` |
| **PUT** | Replace entire resource | `PUT /users/123` |
| **PATCH** | Update part of resource | `PATCH /users/123` |
| **DELETE** | Remove resource | `DELETE /users/123` |

---

### 3. Stateless

Each request is **independent** — the server doesn't remember previous requests.

```
// ❌ Stateful (server remembers)
Request 1: POST /login → Server stores session
Request 2: GET /profile → Server uses stored session

// ✅ Stateless (client sends auth every time)
Request 1: GET /profile (Authorization: Bearer token123)
Request 2: GET /orders (Authorization: Bearer token123)
```

**Why stateless?** Easier to scale (no shared session state), more reliable (no session expiration issues).

---

### 4. HTTP Status Codes

Response codes indicate success or failure:

**2xx Success:**
- `200 OK` — Request succeeded
- `201 Created` — Resource created
- `204 No Content` — Success, but no body (used for DELETE)

**4xx Client Errors:**
- `400 Bad Request` — Invalid data
- `401 Unauthorized` — Missing/invalid authentication
- `403 Forbidden` — Authenticated but not allowed
- `404 Not Found` — Resource doesn't exist
- `409 Conflict` — Resource already exists or version mismatch

**5xx Server Errors:**
- `500 Internal Server Error` — Something broke on the server
- `503 Service Unavailable` — Server overloaded or down

---

## API Design Patterns

### CRUD Operations

**CRUD** = Create, Read, Update, Delete

```
POST   /users          → Create user
GET    /users          → List all users
GET    /users/123      → Get specific user
PUT    /users/123      → Replace user
PATCH  /users/123      → Update user fields
DELETE /users/123      → Delete user
```

---

### Nested Resources

Model relationships through URL structure:

```
GET /users/123/posts           → Get posts by user 123
POST /users/123/posts          → Create post for user 123
GET /posts/456/comments        → Get comments on post 456
```

**Rule:** Don't nest more than 2-3 levels deep (readability).

---

### Filtering, Sorting, Pagination

Use query parameters for list operations:

```
GET /users?role=admin                     # Filter
GET /users?sort=createdAt&order=desc      # Sort
GET /users?page=2&limit=20                # Pagination
GET /users?search=john&status=active      # Search + filter
```

---

### Versioning

Support multiple API versions for backwards compatibility:

**URL path:**
```
https://api.example.com/v1/users
https://api.example.com/v2/users
```

**Header:**
```
GET /users
Accept: application/vnd.myapi.v1+json
```

**Query parameter:**
```
GET /users?version=1
```

**Recommendation:** URL path versioning (most visible and explicit).

---

## Request & Response Format

### JSON Standard

```json
// Request: POST /users
{
  "name": "Alice",
  "email": "alice@example.com",
  "role": "admin"
}

// Response: 201 Created
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "role": "admin",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

---

### Error Responses

Consistent error structure:

```json
// 400 Bad Request
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Email format is invalid",
    "field": "email"
  }
}

// 404 Not Found
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 does not exist"
  }
}
```

---

### Pagination Metadata

```json
// GET /users?page=2&limit=20
{
  "data": [
    { "id": 21, "name": "Alice" },
    { "id": 22, "name": "Bob" }
  ],
  "pagination": {
    "page": 2,
    "limit": 20,
    "totalPages": 5,
    "totalItems": 100,
    "nextPage": "/users?page=3&limit=20",
    "prevPage": "/users?page=1&limit=20"
  }
}
```

---

## Authentication & Authorization

### Bearer Tokens (JWT)

```
GET /profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Flow:**
1. Client sends credentials → Server returns token
2. Client includes token in `Authorization` header on every request
3. Server validates token and grants access

---

### API Keys

```
GET /users
X-API-Key: abc123def456
```

**Use case:** Simple authentication for third-party integrations.

---

### OAuth 2.0

For delegated access (e.g., "Sign in with Google"):

```
1. User grants permission → App receives access token
2. App uses token to access user's data on their behalf
```

---

## REST API Best Practices

### 1. Use Nouns for Resources

```
✅ GET /users
❌ GET /getUsers

✅ POST /orders
❌ POST /createOrder
```

---

### 2. Plural Names for Collections

```
✅ GET /users
❌ GET /user

✅ GET /posts/123/comments
❌ GET /posts/123/comment
```

---

### 3. Use HTTP Methods Correctly

```
✅ GET /users         # Read (safe, idempotent)
✅ POST /users        # Create (not idempotent)
✅ PUT /users/123     # Replace (idempotent)
✅ PATCH /users/123   # Update (idempotent)
✅ DELETE /users/123  # Delete (idempotent)
```

**Idempotent** = Repeating the same request has the same effect as making it once.

---

### 4. Return Appropriate Status Codes

```
✅ 200 OK for successful GET
✅ 201 Created for successful POST
✅ 204 No Content for successful DELETE
✅ 400 Bad Request for invalid input
✅ 404 Not Found for missing resource
❌ 200 OK with error in body
```

---

### 5. Provide Meaningful Error Messages

```json
// ❌ Vague
{ "error": "Invalid request" }

// ✅ Specific
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required and must be valid",
    "field": "email",
    "value": "invalid-email"
  }
}
```

---

### 6. Use HTTPS

Always use HTTPS in production (encrypts data in transit).

---

### 7. Document Your API

Use OpenAPI/Swagger for interactive documentation:

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
```

---

## Common API Patterns

### HATEOAS (Hypermedia)

Include links to related resources:

```json
{
  "id": 123,
  "name": "Alice",
  "links": {
    "self": "/users/123",
    "posts": "/users/123/posts",
    "followers": "/users/123/followers"
  }
}
```

**Benefit:** Clients discover API capabilities dynamically.

---

### Rate Limiting

Prevent abuse with rate limits:

```
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
```

---

### Caching

Use ETags and `Cache-Control` headers:

```
GET /users/123
If-None-Match: "abc123"

# If unchanged:
HTTP/1.1 304 Not Modified

# If changed:
HTTP/1.1 200 OK
ETag: "def456"
{ "id": 123, "name": "Alice Updated" }
```

---

### Webhooks

Push notifications instead of polling:

```
Client registers: POST /webhooks { "url": "https://client.com/callback" }
Event occurs → Server sends: POST https://client.com/callback
```

---

## REST vs GraphQL vs gRPC

| Feature | REST | GraphQL | gRPC |
|---------|------|---------|------|
| **Data format** | JSON | JSON | Protocol Buffers |
| **Querying** | Multiple endpoints | Single endpoint, custom queries | Typed service methods |
| **Over/under-fetching** | Common | Solved (request exactly what you need) | Defined by service schema |
| **Versioning** | URL or header | Schema evolution | Schema versioning |
| **Use case** | General purpose, CRUD | Complex data requirements, mobile apps | Microservices, high performance |

---

## Testing REST APIs

### cURL

```bash
# GET request
curl https://api.example.com/users

# POST request
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@example.com"}'

# With authentication
curl https://api.example.com/profile \
  -H "Authorization: Bearer token123"
```

---

### Postman / Insomnia

GUI tools for API testing:
- Organize requests into collections
- Save environments (dev, staging, prod)
- Write test scripts
- Generate documentation

---

### Automated Testing

```javascript
// Jest example
describe('Users API', () => {
  test('GET /users returns list', async () => {
    const response = await fetch('https://api.example.com/users');
    expect(response.status).toBe(200);

    const data = await response.json();
    expect(Array.isArray(data)).toBe(true);
  });

  test('POST /users creates user', async () => {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: 'Alice', email: 'alice@example.com' }),
    });

    expect(response.status).toBe(201);
    const user = await response.json();
    expect(user.id).toBeDefined();
  });
});
```

---

## Resources

- [REST API Tutorial](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [API Design Guide (Google)](https://cloud.google.com/apis/design)
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)

---

_Designing interfaces one resource at a time._ 🌭

← [Back to Home](../README.md) | [JWTs](../jwts/README.md) | [NestJS](../nestjs/README.md)

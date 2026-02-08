# 🚀 Backend Routing - Complete Revision Notes

> **Repository**: Understanding Backend Routing Concepts  
> **Purpose**: Comprehensive guide to routing in backend development  
> **Level**: Beginner to Advanced
> **Last Updated**: February 2026

---

## 📚 Table of Contents

1. [URL to Server Logic Mapping](#1-url-to-server-logic-mapping)
2. [Relation Between HTTP Methods and Routes](#2-relation-between-http-methods-and-routes)
3. [Route Components](#3-route-components)
4. [Route Types](#4-route-types)
5. [API Versioning](#5-api-versioning)
6. [Deprecation Best Practices](#6-deprecation-best-practices)
7. [Route Grouping](#7-route-grouping)
8. [Securing Routes](#8-securing-routes)
9. [Route Matching Performance Optimization](#9-route-matching-performance-optimization)
10. [Spring Boot Code Examples](#10-spring-boot-code-examples)

---

## 1. URL to Server Logic Mapping

### What is Routing?

Routing maps **incoming HTTP requests** to **specific server-side handlers**.

**Flow**:
```
Client Request (Method + URL)
         ↓
    Router Analysis
         ↓
    Handler Match
         ↓
  Business Logic Execution
         ↓
    Server Response
```

### Real-World Analogy

Think of a **hotel receptionist**:
- Guest: "I need room service" → **Request**
- Receptionist: Checks room number → **Routing**
- Calls room service department → **Handler**
- Room service delivered → **Response**

### Key Components

| Component | Purpose | Example |
|-----------|---------|---------|
| **HTTP Method** | Action type | GET, POST, DELETE |
| **URL Path** | Resource location | `/users/123` |
| **Handler** | Business logic | `getUser Function()` |
| **Router** | Mapping system | Connects request to handler |

---

## 2. Relation Between HTTP Methods and Routes

### HTTP Methods Overview

| Method | Purpose | Idempotent | Safe |
|--------|---------|-----------|------|
| **GET** | Retrieve data | ✅ | ✅ |
| **POST** | Create resource | ❌ | ❌ |
| **PUT** | Replace resource | ✅ | ❌ |
| **PATCH** | Update partial | ❌ | ❌ |
| **DELETE** | Remove resource | ✅ | ❌ |

**Idempotent**: Multiple identical requests have same effect as single request  
**Safe**: Doesn't modify server state

### Method + Route = Unique Action

Same route, different methods = different operations:

```
GET    /api/users/123  → Fetch user 123
PUT    /api/users/123  → Update user 123
PATCH  /api/users/123  → Modify user 123
DELETE /api/users/123  → Delete user 123
```

### RESTful Resource Operations

| Operation | Method | Route | Description |
|-----------|--------|-------|-------------|
| List | GET | `/products` | All products |
| Read | GET | `/products/123` | One product |
| Create | POST | `/products` | New product |
| Replace | PUT | `/products/123` | Full update |
| Modify | PATCH | `/products/123` | Partial update |
| Delete | DELETE | `/products/123` | Remove product |

---

## 3. Route Components

### URL Anatomy

```
https://api.example.com:443/api/v1/users/123?page=1&sort=name#top
│────┘ │─┘ │─────────┘│─┘│──┘│─┘│───┘│──┘│────────────────┘│──┘
│      │   │           │  │   │  │    │   │                 └─ Fragment
│      │   │           │  │   │  │    │   └─ Query Parameters
│      │   │           │  │   │  │    └─ Path Parameter
│      │   │           │  │   │  └─ Resource
│      │   │           │  │   └─ Version
│      │   │           │  └─ Base Path
│      │   │           └─ Port
│      │   └─ Domain
│      └─ Subdomain
└─ Protocol
```

### 3.A Path Parameters

**Definition**: Variable segments in the URL path that identify specific resources.

**Syntax** (framework-dependent):
- `/users/:id` (Express, Node.js)
- `/users/{id}` (Spring Boot, Java)
- `/users/<id>` (Flask, Python)

**Characteristics**:
- ✅ Mandatory (required for route match)
- ✅ Part of URL structure
- ✅ Resource identification
- ❌ Cannot be optional

**Examples**:

| Use Case | Pattern | Matches |
|----------|---------|---------|
| User profile | `/users/{userId}` | `/users/12345` |
| Product | `/products/{id}` | `/products/ABC-123` |
| Order | `/orders/{orderId}` | `/orders/ORD-2024-001` |
| Nested | `/users/{userId}/posts/{postId}` | `/users/1/posts/99` |

**When to Use**:
- Resource identifiers (IDs, slugs)
- Hierarchical relationships
- SEO-friendly URLs

### 3.B Query Parameters

**Definition**: Optional key-value pairs after `?` for filtering/modifying responses.

**Syntax**:
```
/resource?key1=value1&key2=value2
```

**Characteristics**:
- ✅ Optional (not required)
- ✅ Multiple values possible
- ✅ Order-independent
- ❌ Less SEO-friendly than path params

**Examples**:

| Purpose | Query String | Use Case |
|---------|--------------|----------|
| Pagination | `?page=2&size=20` | Page 2, 20 items |
| Filtering | `?category=books&author=Smith` | Filter results |
| Sorting | `?sort=price&order=desc` | Sort by price descending |
| Search | `?q=laptop&brand=dell` | Search products |
| Date range | `?from=2024-01-01&to=2024-12-31` | Filter by dates |

**Special Characters** (URL encoding):
- Space → `%20` or `+`
- `&` → `%26`
- `=` → `%3D`

**Path vs Query Parameters**:

| Aspect | Path | Query |
|--------|------|-------|
| **Required** | Yes | No |
| **Purpose** | Identify resource | Filter/modify |
| **SEO** | Better | Worse |
| **Caching** | Easier | Complex |
| **Example** | `/users/123` | `/users?age=25` |

---

## 4. Route Types

### 4.1 Static Routes

**Definition**: Fixed URL paths with no variables.

```
/api/products    ← Always the same
/api/health      ← Health check
/about           ← About page
```

**Characteristics**:
- Fast matching (exact string comparison)
- Predictable behavior
- Highly cacheable

**When to Use**: Lists, configuration endpoints, health checks, static content

---

### 4.2 Dynamic Routes

**Definition**: Routes with variable segments.

```
/users/:id              ← Variable: id
/products/{productId}   ← Variable: productId
```

**Examples**:
- `/users/123` → userId = "123"
- `/products/ABC` → productId = "ABC"
- `/users/123/posts/456` → userId = "123", postId = "456"

**When to Use**: Individual resource access, detail pages

---

### 4.3 Nested Routes

**Definition**: Hierarchical resource relationships.

```
/users/{userId}/posts/{postId}
```

**Meaning**: Post belongs to specific user.

**Levels**:
```
/companies/{companyId}                          → Company
/companies/{companyId}/departments/{deptId}     → Department in company
/companies/{companyId}/departments/{deptId}/employees → Employees in department
```

**When to Use**: Parent-child relationships, scoped resources

**Best Practices**:
- ✅ Limit to 2-3 nesting levels
- ✅ Validate parent exists
- ❌ Don't over-nest (avoid 4+ levels)

---

### 4.4 Hierarchical Routes

**Definition**: Organizational grouping, not ownership.

```
/admin/users        ← Admin section
/admin/settings     ← Admin settings
/api/public/data    ← Public API
/api/internal/logs  ← Internal API
```

**Difference from Nested**:
- **Nested**: Data ownership (user owns posts)
- **Hierarchical**: Logical organization (admin area)

---

### 4.5 Wildcard/Catch-All Routes

**Definition**: Matches any unmatched path.

**Syntax**:
- `/**` (Spring Boot)
- `*` (Express)
- `/<path:path>` (Flask)

**Purpose**:
- 404 handlers
- SPA fallback routing
- Logging unmatched requests

**Example**:
```
GET /invalid/route → Catch-all returns 404
```

**Best Practices**:
- ✅ Always define one
- ✅ Return helpful error messages
- ✅ Log 404s for monitoring
- ❌ Must be defined last (lowest priority)

---

### 4.6 Regex Routes

**Definition**: Pattern-based matching with regular expressions.

**Examples**:

| Pattern | Matches | Doesn't Match |
|---------|---------|---------------|
| `/users/{id:\\d+}` | `/users/123` | `/users/abc` |
| `/products/{code:[A-Z]{3}-\\d{4}}` | `/products/ABC-1234` | `/products/abc-123` |
| `/date/{date:\\d{4}-\\d{2}-\\d{2}}` | `/date/2024-12-31` | `/date/24-12-31` |

**When to Use**:
- Format validation (UUIDs, codes, dates)
- Legacy URL support

**Considerations**:
- ⚠️ Slower than simple routes
- ⚠️ Complex to debug
- ⚠️ Not all frameworks support them

---

## 5. API Versioning

### Why Version APIs?

**Breaking changes require versioning**:
- Changed response structure
- Removed/renamed fields
- Different authentication
- Modified behavior

### Comparison

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URI** | `/api/v1/users` | Visible, cacheable | URL pollution |
| **Header** | `X-API-Version: 1` | Clean URLs | Less visible |
| **Query** | `/api/users?v=1` | Simple | Not RESTful |

---

### 5.A URI-Based Versioning

**Most Common Approach**

```
/api/v1/products
/api/v2/products
/api/v3/products
```

**Subdomain variant**:
```
https://v1.api.example.com/products
https://v2.api.example.com/products
```

**Example Scenario**:

**V1**:
```json
GET /api/v1/products/123
{
  "id": 123,
  "name": "Laptop",
  "price": 999.99
}
```

**V2** (changed fields):
```json
GET /api/v2/products/123
{
  "id": 123,
  "title": "Laptop",       // name → title
  "cost": 999.99,          // price → cost
  "rating": 4.5,           // new field
  "currency": "USD"        // new field
}
```

**When to Use**:
- ✅ Public APIs
- ✅ Long-term support needed
- ✅ SEO important

---

### 5.B Header-Based Versioning

**Version in HTTP headers**:

```
GET /api/products
X-API-Version: 2
```

Or using content negotiation:
```
Accept: application/vnd.company.v2+json
```

**When to Use**:
- ✅ RESTful purist approach
- ✅ Same resource, different representations
- ❌ Less visible to developers

---

### 5.C Query Parameter-Based Versioning

```
/api/products?version=2
/api/users?v=1
```

**When to Use**:
- ✅ Quick prototyping
- ✅ Internal tools
- ❌ Not recommended for production public APIs

---

## 6. Deprecation Best Practices

### Deprecation Lifecycle

```
Active → Deprecated → Sunset → Removed
```

1. **Active**: Fully supported
2. **Deprecated**: Still works, discouraged
3. **Sunset**: End date announced
4. **Removed**: No longer available

### Timeline Example

```
Jan 2024:  V2 released, V1 active
Apr 2024:  V1 marked deprecated (announce 6-month window)
Oct 2024:  V1 sunset period begins
Jan 2025:  V1 completely removed
```

### HTTP Headers

**Deprecation Warning**:
```
Deprecation: true
Sunset: Sat, 31 Dec 2024 23:59:59 GMT
Link: </api/v2/products>; rel="successor-version"
Warning: 299 - "API v1 deprecated. Migrate by Dec 31, 2024"
```

### Response Status Codes

| Phase | Status Code | Meaning |
|-------|-------------|---------|
| Deprecated | `200 OK` + headers | Still works but warned |
| Sunset | `410 Gone` | Permanently removed |
| Redirect | `301 Moved` | Redirected to new version |

### Recommended Timeline

| API Type | Deprecation Period |
|----------|-------------------|
| Internal | 3-6 months |
| Partner | 6-12 months |
| Public | 12-24 months |

### Communication Checklist

✅ Blog post announcement  
✅ Email to developers  
✅ Documentation updates  
✅ Migration guide  
✅ Code examples  
✅ Support channels  
✅ Usage tracking dashboard

---

## 7. Route Grouping

### Purpose

Apply shared configuration (auth, middleware, rate limiting) to route sets.

### 7.1 Grouping by Version

```
/api/v1/*  → Version 1 routes
/api/v2/*  → Version 2 routes
```

**Benefit**: Independent middleware per version

### 7.2 Grouping by Authentication

```
/api/public/*     → No auth required
/api/protected/*  → Auth required
/api/admin/*      → Admin role required
```

**Benefit**: Apply auth middleware to entire group

### 7.3 Grouping by Resource

```
/api/users/*      → User management
/api/products/*   → Product catalog
/api/orders/*     → Order processing
```

**Benefit**: Domain-driven organization

### Nested Grouping Example

```
/api
  /v1
    /public
      /products     → Public product list
    /protected
      /cart         → Auth required
    /admin
      /users        → Admin only
```

**Middleware Stack**:
```
/api/*            → CORS, Logging
/api/v1/*         → Version-specific middleware
/api/v1/protected → Authentication
/api/v1/admin     → Admin role check
```

---

## 8. Securing Routes

### Security Layers

```
Network (HTTPS) → Authentication → Authorization → Validation → Rate Limiting
```

### 8.1 Authentication Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| **API Keys** | `Authorization: Bearer sk_live_...` | Server-to-server |
| **JWT** | Token with user claims | Stateless auth |
| **OAuth 2.0** | Delegated authorization | Third-party apps |
| **Sessions** | Server-side session | Traditional web |

### 8.2 Authorization Models

**RBAC (Role-Based)**:
```
admin   → Full access
editor  → Create/Update
viewer  → Read only
```

**Permission-Based**:
```
users.create
users.read
users.update
users.delete
```

**Resource-Based**:
```
User can only access their own data
User 123 → /api/users/123/orders ✅
User 123 → /api/users/456/orders ❌
```

### 8.3 Input Validation

**Validate Everything**:
- ✅ Path parameters (format, type)
- ✅ Query parameters (range, allowed values)
- ✅ Request body (schema validation)
- ✅ Headers (injection prevention)

**Security Checks**:
- Type validation
- Format validation (email, phone, URL)
- Range limits
- Length restrictions
- Whitelist allowed values
- Sanitize input

### 8.4 Rate Limiting

**Strategies**:

| Strategy | Description |
|----------|-------------|
| **Fixed Window** | 100 req/minute, resets at :00 |
| **Sliding Window** | 100 req/60 sec, continuous |
| **Token Bucket** | Bucket refills at set rate |

**Tiers**:
```
Anonymous:      10 req/min
Authenticated:  100 req/min
Premium:        1000 req/min
```

**Response when exceeded**:
```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
Retry-After: 42

{
  "error": "Rate limit exceeded",
  "retryAfter": 42
}
```

### 8.5 Security Headers

```http
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
Content-Security-Policy: default-src 'self'
```

### Security Checklist

✅ Use HTTPS everywhere  
✅ Authenticate requests  
✅ Authorize actions  
✅ Validate all input  
✅ Rate limit endpoints  
✅ Use security headers  
✅ Log security events  
✅ Encrypt sensitive data  
✅ Regular security audits  
✅ Keep dependencies updated

---

## 9. Route Matching Performance Optimization

### Why It Matters

Route matching happens on **every request**. Slow routing = slow API.

### 9.1 Route Ordering

**Good Order** (common first):
```
1. /api/products  (90% traffic)
2. /api/users     (5% traffic)
3. /api/admin/*   (1% traffic)
```

**Static before dynamic**:
```
✅ Correct:
1. /products/featured  (static)
2. /products/:id       (dynamic)

❌ Wrong:
1. /products/:id       (matches "featured" as ID!)
2. /products/featured  (never reached)
```

### 9.2 Data Structures

**Traditional Array** (slow):
- Loop through all routes: O(n)

**Trie/Radix Tree** (fast):
- Tree traversal: O(log n)
- Segment-based matching

**Method Indexing**:
```
GET routes  → 600 patterns
POST routes → 300 patterns

Without indexing: Check all 1000
With indexing: Check only 600 for GET
```

### 9.3 Compiled Routes

**Interpretation** (slow):
```
For each request:
  Parse pattern → Match → Extract params
```

**Compilation** (fast):
```
On startup:
  Pre-compile all patterns to regex

For each request:
  Use compiled matcher (10-100x faster)
```

### 9.4 Static Route Fast-Path

```
Static map:
{
  "/api/products": handler1,
  "/api/users": handler2
}

For /api/products:
1. Check map (O(1)) → Found → Execute
2. If not in map → Dynamic matching
```

### 9.5 Middleware Optimization

**Inefficient**:
```
All routes → 10 middleware functions
```

**Efficient**:
```
/api/public/*    → 2 middleware
/api/protected/* → 5 middleware
```

### Performance Targets

- Route matching: < 0.1ms
- Total request: < 10ms (simple operations)

### Benchmarking Tools

- Apache Bench (ab)
- wrk
- k6
- Language-specific profilers

---

## 10. Spring Boot Code Examples

### Setup

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

### 10.1 Basic Routing

**Map HTTP requests to handlers**

```java
@RestController
@RequestMapping("/api")
public class BasicController {
    
    // GET /api/hello
    @GetMapping("/hello")
    public String hello() {
        return "Hello, World!";
    }
    
    // Same route, different methods
    @GetMapping("/users")
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    @PostMapping("/users")
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }
}
```

**Explanation**:
- `@RestController`: Marks class as REST controller
- `@RequestMapping("/api")`: Base path for all methods
- `@GetMapping`, `@PostMapping`: HTTP method + path

---

### 10.2 HTTP Methods Demo

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // GET /api/products - List all
    @GetMapping
    public ResponseEntity<List<Product>> list() {
        return ResponseEntity.ok(productService.findAll());
    }
    
    // GET /api/products/123 - Get one
    @GetMapping("/{id}")
    public ResponseEntity<Product> getOne(@PathVariable Long id) {
        Product product = productService.findById(id);
        if (product == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(product);
    }
    
    // POST /api/products - Create
    @PostMapping
    public ResponseEntity<Product> create(@RequestBody Product product) {
        Product created = productService.save(product);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .header("Location", "/api/products/" + created.getId())
            .body(created);
    }
    
    // PUT /api/products/123 - Replace
    @PutMapping("/{id}")
    public ResponseEntity<Product> replace(
        @PathVariable Long id,
        @RequestBody Product product
    ) {
        if (!productService.exists(id)) {
            return ResponseEntity.notFound().build();
        }
        product.setId(id);
        return ResponseEntity.ok(productService.save(product));
    }
    
    // PATCH /api/products/123 - Partial update
    @PatchMapping("/{id}")
    public ResponseEntity<Product> update(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates
    ) {
        Product product = productService.findById(id);
        if (product == null) {
            return ResponseEntity.notFound().build();
        }
        productService.partialUpdate(product, updates);
        return ResponseEntity.ok(product);
    }
    
    // DELETE /api/products/123 - Delete
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        if (!productService.exists(id)) {
            return ResponseEntity.notFound().build();
        }
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

### 10.3 Path Parameters

```java
@RestController
@RequestMapping("/api")
public class PathParamController {
    
    // Single path variable
    // GET /api/users/123
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // Multiple path variables
    // GET /api/users/123/posts/456
    @GetMapping("/users/{userId}/posts/{postId}")
    public Post getUserPost(
        @PathVariable Long userId,
        @PathVariable Long postId
    ) {
        return postService.findByUserAndId(userId, postId);
    }
    
    // Custom name mapping
    @GetMapping("/orders/{orderId}")
    public Order getOrder(@PathVariable("orderId") String code) {
        return orderService.findByCode(code);
    }
    
    // Regex validation - only numbers
    @GetMapping("/items/{id:\\d+}")
    public Item getItem(@PathVariable Long id) {
        return itemService.findById(id);
    }
    
    // Regex validation - specific format (ABC-1234)
    @GetMapping("/products/{code:[A-Z]{3}-\\d{4}}")
    public Product getByCode(@PathVariable String code) {
        return productService.findByCode(code);
    }
}
```

**Explanation**:
- `@PathVariable`: Extracts value from URL
- `@PathVariable("name")`: Custom parameter name
- `{var:regex}`: Regex validation in route

---

### 10.4 Query Parameters

```java
@RestController
@RequestMapping("/api/products")
public class QueryParamController {
    
    // Simple query params with defaults
    // GET /api/products?page=1&size=20
    @GetMapping
    public List<Product> getProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size
    ) {
        return productService.findAll(page, size);
    }
    
    // Optional query params
    // GET /api/products/search?q=laptop&category=electronics
    @GetMapping("/search")
    public List<Product> search(
        @RequestParam(required = false) String q,
        @RequestParam(required = false) String category,
        @RequestParam(required = false) Double maxPrice
    ) {
        return productService.search(q, category, maxPrice);
    }
    
    // Multiple values for same param
    // GET /api/products?tags=tech&tags=sale
    @GetMapping("/by-tags")
    public List<Product> getByTags(
        @RequestParam List<String> tags
    ) {
        return productService.findByTags(tags);
    }
    
    // All query params as Map
    @GetMapping("/filter")
    public List<Product> filter(
        @RequestParam Map<String, String> filters
    ) {
        return productService.filterBy(filters);
    }
}
```

**Explanation**:
- `@RequestParam`: Extracts query parameter
- `defaultValue`: Provides default if missing
- `required = false`: Makes optional
- `List<String>`: Multiple values
- `Map`: All parameters as key-value pairs

---

### 10.5 Nested Routes

```java
@RestController
@RequestMapping("/api/users")
public class NestedController {
    
    // Level 1: User
    @GetMapping("/{userId}")
    public User getUser(@PathVariable Long userId) {
        return userService.findById(userId);
    }
    
    // Level 2: User's posts
    @GetMapping("/{userId}/posts")
    public List<Post> getUserPosts(@PathVariable Long userId) {
        return postService.findByUserId(userId);
    }
    
    // Level 2: Specific user's post
    @GetMapping("/{userId}/posts/{postId}")
    public ResponseEntity<Post> getUserPost(
        @PathVariable Long userId,
        @PathVariable Long postId
    ) {
        Post post = postService.findById(postId);
        
        // Validate post belongs to user
        if (post == null || !post.getUserId().equals(userId)) {
            return ResponseEntity.notFound().build();
        }
        
        return ResponseEntity.ok(post);
    }
    
    // Level 3: Post comments
    @GetMapping("/{userId}/posts/{postId}/comments")
    public List<Comment> getComments(
        @PathVariable Long userId,
        @PathVariable Long postId
    ) {
        // Verify ownership chain
        Post post = postService.findById(postId);
        if (post == null || !post.getUserId().equals(userId)) {
            throw new ResourceNotFoundException("Post not found");
        }
        
        return commentService.findByPostId(postId);
    }
}
```

**Key Point**: Always validate parent resources exist!

---

### 10.6 Catch-All Routes

```java
// Method 1: Catch-all controller
@RestController
public class NotFoundController {
    
    @RequestMapping("/**")
    public ResponseEntity<ErrorResponse> handleNotFound(
        HttpServletRequest request
    ) {
        ErrorResponse error = new ErrorResponse();
        error.setTimestamp(LocalDateTime.now());
        error.setStatus(404);
        error.setError("Not Found");
        error.setMessage("Route '" + request.getRequestURI() + "' does not exist");
        error.setPath(request.getRequestURI());
        
        return ResponseEntity.status(404).body(error);
    }
}

// Method 2: Using @ControllerAdvice
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(NoHandlerFoundException.class)
    public ResponseEntity<ErrorResponse> handleNoHandler(
        NoHandlerFoundException ex
    ) {
        ErrorResponse error = new ErrorResponse();
        error.setStatus(404);
        error.setError("Route Not Found");
        error.setMessage("The route does not exist");
        
        return ResponseEntity.status(404).body(error);
    }
}
```

**Configuration** (application.properties):
```properties
spring.mvc.throw-exception-if-no-handler-found=true
spring.web.resources.add-mappings=false
```

---

### 10.7 API Versioning (URI-based)

```java
// ===== Version 1 =====
@RestController
@RequestMapping("/api/v1/products")
public class ProductControllerV1 {
    
    @GetMapping("/{id}")
    public ProductV1DTO getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        
        // V1 format: {id, name, price}
        return new ProductV1DTO(
            product.getId(),
            product.getName(),
            product.getPrice()
        );
    }
}

// V1 DTO
class ProductV1DTO {
    private Long id;
    private String name;     // Old field
    private Double price;
    
    // Constructor, getters, setters
}

// ===== Version 2 =====
@RestController
@RequestMapping("/api/v2/products")
public class ProductControllerV2 {
    
    @GetMapping("/{id}")
    public ProductV2DTO getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        
        // V2 format: {id, title, price, rating, imageUrl}
        return new ProductV2DTO(
            product.getId(),
            product.getTitle(),      // "name" → "title"
            product.getPrice(),
            product.getRating(),     // New field
            product.getImageUrl()    // New field
        );
    }
}

// V2 DTO
class ProductV2DTO {
    private Long id;
    private String title;    // Changed from "name"
    private Double price;
    private Double rating;   // New field
    private String imageUrl; // New field
    
    // Constructor, getters, setters
}
```

---

### 10.8 Deprecation Headers

```java
@RestController
@RequestMapping("/api/v1")
@Deprecated  // Mark as deprecated
public class DeprecatedController {
    
    @GetMapping("/products")
    public ResponseEntity<List<ProductV1DTO>> getProducts() {
        List<ProductV1DTO> products = service.getAllV1();
        
        // Add deprecation headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("Deprecation", "true");
        headers.add("Sunset", "Sat, 31 Dec 2024 23:59:59 GMT");
        headers.add("Link", "</api/v2/products>; rel=\"successor-version\"");
        headers.add("Warning", 
            "299 - \"API v1 deprecated. Migrate to v2 by Dec 31, 2024\"");
        
        return ResponseEntity
            .ok()
            .headers(headers)
            .body(products);
    }
}

// After sunset - return 410 Gone
@RestController
@RequestMapping("/api/v1")
public class SunsetController {
    
    @GetMapping("/products")
    public ResponseEntity<Map<String, Object>> gone() {
        return ResponseEntity
            .status(HttpStatus.GONE)  // 410
            .body(Map.of(
                "error", "API no longer available",
                "deprecatedOn", "2024-12-31",
                "successor", "/api/v2/products",
                "migrationGuide", "https://docs.example.com/migration"
            ));
    }
}
```

---

### 10.9 Route Grouping & Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()
                // Public routes
                .antMatchers("/api/public/**").permitAll()
                
                // Protected routes - auth required
                .antMatchers("/api/protected/**").authenticated()
                
                // Admin routes - admin role required
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                
                .anyRequest().authenticated()
            .and()
            .oauth2ResourceServer()
                .jwt();
    }
}

// Public routes
@RestController
@RequestMapping("/api/public")
public class PublicController {
    
    @GetMapping("/products")
    public List<Product> getProducts() {
        return productService.findAll();
    }
}

// Protected routes
@RestController
@RequestMapping("/api/protected")
public class ProtectedController {
    
    @GetMapping("/profile")
    public User getProfile(@AuthenticationPrincipal Jwt jwt) {
        String userId = jwt.getSubject();
        return userService.findById(userId);
    }
}

// Admin routes
@RestController
@RequestMapping("/api/admin")
@PreAuthorize("hasRole('ADMIN')")
public class AdminController {
    
    @GetMapping("/users")
    public List<User> getAllUsers() {
        return userService.findAll();
    }
}
```

---

### 10.10 Rate Limiting

```java
// Custom Rate Limit Interceptor
@Component
public class RateLimitInterceptor implements HandlerInterceptor {
    
    private final Map<String, List<Long>> requestCounts = new ConcurrentHashMap<>();
    private static final int MAX_REQUESTS = 100;
    private static final long TIME_WINDOW = 60_000; // 1 minute
    
    @Override
    public boolean preHandle(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler
    ) throws Exception {
        
        String clientId = getClientId(request);
        long currentTime = System.currentTimeMillis();
        
        requestCounts.putIfAbsent(clientId, new ArrayList<>());
        List<Long> timestamps = requestCounts.get(clientId);
        
        // Remove old timestamps outside window
        timestamps.removeIf(time -> currentTime - time > TIME_WINDOW);
        
        if (timestamps.size() >= MAX_REQUESTS) {
            response.setStatus(429); // Too Many Requests
            response.setHeader("X-RateLimit-Limit", String.valueOf(MAX_REQUESTS));
            response.setHeader("X-RateLimit-Remaining", "0");
            response.setHeader("Retry-After", "60");
            return false;
        }
        
        timestamps.add(currentTime);
        
        response.setHeader("X-RateLimit-Limit", String.valueOf(MAX_REQUESTS));
        response.setHeader("X-RateLimit-Remaining", 
            String.valueOf(MAX_REQUESTS - timestamps.size()));
        
        return true;
    }
    
    private String getClientId(HttpServletRequest request) {
        // Use IP or user ID
        return request.getRemoteAddr();
    }
}

// Register interceptor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Autowired
    private RateLimitInterceptor rateLimitInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(rateLimitInterceptor)
            .addPathPatterns("/api/**");
    }
}
```

---

### 10.11 Input Validation

```java
@RestController
@RequestMapping("/api/users")
@Validated  // Enable validation
public class UserController {
    
    // Validate request body
    @PostMapping
    public ResponseEntity<User> createUser(
        @Valid @RequestBody UserCreateRequest request
    ) {
        User user = userService.create(request);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(user);
    }
    
    // Validate path parameter
    @GetMapping("/{id}")
    public User getUser(
        @PathVariable 
        @Min(1) @Max(Long.MAX_VALUE) 
        Long id
    ) {
        return userService.findById(id);
    }
    
    // Validate query parameters
    @GetMapping
    public List<User> searchUsers(
        @RequestParam 
        @Size(min = 3, max = 50) 
        String name,
        
        @RequestParam 
        @Min(18) @Max(120) 
        Integer age
    ) {
        return userService.search(name, age);
    }
}

// DTO with validation annotations
class UserCreateRequest {
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100)
    private String name;
    
    @NotBlank
    @Email(message = "Invalid email format")
    private String email;
    
    @NotNull
    @Min(18)
    @Max(120)
    private Integer age;
    
    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone")
    private String phone;
    
    // Getters, setters
}

// Global validation exception handler
@ControllerAdvice
public class ValidationExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidation(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(Map.of(
                "error", "Validation failed",
                "fields", errors
            ));
    }
}
```

---

### 10.12 Performance - Route Optimization

```java
// Use specific mappings (faster than wildcards)
@RestController
@RequestMapping("/api")
public class OptimizedController {
    
    // ✅ Good - specific path
    @GetMapping("/products/featured")
    public List<Product> getFeatured() {
        return productService.findFeatured();
    }
    
    // ✅ Good - simple path variable
    @GetMapping("/products/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }
    
    // ⚠️ Slower - regex validation
    @GetMapping("/products/{code:[A-Z]{3}-\\d{4}}")
    public Product getByCode(@PathVariable String code) {
        return productService.findByCode(code);
    }
}

// Use @GetMapping instead of @RequestMapping
// ✅ Faster
@GetMapping("/users")
public List<User> getUsers() { }

// ❌ Slower
@RequestMapping(value = "/users", method = RequestMethod.GET)
public List<User> getUsers() { }

// Order routes: static before dynamic
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // 1. Static routes first (checked first)
    @GetMapping("/featured")
    public List<Product> getFeatured() { }
    
    @GetMapping("/bestsellers")
    public List<Product> getBestsellers() { }
    
    // 2. Dynamic routes last
    @GetMapping("/{id}")
    public Product getById(@PathVariable Long id) { }
}
```

---

## Summary

### Key Takeaways

1. **Routing** = Mapping requests to handlers
2. **HTTP Method + Route** = Unique operation
3. **Path params** = Resource ID (mandatory)
4. **Query params** = Filters (optional)
5. **Static routes** = Fast, cacheable
6. **Dynamic routes** = Flexible, resource-specific
7. **Nested routes** = Parent-child relationships
8. **Versioning** = Backward compatibility
9. **Security** = Auth → Authorization → Validation
10. **Performance** = Ordering + Indexing + Caching

### Best Practices Checklist

✅ Use RESTful conventions  
✅ Version public APIs (URI-based recommended)  
✅ Validate all input  
✅ Authenticate and authorize  
✅ Rate limit endpoints  
✅ Handle errors gracefully  
✅ Return proper HTTP status codes  
✅ Document all endpoints  
✅ Monitor performance  
✅ Deprecate responsibly  

---

## Additional Resources

- [REST API Tutorial](https://restfulapi.net/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [JWT.io](https://jwt.io/)
- [OWASP API Security](https://owasp.org/www-project-api-security/)

---

**Created with ❤️ for learning backend routing concepts**

Last Updated: February 2026

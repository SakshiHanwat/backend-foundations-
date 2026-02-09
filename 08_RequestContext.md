# 🔐 REQUEST CONTEXT - COMPLETE GUIDE

---

## 📌 TABLE OF CONTENTS

1. **What Is Request Context?**
2. **Request Scoped State**
3. **Metadata Management**
4. **User & Session Data**
5. **Trace IDs & Request IDs**
6. **Timeouts & Cancellation**
7. **Best Practices**
8. **Common Pitfalls & Solutions**
9. **Real-World Examples**
10. **Quick Reference**

---

# 1️⃣ WHAT IS REQUEST CONTEXT?

## Definition

**Request Context**: Container/object that holds data specific to a single HTTP request

**Key Word**: **Single Request** - Data belongs only to this request, not shared with other requests

## Visual Concept

```
CLIENT REQUEST #1         CLIENT REQUEST #2         CLIENT REQUEST #3
    │                          │                          │
    └─→ Request Context #1     └─→ Request Context #2     └─→ Request Context #3
        ├─ User: John              ├─ User: Jane              ├─ User: Bob
        ├─ Request ID: abc123      ├─ Request ID: def456      ├─ Request ID: ghi789
        ├─ Trace ID: trace1        ├─ Trace ID: trace2        ├─ Trace ID: trace3
        ├─ Metadata: ...           ├─ Metadata: ...           ├─ Metadata: ...
        └─ Custom data: ...        └─ Custom data: ...        └─ Custom data: ...

IMPORTANT: Each request has its OWN context
           No sharing between requests!
           Data isolated per request!
```

## Why Request Context Matters?

```
Problem without Request Context:
├─ Global variables = shared across requests
├─ Request A sets user_id = 1
├─ Request B reads user_id = 1 (WRONG! Should be different user)
├─ Data leakage between requests!
└─ Major security issue ❌

Solution with Request Context:
├─ Each request gets own context
├─ Request A: context.user_id = 1
├─ Request B: context.user_id = 2 (isolated!)
├─ No data leakage!
└─ Secure! ✓
```

## Request Lifecycle

```
CLIENT REQUEST ARRIVES
    ↓
REQUEST CONTEXT CREATED
├─ New instance per request
├─ Unique request ID
├─ Empty metadata
├─ No user (yet)
└─ Start timer
    ↓
MIDDLEWARE PHASE
├─ Authentication middleware
│  └─ Sets: context.user = user_info
├─ Logging middleware
│  └─ Uses: context.request_id
├─ Custom middleware
│  └─ Adds: context.custom_data
└─ Continue
    ↓
ROUTE HANDLER
├─ Access: context.user
├─ Access: context.request_id
├─ Add: context.result = handler_result
└─ Continue
    ↓
RESPONSE MIDDLEWARE
├─ Log: context.request_id
├─ Log: context.user
├─ Log: response time (from context timer)
└─ Continue
    ↓
SEND RESPONSE
└─ Discard context (garbage collection)

TIMELINE: Request context exists ONLY for this single request
```

## Request Context Structure

```
request.context = {
  // Identification
  request_id: "req_abc123",
  trace_id: "trace_xyz789",
  
  // User & Security
  user: {
    id: 1,
    username: "john",
    role: "admin"
  },
  session: {
    id: "session_abc",
    created_at: "timestamp"
  },
  
  // Metadata
  metadata: {
    source: "web",
    api_version: "v1",
    client_version: "1.2.3"
  },
  
  // Timing
  start_time: timestamp,
  deadline: timestamp (timeout),
  
  // State
  custom_state: {
    cache_data: {...},
    computed_values: {...}
  },
  
  // Cancellation
  cancelled: false,
  cancel_reason: null
}
```

---

# 2️⃣ REQUEST SCOPED STATE

## What Is Request-Scoped State?

**Definition**: Data that exists ONLY for duration of single request

**Characteristics**:
- Created when request arrives
- Destroyed when response sent
- Not shared with other requests
- Thread-safe (if using thread context)
- Isolated per request

## Problem: Global State (BAD)

```
GLOBAL VARIABLE (SHARED):

global current_user = null

Request A:
1. current_user = user_john (ID: 1)
2. Do processing...
3. Use current_user

Request B (same time):
1. current_user = user_jane (ID: 2)
2. current_user changed!
3. Request A still running but current_user changed!

Request A continues:
4. Use current_user
   ├─ Expected: user_john (ID: 1)
   ├─ Actually: user_jane (ID: 2) ❌ WRONG!
   └─ Security breach!

PROBLEM: Both requests share same variable
         Data overwrites each other
         Unpredictable behavior
         Security vulnerabilities
```

## Solution: Request-Scoped State (GOOD)

```
REQUEST-SCOPED CONTEXT:

Request A:
1. context_A.user = user_john (ID: 1)
2. Do processing...
3. Use context_A.user

Request B (same time):
1. context_B.user = user_jane (ID: 2)
2. Different context!
3. Request A continues...

Request A continues:
4. Use context_A.user
   ├─ Expected: user_john (ID: 1)
   ├─ Actually: user_john (ID: 1) ✓ CORRECT!
   └─ No data leakage!

SOLUTION: Each request gets own context
          Data isolated
          Safe concurrent execution
          No conflicts
```

## Storing Request-Scoped Data

### Pattern 1: Request Object (Simple)

```
Web Framework provides: request object

request.context = {
  user: user_info,
  request_id: "abc123",
  custom_data: {}
}

Handler accesses:
├─ request.context.user
├─ request.context.request_id
└─ request.context.custom_data["key"]

Middleware sets:
├─ request.context.user = authenticated_user
├─ request.context.request_id = generate_id()
└─ request.context.custom_data["cache"] = {...}
```

### Pattern 2: Context Manager (Advanced)

```
Language provides: Context storage (thread-local, async context)

Middleware:
ctx = create_context()
ctx.user = authenticated_user
ctx.request_id = generate_id()
set_current_context(ctx)

Handler:
ctx = get_current_context()
user = ctx.user
request_id = ctx.request_id

Cleanup:
clear_current_context()

Advantage:
├─ Don't pass context through function calls
├─ Access from any function
├─ Implicit passing
├─ Clean code
```

### Pattern 3: Dependency Injection (Explicit)

```
Handler receives context as parameter:

handler(context):
  user = context.user
  request_id = context.request_id
  do_something(context)

Function calls pass context:

do_something(context):
  use context.user
  use context.request_id
  call_another(context)

call_another(context):
  use context.request_id
  ...

Advantage:
├─ Explicit dependencies
├─ Testable (mock context)
├─ Clear what data needed
├─ No hidden dependencies
```

## Request-Scoped Data Cleanup

### Why Cleanup Matters?

```
Memory leak example:

Request 1: Create context
├─ Add user data
├─ Add cache data
├─ Add temporary state
└─ Request ends

No cleanup:
├─ Context still in memory
├─ References not cleared
├─ Memory accumulates
├─ After 1000 requests: 1000 contexts in memory
├─ Memory leak!

With cleanup:
├─ Request ends
├─ Context marked for deletion
├─ References cleared
├─ Garbage collection reclaims memory
├─ Memory stays constant
└─ No leak!
```

### Cleanup Pattern

```
Middleware / Framework handles:

On Request Start:
1. Create context
2. Attach to request object

On Request Complete (finally block):
1. Clear all references
2. Remove user data
3. Remove cache data
4. Remove all custom data
5. Mark context for deletion

Code:
try {
  create_context()
  middleware_chain()
  handler()
  return response
} finally {
  clear_context()  ← Important! Always runs
  clear_all_references()
}
```

## Examples of Request-Scoped Data

```
What data should be request-scoped:

✓ REQUEST-SCOPED (YES):
├─ Current user (authenticated user)
├─ Request ID (unique per request)
├─ Trace ID (for tracing)
├─ Request start time
├─ Session data
├─ Temporary cache (computed during request)
├─ Cancellation signal
├─ Deadline/timeout
├─ Request metadata
└─ Temporary state

✗ NOT REQUEST-SCOPED (NO):
├─ Database connections (use connection pool)
├─ Global configuration
├─ Server-wide cache (shared across requests)
├─ Logger instance
├─ Shared utilities
└─ Application constants

Why difference:
├─ Request-scoped = Changes per request
├─ Shared = Same for all requests
```

---

# 3️⃣ METADATA MANAGEMENT

## What Is Metadata?

**Definition**: Information ABOUT the request, not the request data itself

**Examples**:
- API version used
- Client version
- Source of request
- User agent
- Custom headers
- Feature flags
- Experiment flags

## Metadata Examples

### Example 1: API Version

```
Request:
Header: X-API-Version: v2

Metadata:
context.metadata.api_version = "v2"

Handler uses:
if context.metadata.api_version == "v1" {
  use old response format
} else if context.metadata.api_version == "v2" {
  use new response format
} else {
  use latest response format
}

Purpose:
├─ Different API versions coexist
├─ Clients can use older versions
├─ Gradual migration possible
└─ No forced updates
```

### Example 2: Client Version

```
Request:
Header: X-Client-Version: 1.2.3

Metadata:
context.metadata.client_version = "1.2.3"

Handler uses:
if context.metadata.client_version < "2.0.0" {
  return simplified_response
} else {
  return detailed_response
}

Purpose:
├─ Handle older client versions
├─ Gradual feature rollout
├─ Old clients still work
└─ New clients get new features
```

### Example 3: Feature Flags

```
Request:
Header: X-Features: new-dashboard,beta-api

Metadata:
context.metadata.features = ["new-dashboard", "beta-api"]

Handler uses:
if "new-dashboard" in context.metadata.features {
  use new_dashboard_code()
} else {
  use old_dashboard_code()
}

if "beta-api" in context.metadata.features {
  return beta_api_response()
} else {
  return stable_api_response()
}

Purpose:
├─ A/B testing
├─ Feature rollout
├─ Beta testing
├─ Gradual deployment
└─ Easy rollback
```

### Example 4: Source/Platform

```
Request:
Header: X-Source: mobile-ios

Metadata:
context.metadata.source = "mobile-ios"

Handler uses:
if context.metadata.source == "mobile-ios" {
  return ios_optimized_response()
} else if context.metadata.source == "mobile-android" {
  return android_optimized_response()
} else if context.metadata.source == "web" {
  return web_optimized_response()
}

Purpose:
├─ Platform-specific responses
├─ Optimize for each platform
├─ Platform-specific features
└─ Mobile/web/desktop differences
```

### Example 5: User Agent & Device

```
Request:
Header: User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 14_6)

Metadata:
context.metadata.user_agent = "Mozilla/5.0..."
context.metadata.device_type = "mobile"
context.metadata.browser = "Safari"
context.metadata.os = "iOS"

Handler uses:
if context.metadata.device_type == "mobile" {
  return mobile_response()
} else {
  return desktop_response()
}

Purpose:
├─ Device detection
├─ Browser compatibility
├─ Platform-specific handling
└─ Analytics tracking
```

## Extracting Metadata

### From Headers

```
Metadata extraction middleware:

function extract_metadata(request):
  metadata = {}
  
  // Extract from headers
  metadata.api_version = 
    request.headers["X-API-Version"] || "v1"
  
  metadata.client_version = 
    request.headers["X-Client-Version"] || "unknown"
  
  metadata.features = 
    (request.headers["X-Features"] || "").split(",")
  
  metadata.source = 
    request.headers["X-Source"] || "unknown"
  
  metadata.device_type = 
    parse_user_agent(request.headers["User-Agent"])
  
  request.context.metadata = metadata
```

### From URL

```
Metadata from path/query:

Request: GET /api/v2/users?include=profile&format=json

Metadata extraction:
metadata.api_version = "v2" (from path)
metadata.include = ["profile"] (from query)
metadata.format = "json" (from query)

request.context.metadata = metadata
```

### From Custom Headers

```
Custom header patterns:

X-Request-Source: web|mobile|api
X-Client-Type: ios|android|web
X-Feature-Flags: flag1,flag2,flag3
X-Experiment-Group: group_a|group_b
X-Debug-Mode: true|false
X-API-Version: v1|v2|v3

Extraction:
for each header starting with X-:
  key = header_name[2:].lower()  # Remove X-
  metadata[key] = header_value
```

## Using Metadata

### In Handlers

```
Handler uses metadata:

function get_user_profile(request):
  user_id = request.params.user_id
  api_version = request.context.metadata.api_version
  features = request.context.metadata.features
  
  profile = fetch_user_profile(user_id)
  
  if api_version == "v1" {
    return {
      id: profile.id,
      name: profile.name
    }
  } else if api_version == "v2" {
    return {
      id: profile.id,
      name: profile.name,
      email: profile.email,
      created_at: profile.created_at
    }
  }
  
  if "extended" in features {
    return {
      ...profile,
      additional_data: fetch_additional_data(user_id)
    }
  }
  
  return response
```

### In Logging

```
Logging with metadata:

function log_request(request, response):
  log({
    timestamp: now(),
    request_id: request.context.request_id,
    method: request.method,
    path: request.path,
    user_id: request.context.user?.id,
    
    // Include metadata
    api_version: request.context.metadata.api_version,
    client_version: request.context.metadata.client_version,
    source: request.context.metadata.source,
    features: request.context.metadata.features,
    
    status: response.status,
    duration_ms: response.duration
  })
```

### In Metrics

```
Metrics with metadata:

function record_metrics(request, response):
  metrics.increment("requests.total", {
    method: request.method,
    path: request.path,
    status: response.status,
    
    // Metadata tags
    api_version: request.context.metadata.api_version,
    source: request.context.metadata.source,
    device: request.context.metadata.device_type
  })
```

---

# 4️⃣ USER & SESSION DATA

## What Is User Data?

**Definition**: Information about authenticated user making request

**What Includes**:
- User ID
- Username
- Email
- Role/Permissions
- Session ID
- Login time
- Last activity
- Custom user properties

## Authentication to Context

### Step 1: Authentication Middleware

```
Request arrives with token:
Headers: { Authorization: "Bearer token123" }

Authentication Middleware:
1. Extract token from header
2. Verify token (signature, expiration)
3. If valid, decode token
4. Get user information
5. Attach to context
6. Continue

Code:
function authenticate(request):
  token = request.headers.Authorization
  
  if !token {
    return 401 error
  }
  
  try {
    payload = verify_token(token)
    user = get_user(payload.user_id)
    request.context.user = user
  } catch {
    return 401 error
  }
```

### Step 2: User Object in Context

```
request.context.user = {
  id: 1,
  username: "john",
  email: "john@gmail.com",
  first_name: "John",
  last_name: "Doe",
  
  // Roles & Permissions
  role: "admin",
  permissions: ["create_user", "delete_user", "edit_user"],
  
  // Status
  is_active: true,
  is_verified: true,
  
  // Timestamps
  created_at: "2024-01-15",
  last_login: "2025-02-09",
  
  // Custom
  custom_field_1: value1,
  custom_field_2: value2
}
```

## Session Management

### What Is a Session?

**Definition**: Server-side storage of user state across requests

**Contains**:
- Session ID (unique)
- User ID (who owns session)
- Creation time
- Last activity time
- Expiration time
- Session data

### Session Data in Context

```
request.context.session = {
  id: "session_abc123xyz",
  user_id: 1,
  created_at: "2025-02-09T10:00:00Z",
  last_activity: "2025-02-09T13:30:45Z",
  expires_at: "2025-02-10T10:00:00Z",
  
  // Session-specific data
  data: {
    ip_address: "192.168.1.100",
    user_agent: "Mozilla/5.0...",
    device_id: "device_xyz789",
    login_method: "password" | "oauth" | "saml"
  },
  
  // Access & Refresh tokens
  access_token: "...",
  refresh_token: "...",
  token_expires_in: 3600
}
```

### Session Types

#### Type 1: Server-Side Session

```
Session stored on server:

Step 1: Login
├─ User credentials valid? YES
├─ Create session on server
├─ Generate session ID
├─ Store in database/cache
├─ Send session ID to client (in cookie)

Step 2: Subsequent requests
├─ Client sends cookie: sessionid=abc123
├─ Server looks up session in database/cache
├─ Verify session still valid
├─ Load session data
├─ Attach to context

Step 3: Logout
├─ Delete session from database
├─ Session ID becomes invalid
├─ Client sends cookie, server finds nothing
├─ Redirect to login

Pros:
├─ Can revoke sessions immediately
├─ Server controls session state
├─ Good for security-critical apps
└─ Can invalidate on password change

Cons:
├─ Requires server storage
├─ Doesn't scale across servers (sticky sessions)
├─ Database query per request
└─ More server resources
```

#### Type 2: Token-Based Session (JWT)

```
Session stored in token:

Step 1: Login
├─ User credentials valid? YES
├─ Create JWT token
├─ Token contains: user_id, role, permissions, exp
├─ Token signed with server secret
├─ Send token to client

Step 2: Subsequent requests
├─ Client sends: Authorization: Bearer token123
├─ Server verifies token signature
├─ Server decodes token
├─ Load user data from token
├─ Attach to context

Step 3: Logout
├─ Client deletes token locally
├─ Server doesn't need to do anything
├─ Token still valid on server until expiration

Pros:
├─ Stateless (no server storage needed)
├─ Scales across servers (any server can verify)
├─ Fast (no database lookup)
├─ Good for distributed systems
├─ Good for mobile/API

Cons:
├─ Can't revoke immediately (token still valid)
├─ Token size increases payload
├─ Secret management needed
├─ Need token refresh mechanism
└─ Leaked tokens dangerous until expiration
```

## User Authorization

### Authorization in Context

```
request.context.user.permissions = [
  "read_posts",
  "create_posts",
  "delete_own_posts",
  "edit_own_posts",
  "admin_panel"
]

Handler checks:
if "delete_posts" not in request.context.user.permissions {
  return 403 FORBIDDEN
}

OR using roles:

request.context.user.role = "admin"

Handler checks:
if request.context.user.role != "admin" {
  return 403 FORBIDDEN
}
```

### Authorization Middleware

```
Authorization Middleware:

function authorize(required_permission):
  return function(request):
    if !request.context.user {
      return 401 UNAUTHORIZED
    }
    
    if required_permission not in request.context.user.permissions {
      return 403 FORBIDDEN
    }
    
    continue to handler

Usage:
GET /api/admin/users
  - Apply middleware: authorize("admin_users")
  - Only users with "admin_users" permission allowed
  - Others get 403 FORBIDDEN
```

---

# 5️⃣ TRACE IDS & REQUEST IDS

## What Are Request IDs?

**Definition**: Unique identifier for single HTTP request

**Purpose**: Track single request through system

```
Request arrives:
├─ Generate unique ID: req_abc123xyz
├─ Attach to context
├─ Include in all logs
├─ Include in responses
└─ Client can reference this ID for support

Benefit:
├─ Single request tracking
├─ Easy debugging
├─ Customer support reference
├─ Error tracing
└─ Analytics
```

## What Are Trace IDs?

**Definition**: Unique identifier for distributed request flow across multiple services

**Purpose**: Track request across entire system (multiple services/databases)

```
Request Timeline:

Request enters System:
├─ Generate trace ID: trace_xyz789
└─ Request context gets trace ID

Request flows through services:

Service A:
├─ Receives request
├─ Gets trace ID from context
├─ Does work
├─ Calls Service B
└─ Passes trace ID in headers

Service B:
├─ Receives request
├─ Gets trace ID from headers
├─ Does work
├─ Calls Service C
└─ Passes trace ID in headers

Service C:
├─ Receives request
├─ Gets trace ID from headers
├─ Does work
├─ Database query
└─ Returns

Back to Service B:
├─ Receives response
├─ Continues
├─ Returns to Service A

Back to Service A:
├─ Receives response
├─ Returns response to client

All logs have same trace ID:
├─ Service A logs: trace_id=trace_xyz789
├─ Service B logs: trace_id=trace_xyz789
├─ Service C logs: trace_id=trace_xyz789
├─ Database logs: trace_id=trace_xyz789
└─ Can reconstruct entire flow!
```

## Request ID Generation

### Uniqueness

```
Requirements:
├─ Unique (no duplicates)
├─ Random (can't predict)
├─ Short enough (practical)
├─ Sortable (if needed)

Generation methods:

1. UUID (Universal Unique ID)
   ├─ Format: 550e8400-e29b-41d4-a716-446655440000
   ├─ Pros: Truly unique, standard
   └─ Cons: Long (36 characters)

2. Short ID (Base62)
   ├─ Format: abc123xyz789
   ├─ Pros: Short, readable
   └─ Cons: Lower uniqueness guarantee

3. Nanoid
   ├─ Format: V1StGXR_Z5j3eK4x
   ├─ Pros: Short, cryptographically random
   └─ Cons: Library dependency

4. Timestamp-based
   ├─ Format: 1644381045_abc123
   ├─ Pros: Sortable by time
   └─ Cons: Less random

Choice:
├─ Most common: UUID v4
├─ For high performance: Nanoid
├─ For human-readable: Prefix+random (req_abc123)
```

### Generation Middleware

```
function generate_ids(request):
  // Generate request ID
  request.context.request_id = generate_uuid()
  
  // OR extract trace ID from headers (if from another service)
  // OR generate new trace ID
  trace_id = request.headers["X-Trace-ID"]
  if !trace_id {
    trace_id = generate_uuid()
  }
  request.context.trace_id = trace_id
  
  // Add to response headers
  response.headers["X-Request-ID"] = request.context.request_id
  response.headers["X-Trace-ID"] = request.context.trace_id
```

## Using Request & Trace IDs

### In Logging

```
Every log message includes IDs:

log({
  timestamp: "2025-02-09T13:30:45Z",
  request_id: "req_abc123xyz",
  trace_id: "trace_xyz789",
  message: "User login successful",
  user_id: 1,
  level: "INFO"
})

Output:
[2025-02-09T13:30:45Z] req_abc123xyz trace_xyz789 - INFO: User login successful (user_id: 1)

Benefit:
├─ Single request tracked through logs
├─ All logs for request have same ID
├─ Can grep/filter by request ID
├─ Easy debugging
```

### In Error Tracking

```
Error occurs:

Error:
├─ Message: "Database connection timeout"
├─ Request ID: req_abc123xyz
├─ Trace ID: trace_xyz789
├─ Timestamp: 2025-02-09T13:30:45Z
├─ Service: api-users
├─ Stack trace: ...

Error tracking system:
├─ Stores error with request ID
├─ Stores error with trace ID
├─ Can recreate request flow
├─ Can see all logs for that request
├─ Can debug exact issue
```

### In Response Headers

```
Response includes IDs:

HTTP/1.1 200 OK
X-Request-ID: req_abc123xyz
X-Trace-ID: trace_xyz789
Content-Type: application/json

{
  "status": "success",
  "data": {...}
}

Client benefits:
├─ Customer support: "My request ID is req_abc123xyz"
├─ Support team: grep logs for that ID
├─ Find exact request
├─ Solve issue
```

### In Distributed Tracing

```
Distributed Tracing Tool (e.g., Jaeger, Datadog):

Timeline:
├─ 0ms: Request arrives API Gateway
│  └─ Trace ID: trace_xyz789
├─ 5ms: API Gateway → Service A
│  └─ Passes: X-Trace-ID: trace_xyz789
├─ 10ms: Service A processes
├─ 20ms: Service A → Service B
│  └─ Passes: X-Trace-ID: trace_xyz789
├─ 30ms: Service B processes
├─ 40ms: Service B → Database
│  └─ Passes: X-Trace-ID: trace_xyz789
├─ 50ms: Database returns
├─ 60ms: Service B → Service A
├─ 70ms: Service A → API Gateway
├─ 80ms: Response to client

Visualization shows:
├─ Total latency: 80ms
├─ Service A latency: 30ms
├─ Service B latency: 30ms
├─ Database latency: 20ms
├─ Network latency: 5ms
└─ Easy bottleneck identification!
```

---

# 6️⃣ TIMEOUTS & CANCELLATION

## What Is a Timeout?

**Definition**: Maximum time allowed for request to complete

**Purpose**: Prevent hanging requests, resource exhaustion

```
Example:

Timeout: 30 seconds

Request starts: 10:00:00
Still processing: 10:00:15 (15 seconds elapsed)
Still processing: 10:00:25 (25 seconds elapsed)
Timeout reached: 10:00:30 (30 seconds elapsed!)

Action:
├─ Cancel request
├─ Stop processing
├─ Return 504 GATEWAY TIMEOUT
├─ Client gets error
└─ Server resources freed
```

## Timeout in Request Context

```
request.context = {
  ...
  
  // Timing
  start_time: 1644381045000,  // milliseconds
  timeout: 30000,             // 30 seconds
  deadline: 1644381075000,    // start_time + timeout
  
  // Check timeout
  is_expired(): {
    return current_time() > deadline
  },
  
  remaining_time(): {
    return deadline - current_time()
  }
}
```

## Timeout Implementation

### Timeout Middleware

```
function timeout_middleware(max_timeout):
  return function(request):
    request.context.start_time = current_time()
    request.context.timeout = max_timeout
    request.context.deadline = start_time + max_timeout
    
    timer = set_timer(max_timeout, function():
      request.context.timed_out = true
      cancel_request()
    )
    
    try {
      call_next_middleware()
    } finally {
      clear_timer(timer)
    }
```

### Timeout Checking in Handler

```
function process_large_query(request):
  results = []
  
  for item in large_list {
    // Check if timeout reached
    if request.context.is_expired() {
      return 504 error: "Request timeout"
    }
    
    // Process item
    results.push(process(item))
  }
  
  return results
```

### Timeout in Database Queries

```
function query_database(request):
  remaining = request.context.remaining_time()
  
  query = database.query(sql)
  query.timeout = remaining  // Use remaining time as timeout
  
  try {
    results = query.execute()
  } catch TimeoutException {
    return 504 error: "Database query timeout"
  }
  
  return results
```

## What Is Cancellation?

**Definition**: Ability to stop request processing early

**Purpose**: Clean resource cleanup, immediate response to client

```
Reasons for cancellation:

1. Client disconnects
   ├─ Browser closes tab
   ├─ Mobile app loses connection
   ├─ User stops request
   └─ Should stop processing

2. Timeout reached
   ├─ Request exceeded max time
   ├─ Should stop processing
   └─ Return error to client

3. Error occurs
   ├─ Unrecoverable error found
   ├─ Should stop processing
   └─ Return error immediately

4. Resource limit exceeded
   ├─ Memory usage too high
   ├─ Should stop processing
   └─ Return error
```

## Cancellation in Request Context

```
request.context = {
  ...
  
  // Cancellation
  cancelled: false,
  cancel_reason: null,
  cancel_func: null,
  
  cancel(reason): {
    this.cancelled = true
    this.cancel_reason = reason
    if this.cancel_func {
      this.cancel_func()
    }
  }
}
```

## Cancellation Implementation

### Listen for Cancellation

```
function handle_request(request):
  
  // Client disconnected
  request.on_disconnect(function():
    request.context.cancel("client_disconnected")
  )
  
  // Timeout reached
  request.on_timeout(function():
    request.context.cancel("timeout")
  )
  
  try {
    // Process request
    result = handler(request)
    return result
  } except:
    if request.context.cancelled {
      log("Request cancelled: " + request.context.cancel_reason)
      return null  // Don't return error (client already gone)
    } else {
      // Normal error
      return error_response()
    }
```

### Check Cancellation in Loop

```
function process_items(request):
  results = []
  
  for item in items {
    // Check if cancelled
    if request.context.cancelled {
      return error: "Request cancelled: " + request.context.cancel_reason
    }
    
    // Process item
    results.push(process(item))
  }
  
  return results
```

### Cooperative Cancellation

```
function long_operation(request):
  
  // Register cancellation handler
  def cleanup():
    cleanup_resources()
  
  request.context.cancel_func = cleanup
  
  try {
    // Do work
    for i in range(1000000):
      if i % 1000 == 0:  // Check every 1000 iterations
        if request.context.cancelled:
          cleanup()
          raise CancelledException()
      
      do_work(i)
    
    return result
  finally:
    cleanup()
```

## Timeout Best Practices

```
Set appropriate timeouts:

├─ Fast endpoints (< 100ms): 5 seconds
├─ Normal endpoints (100-500ms): 30 seconds
├─ Slow endpoints (> 500ms): 60+ seconds
├─ Report generation: 5+ minutes
├─ File upload: 10+ minutes (based on size)

Timeout per layer:

Client timeout: 60 seconds
  ↓
API Gateway timeout: 55 seconds
  ↓
Service timeout: 50 seconds
  ↓
Database timeout: 40 seconds

Reason:
├─ Each layer shorter timeout
├─ Inner layers fail first
├─ Resources freed promptly
├─ Prevents cascading timeouts
```

---

# 7️⃣ BEST PRACTICES

## Best Practice 1: AVOID MEMORY LEAKS

### What Are Memory Leaks?

```
Memory leak example:

Request 1: Create context
├─ Add user data (1 KB)
├─ Add cache data (100 KB)
├─ Add custom state (50 KB)
└─ Total: 151 KB

Request complete:
├─ Context not cleaned up
├─ Memory not freed
├─ Reference still exists
└─ 151 KB wasted

Requests: 10,000
Memory wasted: 10,000 × 151 KB = 1.5 GB!

Problem:
├─ Server runs out of memory
├─ Performance degrades
├─ Eventually crashes
└─ Service unavailable
```

### Prevention: Proper Cleanup

```
Cleanup pattern:

function handle_request(request):
  try {
    context = create_context()
    request.context = context
    
    // Process request
    response = call_handler(request)
    
    return response
    
  } finally {
    // ALWAYS cleanup, even if error
    cleanup_context(request.context)
    request.context = null
    request = null
  }
```

### Cleanup Checklist

```
Cleanup tasks:

☐ Clear user data reference
  ├─ context.user = null

☐ Clear session data reference
  ├─ context.session = null

☐ Clear temporary state
  ├─ context.cache = null
  ├─ context.computed = null

☐ Clear metadata
  ├─ context.metadata = null

☐ Cancel pending operations
  ├─ If timeout still running: cancel it
  ├─ If async operations: cancel them

☐ Close resources
  ├─ If file handles: close them
  ├─ If streams: close them

☐ Remove request reference
  ├─ request = null
  ├─ context = null

Result:
└─ All references cleared
└─ Garbage collection reclaims memory
└─ No memory leak
```

### Memory Leak Detection

```
Monitor memory usage:

Server startup: 500 MB (baseline)

After 1000 requests: 550 MB (50 MB used, expected)
After 10,000 requests: 1.5 GB (1000 MB leaked!)

Signs of leak:
├─ Memory usage increases over time
├─ Doesn't decrease when no requests
├─ Becomes unbounded
├─ Eventually crashes

Tools:
├─ Memory profilers (track heap)
├─ Monitoring dashboards (track memory over time)
├─ Load testing (run many requests, watch memory)

Fix:
├─ Find objects not cleaned up
├─ Add cleanup code
├─ Verify cleanup runs in finally block
└─ Re-test with memory profiler
```

## Best Practice 2: AVOID TIGHT COUPLING

### What Is Tight Coupling?

```
Tight coupling example:

Middleware directly accesses request object:

function custom_middleware(request):
  user_id = request.body.user_id
  request.context.user = fetch_user(user_id)
  request.context.extra_data = fetch_extra(user_id)
  request.context.cache = fetch_cache(user_id)

Handler directly calls middleware functions:

function handler(request):
  add_user_context(request)
  add_extra_data(request)
  add_cache(request)
  
  do_business_logic(request)

Problems:
├─ Hard to test (mock request object)
├─ Hard to refactor (middleware tightly bound)
├─ Hard to reuse (can't use different context)
├─ Changes in one place break others
├─ Hard to reason about dependencies
└─ Hard to change implementation
```

### Solution: Loose Coupling (Dependency Injection)

```
Loose coupling with dependency injection:

Middleware returns function:

function create_auth_middleware(user_service):
  return function middleware(request):
    token = request.headers.Authorization
    user = user_service.verify_token(token)
    request.context.user = user

Middleware chained:

middlewares = [
  create_auth_middleware(user_service),
  create_logging_middleware(logger),
  create_validation_middleware(validator)
]

Handler signature:

function handler(request):
  user = request.context.user  // Filled by middleware
  do_business_logic(user)

Benefits:
├─ Easy to test (mock services)
├─ Easy to refactor (just swap middleware)
├─ Easy to reuse (same middleware, different services)
├─ Changes isolated
├─ Clear dependencies
└─ Testable
```

### Dependency Injection Pattern

```
Pattern 1: Constructor injection

class UserHandler:
  constructor(user_service, auth_service):
    this.user_service = user_service
    this.auth_service = auth_service
  
  handle(request):
    user = this.user_service.get(request.context.user_id)
    return user

Usage:
handler = UserHandler(user_service, auth_service)
response = handler.handle(request)


Pattern 2: Function parameter injection

function handle_user(request, user_service):
  user = user_service.get(request.context.user_id)
  return user

Usage:
response = handle_user(request, user_service)


Pattern 3: Container/Factory

class Container:
  constructor():
    this.services = {}
  
  register(name, factory):
    this.services[name] = factory
  
  get(name):
    return this.services[name]()

container.register("user_service", () => UserService())
container.register("handler", () => 
  UserHandler(
    container.get("user_service"),
    container.get("auth_service")
  )
)

handler = container.get("handler")
```

### Avoiding Coupling: Rules

```
✓ DO:
├─ Pass dependencies as parameters
├─ Use interfaces/contracts
├─ Inject services
├─ Use factory functions
├─ Use container/DI framework
└─ Decouple from frameworks

✗ DON'T:
├─ Use global variables
├─ Import services directly in functions
├─ Tightly couple to request object
├─ Create dependencies inside functions
├─ Hardcode dependencies
└─ Mix concerns
```

## Best Practice 3: CONTEXT ISOLATION

### Why Isolation Matters?

```
Problem without isolation:

Request A modifies context:
├─ context.user = john
├─ context.cache = {...}
└─ context.state = "processed"

Request B should get own context:
├─ But somehow gets Request A's context
├─ context.user = john (WRONG! Should be different user)
├─ context.cache = {...} (WRONG! Shared data)
├─ Data contamination!

This happens when:
├─ Not creating separate context per request
├─ Context stored in global variable
├─ Context not cleaned up
└─ Threading issues
```

### Isolation Implementation

```
Each request gets own context:

function create_request_context():
  return {
    request_id: generate_uuid(),
    user: null,
    metadata: {},
    cache: {},
    state: {}
  }

Middleware attaches context:

function context_middleware(request):
  request.context = create_request_context()

Handler receives isolated context:

function handler(request):
  // This context is ONLY for this request
  user = request.context.user
  cache = request.context.cache
  
  // Changes don't affect other requests
  request.context.cache["key"] = "value"

Cleanup removes context:

finally {
  request.context = null  // Isolated context destroyed
}
```

## Best Practice 4: THREAD SAFETY

### Concurrency Issues

```
Problem: Two requests running simultaneously

Request A (Thread 1):
  context_A.user = john
  
Request B (Thread 2):
  context_B.user = jane
  
If using global context (BAD):
  global_context.user = john
  global_context.user = jane (overwrites!)
  
Request A continues:
  Use global_context.user
  Expected: john
  Actually: jane
  WRONG!

Solution:
├─ Use thread-local storage
├─ Each thread gets own context
├─ No interference
└─ Thread-safe by design
```

### Thread-Local Implementation

```
Thread-local storage:

thread_local_context = {}

function set_context(context):
  thread_local_context[current_thread_id()] = context

function get_context():
  return thread_local_context[current_thread_id()]

Request A (Thread 1):
  context_A = create_context()
  set_context(context_A)
  context = get_context()  // Gets context_A
  
Request B (Thread 2):
  context_B = create_context()
  set_context(context_B)
  context = get_context()  // Gets context_B

Each thread has own context:
├─ No conflicts
├─ Thread-safe
└─ Works correctly
```

## Best Practice 5: TIMEOUT HANDLING

### Set Reasonable Timeouts

```
Rule: Timeout = Expected Time + Buffer

Fast endpoint (10ms typical):
  Timeout = 10ms + 50ms (5x buffer) = 60ms

Normal endpoint (100ms typical):
  Timeout = 100ms + 900ms (10x buffer) = 1 second

Slow endpoint (1 second typical):
  Timeout = 1s + 4s (5x buffer) = 5 seconds

Never:
├─ No timeout (request can hang forever!)
├─ Timeout too short (normal requests fail)
├─ Timeout same as expected time (no buffer)
```

### Timeout Propagation

```
Set timeout at each layer:

Client: 60 second timeout
  ↓
API Gateway: 55 second timeout
  ├─ Request starts: 0ms
  ├─ Timeout would be: 55 seconds from now
  ├─ Tells Service A: You have 50 seconds
  └─ Request starts: 0ms
    ↓
    Service A: 50 second timeout
    ├─ Does work: 10 seconds
    ├─ Calls Service B with 40 seconds remaining
    └─ Request starts: 0ms
      ↓
      Service B: 40 second timeout
      ├─ Does work: 30 seconds
      ├─ Calls Database with 10 seconds remaining
      └─ Request starts: 0ms
        ↓
        Database: 10 second timeout

Result:
├─ Service B times out first (10s)
├─ Service A gets error, can recover
├─ API Gateway gets error, can handle
├─ Client gets error response
└─ No cascading timeouts, clean failure
```

## Best Practice 6: ERROR HANDLING IN CONTEXT

### Clean Error Context

```
When error occurs:

try {
  do_work()
} catch error {
  // Store error in context
  request.context.error = {
    type: error.type,
    message: error.message,
    stack_trace: error.stack_trace (development only!)
  }
  
  // Log with context
  logger.error({
    request_id: request.context.request_id,
    user_id: request.context.user?.id,
    error: error.message
  })
  
  // Still cleanup on finally!
} finally {
  cleanup_context()  // Happens even if error!
}
```

### Context Propagation on Error

```
Error handling with context:

function middleware_chain():
  context = create_context()
  
  try {
    auth_middleware(context)
    validation_middleware(context)
    handler(context)
  } catch error {
    error_middleware(context, error)
  } finally {
    cleanup_middleware(context)  // Always runs!
  }

Each middleware:
├─ Adds to context
├─ Or handles error
├─ Or passes to next

Finally block:
└─ Always cleans up
```

---

# 8️⃣ COMMON PITFALLS & SOLUTIONS

## Pitfall 1: Global Request Context

### Problem

```
❌ BAD: Global request context

request_context = {}

function authenticate(request):
  request_context.user = authenticate_user()

function handle():
  use request_context.user

Issues:
├─ Concurrent requests overwrite each other
├─ Request A sets user, Request B sets different user
├─ Request A reads Request B's user!
├─ Thread-safety issues
├─ Unpredictable behavior
└─ Data leakage
```

### Solution

```
✓ GOOD: Request-scoped context

function middleware(request):
  request.context = create_context()
  
  request.context.user = authenticate_user()

function handler(request):
  use request.context.user

Benefits:
├─ Each request has own context
├─ No overwrites
├─ Thread-safe
├─ Predictable behavior
├─ No data leakage
└─ Isolated per request
```

## Pitfall 2: Forgetting to Cleanup

### Problem

```
❌ BAD: No cleanup

function handle_request(request):
  context = create_context()
  request.context = context
  
  // Process request
  response = handler(request)
  
  return response
  // Context NOT cleaned up!
  // Memory leak!

Issues:
├─ Context objects accumulate
├─ Memory usage grows
├─ Eventually crashes
├─ Hard to debug
└─ Intermittent failures
```

### Solution

```
✓ GOOD: Cleanup in finally

function handle_request(request):
  context = create_context()
  request.context = context
  
  try {
    // Process request
    response = handler(request)
    return response
  } finally {
    // ALWAYS cleanup!
    request.context = null
    context = null
    // Garbage collection frees memory
  }

Benefits:
├─ No memory leaks
├─ Predictable memory usage
├─ Reliable performance
├─ No crashes
└─ Runs even if error!
```

## Pitfall 3: Exposing Internal Data

### Problem

```
❌ BAD: Exposing sensitive data in errors

function handler(request):
  try {
    query_database()
  } catch error {
    return {
      error: error.message,  // "Connection refused to MySQL at localhost:3306"
      stack_trace: error.stack  // Full stack trace exposing paths
    }
  }

Issues:
├─ Database connection details exposed
├─ Server paths exposed
├─ Secrets might be in stack trace
├─ Information disclosure
├─ Security vulnerability!
```

### Solution

```
✓ GOOD: Safe error messages

function handler(request):
  try {
    query_database()
  } catch error {
    logger.error({
      request_id: request.context.request_id,
      error: error.message,  // Logged securely
      stack_trace: error.stack  // Not sent to client
    })
    
    return {
      error: "Server error",  // Generic message
      request_id: request.context.request_id  // For support
    }
  }

Benefits:
├─ No sensitive data exposed
├─ User gets generic error
├─ Support can trace with request_id
├─ Secure
└─ Professional
```

## Pitfall 4: Tight Coupling to Request Object

### Problem

```
❌ BAD: Tightly coupled functions

function validate_user(request):
  username = request.body.username
  email = request.body.email
  // Directly uses request object
  return validate(username, email)

function process_user(request):
  if !validate_user(request) {
    return error
  }
  // Can't test without request object
  // Can't reuse with different input
  // Coupled to request structure
```

### Solution

```
✓ GOOD: Loosely coupled with passed parameters

function validate_user(username, email):
  // No dependency on request object
  return validate(username, email)

function process_user(request):
  if !validate_user(request.body.username, request.body.email) {
    return error
  }
  // Easy to test
  // Easy to reuse
  // No coupling
  // Can pass any values

Usage:
├─ In handler: validate_user(request.body.username, ...)
├─ In tests: validate_user("john", "john@example.com")
├─ In scheduled tasks: validate_user(user.username, user.email)
└─ Flexible!
```

## Pitfall 5: Not Propagating Context

### Problem

```
❌ BAD: Not passing context to functions

function handler(request):
  user_id = request.context.user.id
  
  result = fetch_user_details(user_id)
  // Lost trace_id when calling function!
  // Hard to debug
  // No logging correlation

function fetch_user_details(user_id):
  // Doesn't have request context
  // Can't log with trace_id
  // Can't check timeout
  // Can't access user info
```

### Solution

```
✓ GOOD: Propagate context to functions

function handler(request):
  user_id = request.context.user.id
  result = fetch_user_details(request.context, user_id)

function fetch_user_details(context, user_id):
  logger.info({
    trace_id: context.trace_id,  // Can log with trace!
    message: "Fetching user details"
  })
  
  if context.is_expired() {
    return error  // Can check timeout!
  }
  
  return fetch_from_db(user_id)

Benefits:
├─ Context available in all functions
├─ Consistent logging
├─ Timeout checking possible
├─ Tracing works end-to-end
└─ Easy debugging
```

---

# 9️⃣ REAL-WORLD EXAMPLES

## Example 1: User Authentication Flow

```
REQUEST: POST /api/auth/login

Step 1: Create Request Context
├─ Middleware: context_creation_middleware
├─ Creates: new request context
├─ Sets: request_id = "req_abc123xyz"
├─ Sets: trace_id = "trace_xyz789"
├─ Sets: start_time = now()
└─ Sets: deadline = now() + 30 seconds

Step 2: Extract Metadata
├─ Middleware: metadata_middleware
├─ Sets: context.metadata.api_version = "v2"
├─ Sets: context.metadata.source = "mobile-ios"
├─ Sets: context.metadata.device_type = "iPhone"
└─ Sets: context.metadata.features = ["new-auth"]

Step 3: Body Parser
├─ Middleware: body_parser_middleware
├─ Parses: JSON body
├─ Sets: request.body = { email: "john@gmail.com", password: "pass123" }
└─ Validates: Body parsed correctly

Step 4: Rate Limiting
├─ Middleware: rate_limiting_middleware
├─ Checks: Login attempts from this IP
├─ Count: 3/5 attempts (within limit)
├─ Allows: Continue

Step 5: Validation
├─ Middleware: validation_middleware
├─ Validates: Email format correct
├─ Validates: Password length >= 8
├─ All pass: Continue

Step 6: Handler Execution
├─ Handler: login_handler
├─ Queries: Find user by email
├─ Found: User exists
├─ Verifies: Password matches
├─ Correct: Password verified
├─ Creates: Session
├─ Sets: context.session = { id: "session_abc", user_id: 1 }
├─ Creates: Token
├─ Sets: context.token = "jwt_token_xyz"
└─ Creates: Response

Step 7: Logging
├─ Middleware: logging_middleware
├─ Logs: {
│   timestamp: "2025-02-09T13:30:45Z",
│   request_id: "req_abc123xyz",
│   trace_id: "trace_xyz789",
│   event: "user_login",
│   user_id: 1,
│   ip: "192.168.1.100",
│   status: "success"
├─ }
└─ Continues

Step 8: Response
├─ Status: 200 OK
├─ Headers: {
│   X-Request-ID: "req_abc123xyz",
│   X-Trace-ID: "trace_xyz789"
├─ }
├─ Body: {
│   session_id: "session_abc",
│   token: "jwt_token_xyz",
│   user: { id: 1, username: "john" }
├─ }
└─ Sent to client

Step 9: Cleanup
├─ Finally block runs
├─ Clears: context.user = null
├─ Clears: context.session = null
├─ Clears: context.token = null
├─ Clears: request.context = null
└─ Memory freed

Total time: ~100ms
```

## Example 2: Protected Resource with Timeout

```
REQUEST: GET /api/reports/generate

Step 1-2: Context & Metadata (same as above)
├─ request_id: "req_def456uvw"
├─ trace_id: "trace_uvw123"
├─ start_time: 13:31:00
├─ deadline: 13:31:30 (30 second timeout!)
└─ api_version: "v2"

Step 3: Authentication
├─ Middleware: auth_middleware
├─ Checks: Authorization header
├─ Token: "Bearer jwt_token_xyz"
├─ Verifies: Token signature valid
├─ Decodes: user_id = 1, role = "admin"
├─ Sets: context.user = { id: 1, role: "admin" }
└─ Continues

Step 4: Authorization
├─ Middleware: authz_middleware
├─ Requires: "report_generation" permission
├─ User has: ["reports_read", "reports_generate"]
├─ Check: "reports_generate" in permissions? YES
└─ Continues

Step 5: Handler Execution
├─ Handler: generate_report_handler
├─ Starts: 13:31:00
├─ Loop through data:
│  ├─ Iteration 1: 13:31:02 (2 seconds elapsed, 28 remaining)
│  ├─ Check: context.is_expired()? NO
│  ├─ Iteration 2: 13:31:05 (5 seconds elapsed, 25 remaining)
│  ├─ Check: context.is_expired()? NO
│  ├─ ...
│  ├─ Iteration 50: 13:31:28 (28 seconds elapsed, 2 remaining)
│  ├─ Check: context.is_expired()? NO
│  ├─ Iteration 51: Starts query (2 seconds remaining)
│  ├─ Query takes: 3 seconds
│  ├─ Timeout reached at: 13:31:30
│  └─ Timeout fires!

Step 6: Timeout Handling
├─ Timeout middleware triggers
├─ Sets: context.cancelled = true
├─ Sets: context.cancel_reason = "timeout"
├─ Handler detects cancellation
├─ Stops processing
├─ Cleanup: Cancel database query
├─ Returns: 504 GATEWAY TIMEOUT

Step 7: Error Response
├─ Status: 504 GATEWAY TIMEOUT
├─ Headers: {
│   X-Request-ID: "req_def456uvw",
│   X-Trace-ID: "trace_uvw123"
├─ }
├─ Body: {
│   error: "Request timeout",
│   request_id: "req_def456uvw",
│   message: "Report generation exceeded 30 second limit"
├─ }
└─ Sent to client

Step 8: Logging
├─ Logs: {
│   request_id: "req_def456uvw",
│   event: "report_timeout",
│   duration_ms: 30000,
│   user_id: 1
├─ }
└─ Continues

Step 9: Cleanup
├─ Finally block runs
├─ Cancels pending operations
├─ Frees memory
└─ Done

Total time: 30 seconds (hit timeout)
```

## Example 3: Error with Request Tracing

```
REQUEST: POST /api/users/profile

Step 1-5: Setup (as before)
├─ request_id: "req_ghi789xyz"
├─ trace_id: "trace_xyz456"
└─ context.user = { id: 1, username: "john" }

Step 6: Handler Execution
├─ Handler: update_profile_handler
├─ Validates: Input data valid
├─ Calls: user_service.update(user_id, data)
│
├─ Service method: user_service.update()
│  ├─ Logs: {
│  │   trace_id: "trace_xyz456",
│  │   event: "updating_user",
│  │   user_id: 1
│  ├─ }
│  ├─ Calls: database.update_user()
│  │
│  ├─ Database: update_user()
│  │  ├─ Logs: {
│  │  │   trace_id: "trace_xyz456",
│  │  │   event: "db_update_start",
│  │  │   table: "users"
│  │  ├─ }
│  │  ├─ Executes: UPDATE users SET ...
│  │  ├─ Error: Foreign key constraint violated!
│  │  ├─ Throws: ForeignKeyError()
│  │  └─ Logs: {
│  │      trace_id: "trace_xyz456",
│  │      event: "db_error",
│  │      error: "Foreign key constraint violated"
│  │    }
│  │
│  ├─ Catches: ForeignKeyError()
│  ├─ Logs: {
│  │   trace_id: "trace_xyz456",
│  │   event: "update_failed",
│  │   error: "FK violation"
│  ├─ }
│  └─ Throws: ServiceError("Invalid profile data")
│
├─ Handler catches: ServiceError()
├─ Logs: {
│   request_id: "req_ghi789xyz",
│   trace_id: "trace_xyz456",
│   event: "update_error",
│   user_id: 1,
│   error: "Invalid profile data"
├─ }
└─ Returns: Error response

Step 7: Error Response
├─ Status: 422 UNPROCESSABLE ENTITY
├─ Headers: {
│   X-Request-ID: "req_ghi789xyz",
│   X-Trace-ID: "trace_xyz456"
├─ }
├─ Body: {
│   error: "Failed to update profile",
│   request_id: "req_ghi789xyz",
│   message: "Invalid profile data"
├─ }
└─ Sent to client

Step 8: Tracing
├─ Support gets trace_id: "trace_xyz456"
├─ Searches logs for: trace_id = "trace_xyz456"
├─ Finds: All logs for this request
│  ├─ Handler log: update started
│  ├─ Service log: update attempted
│  ├─ Database log: constraint violation
│  ├─ Service log: error caught
│  └─ Handler log: error returned
├─ Can see: Exact error point
├─ Can see: All operations performed
└─ Can debug: Issue completely

Step 9: Cleanup
├─ Finally block runs
├─ Context cleaned
└─ Done

Total time: ~50ms (quick error)

VALUE: Entire request flow traceable with single trace_id!
```

---

# 🔟 QUICK REFERENCE

## Request Context Structure

```
request.context = {
  // Identification
  request_id: string,
  trace_id: string,
  
  // User & Auth
  user: {
    id: number,
    username: string,
    role: string,
    permissions: array,
    is_active: boolean
  },
  
  session: {
    id: string,
    user_id: number,
    created_at: timestamp,
    expires_at: timestamp,
    data: object
  },
  
  // Metadata
  metadata: {
    api_version: string,
    client_version: string,
    source: string,
    device_type: string,
    features: array
  },
  
  // Timing & Cancellation
  start_time: timestamp,
  timeout: number,
  deadline: timestamp,
  cancelled: boolean,
  cancel_reason: string,
  
  // State
  cache: object,
  state: object,
  error: object
}
```

## Middleware Order with Context

```
1. Context Creation → Create new context, generate IDs
2. Metadata Extraction → Parse headers, fill metadata
3. Body Parser → Parse request body
4. Rate Limiting → Check if within limits
5. CORS → Check origin
6. Authentication → Fill context.user
7. Authorization → Check permissions
8. Validation → Validate input
9. Handler → Use context, process request
10. Error Handler → Format errors
11. Logging → Log with context
12. Cleanup → Clear context, free memory
```

## Common Context Operations

```
Set user:
context.user = { id: 1, username: "john" }

Get user:
user_id = context.user.id

Check authenticated:
if context.user:
  authenticated = true

Check permission:
if "admin" in context.user.permissions:
  allowed = true

Get trace ID for logging:
log({trace_id: context.trace_id, message: "..."})

Check timeout:
if context.is_expired():
  return error

Cancel request:
context.cancel("reason")

Get remaining time:
remaining = context.remaining_time()
```

## Best Practices Checklist

```
☐ Create new context per request
☐ Never use global context
☐ Always cleanup in finally block
☐ Set appropriate timeouts
☐ Propagate context to functions
☐ Don't expose sensitive data in errors
☐ Use request_id for support
☐ Use trace_id for debugging
☐ Check cancellation in loops
☐ Avoid tight coupling
☐ Thread-safe (use thread-local or request object)
☐ Isolate per request
☐ Extract metadata from headers
☐ Store authenticated user in context
☐ Monitor for memory leaks
```

## Quick Decision Tree

```
Do I need to store data for this request?
├─ YES
│  ├─ One-time use (cache, computed value)?
│  │  └─ Store in context.cache
│  ├─ Authenticated user?
│  │  └─ Store in context.user
│  ├─ Request info (ID, trace)?
│  │  └─ Generate and store in context
│  └─ Session data?
│     └─ Store in context.session
└─ NO → Don't store

Data needs to survive request?
├─ YES → Store in database/cache (not context!)
└─ NO → Store in context

Multiple requests need this data?
├─ YES → Store in database/cache (not context!)
└─ NO → Store in context

Data changes per request?
├─ YES → Store in context (request-scoped)
└─ NO → Store globally or in database

Need to track request flow?
├─ YES → Use trace_id
├─ Need to reference request?
│  └─ YES → Use request_id
└─ NO → Don't track
```

---

## 🎯 SUMMARY: REQUEST CONTEXT

### What
- Container holding data for single HTTP request
- Created when request arrives, destroyed when response sent
- Not shared with other requests

### Why
- Prevents data leakage between requests
- Enables request-scoped state
- Supports tracing and logging
- Thread-safe concurrent processing
- Timeout and cancellation management

### How
1. Create context at request start
2. Attach authenticated user
3. Store metadata
4. Pass to handlers/functions
5. Log with request/trace IDs
6. Handle timeouts
7. Support cancellation
8. Cleanup in finally block

### Key Components
- Request ID (unique per request)
- Trace ID (unique per user flow)
- Authenticated user
- Session data
- Metadata
- Timeout/deadline
- Cancellation signal

### Best Practices
- One context per request
- Always cleanup
- Propagate to functions
- Check timeout in loops
- Avoid tight coupling
- Thread-safe
- Safe error messages
- Monitor memory

**Master request context, and your backend will be robust, debuggable, and secure!** 🚀

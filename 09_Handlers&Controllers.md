# 🏗️ CONTROLLERS, HANDLERS & MVC ARCHITECTURE - COMPLETE GUIDE

---

## 📌 TABLE OF CONTENTS

1. **What Is MVC Architecture?**
2. **MVC Architecture Deep Dive**
3. **Controllers Explained**
4. **Handlers Explained**
5. **Services Explained**
6. **Roles & Relationships**
7. **Centralized Error Handling**
8. **Consistent API Responses**
9. **Best Practices**
10. **Real-World Examples**
11. **Quick Reference**

---

# 1️⃣ WHAT IS MVC ARCHITECTURE?

## Definition

**MVC**: Model-View-Controller - A design pattern that separates application into 3 interconnected components

**Each Component**:
- **Model**: Data & business logic
- **View**: User interface (presentation)
- **Controller**: Handles user input, coordinates Model & View

## Visual Overview

```
USER/CLIENT
    ↓
USER INTERACTION
    ↓
CONTROLLER
├─ Receives input
├─ Processes request
├─ Calls Model if needed
├─ Selects View
└─ Sends to View
    ↓
MODEL
├─ Stores data
├─ Business logic
├─ Validation
└─ Returns data
    ↓
VIEW
├─ Formats data
├─ Renders output
└─ Sends to user
    ↓
USER SEES OUTPUT
```

## Why MVC?

```
Problems it solves:

1. SEPARATION OF CONCERNS
   ├─ Model: Only handles data
   ├─ View: Only handles presentation
   ├─ Controller: Only handles coordination
   └─ Each has single responsibility

2. CODE REUSABILITY
   ├─ Model reused by multiple Controllers
   ├─ View can use data from different Models
   ├─ Controller logic portable
   └─ Less duplication

3. TESTABILITY
   ├─ Model easy to unit test (no UI)
   ├─ Controller easy to test (mock Model)
   ├─ View testable separately
   └─ Components isolated

4. MAINTAINABILITY
   ├─ Easy to locate code
   ├─ Easy to change one layer
   ├─ Changes don't affect others
   ├─ Clear structure
   └─ New developers understand quickly

5. SCALABILITY
   ├─ Can add new Models
   ├─ Can add new Views
   ├─ Can add new Controllers
   ├─ Independent scaling
   └─ No tight coupling
```

---

# 2️⃣ MVC ARCHITECTURE DEEP DIVE

## Traditional Web MVC (HTML/Browser)

```
USER → BROWSER
    ↓
REQUEST (e.g., GET /users)
    ↓
CONTROLLER
├─ Routes request
├─ Validates input
├─ Calls Model
├─ Gets data
└─ Sends to View
    ↓
MODEL
├─ Queries database
├─ Processes business logic
├─ Returns data to Controller
└─ No knowledge of View
    ↓
VIEW
├─ Receives data from Controller
├─ Renders HTML with data
├─ Formats presentation
└─ Sends HTML to Browser
    ↓
BROWSER
├─ Receives HTML
├─ Renders page
├─ User sees result
└─ USER SATISFIED ✓
```

## API/REST MVC (Different from Web MVC)

```
CLIENT (Mobile/Frontend)
    ↓
HTTP REQUEST (e.g., GET /api/users)
    ↓
CONTROLLER (API Controller)
├─ Routes request
├─ Validates input
├─ Calls Service/Model
├─ Gets data
└─ Sends to formatter
    ↓
MODEL/SERVICE
├─ Queries database
├─ Processes business logic
├─ Returns data
└─ No knowledge of response format
    ↓
RESPONSE FORMATTER
├─ Converts data to JSON
├─ Adds metadata
├─ Adds headers
└─ Ready for network
    ↓
HTTP RESPONSE (JSON)
    ↓
CLIENT
├─ Receives JSON
├─ Parses data
├─ Renders UI
└─ USER SATISFIED ✓

NOTE: In APIs, "View" is JSON/Response formatter
      Not HTML rendering
```

## MVC Component Details

### Model Component

```
MODEL: Data & Business Logic

Responsibilities:
├─ Store & retrieve data
├─ Business logic
├─ Validation
├─ Calculations
├─ Database operations
├─ State management

Does NOT:
├─ Know about UI/View
├─ Know about HTTP
├─ Know about user actions
├─ Handle requests directly

Example:
User Model:
├─ Properties: id, name, email, password
├─ Methods:
│  ├─ create_user()
│  ├─ get_user(id)
│  ├─ update_user()
│  ├─ delete_user()
│  ├─ validate_email()
│  ├─ hash_password()
│  └─ is_active()
```

### Controller Component

```
CONTROLLER: Request Handler & Coordinator

Responsibilities:
├─ Receive HTTP request
├─ Parse request data
├─ Call appropriate Model/Service
├─ Handle response
├─ Return to client

Does NOT:
├─ Know database details
├─ Know business logic details
├─ Know UI rendering details
├─ Do business calculations

Example:
UserController:
├─ get_user_handler(id)
│  ├─ Parse request
│  ├─ Validate id
│  ├─ Call User.get_user(id)
│  ├─ Format response
│  └─ Return
│
├─ create_user_handler(request)
│  ├─ Parse request body
│  ├─ Validate data
│  ├─ Call User.create_user(data)
│  ├─ Format response
│  └─ Return
│
└─ update_user_handler(id, request)
   ├─ Parse request
   ├─ Validate data
   ├─ Call User.update_user(id, data)
   ├─ Format response
   └─ Return
```

### View Component (Web)

```
VIEW: Presentation Layer (HTML)

Responsibilities:
├─ Receive data from Controller
├─ Format/render data
├─ Display to user
├─ Handle user interactions

Does NOT:
├─ Access database directly
├─ Do business logic
├─ Know HTTP details

Example:
User Profile View:
├─ Receives: user object
├─ Renders: HTML with user info
├─ Displays: Name, email, profile pic
├─ Interactive: Edit button, delete button
└─ HTML sent to browser
```

### Response Formatter (API)

```
RESPONSE FORMATTER: Presentation Layer (JSON)

Responsibilities:
├─ Receive data from Controller
├─ Convert to JSON
├─ Add metadata
├─ Add headers
├─ Format for network

Example:
API Response Formatter:
├─ Receives: user object
├─ Formats: JSON with user info
├─ Adds: status, code, message
├─ Adds: headers (Content-Type, etc.)
└─ JSON sent to client

Response:
{
  "status": "success",
  "data": {
    "id": 1,
    "name": "John",
    "email": "john@gmail.com"
  },
  "timestamp": "2025-02-09T13:30:45Z"
}
```

## MVC Data Flow

```
Complete Request Lifecycle:

REQUEST ARRIVES
    ↓
ROUTING
├─ Determine which Controller
├─ Determine which method
└─ Route to appropriate handler

CONTROLLER
├─ Receive request
├─ Extract parameters
├─ Validate input
├─ Call Model/Service
└─ Await response

MODEL/SERVICE
├─ Receive parameters
├─ Apply business logic
├─ Query database
├─ Process data
├─ Return result

CONTROLLER (contd.)
├─ Receive data from Model
├─ Format result
├─ Create response object
└─ Send to View/Formatter

VIEW/FORMATTER
├─ Receive data
├─ Format for presentation
├─ Add metadata
├─ Return to Controller

CONTROLLER (final)
├─ Send response to client
└─ Request complete

RESPONSE SENT
```

---

# 3️⃣ CONTROLLERS EXPLAINED

## What Is a Controller?

**Definition**: Component that handles HTTP requests and coordinates between Model and View

**Core Responsibility**: Act as intermediary between client and business logic

## Controller Characteristics

```
A Controller:

✓ Receives HTTP requests
✓ Parses request data
✓ Validates input
✓ Calls appropriate service/model
✓ Formats response
✓ Returns response to client

✗ Does NOT:
✗ Do business logic itself
✗ Query database directly
✗ Know View implementation
✗ Make business decisions

Analogy:
Controller = Restaurant manager
├─ Takes customer order
├─ Validates order
├─ Sends to kitchen (Model)
├─ Receives prepared food
├─ Presents to customer
└─ Makes sure customer satisfied

Manager doesn't cook!
Model/Kitchen does cooking!
```

## Controller Structure

```
Controller Class:

class UserController:
  
  // Properties
  - user_service
  - logger
  - validator
  
  // Constructor (Dependency Injection)
  constructor(user_service, logger, validator):
    this.user_service = user_service
    this.logger = logger
    this.validator = validator
  
  // Handler Methods
  - get_user(request)
  - create_user(request)
  - update_user(request)
  - delete_user(request)
  - list_users(request)
  - search_users(request)
```

## Handler Methods in Controller

### Handler Method Structure

```
Generic handler method pattern:

handler_method(request):
  try {
    // Step 1: Extract & validate input
    input = extract_request_data(request)
    validation_result = validate(input)
    if validation_result.has_errors:
      return error_response(400, validation_result.errors)
    
    // Step 2: Call service
    result = service.perform_action(input)
    
    // Step 3: Format response
    response = format_response(result)
    
    // Step 4: Return response
    return response
  
  } catch error:
    // Handle error
    log_error(error)
    return error_response(500, error.message)
```

### Example 1: Get User Handler

```
Handler: get_user

Purpose: Retrieve single user by ID

get_user(request):
  try {
    // Step 1: Extract ID from URL path
    user_id = request.params.id
    
    // Step 2: Validate ID
    if !user_id:
      return error_response(400, "User ID required")
    
    if !is_valid_id(user_id):
      return error_response(400, "Invalid user ID format")
    
    // Step 3: Call service
    user = user_service.get_user(user_id)
    
    if !user:
      return error_response(404, "User not found")
    
    // Step 4: Format response
    response = {
      status: "success",
      data: user,
      timestamp: now()
    }
    
    return success_response(200, response)
  
  } catch error:
    return error_response(500, "Server error")
```

### Example 2: Create User Handler

```
Handler: create_user

Purpose: Create new user

create_user(request):
  try {
    // Step 1: Extract request body
    body = request.body  // Should be JSON
    
    // Step 2: Validate input
    validation_errors = {}
    
    if !body.email:
      validation_errors.email = "Email required"
    elif !is_valid_email(body.email):
      validation_errors.email = "Email format invalid"
    
    if !body.password:
      validation_errors.password = "Password required"
    elif body.password.length < 8:
      validation_errors.password = "Password min 8 chars"
    
    if !body.name:
      validation_errors.name = "Name required"
    
    if validation_errors not empty:
      return error_response(400, validation_errors)
    
    // Step 3: Check if email already exists
    existing_user = user_service.find_by_email(body.email)
    if existing_user:
      return error_response(409, "Email already registered")
    
    // Step 4: Call service to create user
    new_user = user_service.create_user({
      name: body.name,
      email: body.email,
      password: body.password
    })
    
    // Step 5: Format response
    response = {
      status: "success",
      data: {
        id: new_user.id,
        name: new_user.name,
        email: new_user.email
      },
      message: "User created successfully",
      timestamp: now()
    }
    
    return success_response(201, response)
  
  } catch error:
    return error_response(500, "Failed to create user")
```

### Example 3: Update User Handler

```
Handler: update_user

Purpose: Update existing user

update_user(request):
  try {
    // Step 1: Extract parameters
    user_id = request.params.id
    body = request.body
    
    // Step 2: Validate ID
    if !user_id:
      return error_response(400, "User ID required")
    
    // Step 3: Check authorization
    if !is_authenticated(request):
      return error_response(401, "Authentication required")
    
    if !can_edit_user(request.context.user, user_id):
      return error_response(403, "Not permitted to edit this user")
    
    // Step 4: Validate update data
    validation_errors = {}
    
    if body.email && !is_valid_email(body.email):
      validation_errors.email = "Email format invalid"
    
    if body.password && body.password.length < 8:
      validation_errors.password = "Password min 8 chars"
    
    if validation_errors not empty:
      return error_response(400, validation_errors)
    
    // Step 5: Call service
    updated_user = user_service.update_user(user_id, body)
    
    if !updated_user:
      return error_response(404, "User not found")
    
    // Step 6: Format response
    response = {
      status: "success",
      data: updated_user,
      message: "User updated successfully"
    }
    
    return success_response(200, response)
  
  } catch error:
    return error_response(500, "Failed to update user")
```

## Controller Responsibilities

```
What Controller DOES:

1. REQUEST RECEPTION
   ├─ Receive HTTP request
   ├─ Extract method, path, params
   ├─ Extract headers, body
   └─ Parse data

2. INPUT VALIDATION
   ├─ Check required fields
   ├─ Validate format
   ├─ Validate types
   └─ Reject invalid input early

3. AUTHORIZATION
   ├─ Check authentication
   ├─ Check permissions
   ├─ Ensure user can perform action
   └─ Reject unauthorized access

4. SERVICE COORDINATION
   ├─ Call appropriate service
   ├─ Pass validated data
   ├─ Await result
   └─ Handle service errors

5. RESPONSE FORMATTING
   ├─ Convert result to response format
   ├─ Add metadata
   ├─ Add status codes
   └─ Add headers

6. ERROR HANDLING
   ├─ Catch errors
   ├─ Log errors
   ├─ Format error response
   └─ Return appropriate status code

What Controller DOESN'T DO:

✗ Business logic (belongs in Service)
✗ Database queries (belongs in Model)
✗ Detailed calculations (belongs in Service)
✗ Know about database structure
✗ Know about UI rendering
✗ Know about presentation details
```

## Types of Controllers

### Type 1: REST Controller (API)

```
REST API Controller:

Routes:
GET /api/users → list_users()
GET /api/users/:id → get_user()
POST /api/users → create_user()
PUT /api/users/:id → update_user()
DELETE /api/users/:id → delete_user()

Response format: JSON
Status codes: Standard HTTP codes
Error handling: JSON error messages
```

### Type 2: Web Controller (HTML)

```
Web Controller:

Routes:
GET /users → show_list_view()
GET /users/:id → show_detail_view()
POST /users → create_user_and_redirect()
PUT /users/:id → update_user_and_redirect()
DELETE /users/:id → delete_user_and_redirect()

Response format: HTML
Status codes: 200, 302 (redirect), 404, 500
Error handling: HTML error pages
```

### Type 3: Hybrid Controller

```
Hybrid Controller:

Routes:
GET /api/users → JSON response
GET /users.html → HTML response
POST /api/users → JSON response
POST /users → Form response + redirect

Response format: JSON or HTML (based on request)
Status codes: Appropriate for format
Error handling: Format-specific errors
```

---

# 4️⃣ HANDLERS EXPLAINED

## What Is a Handler?

**Definition**: Specific method in a Controller that handles one HTTP endpoint

**Scope**: Individual request handling (one request → one handler method)

## Handler vs Controller

```
RELATIONSHIP:

Controller:
├─ Class/Component
├─ Contains multiple handlers
├─ One Controller handles related operations
└─ Example: UserController

Handler:
├─ Method in Controller
├─ Handles single endpoint
├─ One handler per endpoint
└─ Example: get_user(), create_user(), etc.

ANALOGY:

Restaurant:
├─ Server (Controller)
│  ├─ Takes order (handler 1)
│  ├─ Serves food (handler 2)
│  ├─ Processes payment (handler 3)
│  └─ Handles complaints (handler 4)

Each server action is a handler!
Server itself is the controller!
```

## Handler Anatomy

```
Handler Structure:

handler_name(request):
  // Input Phase
  ├─ Extract data from request
  ├─ Parse parameters
  └─ Get request context
  
  // Validation Phase
  ├─ Validate input format
  ├─ Validate business rules
  └─ Return error if invalid
  
  // Processing Phase
  ├─ Call service/model
  ├─ Execute business logic
  ├─ Get results
  └─ Handle processing errors
  
  // Formatting Phase
  ├─ Format response
  ├─ Add metadata
  ├─ Add status code
  └─ Create response object
  
  // Return Phase
  └─ Return response to client
```

## Handler Best Practices

### Best Practice 1: Single Responsibility

```
✗ BAD: Handler doing too much

create_user(request):
  // Extract
  // Validate
  // Hash password
  // Check email exists
  // Save to database
  // Create session
  // Send email
  // Update cache
  // Log audit trail
  // Format response
  // Return
  
  TOO MUCH! All in one handler!

✓ GOOD: Handler delegates

create_user(request):
  // Extract
  user_data = request.body
  
  // Validate
  validate(user_data)
  
  // Delegate to service
  result = user_service.create_user(user_data)
  
  // Format response
  response = format_response(result)
  
  // Return
  return response
  
  Service handles: password, email check, database, session, email, cache, audit, etc.
```

### Best Practice 2: Input Validation First

```
✗ BAD: Validate after using

create_user(request):
  user_data = request.body
  
  // Process without checking
  user = user_service.create_user(user_data)
  
  // Validate later (too late!)
  if !user.email:
    return error  // Should have validated first!

✓ GOOD: Validate immediately

create_user(request):
  user_data = request.body
  
  // Validate first!
  errors = validate(user_data)
  if errors:
    return error_response(400, errors)
  
  // Only process valid data
  user = user_service.create_user(user_data)
  
  return success_response(201, user)
```

### Best Practice 3: Early Returns for Errors

```
✗ BAD: Nested conditions

update_user(request):
  user_id = request.params.id
  
  if user_id:
    if is_authenticated(request):
      if has_permission(request.user, user_id):
        if is_valid_data(request.body):
          result = user_service.update(user_id, request.body)
          return response(result)
        else:
          return error_response(400, errors)
      else:
        return error_response(403, "Not permitted")
    else:
      return error_response(401, "Not authenticated")
  else:
    return error_response(400, "ID required")

Deep nesting! Hard to read!

✓ GOOD: Early returns

update_user(request):
  user_id = request.params.id
  
  // Check each condition, return early if fails
  if !user_id:
    return error_response(400, "ID required")
  
  if !is_authenticated(request):
    return error_response(401, "Not authenticated")
  
  if !has_permission(request.user, user_id):
    return error_response(403, "Not permitted")
  
  if !is_valid_data(request.body):
    return error_response(400, "Invalid data")
  
  // Only reach here if all checks pass
  result = user_service.update(user_id, request.body)
  return response(result)

Clear flow! Easy to read!
```

### Best Practice 4: Proper Error Handling

```
✗ BAD: Silent failures

create_user(request):
  data = request.body
  user = user_service.create_user(data)
  return response(user)  // What if service throws error?

No error handling!

✓ GOOD: Explicit error handling

create_user(request):
  try {
    data = request.body
    user = user_service.create_user(data)
    return success_response(201, user)
  
  } catch DuplicateEmailError as e:
    return error_response(409, "Email already exists")
  
  } catch ValidationError as e:
    return error_response(422, e.message)
  
  } catch error:
    log_error(error)
    return error_response(500, "Server error")
```

---

# 5️⃣ SERVICES EXPLAINED

## What Is a Service?

**Definition**: Component that encapsulates business logic

**Core Responsibility**: Execute business operations independent of how they're called

## Service Characteristics

```
A Service:

✓ Contains business logic
✓ Performs operations (create, update, delete, etc.)
✓ Handles data processing
✓ Manages transactions
✓ Calls Models/Repositories
✓ Reusable across Controllers
✓ No knowledge of HTTP/Request

✗ Does NOT:
✗ Know about HTTP requests
✗ Know about response format
✗ Know about clients
✗ Know about databases directly (uses Repository)
✗ Do HTTP-level validation

Analogy:
Service = Kitchen in restaurant
├─ Receives order (from waiter/controller)
├─ Executes cooking logic
├─ Uses ingredients (models/data)
├─ Doesn't know about customers
├─ Just cooks the food
└─ Returns finished dish
```

## Service Structure

```
Service Class:

class UserService:
  
  // Properties
  - user_repository
  - email_service
  - password_hasher
  - logger
  
  // Constructor (Dependency Injection)
  constructor(user_repository, email_service, hasher, logger):
    this.user_repository = user_repository
    this.email_service = email_service
    this.password_hasher = hasher
    this.logger = logger
  
  // Service Methods
  - create_user(data)
  - get_user(id)
  - update_user(id, data)
  - delete_user(id)
  - find_by_email(email)
  - authenticate(email, password)
  - validate_user_data(data)
  - change_password(user_id, old_pass, new_pass)
```

## Service Responsibilities

```
What Service DOES:

1. BUSINESS LOGIC
   ├─ Complex calculations
   ├─ Multi-step operations
   ├─ Business rules
   └─ Orchestration

2. DATA OPERATIONS
   ├─ Call repository
   ├─ Get data
   ├─ Save data
   ├─ Delete data
   └─ Process data

3. VALIDATION (Business level)
   ├─ Email unique?
   ├─ Password meets policy?
   ├─ Business rules satisfied?
   └─ Cannot perform?

4. SIDE EFFECTS
   ├─ Send emails
   ├─ Log activities
   ├─ Update cache
   ├─ Call external APIs
   └─ Trigger events

5. TRANSACTIONS
   ├─ Multiple operations
   ├─ All succeed or all fail
   ├─ Data consistency
   └─ Rollback on error

What Service DOESN'T DO:

✗ Parse HTTP requests
✗ Know about response format
✗ Know about clients/users
✗ Query database directly (uses Repository)
✗ Return HTTP responses
✗ Know about HTTP status codes
```

## Service Examples

### Example 1: User Creation Service

```
Service method: create_user

Purpose: Create new user with all business logic

create_user(user_data):
  try {
    // Step 1: Validate business rules
    if !this.is_valid_email(user_data.email):
      throw ValidationError("Invalid email format")
    
    if user_data.password.length < 8:
      throw ValidationError("Password too short")
    
    // Step 2: Check unique constraints
    existing = this.user_repository.find_by_email(user_data.email)
    if existing:
      throw DuplicateError("Email already registered")
    
    // Step 3: Transform data
    user_to_save = {
      email: user_data.email.lower(),  // normalize email
      name: user_data.name.trim(),     // normalize name
      password: this.password_hasher.hash(user_data.password),  // hash password
      created_at: now(),
      is_active: true
    }
    
    // Step 4: Save to database
    saved_user = this.user_repository.save(user_to_save)
    
    // Step 5: Side effects
    this.logger.info("User created: " + saved_user.id)
    this.email_service.send_welcome_email(saved_user.email)
    
    // Step 6: Return result
    return {
      id: saved_user.id,
      email: saved_user.email,
      name: saved_user.name,
      created_at: saved_user.created_at
    }
  
  } catch error:
    this.logger.error("Failed to create user: " + error.message)
    throw error
```

### Example 2: Authentication Service

```
Service method: authenticate

Purpose: Authenticate user with email/password

authenticate(email, password):
  try {
    // Step 1: Find user
    user = this.user_repository.find_by_email(email)
    
    if !user:
      throw AuthenticationError("Invalid credentials")
    
    // Step 2: Check if active
    if !user.is_active:
      throw AuthenticationError("Account disabled")
    
    // Step 3: Verify password
    password_correct = this.password_hasher.verify(password, user.password)
    
    if !password_correct:
      throw AuthenticationError("Invalid credentials")
    
    // Step 4: Update last login
    this.user_repository.update(user.id, {
      last_login: now()
    })
    
    // Step 5: Create session/token
    token = this.generate_token(user)
    
    // Step 6: Log
    this.logger.info("User authenticated: " + user.id)
    
    // Step 7: Return
    return {
      user_id: user.id,
      email: user.email,
      name: user.name,
      token: token,
      token_expires: token_expiration
    }
  
  } catch error:
    this.logger.warn("Authentication failed: " + email)
    throw error
```

### Example 3: Update User Service

```
Service method: update_user

Purpose: Update user with validation and side effects

update_user(user_id, update_data):
  try {
    // Step 1: Get existing user
    user = this.user_repository.get(user_id)
    
    if !user:
      throw NotFoundError("User not found")
    
    // Step 2: Validate update data
    if update_data.email && !this.is_valid_email(update_data.email):
      throw ValidationError("Invalid email format")
    
    // Step 3: Check unique constraints
    if update_data.email && update_data.email != user.email:
      existing = this.user_repository.find_by_email(update_data.email)
      if existing:
        throw DuplicateError("Email already in use")
    
    // Step 4: Prepare update
    updates = {}
    
    if update_data.name:
      updates.name = update_data.name.trim()
    
    if update_data.email:
      updates.email = update_data.email.lower()
      updates.email_verified = false  // Need to re-verify
    
    if update_data.password:
      if update_data.password.length < 8:
        throw ValidationError("Password too short")
      updates.password = this.password_hasher.hash(update_data.password)
    
    updates.updated_at = now()
    
    // Step 5: Save updates
    updated_user = this.user_repository.update(user_id, updates)
    
    // Step 6: Side effects
    this.logger.info("User updated: " + user_id)
    
    if update_data.email:
      this.email_service.send_verification_email(update_data.email)
    
    // Step 7: Return
    return updated_user
  
  } catch error:
    this.logger.error("Failed to update user: " + error.message)
    throw error
```

---

# 6️⃣ ROLES & RELATIONSHIPS

## Complete Architecture Overview

```
REQUEST ARRIVES
    ↓
ROUTE MATCHING
├─ Determine Controller
├─ Determine Handler
└─ Pass request to handler
    ↓
CONTROLLER (Handler Method)
├─ Parse request
├─ Validate input
├─ Call Service
└─ Format response
    ↓
SERVICE
├─ Execute business logic
├─ Call Repository
├─ Do side effects
└─ Return result
    ↓
REPOSITORY/MODEL
├─ Query database
├─ Retrieve/save data
├─ No business logic
└─ Return data
    ↓
SERVICE (continued)
├─ Receive data from Repository
├─ Process/transform
├─ Handle side effects
└─ Return to Controller
    ↓
CONTROLLER (continued)
├─ Receive result from Service
├─ Format response
├─ Add metadata
└─ Return to client
    ↓
RESPONSE SENT TO CLIENT
```

## Layer Separation

```
REQUEST LAYER (Controller)
├─ HTTP request/response
├─ Parse parameters
├─ Validate input format
├─ Format response
└─ Return HTTP response

BUSINESS LOGIC LAYER (Service)
├─ Core business logic
├─ Complex calculations
├─ Transactions
├─ Orchestration
└─ Side effects

DATA LAYER (Repository/Model)
├─ Database queries
├─ Data persistence
├─ Data retrieval
└─ Simple CRUD operations
```

## Who Calls Whom

```
HTTP Request
    ↓
    CONTROLLER
    ├─ Called by: Framework/Router
    ├─ Calls: Service
    ├─ Returns to: Framework → Client
    └─ Knows about: Request, Response
    
    SERVICE
    ├─ Called by: Controller
    ├─ Calls: Repository, External Services
    ├─ Returns to: Controller
    └─ Knows about: Business logic
    
    REPOSITORY/MODEL
    ├─ Called by: Service
    ├─ Calls: Database driver
    ├─ Returns to: Service
    └─ Knows about: Data structure
    
    DATABASE
    ├─ Called by: Repository
    └─ Returns: Raw data
```

## Dependency Direction

```
Controller ──depends on──> Service
    │                          │
    │                          ↓
    └──────────> Repository <──┘

Dependency Flow:
├─ Controller depends on Service (calls it)
├─ Service depends on Repository (calls it)
├─ Repository depends on Database (queries it)
└─ Nothing depends on Controller (it's top-level)

Important:
├─ Higher layers depend on lower
├─ Lower layers don't depend on higher
├─ Each layer replaceable/testable
└─ No circular dependencies
```

## Responsibilities Table

```
┌──────────────┬─────────────────┬─────────────────────────┐
│ Component    │ Responsibility  │ Example Methods         │
├──────────────┼─────────────────┼─────────────────────────┤
│ CONTROLLER   │ Request handling│ get_user()              │
│              │ Input validation│ create_user()           │
│              │ Coordination    │ update_user()           │
│              │ Response format │ delete_user()           │
├──────────────┼─────────────────┼─────────────────────────┤
│ SERVICE      │ Business logic  │ create_user_with...()   │
│              │ Transactions    │ authenticate()          │
│              │ Side effects    │ change_password()       │
│              │ Orchestration   │ transfer_funds()        │
├──────────────┼─────────────────┼─────────────────────────┤
│ REPOSITORY   │ Data access     │ save()                  │
│              │ CRUD ops        │ get()                   │
│              │ Queries         │ find_by_email()         │
│              │ Persistence     │ delete()                │
└──────────────┴─────────────────┴─────────────────────────┘
```

---

# 7️⃣ CENTRALIZED ERROR HANDLING

## What Is Centralized Error Handling?

**Definition**: Single, unified way to handle all errors in application

**Purpose**: Consistent error responses, easier debugging, better user experience

## Why Centralized?

```
Problem without centralization:

❌ Each handler catches errors differently:

Handler 1:
  catch error:
    return { error: error.message, code: 400 }

Handler 2:
  catch error:
    return error.message (just string!)

Handler 3:
  catch error:
    return { status: "failed", error: error }

Issues:
├─ Inconsistent response format
├─ Client confused (different formats)
├─ Hard to parse errors on client
├─ Unprofessional API
├─ Difficult to debug
└─ Difficult to maintain

✓ Solution: Centralized error handling

All errors pass through one place:
├─ Consistent format
├─ Consistent logging
├─ Consistent status codes
├─ Professional API
├─ Easy to debug
└─ Easy to maintain
```

## Error Hierarchy

```
Error Types:

1. VALIDATION ERRORS (4xx)
   ├─ Bad Request (400)
   ├─ Unprocessable Entity (422)
   └─ Should be caught by handlers

2. AUTHENTICATION ERRORS (401)
   ├─ Invalid credentials
   ├─ Expired token
   └─ Should be caught by auth middleware

3. AUTHORIZATION ERRORS (403)
   ├─ Insufficient permissions
   ├─ User not allowed
   └─ Should be caught by auth middleware

4. RESOURCE ERRORS (404)
   ├─ Resource not found
   └─ Should be caught by handlers

5. CONFLICT ERRORS (409)
   ├─ Duplicate resource
   ├─ State conflict
   └─ Should be caught by handlers

6. BUSINESS LOGIC ERRORS (422)
   ├─ Cannot perform operation
   ├─ Business rule violated
   └─ Should be caught by handlers

7. SERVER ERRORS (5xx)
   ├─ Unexpected errors
   ├─ Database errors
   ├─ External service errors
   └─ Should be caught by global handler
```

## Centralized Error Handler Pattern

### Pattern 1: Error Handler Middleware

```
Error Handler Middleware:

function error_handler_middleware(error, request, response):
  
  // Log error
  logger.error({
    timestamp: now(),
    request_id: request.context.request_id,
    path: request.path,
    method: request.method,
    error: error.message,
    stack: error.stack
  })
  
  // Determine error type
  error_type = determine_error_type(error)
  
  // Get status code
  status_code = get_status_code(error_type, error)
  
  // Format error response
  error_response = {
    status: "error",
    error: error_type,
    message: get_user_friendly_message(error),
    request_id: request.context.request_id,
    timestamp: now()
  }
  
  // Add details in development
  if is_development():
    error_response.details = {
      stack_trace: error.stack,
      technical_message: error.message
    }
  
  // Send response
  response.status(status_code).json(error_response)

// Register middleware (MUST be last!)
app.use(error_handler_middleware)
```

### Pattern 2: Error Classes

```
Structured errors:

Base Error Class:
class ApplicationError(error_message, status_code):
  this.message = error_message
  this.status_code = status_code
  this.timestamp = now()

Validation Error:
class ValidationError(message) extends ApplicationError:
  constructor(message):
    super(message, 400)
    this.type = "VALIDATION_ERROR"

Not Found Error:
class NotFoundError(message) extends ApplicationError:
  constructor(message):
    super(message, 404)
    this.type = "NOT_FOUND_ERROR"

Duplicate Error:
class DuplicateError(message) extends ApplicationError:
  constructor(message):
    super(message, 409)
    this.type = "DUPLICATE_ERROR"

Server Error:
class ServerError(message) extends ApplicationError:
  constructor(message):
    super(message, 500)
    this.type = "SERVER_ERROR"

Usage:
create_user(data):
  if !is_valid(data):
    throw ValidationError("Invalid data")
  
  if email_exists(data.email):
    throw DuplicateError("Email already registered")
  
  try {
    save_user(data)
  } catch error:
    throw ServerError("Failed to save user")
```

### Pattern 3: Error Handler Function

```
Error handler function:

function handle_error(error, request):
  
  // Determine error type and code
  switch error.type:
    case "VALIDATION_ERROR":
      return {
        status_code: 400,
        type: "VALIDATION_ERROR",
        message: "Invalid input",
        details: error.details
      }
    
    case "NOT_FOUND_ERROR":
      return {
        status_code: 404,
        type: "NOT_FOUND_ERROR",
        message: "Resource not found"
      }
    
    case "DUPLICATE_ERROR":
      return {
        status_code: 409,
        type: "DUPLICATE_ERROR",
        message: "Resource already exists"
      }
    
    case "AUTHENTICATION_ERROR":
      return {
        status_code: 401,
        type: "AUTHENTICATION_ERROR",
        message: "Invalid credentials"
      }
    
    case "AUTHORIZATION_ERROR":
      return {
        status_code: 403,
        type: "AUTHORIZATION_ERROR",
        message: "Insufficient permissions"
      }
    
    default:
      return {
        status_code: 500,
        type: "SERVER_ERROR",
        message: "Internal server error",
        request_id: request.context.request_id
      }
```

## Error Response Format

```
Standard error response:

{
  "status": "error",
  "error": {
    "type": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "code": "VALIDATION_FAILED",
    
    // Optional: Field-level errors
    "details": {
      "email": "Invalid email format",
      "password": "Password too short"
    }
  },
  
  "request_id": "req_abc123xyz",
  "trace_id": "trace_xyz789",
  "timestamp": "2025-02-09T13:30:45Z"
}

Different error types:

1. Validation Error (400):
{
  "status": "error",
  "error": {
    "type": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": {
      "email": "Invalid format",
      "password": "Too short"
    }
  }
}

2. Not Found Error (404):
{
  "status": "error",
  "error": {
    "type": "NOT_FOUND_ERROR",
    "message": "User not found"
  }
}

3. Duplicate Error (409):
{
  "status": "error",
  "error": {
    "type": "DUPLICATE_ERROR",
    "message": "Email already registered"
  }
}

4. Server Error (500):
{
  "status": "error",
  "error": {
    "type": "SERVER_ERROR",
    "message": "Internal server error"
  },
  "request_id": "req_abc123xyz"
}
```

## Error Handling Workflow

```
Error occurs anywhere in application:

Handler:
  try {
    service.do_something()
  } catch error {
    throw error  // Let error bubble up
  }

Service:
  try {
    repository.save(data)
  } catch error {
    throw ServiceError("Failed to save")
  }

Repository:
  try {
    database.query()
  } catch error {
    throw error  // Propagate
  }

Error bubbles up → reaches Global Error Handler

Global Error Handler:
  1. Catches error
  2. Determines type
  3. Logs error
  4. Formats response
  5. Sends to client

Client receives:
{
  "status": "error",
  "error": {...},
  "request_id": "..."
}

Client can:
├─ Display error to user
├─ Use request_id for support
├─ Handle based on error type
└─ Retry if appropriate
```

---

# 8️⃣ CONSISTENT API RESPONSES

## What Is Consistent Response Format?

**Definition**: All API responses follow same structure regardless of success/failure

**Purpose**: Easy client parsing, predictability, professional API

## Response Structure

```
Every API response has same structure:

Success Response:
{
  "status": "success",
  "code": 200,
  "message": "Operation successful",
  "data": { ... },
  "metadata": { ... },
  "request_id": "req_abc123xyz",
  "timestamp": "2025-02-09T13:30:45Z"
}

Error Response:
{
  "status": "error",
  "code": 400,
  "message": "Invalid input",
  "error": {
    "type": "VALIDATION_ERROR",
    "details": { ... }
  },
  "request_id": "req_abc123xyz",
  "timestamp": "2025-02-09T13:30:45Z"
}

Consistent fields:
├─ status: "success" or "error"
├─ code: HTTP status code
├─ message: Human-readable message
├─ request_id: For tracking
├─ timestamp: When response created
└─ data OR error: Specific content
```

## Response Builders

### Pattern 1: Response Factory

```
Response factory functions:

success_response(data, message = "", metadata = null):
  return {
    status: "success",
    code: 200,
    message: message || "Success",
    data: data,
    metadata: metadata,
    request_id: current_request.context.request_id,
    timestamp: now()
  }

created_response(data, message = ""):
  return {
    status: "success",
    code: 201,
    message: message || "Created",
    data: data,
    request_id: current_request.context.request_id,
    timestamp: now()
  }

no_content_response():
  return {
    status: "success",
    code: 204,
    message: "No content",
    request_id: current_request.context.request_id,
    timestamp: now()
  }

error_response(error_type, message, details = null):
  return {
    status: "error",
    code: error_type.status_code,
    message: message,
    error: {
      type: error_type.name,
      details: details
    },
    request_id: current_request.context.request_id,
    timestamp: now()
  }

Usage:

get_user(request):
  user = service.get_user(id)
  return success_response(user, "User retrieved")

create_user(request):
  user = service.create_user(data)
  return created_response(user, "User created")

delete_user(request):
  service.delete_user(id)
  return no_content_response()

try {
  ...
} catch error {
  return error_response(
    ValidationError,
    "Invalid input",
    { field: "email", message: "Invalid format" }
  )
}
```

### Pattern 2: Response Wrapper

```
Response wrapper class:

class ApiResponse:
  
  static success(data, message = "Success", code = 200):
    return {
      status: "success",
      code: code,
      message: message,
      data: data,
      request_id: get_request_id(),
      timestamp: now()
    }
  
  static created(data, message = "Created"):
    return this.success(data, message, 201)
  
  static error(type, message, details = null, code = 400):
    return {
      status: "error",
      code: code,
      message: message,
      error: {
        type: type,
        details: details
      },
      request_id: get_request_id(),
      timestamp: now()
    }
  
  static not_found(message = "Not found"):
    return this.error("NOT_FOUND", message, null, 404)
  
  static validation_error(message, details):
    return this.error("VALIDATION_ERROR", message, details, 400)
  
  static conflict(message):
    return this.error("CONFLICT", message, null, 409)
  
  static server_error(message, request_id = null):
    return this.error("SERVER_ERROR", message, null, 500)

Usage:

return ApiResponse.success(user, "User retrieved")
return ApiResponse.created(user, "User created")
return ApiResponse.not_found("User not found")
return ApiResponse.validation_error("Invalid input", errors)
return ApiResponse.conflict("Email already exists")
```

## Real Examples

### Example 1: Success Response

```
Request: GET /api/users/1

Handler:
get_user(request):
  user_id = request.params.id
  
  if !user_id:
    return error_response("VALIDATION_ERROR", "ID required", {
      field: "id",
      message: "ID parameter required"
    })
  
  user = service.get_user(user_id)
  
  if !user:
    return error_response("NOT_FOUND", "User not found")
  
  return success_response(user, "User retrieved successfully")

Response:
{
  "status": "success",
  "code": 200,
  "message": "User retrieved successfully",
  "data": {
    "id": 1,
    "name": "John",
    "email": "john@gmail.com",
    "created_at": "2025-01-15"
  },
  "request_id": "req_abc123xyz",
  "timestamp": "2025-02-09T13:30:45Z"
}
```

### Example 2: Created Response

```
Request: POST /api/users
Body: { name: "Jane", email: "jane@gmail.com", password: "password123" }

Handler:
create_user(request):
  data = request.body
  
  // Validate
  errors = validate_user_data(data)
  if errors:
    return error_response("VALIDATION_ERROR", "Invalid input", errors)
  
  // Check duplicate
  if service.email_exists(data.email):
    return error_response("CONFLICT", "Email already registered")
  
  // Create
  user = service.create_user(data)
  
  return created_response(user, "User created successfully")

Response:
{
  "status": "success",
  "code": 201,
  "message": "User created successfully",
  "data": {
    "id": 2,
    "name": "Jane",
    "email": "jane@gmail.com",
    "created_at": "2025-02-09T13:30:45Z"
  },
  "request_id": "req_def456uvw",
  "timestamp": "2025-02-09T13:30:46Z"
}
```

### Example 3: Error Response

```
Request: POST /api/users
Body: { name: "John" }  // Missing email and password

Handler:
create_user(request):
  data = request.body
  
  // Validate
  errors = {}
  
  if !data.email:
    errors.email = "Email required"
  
  if !data.password:
    errors.password = "Password required"
  
  if errors is not empty:
    return error_response("VALIDATION_ERROR", "Invalid input", errors)
  
  ...

Response:
{
  "status": "error",
  "code": 400,
  "message": "Invalid input",
  "error": {
    "type": "VALIDATION_ERROR",
    "details": {
      "email": "Email required",
      "password": "Password required"
    }
  },
  "request_id": "req_ghi789xyz",
  "timestamp": "2025-02-09T13:30:47Z"
}
```

### Example 4: List Response with Metadata

```
Request: GET /api/users?page=1&limit=10

Handler:
list_users(request):
  page = request.query.page || 1
  limit = request.query.limit || 10
  
  // Validate
  if page < 1 or limit < 1:
    return error_response("VALIDATION_ERROR", "Invalid pagination")
  
  // Get data
  result = service.list_users(page, limit)
  
  // Return with metadata
  return success_response(
    result.users,
    "Users retrieved",
    {
      pagination: {
        page: page,
        limit: limit,
        total: result.total_count,
        total_pages: result.total_pages,
        has_next: page < result.total_pages,
        has_prev: page > 1
      }
    }
  )

Response:
{
  "status": "success",
  "code": 200,
  "message": "Users retrieved",
  "data": [
    { "id": 1, "name": "John", "email": "john@gmail.com" },
    { "id": 2, "name": "Jane", "email": "jane@gmail.com" },
    ...
  ],
  "metadata": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 50,
      "total_pages": 5,
      "has_next": true,
      "has_prev": false
    }
  },
  "request_id": "req_jkl012mno",
  "timestamp": "2025-02-09T13:30:48Z"
}
```

---

# 9️⃣ BEST PRACTICES

## Best Practice 1: Separation of Concerns

```
✓ GOOD: Clear separation

Controller:
├─ Handles HTTP
├─ Parses requests
├─ Calls service
└─ Formats response

Service:
├─ Business logic
├─ Calls repository
├─ Side effects
└─ Returns result

Repository:
├─ Database queries
├─ Data persistence
└─ CRUD operations

Benefits:
├─ Easy to understand
├─ Easy to test
├─ Easy to change one layer
└─ Reusable components
```

## Best Practice 2: Dependency Injection

```
✓ GOOD: Dependencies injected

class UserController:
  constructor(user_service, logger):
    this.user_service = user_service
    this.logger = logger

class UserService:
  constructor(user_repository, email_service):
    this.user_repository = user_repository
    this.email_service = email_service

Benefits:
├─ Testable (mock dependencies)
├─ Flexible (swap implementations)
├─ Loose coupling
└─ Easy to manage

NOT:

❌ BAD: Hard-coded dependencies

class UserController:
  get_user():
    service = UserService()  // Hard-coded
    user = service.get_user(id)
    return user

Problems:
├─ Hard to test
├─ Hard to swap implementations
├─ Tight coupling
└─ Hard to manage
```

## Best Practice 3: Input Validation First

```
✓ GOOD: Validate input immediately

handler(request):
  // Validate first
  if !is_valid_input(request):
    return error_response()
  
  // Only process valid input
  result = service.process(request)
  return success_response(result)

Benefits:
├─ Fail fast
├─ Prevent invalid processing
├─ Save resources
└─ Better errors
```

## Best Practice 4: Error Handling in Service Layer

```
✓ GOOD: Let services throw meaningful errors

service.create_user(data):
  if email_exists(data.email):
    throw DuplicateError("Email already registered")
  
  if !is_valid_password(data.password):
    throw ValidationError("Password too weak")
  
  return save_user(data)

Controller:
create_user(request):
  try {
    user = service.create_user(request.body)
    return created_response(user)
  } catch DuplicateError as e:
    return error_response("DUPLICATE", e.message)
  } catch ValidationError as e:
    return error_response("VALIDATION", e.message)
  } catch error:
    return error_response("SERVER", "Failed to create user")

Benefits:
├─ Clear error types
├─ Easy to handle
├─ Consistent responses
└─ Service doesn't know about HTTP
```

## Best Practice 5: Consistent Response Format

```
✓ GOOD: Always use same format

all_handlers_return():
  {
    status: "success" or "error",
    code: HTTP status code,
    message: user message,
    data: response data,
    request_id: request id,
    timestamp: timestamp
  }

Benefits:
├─ Client can parse consistently
├─ Easy to document
├─ Professional API
├─ Easy to debug
└─ Predictable
```

---

# 🔟 REAL-WORLD EXAMPLES

## Complete Example 1: User Registration Flow

```
REQUEST: POST /api/auth/register
Body: {
  name: "John Doe",
  email: "john@gmail.com",
  password: "SecurePass123"
}

STEP 1: Router directs to AuthController.register_handler()

STEP 2: Controller (Handler)
register_handler(request):
  try {
    // Extract
    data = request.body
    
    // Validate format
    errors = {}
    if !data.name: errors.name = "Name required"
    if !data.email: errors.email = "Email required"
    if !data.password: errors.password = "Password required"
    
    if errors not empty:
      return error_response("VALIDATION_ERROR", "Invalid input", errors)
    
    // Call service
    user = auth_service.register_user({
      name: data.name,
      email: data.email,
      password: data.password
    })
    
    // Success
    return created_response(
      {
        id: user.id,
        name: user.name,
        email: user.email
      },
      "User registered successfully"
    )
  
  } catch DuplicateError as e:
    return error_response("CONFLICT", "Email already registered")
  
  } catch ValidationError as e:
    return error_response("VALIDATION_ERROR", e.message)
  
  } catch error:
    return error_response("SERVER_ERROR", "Failed to register")

STEP 3: Service
register_user(data):
  try {
    // Validate business rules
    if !is_valid_email(data.email):
      throw ValidationError("Invalid email format")
    
    if data.password.length < 8:
      throw ValidationError("Password minimum 8 characters")
    
    // Check unique
    existing = user_repository.find_by_email(data.email)
    if existing:
      throw DuplicateError("Email already registered")
    
    // Transform
    user_to_save = {
      name: data.name.trim(),
      email: data.email.lower(),
      password: password_hasher.hash(data.password),
      created_at: now(),
      is_active: true
    }
    
    // Save
    saved_user = user_repository.save(user_to_save)
    
    // Side effects
    email_service.send_welcome_email(saved_user.email)
    logger.info("User registered: " + saved_user.id)
    
    return saved_user
  
  } catch error:
    logger.error("Registration failed: " + error.message)
    throw error

STEP 4: Repository
save(user_data):
  return database.insert("users", user_data)

STEP 5: Response
{
  "status": "success",
  "code": 201,
  "message": "User registered successfully",
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "john@gmail.com"
  },
  "request_id": "req_abc123xyz",
  "timestamp": "2025-02-09T13:30:45Z"
}
```

## Complete Example 2: Update User with Error Handling

```
REQUEST: PUT /api/users/1
Headers: { Authorization: "Bearer token123" }
Body: {
  email: "newemail@gmail.com",
  name: "John Updated"
}

STEP 1: Router directs to UserController.update_user_handler()

STEP 2: Controller (Handler)
update_user_handler(request):
  try {
    user_id = request.params.id
    
    // Validate ID
    if !user_id:
      return error_response("VALIDATION_ERROR", "ID required")
    
    // Check auth
    if !request.context.user:
      return error_response("UNAUTHORIZED", "Login required", null, 401)
    
    // Check permission
    if request.context.user.id != user_id:
      return error_response("FORBIDDEN", "Cannot edit other users", null, 403)
    
    // Validate data
    data = request.body
    errors = {}
    
    if data.email && !is_valid_email(data.email):
      errors.email = "Invalid email format"
    
    if data.name && !data.name.trim():
      errors.name = "Name cannot be empty"
    
    if errors not empty:
      return error_response("VALIDATION_ERROR", "Invalid input", errors)
    
    // Call service
    updated_user = user_service.update_user(user_id, data)
    
    if !updated_user:
      return error_response("NOT_FOUND", "User not found", null, 404)
    
    // Success
    return success_response(updated_user, "User updated successfully")
  
  } catch DuplicateError as e:
    return error_response("CONFLICT", "Email already in use")
  
  } catch error:
    logger.error("Update failed: " + error.message)
    return error_response("SERVER_ERROR", "Failed to update user")

STEP 3: Service
update_user(user_id, update_data):
  try {
    // Get existing user
    user = user_repository.get(user_id)
    if !user:
      throw NotFoundError("User not found")
    
    // Check unique email
    if update_data.email && update_data.email != user.email:
      existing = user_repository.find_by_email(update_data.email)
      if existing:
        throw DuplicateError("Email already in use")
    
    // Prepare updates
    updates = {}
    
    if update_data.email:
      updates.email = update_data.email.lower()
    
    if update_data.name:
      updates.name = update_data.name.trim()
    
    updates.updated_at = now()
    
    // Save
    updated_user = user_repository.update(user_id, updates)
    
    // Side effects
    if update_data.email:
      email_service.send_change_confirmation(update_data.email)
    
    logger.info("User updated: " + user_id)
    
    return updated_user
  
  } catch error:
    logger.error("Update service failed: " + error.message)
    throw error

STEP 4: Response
{
  "status": "success",
  "code": 200,
  "message": "User updated successfully",
  "data": {
    "id": 1,
    "name": "John Updated",
    "email": "newemail@gmail.com",
    "updated_at": "2025-02-09T13:35:20Z"
  },
  "request_id": "req_def456uvw",
  "timestamp": "2025-02-09T13:35:20Z"
}
```

---

# 1️⃣1️⃣ QUICK REFERENCE

## MVC Architecture Diagram

```
REQUEST
    ↓
CONTROLLER (Handle HTTP)
    ├─ Parse request
    ├─ Validate input
    ├─ Call service
    └─ Format response
    ↓
SERVICE (Business Logic)
    ├─ Execute operations
    ├─ Call repository
    ├─ Side effects
    └─ Return result
    ↓
REPOSITORY (Data Access)
    ├─ Query database
    └─ Persist data
    ↓
RESPONSE → CLIENT
```

## Components at a Glance

```
CONTROLLER
├─ What: HTTP request handler
├─ Knows: Request, Response, HTTP
├─ Calls: Service
├─ Returns: HTTP Response
└─ Not responsible for: Business logic

SERVICE
├─ What: Business logic executor
├─ Knows: Business rules, operations
├─ Calls: Repository, External Services
├─ Returns: Domain objects
└─ Not responsible for: HTTP, Database

REPOSITORY
├─ What: Data access layer
├─ Knows: Database, Queries
├─ Calls: Database driver
├─ Returns: Raw data
└─ Not responsible for: Business logic, HTTP
```

## Handler Template

```
handler_name(request):
  try {
    // 1. Extract
    input = extract_data(request)
    
    // 2. Validate
    if !is_valid(input):
      return error_response(400, validation_errors)
    
    // 3. Authorize
    if !can_perform(request.user, operation):
      return error_response(403, "Not permitted")
    
    // 4. Process
    result = service.perform_operation(input)
    
    // 5. Format response
    return success_response(result, "Success message")
  
  } catch SpecificError as e:
    return error_response(appropriate_code, e.message)
  
  } catch error:
    return error_response(500, "Server error")
```

## Error Types & Status Codes

```
Validation Error → 400 Bad Request
Unauthorized → 401 Unauthorized
Forbidden → 403 Forbidden
Not Found → 404 Not Found
Duplicate → 409 Conflict
Unprocessable → 422 Unprocessable Entity
Server Error → 500 Internal Server Error
```

## Response Format

```
Success:
{
  "status": "success",
  "code": 200,
  "message": "...",
  "data": {...},
  "request_id": "...",
  "timestamp": "..."
}

Error:
{
  "status": "error",
  "code": 400,
  "message": "...",
  "error": {
    "type": "...",
    "details": {...}
  },
  "request_id": "...",
  "timestamp": "..."
}
```

## Best Practices Checklist

- [ ] Clear separation: Controller, Service, Repository
- [ ] Dependency injection: No hard-coded dependencies
- [ ] Input validation: Validate immediately
- [ ] Error handling: Meaningful error types
- [ ] Consistent responses: Same format always
- [ ] Single responsibility: Each component has one job
- [ ] No business logic in Controller
- [ ] No HTTP knowledge in Service
- [ ] Proper error codes: Use appropriate status codes
- [ ] Centralized error handling: One place for all errors

---

## 🎯 SUMMARY: COMPLETE MVC

### What
- **MVC**: Model-View-Controller architecture pattern
- **Controller**: Handles HTTP requests
- **Service**: Executes business logic
- **Repository**: Accesses data

### Why
- **Separation**: Each component has single responsibility
- **Testability**: Easy to test each layer
- **Reusability**: Components reused across controllers
- **Maintainability**: Clear structure, easy to change

### How
1. Request arrives → Controller receives
2. Controller validates → Calls Service
3. Service executes logic → Calls Repository
4. Repository queries database → Returns data
5. Service processes → Returns to Controller
6. Controller formats → Returns response

### Key Points
- Layers don't skip: Controller → Service → Repository
- Dependencies downward: Higher layers depend on lower
- No circular dependencies
- Each layer independent and testable
- Consistent error handling
- Consistent response format

**Master MVC, and your APIs will be clean, maintainable, and professional!** 🚀

# Authentication & Authorization - Complete Revision Guide 📚

## Table of Contents
1. [Introduction & Core Concepts](#introduction--core-concepts)
2. [Historical Evolution](#historical-evolution)
3. [Sessions](#sessions)
4. [JWT (JSON Web Tokens)](#jwt-json-web-tokens)
5. [Cookies](#cookies)
6. [Types of Authentication](#types-of-authentication)
7. [OAuth 2.0 & OpenID Connect](#oauth-20--openid-connect)
8. [Authorization & RBAC](#authorization--rbac)
9. [Security Best Practices](#security-best-practices)
10. [Flowcharts & Diagrams](#flowcharts--diagrams)

---

## Introduction & Core Concepts

### Two Fundamental Questions

**AUTHENTICATION**: "Who are you?"
- Process of verifying identity
- Assigns an identity to a subject
- Examples: Login screens, sign-up forms

**AUTHORIZATION**: "What can you do?"
- Process of determining permissions
- Defines capabilities and access levels
- Examples: Admin vs User roles, read/write permissions

### Simple Summary

```
AUTHENTICATION = Identity Verification (Who you are)
AUTHORIZATION = Permission Management (What you can do)
```

---

## Historical Evolution

### 1. Pre-Industrial Era: Trust-Based Authentication

**How it worked:**
- Village elders vouched for people
- Handshakes symbolized trust
- Identity = Personal recognition
- Based on human contextual trust

**Problem:**
- ❌ Didn't scale beyond familiar circles
- ❌ Required personal acquaintance
- ❌ Failed as populations grew

### 2. Medieval Period: Seals & Cryptography

**Wax Seals - First Authentication Tokens**

```
┌─────────────────────────────────────┐
│     WAX SEAL (Physical Token)       │
├─────────────────────────────────────┤
│ ✓ Unique pattern                    │
│ ✓ Attached to documents             │
│ ✓ Acted as signature                │
│ ✓ Principle: "Something you have"   │
└─────────────────────────────────────┘
```

**Vulnerabilities:**
- Prone to forgery
- First "bypass attacks" (forged seals)
- Led to watermarks and encrypted codes

### 3. Industrial Revolution: Passphrases

**Telegraph Era (Shared Secrets)**

- Pre-agreed passphrases between operators
- Static passwords (not dynamic)
- **Principle evolved**: "Something you have" → "Something you know"

**Advantage:**
- More secure than physical tokens
- Could be shared verbally/written

### 4. Modern Computation Era

#### Mainframes (1960s)

**MIT's CTSS (1961) - First Password System**

```
Problem: Multi-user systems needed isolation
Solution: Password for each user
CRITICAL MISTAKE: Stored passwords in PLAIN TEXT

Incident: Someone printed the password file
Result: Birth of secure password storage
```

**Innovations that followed:**
- ✅ **Hashing algorithms** (irreversible transformation)
- ✅ **Fixed-length representations**
- ✅ Cryptographic security principles

#### Cryptographic Revolution (1970s)

**Diffie-Hellman Key Exchange**
- Introduced **asymmetric cryptography**
- Two parties establish shared secret over untrusted medium
- Foundation for modern authentication

**Kerberos Protocol**
- Ticket-based authentication
- Trusted third-party issuing tickets
- Precursor to token-based systems

#### Internet Era (1990s)

**Problems:**
- Brute force attacks
- Dictionary attacks
- Username/password not enough

**Solution: Multi-Factor Authentication (MFA)**

```
MFA = Something you know + Something you have + Something you are

Examples:
├─ Something you know: Password, PIN
├─ Something you have: Smart card, OTP generator
└─ Something you are: Fingerprint, retina scan (biometrics)
```

**Challenges:**
- False positives/negatives in biometrics
- Template security issues

### 5. Modern Era (21st Century)

**Drivers:**
- Cloud computing
- Mobile devices
- API-based architectures

**Key Technologies:**
- ✅ OAuth 2.0
- ✅ JWT (JSON Web Tokens)
- ✅ Zero Trust Architecture
- ✅ Passwordless authentication (WebAuthn)

### 6. Future Candidates

```
┌──────────────────────────────────────────────┐
│         FUTURE OF AUTHENTICATION             │
├──────────────────────────────────────────────┤
│                                              │
│ 1. Decentralized Identity (Blockchain)       │
│    - Self-sovereign identity                 │
│    - No central authority                    │
│                                              │
│ 2. Behavioral Biometrics                     │
│    - Typing patterns                         │
│    - Mouse movements                         │
│    - Gait analysis                           │
│                                              │
│ 3. Post-Quantum Cryptography                 │
│    - Algorithms secure against quantum       │
│      computers                               │
│    - RSA will break with quantum computing   │
│    - New cryptographic techniques needed     │
│                                              │
└──────────────────────────────────────────────┘
```

---

## Sessions

### The Problem: HTTP is Stateless

**What does "Stateless" mean?**
- HTTP treats every request as isolated
- No memory of past exchanges
- Each request must contain ALL information needed

**Why was this a problem?**

```
Early Web: Static pages → Stateless was FINE
           Just reading content, no interaction

Modern Web: Dynamic content → Stateless is BOTTLENECK
            ├─ E-commerce: Remember cart items
            ├─ User login: Stay logged in across pages
            └─ Personalization: Remember preferences
```

### The Solution: Sessions

**Session = Temporary server-side context for each user**

### How Sessions Work

```
┌─────────────────────────────────────────────────────────────┐
│                    SESSION WORKFLOW                         │
└─────────────────────────────────────────────────────────────┘

PHASE 1: SESSION CREATION
┌─────────────────────┐
│   User Logs In      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ Server creates unique SESSION ID        │
│ e.g., "sess_a1b2c3d4e5f6"              │
└──────────┬──────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ Store in Persistent Storage:            │
│                                         │
│ Session ID: sess_a1b2c3d4e5f6           │
│ Data: {                                 │
│   userId: 123,                          │
│   role: "user",                         │
│   cartItems: [...],                     │
│   isAuthenticated: true                 │
│ }                                       │
│                                         │
│ Storage: Redis / Database               │
└──────────┬──────────────────────────────┘
           │
           ▼

PHASE 2: SESSION ID TRANSMISSION
┌─────────────────────────────────────────┐
│ Session ID sent to client as COOKIE     │
│                                         │
│ Set-Cookie: sessionId=sess_a1b2c3d4e5f6 │
└──────────┬──────────────────────────────┘
           │
           ▼

PHASE 3: SUBSEQUENT REQUESTS
┌─────────────────────────────────────────┐
│ Client sends cookie with every request  │
│                                         │
│ Cookie: sessionId=sess_a1b2c3d4e5f6     │
└──────────┬──────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│ Server receives session ID              │
│    ↓                                    │
│ Fetches user data from Redis/DB         │
│    ↓                                    │
│ Server has context about the user       │
└─────────────────────────────────────────┘

PHASE 4: EXPIRATION
┌─────────────────────────────────────────┐
│ Sessions have expiry (e.g., 15 minutes) │
│    ↓                                    │
│ After expiry, create new session        │
│    ↓                                    │
│ User may need to re-authenticate        │
└─────────────────────────────────────────┘
```

### Session Storage Evolution

```
1. FILE-BASED SESSIONS (Early days)
   ├─ Stored in files on server
   ├─ Simple but NOT scalable
   └─ Problem: As users grew, performance dropped

2. DATABASE-BACKED SESSIONS
   ├─ Stored in databases (PostgreSQL, MySQL)
   ├─ Faster lookups
   ├─ Persistent across server restarts
   └─ Better for large user bases

3. DISTRIBUTED IN-MEMORY STORAGE (Modern)
   ├─ Redis / Memcached
   ├─ Stored in RAM (not hard disk)
   ├─ VERY fast access times
   └─ Scales across multiple servers
```

### Session Storage Comparison

| Storage Type | Speed | Persistence | Scalability | Use Case |
|--------------|-------|-------------|-------------|----------|
| File-based | Slow | Yes | Poor | Small apps |
| Database | Medium | Yes | Good | Medium apps |
| Redis/Memcached | Very Fast | Configurable | Excellent | Large-scale apps |

---

## JWT (JSON Web Tokens)

### The Problem Sessions Created

By mid-2000s, web applications became globally distributed:

```
CHALLENGES WITH SESSIONS:

1. MEMORY OVERHEAD
   ├─ Millions/billions of users
   ├─ Storing session data = costly
   └─ Server memory consumed

2. REPLICATION ISSUES
   ├─ Server in USA
   ├─ Server in Europe
   ├─ Server in Asia
   └─ Synchronizing session data = LATENCY + CONSISTENCY problems

3. SCALABILITY BOTTLENECK
   └─ Stateful systems don't scale well
```

### The Solution: JWT (Stateless Tokens)

**JWT formalized in 2015**

**Key Innovation: Self-contained tokens**
- User data stored IN the token itself
- Cryptographically signed
- No server-side storage needed

### JWT Structure

```
┌────────────────────────────────────────────────────────────┐
│                    JWT TOKEN                               │
│                                                            │
│  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.                    │
│  eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIn0.     │
│  SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c              │
│                                                            │
│       HEADER    .    PAYLOAD    .    SIGNATURE             │
└────────────────────────────────────────────────────────────┘
```

### Three Parts of JWT

#### 1. HEADER (Metadata)

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Contains:**
- Signing algorithm (e.g., HS256, RS256)
- Token type (JWT)

#### 2. PAYLOAD (User Data)

```json
{
  "sub": "USER-456",        // Subject (User ID)
  "iat": 1675872600,        // Issued At timestamp
  "name": "John Doe",       // Optional: User name
  "role": "admin",          // Optional: User role
  "email": "john@example.com"
}
```

**Standard fields:**
- `sub`: User ID (from database or auth provider)
- `iat`: When JWT was issued
- `exp`: Expiration time
- Custom fields: name, role, permissions, etc.

#### 3. SIGNATURE (Verification)

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

**Purpose:**
- Verify token hasn't been tampered
- Only server with secret key can verify
- If data changes, signature validation fails

### How JWT Works

```
┌──────────────────────────────────────────────────────────┐
│                  JWT AUTHENTICATION FLOW                 │
└──────────────────────────────────────────────────────────┘

1. USER LOGS IN
   ├─ Send username/password
   └─ Server verifies credentials

2. SERVER CREATES JWT
   ├─ Include user data in payload
   ├─ Sign with secret key
   └─ Return JWT to client

3. CLIENT STORES JWT
   ├─ Store in cookie (HTTP-only)
   ├─ Or in memory (not localStorage - security risk)
   └─ Send with every request

4. SUBSEQUENT REQUESTS
   ┌────────────────────────────────┐
   │ Client sends JWT in header:    │
   │ Authorization: Bearer <token>  │
   └───────────┬────────────────────┘
               ▼
   ┌────────────────────────────────┐
   │ Server receives JWT            │
   │    ↓                           │
   │ Verify signature with secret   │
   │    ↓                           │
   │ Extract user data from payload │
   │    ↓                           │
   │ Authenticate user              │
   └────────────────────────────────┘
```

### JWT Advantages

```
✅ STATELESS
   - No server-side storage
   - Reduces infrastructure costs

✅ SCALABLE
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │ Server 1 │  │ Server 2 │  │ Server 3 │
   └─────┬────┘  └─────┬────┘  └─────┬────┘
         │             │             │
         └─────────────┴─────────────┘
              All share secret key
         Any server can verify JWT
         Perfect for microservices!

✅ PORTABLE
   - URL-friendly (Base64 encoded)
   - Can be stored in cookies
   - Can be passed between systems
   - Lightweight

✅ CROSS-DOMAIN
   - Works across different domains
   - Microservices can share authentication
```

### JWT Disadvantages

```
❌ TOKEN THEFT
   - If someone steals JWT, they can impersonate user
   - No way to invalidate until expiry
   - Stateless = no server-side revocation

❌ REVOCATION PROBLEM
   - Can't revoke a single JWT without affecting all users
   - Only solution: Change secret key (all users logout)

❌ SIZE
   - Larger than session IDs
   - Sent with every request
```

### Hybrid Approach (Best Practice)

**Combines stateless + stateful**

```
┌─────────────────────────────────────────────────┐
│         HYBRID JWT AUTHENTICATION               │
└─────────────────────────────────────────────────┘

1. User logs in → Get JWT (normal flow)

2. On every request:
   ├─ Verify JWT signature ✓
   ├─ Extract user data ✓
   └─ Check JWT blacklist in Redis
       │
       ├─ If blacklisted → REJECT (revoked)
       └─ If not blacklisted → ALLOW

3. To revoke a user's access:
   └─ Add their JWT to blacklist in Redis
```

**Blacklist Storage:**
```
Redis Blacklist:
├─ jwt_abc123xyz → expires in 1 hour
├─ jwt_def456uvw → expires in 30 minutes
└─ jwt_ghi789rst → expires in 2 hours
```

**Question: Why not just use sessions?**

**Answer:**
- JWT still provides scalability benefits
- Blacklist is only for revoked tokens (small subset)
- Normal flow is still stateless and fast
- Blacklist lookup is much faster than full session storage

---

## Cookies

### What are Cookies?

**Cookie = Small piece of data stored in user's browser by the server**

```
┌─────────────────────────────────────────┐
│         COOKIE MECHANISM                │
├─────────────────────────────────────────┤
│                                         │
│ Server → Stores data in client browser  │
│ Client → Sends data back automatically  │
│                                         │
│ Key Feature:                            │
│ Cookie set by ServerA only visible      │
│ to ServerA (security feature)           │
└─────────────────────────────────────────┘
```

### How Cookies Work in Authentication

```
STEP 1: SERVER SETS COOKIE
┌──────────────────────────────────┐
│ User logs in successfully        │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ Server sends response:           │
│                                  │
│ Set-Cookie: authToken=sess_123;  │
│            HttpOnly;              │
│            Secure;                │
│            SameSite=Strict        │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ Browser stores cookie            │
└──────────────────────────────────┘


STEP 2: BROWSER SENDS COOKIE AUTOMATICALLY
┌──────────────────────────────────┐
│ User makes another request       │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ Browser automatically includes:  │
│                                  │
│ Cookie: authToken=sess_123       │
│                                  │
│ (No JavaScript code needed!)     │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ Server receives cookie           │
│ Validates token                  │
│ Identifies user                  │
└──────────────────────────────────┘
```

### Cookie Attributes (Security)

```
Set-Cookie: sessionId=abc123; 
            HttpOnly; 
            Secure; 
            SameSite=Strict; 
            Max-Age=3600
            
┌────────────────────────────────────────────┐
│ HttpOnly                                   │
│ ├─ JavaScript CANNOT access this cookie    │
│ └─ Prevents XSS attacks                    │
├────────────────────────────────────────────┤
│ Secure                                     │
│ ├─ Only sent over HTTPS                    │
│ └─ Not sent over HTTP (unencrypted)        │
├────────────────────────────────────────────┤
│ SameSite=Strict                            │
│ ├─ Only sent to same domain                │
│ └─ Prevents CSRF attacks                   │
├────────────────────────────────────────────┤
│ Max-Age=3600                               │
│ └─ Cookie expires after 1 hour             │
└────────────────────────────────────────────┘
```

### Why Cookies for Authentication?

✅ **Automatic**: Browser sends with every request
✅ **Secure**: HttpOnly prevents JavaScript access
✅ **Convenient**: No manual token management needed
✅ **Standard**: Works across all browsers

---

## Types of Authentication

### 1. Stateful Authentication

**Uses**: Sessions stored on server

```
┌─────────────────────────────────────────────────┐
│         STATEFUL AUTHENTICATION                 │
└─────────────────────────────────────────────────┘

CLIENT                                   SERVER
  │                                         │
  │  1. POST /login                         │
  │     username: john                      │
  │     password: secret123                 │
  │────────────────────────────────────────>│
  │                                         │
  │                              2. Verify credentials
  │                                 3. Generate Session ID
  │                                    sess_xyz789
  │                                 4. Store in Redis:
  │                                    {
  │                                      userId: 123,
  │                                      role: "user",
  │                                      cart: [...]
  │                                    }
  │                                         │
  │  5. Set-Cookie: sessionId=sess_xyz789   │
  │<────────────────────────────────────────│
  │                                         │
  │  6. GET /api/profile                    │
  │     Cookie: sessionId=sess_xyz789       │
  │────────────────────────────────────────>│
  │                                         │
  │                              7. Get sess_xyz789
  │                                 8. Lookup in Redis
  │                                 9. Find user data
  │                                10. Authorize
  │                                         │
  │  11. Response with user data            │
  │<────────────────────────────────────────│
  │                                         │
```

**Pros:**
- ✅ Centralized control
- ✅ Real-time session information
- ✅ Easy revocation (delete from store)
- ✅ More secure (server controls state)

**Cons:**
- ❌ Memory overhead (store all sessions)
- ❌ Scalability issues with distributed systems
- ❌ Latency in multi-region setups

**Best for:**
- Web applications
- Admin dashboards
- Systems requiring strict session control

---

### 2. Stateless Authentication

**Uses**: JWT tokens

```
┌─────────────────────────────────────────────────┐
│        STATELESS AUTHENTICATION                 │
└─────────────────────────────────────────────────┘

CLIENT                                   SERVER
  │                                         │
  │  1. POST /login                         │
  │     username: john                      │
  │     password: secret123                 │
  │────────────────────────────────────────>│
  │                                         │
  │                              2. Verify credentials
  │                                 3. Create JWT:
  │                                    {
  │                                      sub: "123",
  │                                      role: "user",
  │                                      iat: 1234567890
  │                                    }
  │                                 4. Sign with secret
  │                                         │
  │  5. Return JWT token                    │
  │     eyJhbGc...                           │
  │<────────────────────────────────────────│
  │                                         │
  │  6. GET /api/profile                    │
  │     Authorization: Bearer eyJhbGc...    │
  │────────────────────────────────────────>│
  │                                         │
  │                              7. Verify JWT signature
  │                                 8. Extract user data
  │                                 9. Authorize
  │                                         │
  │  10. Response with user data            │
  │<────────────────────────────────────────│
  │                                         │
```

**Pros:**
- ✅ No server-side storage
- ✅ Highly scalable
- ✅ Perfect for microservices
- ✅ Mobile-friendly

**Cons:**
- ❌ Token revocation is complex
- ❌ Security risk if stolen
- ❌ Larger payload than session ID

**Best for:**
- APIs
- Microservices
- Mobile applications
- Distributed systems

---

### 3. API Key Authentication

**Purpose**: Machine-to-machine communication

```
┌─────────────────────────────────────────────────┐
│           API KEY AUTHENTICATION                │
└─────────────────────────────────────────────────┘

USE CASE EXAMPLE: Using ChatGPT API

1. USER INTERFACE
   ┌─────────────────────┐
   │  ChatGPT Website    │  ← Humans interact here
   │  (chat.openai.com)  │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  ChatGPT Servers    │
   └─────────────────────┘


2. PROGRAMMATIC ACCESS (API Key)
   ┌─────────────────────┐
   │  Your Server/App    │
   └──────────┬──────────┘
              │ Wants ChatGPT capabilities
              │ without using the UI
              │
              │ Uses API Key
              │ sk-abc123xyz456...
              │
              ▼
   ┌─────────────────────┐
   │  ChatGPT API Server │
   └─────────────────────┘
```

### How API Keys Work

```
STEP 1: GENERATE API KEY
User goes to platform → Click "Generate API Key"
                     ↓
           Platform returns: sk-abc123xyz456789

STEP 2: USE API KEY
┌──────────────────────────────────────┐
│ Your Server Code:                    │
│                                      │
│ fetch('https://api.openai.com/v1/chat', {
│   headers: {                         │
│     'Authorization': 'Bearer sk-abc123xyz456789'
│   }                                  │
│ })                                   │
└──────────────────────────────────────┘

STEP 3: SERVER VALIDATES
API Server:
├─ Receives API key
├─ Checks validity
├─ Checks permissions
├─ Checks quota/limits
└─ Authorizes request
```

### API Key Characteristics

```
✅ EASY TO GENERATE
   - One click in UI
   - Get cryptographically random string

✅ IDEAL FOR MACHINE-TO-MACHINE
   ┌──────────┐         ┌──────────┐
   │ Server A │────────>│ Server B │
   └──────────┘         └──────────┘
   No UI, no human interaction
   Just programmatic access

✅ PERMISSIONS & QUOTAS
   - Each key has specific permissions
   - Rate limits enforced
   - Can be scoped (read-only, write, etc.)

✅ EASY REVOCATION
   - Delete key from dashboard
   - Immediate effect
```

### Human vs Machine Interaction

```
HUMAN INTERACTION (Client-to-Server)
┌─────────┐     ┌──────┐     ┌──────────┐
│ Browser │────>│  UI  │────>│  Server  │
└─────────┘     └──────┘     └──────────┘
    ↑               │
    └── Visual ─────┘
        Login form, clicks, etc.


MACHINE INTERACTION (Server-to-Server)
┌──────────┐    API Key     ┌──────────┐
│ Server A │───────────────>│ Server B │
└──────────┘                └──────────┘
  
  No UI, no human
  Just code talking to code
```

**When to use:**
- Server-to-server communication
- Third-party integrations
- Programmatic access to services
- IoT devices

---

## OAuth 2.0 & OpenID Connect

### The Delegation Problem

**Problem**: One website needs access to another website's resources

```
SCENARIOS:

1. Travel App needs Gmail access
   └─ To scan flight tickets from email

2. Social Media App needs Google Contacts
   └─ To import contacts

3. Photo App needs Dropbox access
   └─ To save photos

PATTERN: App A wants Resource from App B
```

### Initial (BAD) Solution: Password Sharing

```
❌ DISASTROUS APPROACH

User shares Google password with Travel App
                    ↓
          HUGE SECURITY RISKS:
          ├─ App has FULL access to everything
          ├─ No way to limit permissions
          ├─ Can't revoke without changing password everywhere
          └─ Password exposed to third party
```

### OAuth 1.0 (2007) - Token Sharing

**Revolutionary Idea: Share TOKENS, not PASSWORDS**

```
PASSWORD vs TOKEN:

PASSWORD
├─ Full access to everything
├─ All or nothing
└─ Can't revoke without changing everywhere

TOKEN
├─ Specific permissions (read contacts only)
├─ Limited scope
├─ Can be revoked anytime
└─ Expiration time
```

### OAuth Components

```
┌────────────────────────────────────────────────┐
│         OAUTH 2.0 COMPONENTS                   │
├────────────────────────────────────────────────┤
│                                                │
│ 1. RESOURCE OWNER (User)                       │
│    └─ You (person who owns the data)           │
│                                                │
│ 2. CLIENT (Requesting App)                     │
│    └─ Facebook (wants your Google contacts)    │
│                                                │
│ 3. RESOURCE SERVER                             │
│    └─ Google (stores your contacts)            │
│                                                │
│ 4. AUTHORIZATION SERVER                        │
│    └─ Google OAuth (issues tokens)             │
│                                                │
└────────────────────────────────────────────────┘
```

### OAuth 2.0 Flow

```
┌──────────────────────────────────────────────────────────┐
│              OAUTH 2.0 AUTHORIZATION FLOW                │
└──────────────────────────────────────────────────────────┘

USER (Resource Owner)
  │
  │ 1. User clicks "Import Google Contacts"
  │    on Facebook
  │
  ▼
┌─────────────────────────────────┐
│ FACEBOOK (Client)               │
└────────────┬────────────────────┘
             │
             │ 2. Redirects to Google OAuth
             │
             ▼
┌─────────────────────────────────┐
│ GOOGLE OAUTH SERVER             │
│ (Authorization Server)          │
├─────────────────────────────────┤
│ Login Screen:                   │
│ "Facebook wants to access       │
│  your Google Contacts"          │
│                                 │
│ Permissions:                    │
│ ☑ Read contacts                 │
│ ☐ Delete contacts               │
│                                 │
│ [Allow] [Deny]                  │
└────────────┬────────────────────┘
             │
             │ 3. User clicks "Allow"
             │
             ▼
┌─────────────────────────────────┐
│ GOOGLE OAUTH SERVER             │
│ Issues TOKEN                    │
│                                 │
│ Token: oauth_abc123xyz          │
│ Permissions: read_contacts      │
│ Expires: 1 hour                 │
└────────────┬────────────────────┘
             │
             │ 4. Sends token to Facebook
             │
             ▼
┌─────────────────────────────────┐
│ FACEBOOK (Client)               │
│ Stores token: oauth_abc123xyz   │
└────────────┬────────────────────┘
             │
             │ 5. Uses token to access resources
             │
             ▼
┌─────────────────────────────────┐
│ GOOGLE RESOURCE SERVER          │
│ (Contacts API)                  │
├─────────────────────────────────┤
│ Request:                        │
│ GET /contacts                   │
│ Authorization: oauth_abc123xyz  │
│                                 │
│ Response:                       │
│ [Contact1, Contact2, ...]       │
└─────────────────────────────────┘
```

### OAuth 2.0 Evolution

**OAuth 1.0 → OAuth 2.0 (2010)**

```
OAUTH 1.0 LIMITATIONS:
├─ Very complex for developers
└─ Cryptographic signatures (error-prone)

OAUTH 2.0 IMPROVEMENTS:
├─ Bearer tokens (simpler)
└─ Multiple flows for different use cases
```

### OAuth 2.0 Flows

```
┌─────────────────────────────────────────────────┐
│           OAUTH 2.0 GRANT TYPES                 │
├─────────────────────────────────────────────────┤
│                                                 │
│ 1. AUTHORIZATION CODE FLOW                      │
│    └─ For server-side apps                      │
│       Most secure                               │
│                                                 │
│ 2. IMPLICIT FLOW                                │
│    └─ For browser-based apps                    │
│       (NOW DISCOURAGED - security risks)        │
│                                                 │
│ 3. CLIENT CREDENTIALS FLOW                      │
│    └─ For machine-to-machine                    │
│       No user involved                          │
│                                                 │
│ 4. DEVICE CODE FLOW                             │
│    └─ For devices with limited input            │
│       Example: Smart TV authentication          │
│                                                 │
└─────────────────────────────────────────────────┘
```

### OpenID Connect (2014)

**Problem OAuth 2.0 Solved:**
- ✅ Authorization (what you can do)

**Problem OAuth 2.0 DIDN'T Solve:**
- ❌ Authentication (who you are)

**OpenID Connect = OAuth 2.0 + Identity Layer**

### What OpenID Connect Added

```
┌────────────────────────────────────────────┐
│     OPENID CONNECT = OAuth 2.0 + ID        │
├────────────────────────────────────────────┤
│                                            │
│ Added: ID TOKEN (JWT)                      │
│                                            │
│ ID Token contains:                         │
│ ├─ User ID                                 │
│ ├─ Name                                    │
│ ├─ Email                                   │
│ ├─ Profile picture                         │
│ ├─ Issued at (iat)                         │
│ └─ Issuing authority (iss)                 │
│                                            │
└────────────────────────────────────────────┘
```

### "Sign in with Google" Flow

```
┌──────────────────────────────────────────────────────────┐
│         OPENID CONNECT AUTHENTICATION                    │
└──────────────────────────────────────────────────────────┘

STEP 1: USER CLICKS "SIGN IN WITH GOOGLE"
┌─────────────────────────┐
│ Note-taking App         │
│                         │
│ [Sign in with Google]   │
└──────────┬──────────────┘
           │
           │ Redirects to Google
           ▼
┌─────────────────────────┐
│ Google OAuth Server     │
│                         │
│ Login + Grant permission│
└──────────┬──────────────┘
           │
           │ Returns Authorization Code
           ▼
┌─────────────────────────┐
│ Note-taking App         │
└──────────┬──────────────┘
           │
           │ STEP 2: Exchange code for tokens
           ▼
┌─────────────────────────────────────────┐
│ Google OAuth Server                     │
│                                         │
│ Returns:                                │
│ ├─ ACCESS TOKEN (for API access)        │
│ └─ ID TOKEN (JWT with user info)        │
│                                         │
│ ID Token (JWT):                         │
│ {                                       │
│   "sub": "google-user-123",             │
│   "name": "John Doe",                   │
│   "email": "john@gmail.com",            │
│   "picture": "https://..."              │
│ }                                       │
└─────────────┬───────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│ Note-taking App                         │
│                                         │
│ Stores user info from ID Token          │
│ Creates account or logs in user         │
│ No password needed!                     │
└─────────────────────────────────────────┘
```

### OAuth 2.0 + OpenID Connect Together

```
OAUTH 2.0 (Authorization)
"What can the app do with your data?"
├─ Access Google Drive files
├─ Read Gmail
└─ Post to Facebook

OPENID CONNECT (Authentication)
"Who are you?"
├─ Your identity (email, name)
├─ Profile information
└─ Used for "Sign in with..."
```

---

## Authorization & RBAC

### The Authorization Problem

**Scenario: Note-taking Platform**

```
REQUIREMENTS:

1. Regular users:
   ├─ Create notes
   ├─ Edit own notes
   └─ Delete own notes (to trash)

2. Admin/Creator needs:
   ├─ All user permissions +
   ├─ Access "Dead Zone" (permanently deleted notes)
   ├─ View all users' notes
   └─ Manage platform settings

HOW TO IMPLEMENT THIS?
```

### Bad Solution: God Mode String

```
❌ SECURITY NIGHTMARE

Admin sends special string with API:
GET /api/dead-zone?secret=admin_magic_key_123

PROBLEMS:
├─ If string leaked → disaster
├─ Hard to manage multiple admins
├─ Can't easily revoke access
└─ Not scalable
```

### RBAC (Role-Based Access Control)

**Best Practice for Authorization**

```
┌────────────────────────────────────────────┐
│    RBAC = Roles + Permissions              │
├────────────────────────────────────────────┤
│                                            │
│ ROLES:                                     │
│ ├─ User                                    │
│ ├─ Admin                                   │
│ ├─ Moderator                               │
│ └─ Editor                                  │
│                                            │
│ PERMISSIONS:                               │
│ ├─ Read                                    │
│ ├─ Write                                   │
│ ├─ Delete                                  │
│ └─ Access_Dead_Zone                        │
│                                            │
│ MAPPING:                                   │
│ User role → [Read, Write, Delete] on notes │
│ Admin role → [All permissions] + Dead_Zone │
│ Moderator → [Read, Write] on all notes     │
│                                            │
└────────────────────────────────────────────┘
```

### RBAC Workflow

```
┌──────────────────────────────────────────────────────────┐
│              RBAC AUTHORIZATION FLOW                     │
└──────────────────────────────────────────────────────────┘

1. USER REGISTERS
   ↓
   Server assigns role: "user"
   Stored in database:
   {
     userId: 123,
     role: "user"
   }

2. USER MAKES REQUEST
   ┌────────────────────────────────┐
   │ GET /api/dead-zone             │
   │ Cookie: sessionId=xyz          │
   └────────────┬───────────────────┘
                │
                ▼
   ┌────────────────────────────────┐
   │ SERVER MIDDLEWARE CHAIN        │
   ├────────────────────────────────┤
   │                                │
   │ Step 1: Authentication         │
   │ ├─ Get sessionId from cookie   │
   │ ├─ Lookup in Redis/DB          │
   │ └─ Identify user (userId: 123) │
   │                                │
   │ Step 2: Get User Role          │
   │ ├─ Lookup userId in DB         │
   │ └─ role = "user"               │
   │                                │
   │ Step 3: Authorization Check    │
   │ ├─ Resource: /api/dead-zone    │
   │ ├─ Required role: "admin"      │
   │ ├─ User role: "user"           │
   │ └─ user ≠ admin → REJECT       │
   │                                │
   └────────────┬───────────────────┘
                │
                ▼
   ┌────────────────────────────────┐
   │ Response: 403 Forbidden        │
   │ "You don't have permission"    │
   └────────────────────────────────┘
```

### Permission Matrix Example

| Role | Create Notes | Edit Own | Delete Own | Edit Others | Access Dead Zone | Manage Users |
|------|--------------|----------|------------|-------------|------------------|--------------|
| **User** | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Moderator** | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Admin** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

### Granular Permissions

```
RESOURCE-LEVEL PERMISSIONS:

Notes:
├─ User: Read, Write, Delete (own notes only)
├─ Moderator: Read, Write (all notes)
└─ Admin: Full access

Dead Zone:
├─ User: No access
├─ Moderator: No access
└─ Admin: Full access

User Management:
├─ User: No access
├─ Moderator: Read only
└─ Admin: Full access
```

### Multi-Tenant Example

```
ORGANIZATION-BASED RBAC:

Organization A:
├─ Owner: Full control
├─ Admin: Manage members, edit all docs
├─ Editor: Edit all docs
└─ Viewer: Read-only access

Organization B:
├─ Owner: Full control
├─ Admin: Manage members, edit all docs
└─ Member: Edit own docs only
```

---

## Security Best Practices

### 1. Error Messages (CRITICAL)

**Problem: Helpful errors leak information**

```
❌ BAD - Information Leakage

User tries to login:
├─ Email: john@example.com
└─ Password: wrong123

Server responses:
├─ "User not found" → Attacker knows email doesn't exist
├─ "Incorrect password" → Attacker knows email is CORRECT
└─ "Account locked" → Attacker knows account exists but is locked

ATTACKER CAN:
├─ Enumerate valid usernames
├─ Focus attack on password only
└─ Build list of valid accounts
```

```
✅ GOOD - Generic Messages

For ALL authentication failures:
└─ "Authentication failed. Please check your credentials."

ALWAYS use generic messages for:
├─ Invalid username
├─ Invalid password
├─ Account locked
├─ Account suspended
└─ Any auth-related error
```

### 2. Timing Attacks

**Problem: Response time reveals information**

```
AUTHENTICATION WORKFLOW:

Step 1: Find user in database
        ├─ User not found → FAST response (100ms)
        └─ User found → Continue

Step 2: Check if account locked
        └─ Not locked → Continue

Step 3: Hash password and compare
        └─ Password hashing takes time (200ms)

TIMING ATTACK:
├─ Invalid username → 100ms response
├─ Valid username + wrong password → 300ms response
└─ Attacker measures timing difference!
```

**How attackers exploit:**

```
Attacker tries 1000 usernames:
├─ 950 respond in ~100ms → Invalid usernames
└─ 50 respond in ~300ms → VALID usernames!

Now attacker knows:
├─ 50 valid usernames
└─ Focus brute-force on these only
```

**Defense Mechanisms:**

```
✅ SOLUTION 1: Constant-Time Operations

Use cryptographic constant-time comparison:
├─ Password hash comparison takes same time
├─ Whether match or not → same duration
└─ Prevents timing analysis

✅ SOLUTION 2: Artificial Delay

Even if username not found:
├─ Simulate password hashing delay
├─ Add fixed delay (200ms)
└─ Response time is consistent

Example:
if (!userFound) {
  await simulateDelay(200); // Fake the hashing time
  return "Authentication failed";
}
```

### 3. Password Security

```
┌────────────────────────────────────────────┐
│       PASSWORD STORAGE EVOLUTION           │
├────────────────────────────────────────────┤
│                                            │
│ ❌ NEVER: Plain text                       │
│    password: "mySecret123"                 │
│                                            │
│ ❌ BAD: Simple hashing                     │
│    password: md5("mySecret123")            │
│    Problem: Rainbow tables                 │
│                                            │
│ ✅ GOOD: Hashing + Salt                    │
│    salt: random_string                     │
│    password: bcrypt("mySecret123" + salt)  │
│                                            │
│ ✅ BEST: Modern algorithms                 │
│    - bcrypt (industry standard)            │
│    - Argon2 (newer, more secure)           │
│    - scrypt                                │
│                                            │
└────────────────────────────────────────────┘
```

### 4. Token Security

```
JWT BEST PRACTICES:

✅ DO:
├─ Use HTTPS always
├─ Store in HttpOnly cookies
├─ Short expiration times (15-60 min)
├─ Use refresh tokens for longevity
└─ Implement token blacklist for revocation

❌ DON'T:
├─ Store in localStorage (XSS vulnerable)
├─ Include sensitive data in payload
├─ Use weak secret keys
├─ Share tokens across domains
└─ Have very long expiration (hours/days)
```

### 5. Rate Limiting

```
PREVENT BRUTE FORCE ATTACKS:

After 5 failed login attempts:
├─ Lock account for 15 minutes
├─ Or require CAPTCHA
└─ Or send email notification

Implementation:
┌────────────────────────────────┐
│ Redis Store:                   │
│                                │
│ login_attempts:user@email.com  │
│ Count: 5                       │
│ Locked until: 2026-02-08 15:30 │
└────────────────────────────────┘
```

### 6. Use Auth Providers

```
RECOMMENDATION:

Instead of building your own:
├─ Use Auth0
├─ Use Clerk
├─ Use Firebase Auth
├─ Use AWS Cognito
└─ Use Supabase Auth

WHY?
├─ Security is their core business
├─ Constantly updated against new threats
├─ Compliance handled (GDPR, SOC2)
├─ Battle-tested at scale
└─ Less liability for you

WHEN TO BUILD YOUR OWN:
├─ Learning purposes ✅
├─ Very specific custom requirements
└─ High security needs + expert team
```

---

## Flowcharts & Diagrams

### Complete Authentication Flow

```
┌──────────────────────────────────────────────────────────────┐
│           COMPLETE AUTHENTICATION FLOW                       │
└──────────────────────────────────────────────────────────────┘

                    START
                      │
                      ▼
            ┌─────────────────┐
            │ User Registration│
            └────────┬─────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │ Choose Auth Method:   │
         │ ├─ Email/Password     │
         │ ├─ OAuth (Google)     │
         │ └─ Passwordless       │
         └───────┬───────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
    ▼                         ▼
┌─────────┐            ┌──────────────┐
│Password │            │OAuth/Social  │
│  Auth   │            │    Login     │
└────┬────┘            └──────┬───────┘
     │                        │
     │ Credentials            │ Redirect to provider
     ▼                        ▼
┌─────────────┐        ┌──────────────┐
│Verify       │        │User grants   │
│credentials  │        │permission    │
└────┬────────┘        └──────┬───────┘
     │                        │
     │ Valid?                 │ Get tokens
     ▼                        ▼
   ┌─────┐              ┌──────────┐
   │ Yes │              │Get user  │
   └──┬──┘              │info      │
      │                 └────┬─────┘
      │                      │
      └──────────┬───────────┘
                 │
                 ▼
        ┌────────────────┐
        │Choose Session  │
        │Management:     │
        │├─ Stateful     │
        │└─ Stateless    │
        └────┬───────────┘
             │
    ┌────────┴────────┐
    │                 │
    ▼                 ▼
┌────────┐      ┌──────────┐
│Session │      │   JWT    │
│+ Cookie│      │  Token   │
└────┬───┘      └─────┬────┘
     │                │
     │                │
     └────────┬───────┘
              │
              ▼
    ┌──────────────────┐
    │Return to client  │
    │with auth token   │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │Subsequent        │
    │requests include  │
    │token             │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │Server validates  │
    │on each request   │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │AUTHORIZATION     │
    │Check permissions │
    │based on role     │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │Execute request   │
    │or return 403     │
    └──────────────────┘
```

### OAuth 2.0 Detailed Flow

```
┌──────────────────────────────────────────────────────────────┐
│         OAUTH 2.0 AUTHORIZATION CODE FLOW                    │
└──────────────────────────────────────────────────────────────┘

USER                CLIENT APP              AUTH SERVER         RESOURCE SERVER
  │                     │                         │                    │
  │  1. Access app      │                         │                    │
  │────────────────────>│                         │                    │
  │                     │                         │                    │
  │                     │  2. Redirect to auth    │                    │
  │                     │────────────────────────>│                    │
  │                     │    + client_id          │                    │
  │                     │    + redirect_uri       │                    │
  │                     │    + scope              │                    │
  │                     │                         │                    │
  │  3. Login + Grant permission                  │                    │
  │<──────────────────────────────────────────────│                    │
  │                     │                         │                    │
  │  4. Approve         │                         │                    │
  │────────────────────────────────────────────────>                    │
  │                     │                         │                    │
  │  5. Redirect with Authorization Code          │                    │
  │<────────────────────┤                         │                    │
  │                     │                         │                    │
  │                     │  6. Exchange code       │                    │
  │                     │    for tokens           │                    │
  │                     │────────────────────────>│                    │
  │                     │    + code               │                    │
  │                     │    + client_secret      │                    │
  │                     │                         │                    │
  │                     │  7. Return tokens       │                    │
  │                     │<────────────────────────│                    │
  │                     │    - access_token       │                    │
  │                     │    - refresh_token      │                    │
  │                     │    - (id_token)         │                    │
  │                     │                         │                    │
  │                     │  8. API request with access_token            │
  │                     │─────────────────────────────────────────────>│
  │                     │                         │                    │
  │                     │  9. Return protected resources               │
  │                     │<─────────────────────────────────────────────│
  │                     │                         │                    │
  │  10. Display data   │                         │                    │
  │<────────────────────│                         │                    │
  │                     │                         │                    │
```

### RBAC Decision Tree

```
                    User Makes Request
                           │
                           ▼
                ┌──────────────────┐
                │ Authenticate     │
                │ Who are you?     │
                └────────┬─────────┘
                         │
                    ┌────┴────┐
                    │         │
                Invalid    Valid
                    │         │
                    ▼         ▼
              ┌─────────┐  ┌──────────────┐
              │ 401     │  │ Get User Role│
              │ Unauthorized  └──────┬───────┘
              └─────────┘         │
                                  ▼
                        ┌──────────────────┐
                        │ Check Permission │
                        │ What can you do? │
                        └────────┬─────────┘
                                 │
                    ┌────────────┼────────────┐
                    │            │            │
                  User       Moderator      Admin
                    │            │            │
                    ▼            ▼            ▼
         ┌──────────────┐ ┌──────────┐ ┌──────────┐
         │Access own    │ │Access all│ │Full      │
         │notes only    │ │notes     │ │access    │
         └──────┬───────┘ └────┬─────┘ └────┬─────┘
                │              │            │
                └──────────────┼────────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                Allowed              Not Allowed
                    │                     │
                    ▼                     ▼
              ┌──────────┐          ┌─────────┐
              │ 200 OK   │          │ 403     │
              │ + Data   │          │Forbidden│
              └──────────┘          └─────────┘
```

### Session vs JWT Comparison

```
┌──────────────────────────────────────────────────────────────┐
│              SESSION vs JWT COMPARISON                       │
└──────────────────────────────────────────────────────────────┘

SESSION-BASED (Stateful)
─────────────────────────
CLIENT                          SERVER
  │                               │
  │ Login                         │
  │──────────────────────────────>│
  │                               │ Create session
  │                               │ Store in Redis:
  │                               │ sess_123 → {userId, role}
  │                               │
  │ sessionId=sess_123            │
  │<──────────────────────────────│
  │ (in cookie)                   │
  │                               │
  │ Request + Cookie              │
  │──────────────────────────────>│
  │                               │ 1. Get session ID
  │                               │ 2. Lookup Redis
  │                               │ 3. Get user data
  │                               │ 4. Validate
  │                               │
  │ Response                      │
  │<──────────────────────────────│
  │                               │

Storage: Redis/Database
Revocation: Delete from store ✅
Scalability: Medium ⚠️
Security: High ✅


JWT-BASED (Stateless)
──────────────────────
CLIENT                          SERVER
  │                               │
  │ Login                         │
  │──────────────────────────────>│
  │                               │ Create JWT
  │                               │ Sign with secret
  │                               │ No storage needed
  │                               │
  │ JWT token                     │
  │<──────────────────────────────│
  │ eyJhbGc...                    │
  │                               │
  │ Request + JWT                 │
  │──────────────────────────────>│
  │ Authorization: Bearer token   │
  │                               │ 1. Verify signature
  │                               │ 2. Extract payload
  │                               │ 3. Validate
  │                               │ (NO database lookup)
  │                               │
  │ Response                      │
  │<──────────────────────────────│
  │                               │

Storage: None ✅
Revocation: Complex (need blacklist) ⚠️
Scalability: Excellent ✅
Security: Medium ⚠️
```

### Authentication Decision Tree

```
                Start: Need to authenticate users
                            │
                            ▼
                   ┌────────────────┐
                   │ What type of   │
                   │ application?   │
                   └────────┬───────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
   ┌─────────┐       ┌──────────┐       ┌──────────┐
   │ Web App │       │ Mobile   │       │ API only │
   │(Browser)│       │   App    │       │(M2M)     │
   └────┬────┘       └────┬─────┘       └────┬─────┘
        │                 │                   │
        │                 │                   │
        ▼                 ▼                   ▼
   ┌─────────┐       ┌─────────┐       ┌──────────┐
   │Stateful │       │Stateless│       │ API Key  │
   │(Session)│       │  (JWT)  │       │   or     │
   │   or    │       │         │       │  OAuth   │
   │Stateless│       │         │       │          │
   └────┬────┘       └────┬────┘       └────┬─────┘
        │                 │                  │
        ▼                 ▼                  ▼
   Need easy       Need offline       Service-to-
   revocation?     capability?        service?
        │                 │                  │
      Yes│               Yes│               Yes│
        ▼                 ▼                  ▼
   Use Session     Use JWT with       Use API Keys
   + Redis         Refresh Token      or OAuth Client
                                      Credentials
```

---

## Quick Reference

### When to Use What

```
┌─────────────────────────────────────────────────────────┐
│              AUTHENTICATION METHOD SELECTOR             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ STATEFUL (Sessions)                                     │
│ ✅ Use when:                                            │
│    - Web applications                                   │
│    - Need easy revocation                               │
│    - Security is top priority                           │
│    - Moderate traffic                                   │
│                                                         │
│ STATELESS (JWT)                                         │
│ ✅ Use when:                                            │
│    - APIs                                               │
│    - Mobile apps                                        │
│    - Microservices                                      │
│    - High scalability needed                            │
│                                                         │
│ API KEYS                                                │
│ ✅ Use when:                                            │
│    - Server-to-server                                   │
│    - Third-party integrations                           │
│    - IoT devices                                        │
│    - Programmatic access                                │
│                                                         │
│ OAUTH 2.0 / OpenID Connect                              │
│ ✅ Use when:                                            │
│    - "Sign in with..." feature                          │
│    - Third-party data access                            │
│    - Delegation needed                                  │
│    - Identity federation                                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Security Checklist

```
AUTHENTICATION SECURITY CHECKLIST:

□ Use HTTPS for all authentication endpoints
□ Implement rate limiting (5 attempts → lockout)
□ Hash passwords with bcrypt/Argon2
□ Use salt with password hashing
□ Return generic error messages
□ Implement constant-time comparisons
□ Add artificial delays to prevent timing attacks
□ Use HttpOnly cookies for tokens
□ Implement CSRF protection
□ Set short JWT expiration times
□ Use refresh tokens for longevity
□ Implement token blacklist for revocation
□ Monitor for suspicious activity
□ Enable MFA for sensitive operations
□ Regularly rotate secrets/keys
□ Use auth provider for production (recommended)
```

### Common Attacks & Defenses

| Attack Type | Description | Defense |
|-------------|-------------|---------|
| **Brute Force** | Try many passwords | Rate limiting, CAPTCHA, account lockout |
| **Credential Stuffing** | Use leaked passwords | MFA, password breach detection |
| **Timing Attack** | Measure response time | Constant-time operations, artificial delays |
| **Session Hijacking** | Steal session tokens | HttpOnly cookies, HTTPS, short expiration |
| **CSRF** | Trick user into unwanted action | CSRF tokens, SameSite cookies |
| **XSS** | Inject malicious scripts | Content Security Policy, sanitize input |
| **Man-in-the-Middle** | Intercept communication | HTTPS/TLS, certificate pinning |

---

## Summary

### Core Principles

```
1. AUTHENTICATION = WHO YOU ARE
   - Verifies identity
   - Login process
   - Credentials validation

2. AUTHORIZATION = WHAT YOU CAN DO
   - Checks permissions
   - Role-based access
   - Resource protection

3. ALWAYS USE BOTH
   - First authenticate (who?)
   - Then authorize (what?)
   - Both are essential
```

### Evolution Timeline

```
Pre-Industrial → Trust & handshakes
Medieval → Wax seals (physical tokens)
Industrial → Passphrases (shared secrets)
1960s → Passwords in computers
1970s → Cryptography & hashing
1990s → MFA & biometrics
2007 → OAuth 1.0
2010 → OAuth 2.0
2014 → OpenID Connect
2015 → JWT formalized
Today → Passwordless, Zero Trust
Future → Blockchain identity, Post-quantum crypto
```

### Best Practices Recap

```
✅ DO:
├─ Use auth providers (Auth0, Clerk, etc.)
├─ Implement MFA
├─ Use HTTPS everywhere
├─ Hash + salt passwords
├─ Set short token expiration
├─ Return generic error messages
├─ Implement rate limiting
├─ Use RBAC for authorization
└─ Regular security audits

❌ DON'T:
├─ Store passwords in plain text
├─ Use MD5 or SHA1 for passwords
├─ Store JWT in localStorage
├─ Return specific error messages
├─ Ignore timing attacks
├─ Skip HTTPS
├─ Hard-code secrets
└─ Forget about authorization after authentication
```

---

**End of Authentication & Authorization Guide**

*Remember: Security is not a feature, it's a requirement!*

*Last Updated: February 2026*

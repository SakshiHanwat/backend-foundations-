# HTTP for Backend Engineers - Part 1: Basics

## Table of Contents
1. [Backend Systems Overview](#1-backend-systems-overview)
2. [HTTP Protocol Basics](#2-http-protocol-basics)
3. [TCP Connection](#3-tcp-connection)
4. [HTTPS and Security](#4-https-and-security)
5. [HTTP Message Structure](#5-http-message-structure)
6. [HTTP Headers](#6-http-headers)

---

## 1. Backend Systems Overview

### Definition
Backend systems are server-side applications that process client requests, manage data, and send responses.

### Theory
When you use a website or app, the backend handles all processing behind the scenes - database operations, business logic, authentication, and data management.

### Technical Terms
- **Client**: Application making requests (browser, mobile app)
- **Server**: System that processes requests and sends responses
- **Request**: Message from client asking for data/action
- **Response**: Message from server with data/confirmation
- **Firewall**: Security layer filtering requests
- **Proxy**: Intermediate server (like Nginx)
- **Backend Server**: Application server (Spring Boot, Node.js)
- **Database**: Where data is stored

### Real-World Example
**Ordering food on Swiggy:**
- You (Client) open app and order biryani
- App sends order to Swiggy servers
- Firewall checks if request is safe
- Nginx routes to correct service
- Spring Boot processes order, checks restaurant, calculates price
- Database stores order
- Response: "Order confirmed! 30 mins"

### Complete Flow
```
USER types URL: www.facebook.com
    ↓
DNS LOOKUP
"What is IP of facebook.com?"
DNS: "157.240.22.35"
    ↓
TCP CONNECTION (3-way handshake)
SYN → SYN-ACK → ACK
✓ Connection established
    ↓
TLS HANDSHAKE (if HTTPS)
Exchange certificates
Setup encryption
✓ Secure connection
    ↓
HTTP REQUEST
"GET /home HTTP/1.1"
    ↓
FIREWALL
"Is this safe? ✓ Yes"
    ↓
PROXY (Nginx)
"Route to correct server"
    ↓
BACKEND SERVER (Spring Boot)
- Check authentication
- Query database
- Apply business logic
- Generate response
    ↓
HTTP RESPONSE
"200 OK + Data"
    ↓
BROWSER
Render and display page
```

---

## 2. HTTP Protocol Basics

### Definition
HTTP (HyperText Transfer Protocol) is a set of rules defining how clients and servers communicate.

### Theory
HTTP is the foundation of the web - a standard protocol that browsers and servers follow to exchange information. Without HTTP, different systems couldn't talk to each other.

### Two Core Principles

#### Principle 1: Statelessness

**Definition**: HTTP has no memory - each request is independent.

**Theory**
The server doesn't remember previous requests. Every request must include all necessary information (authentication, data, etc.). After responding, the server forgets everything.

**Technical Terms**
- **Stateless**: No memory between requests
- **Self-contained**: Each request has complete information
- **Independent**: Requests don't depend on previous ones

**Why Important?**
Since server has no memory, we use:
- **Cookies**: Small data stored in browser
- **Sessions**: Server-side storage with session ID
- **Tokens**: JWT (JSON Web Tokens) for authentication

**Example**
```
Request 1:
Browser: "Show me profile page"
Server: "Who are you? Don't know you!"
Browser: "Here's my token: abc123"
Server: "Verified! Here's your profile"
[Server forgets after response]

Request 2 (1 minute later):
Browser: "Show my messages"
Server: "Who are you?" (no memory!)
Browser: "Token: abc123" (must send again)
Server: "Verified! Here's messages"
```

**Real-World Analogy**
Like talking to someone with complete amnesia:
- You: "Hi, I'm Rahul, I want to buy a shirt"
- Shopkeeper: "OK, here's catalog"
- You: "Hi, I'm Rahul, I want this blue shirt" (reintroduce!)
- Shopkeeper: "OK, here's price"

**Benefits**
1. **Simplicity**: Server doesn't store session data for millions of users
2. **Scalability**: Request can go to any server
3. **Reliability**: If server crashes, no data lost

---

#### Principle 2: Client-Server Model

**Definition**: Communication always started by client, server only responds.

**Theory**
Client initiates conversation, server waits and responds. Server never sends unsolicited messages (in standard HTTP).

**Roles**

**Client Responsibilities:**
- Initiate requests
- Provide URL
- Provide headers (metadata)
- Provide body data (if needed)
- Wait for response

**Server Responsibilities:**
- Wait for requests
- Process requests
- Access databases
- Apply business logic
- Generate and send responses

**Technical Terms**
- **Request-Response Cycle**: One complete round
- **Initiator**: Who starts (always client)
- **Responder**: Who replies (always server)

**Example**
```
Instagram App:

Client: "GET /api/feed - Give me posts"
Server: "Here are 20 posts"

Client: "POST /api/like/post123 - I liked this"
Server: "Like added successfully"

Server CANNOT say:
"Hey client, new notification!" (without client asking)

For real-time: Use WebSockets or polling
```

**Real-World Analogy**
Customer service call:
- You call customer care (initiate)
- Agent picks up and waits
- You: "What's my balance?"
- Agent: "Balance is ₹5000"
- You: "Transfer ₹1000"
- Agent: "Transfer done"

Agent never calls you randomly!

---

### HTTP vs HTTPS

**HTTP (Plain)**
- **Definition**: Original protocol without encryption
- **Port**: 80
- **Security**: None - plain text
- **Use**: Insecure, anyone can read data

**HTTPS (Secure)**
- **Definition**: HTTP with TLS/SSL encryption
- **Port**: 443
- **Security**: Encrypted
- **Use**: Secure, only sender/receiver can read

**Comparison Example**
```
Sending password via HTTP:
Browser → Internet → Server
Password: "mypass123" (visible to anyone!)

Hacker sees: "mypass123" ❌

---

Sending via HTTPS:
Browser → [ENCRYPTED] → Server
Encrypted: "8k3#mF9@xL2$"

Hacker sees: "8k3#mF9@xL2$" (meaningless!) ✓
```

**Real-World Analogy**
- **HTTP**: Postcard - anyone handling it can read
- **HTTPS**: Sealed envelope - only recipient can open

---

## 3. TCP Connection

### Definition
TCP (Transmission Control Protocol) provides reliable, ordered data delivery between applications.

### Theory
Before HTTP communication, TCP connection must be established. TCP ensures data arrives in correct order and retransmits lost packets.

Think:
- **TCP**: Road/highway (data travels on)
- **HTTP**: Car/vehicle (carries actual data)

### Technical Terms
- **TCP**: Transmission Control Protocol
- **Reliable**: Guarantees data delivery
- **Ordered**: Data arrives in sequence
- **Connection-oriented**: Connection before data transfer
- **Port**: Number identifying application (HTTP=80, HTTPS=443)
- **Socket**: IP address + port (192.168.1.1:80)

### 3-Way Handshake

**Definition**: Three-step process to establish TCP connection.

**Theory**
Before exchanging data, client and server must agree to communicate and synchronize. Like a formal greeting before conversation.

**Step 1: SYN (Synchronize)**

**What happens**: Client sends SYN packet.

**Technical Details**
```
Browser → Server
- SYN flag = 1 (connection request)
- Sequence number = 1000 (random starting number)
- Source port = 54321 (browser's port)
- Destination port = 443 (server's HTTPS port)
```

**Meaning**: "Hello Server! I want to connect. My sequence starts at 1000."

---

**Step 2: SYN-ACK (Synchronize-Acknowledge)**

**What happens**: Server responds with SYN-ACK.

**Technical Details**
```
Server → Browser
- SYN flag = 1 (I'm ready too)
- ACK flag = 1 (received your SYN)
- Sequence number = 5000 (server's starting number)
- Acknowledgment = 1001 (client's number + 1)
```

**Meaning**: "Hello! I received your request. I'm ready. My sequence starts at 5000."

---

**Step 3: ACK (Acknowledge)**

**What happens**: Client sends final ACK.

**Technical Details**
```
Browser → Server
- ACK flag = 1 (confirmed)
- Acknowledgment = 5001 (server's number + 1)
```

**Meaning**: "Perfect! Connection established. Let's transfer data!"

---

**Complete Visual Flow**
```
CLIENT                      SERVER
  |                            |
  | SYN (seq=1000)            |
  |-------------------------→ |
  | "I want to connect"       |
  |                           |
  | SYN-ACK (seq=5000,        |
  |         ack=1001)         |
  | ←-------------------------|
  | "I'm ready, acknowledged" |
  |                           |
  | ACK (ack=5001)            |
  |-------------------------→ |
  | "Perfect, let's begin"    |
  |                           |
  | ✓ CONNECTION ESTABLISHED  |
  |                           |
  | Data Transfer             |
  |==========================|
```

**Real-World Analogies**

**Phone Call:**
1. You dial (SYN) - "Ring ring..."
2. Friend picks up (SYN-ACK) - "Hello?"
3. You respond (ACK) - "Hi, it's Rahul"
4. Conversation starts (data transfer)

**Job Interview:**
1. You arrive (SYN) - "Hello, I'm here for interview"
2. Receptionist (SYN-ACK) - "Welcome! Please sit"
3. You sit (ACK) - "Thank you"
4. Interview begins (data transfer)

**Why 3-Way Handshake?**

1. **Confirm both sides ready**
   - Client ready to send
   - Server ready to receive
   - Both acknowledge each other

2. **Synchronize sequence numbers**
   - Both agree on starting numbers
   - Track packets and maintain order

3. **Detect lost packets**
   - If SYN lost, client resends
   - If SYN-ACK lost, server resends
   - Ensures reliable connection

**After Connection**
```
HTTP can now work:

Browser → Server:
GET /home HTTP/1.1
Host: facebook.com

Server → Browser:
HTTP/1.1 200 OK
Content-Type: text/html
[HTML data]
```

---

## 4. HTTPS and Security

### TLS/SSL Handshake

**Definition**: Process of establishing encrypted connection.

**Theory**
After TCP connection, if site uses HTTPS, TLS handshake occurs. This sets up encryption so all data is unreadable to anyone in the middle.

### Technical Terms
- **SSL**: Secure Sockets Layer (older, deprecated)
- **TLS**: Transport Layer Security (modern, secure)
- **Encryption**: Converting data to unreadable format
- **Decryption**: Converting back to readable
- **Certificate**: Digital document proving server identity
- **Public Key**: Used to encrypt (anyone can have)
- **Private Key**: Used to decrypt (only server has)
- **Certificate Authority (CA)**: Trusted organization issuing certificates

### TLS Handshake Steps

**Step 1: Client Hello**
```
Browser → Server:
"Hello! I want HTTPS.
I support TLS 1.3
I can use these encryptions: AES, RSA..."
```

**Step 2: Server Hello + Certificate**
```
Server → Browser:
"Hello! Let's use TLS 1.3
We'll use AES encryption

Here's my SSL Certificate:
- Domain: facebook.com
- Owner: Meta Platforms Inc.
- Valid until: Dec 31, 2026
- Issued by: DigiCert (CA)
- Public Key: [key data]"
```

**Step 3: Certificate Verification**
```
Browser checks:
✓ Certificate expired? No
✓ Issued by trusted CA? Yes (DigiCert)
✓ Domain matches? Yes (facebook.com)
✓ Certificate revoked? No

Result: Certificate VALID ✓
```

**Step 4: Key Exchange**
```
Browser:
1. Generates random secret key: "abc123xyz"
2. Encrypts key using server's public key
3. Sends encrypted key

Server:
1. Receives encrypted key
2. Decrypts using private key
3. Now both have same secret key!

✓ Symmetric encryption established
```

**Step 5: Encrypted Communication**
```
Browser → Server (encrypted):
Plain: "GET /profile"
Encrypted: "k3#8f@2dx9..."

Server → Browser (encrypted):
Plain: "Here's data"
Encrypted: "p9@xL2#mF3..."

No one in middle can read! 🔒
```

**Visual Flow**
```
CLIENT                          SERVER
  |                               |
  | Client Hello                  |
  |----------------------------→ |
  |                               |
  | ←-----------------------------|
  | Server Hello + Certificate    |
  | + Public Key                  |
  |                               |
  | Verify Certificate            |
  | ✓ Valid                       |
  |                               |
  | Generate Secret Key           |
  | Encrypt with Public Key       |
  |----------------------------→ |
  | [Encrypted Secret Key]        |
  |                               |
  |                        Decrypt|
  |                   with Private|
  |                            Key|
  |                               |
  | ✓ BOTH HAVE SECRET KEY       |
  |                               |
  |==============================|
  | All data encrypted with       |
  | symmetric key                 |
  |==============================|
```

---

### SSL Certificate

**Definition**: Digital document proving website identity.

**Theory**
Like Aadhaar card or passport proves your identity, SSL certificate proves website is who it claims to be.

**What Certificate Contains**
```
SSL Certificate for facebook.com:

Domain Name: facebook.com
Owner: Meta Platforms, Inc.
Address: 1 Hacker Way, Menlo Park, CA
Valid From: Jan 1, 2026
Valid Until: Dec 31, 2026
Public Key: [long string]
Certificate Authority: DigiCert Inc.
CA Signature: [digital signature]
```

**Real-World Analogy**

**Driving License:**
```
Name: Rahul Kumar
DOB: Jan 1, 2000
License #: DL-1234567890
Valid Until: Dec 31, 2030
Issued By: RTO Delhi (Trusted)
Photo + Signature
```
Proves you're Rahul and authorized to drive.

**SSL Certificate:**
```
Domain: facebook.com
Owner: Meta Inc.
Valid Until: Dec 31, 2026
Issued By: DigiCert (Trusted CA)
Public Key + Signature
```
Proves website is really facebook.com, not fake.

---

### Certificate Authority (CA)

**Definition**: Trusted organizations that verify and issue SSL certificates.

**Theory**
Like government issues Aadhaar after verifying identity, CAs verify websites and issue certificates.

**Popular CAs**
- DigiCert
- Let's Encrypt (Free!)
- GlobalSign
- Comodo
- GoDaddy

**Example Process**
```
Meta (Facebook) wants certificate:

Step 1: Apply to DigiCert
"We own facebook.com, need certificate"

Step 2: DigiCert verifies
- Really own facebook.com? ✓
- Legitimate company? ✓
- DNS records verified? ✓

Step 3: DigiCert issues
"Here's certificate, valid 1 year"

Step 4: Browsers trust it
Browser has list of trusted CAs (includes DigiCert)
"DigiCert signed this, I trust it" ✓
```

---

### Public Key and Private Key

**Definition**: Pair of cryptographic keys for encryption/decryption.

**Theory**
Public key = lock (anyone can use to lock)
Private key = only key that can unlock

**How It Works**

**Lock and Key Analogy**
```
Server's Public Key = Lock (everyone gets copy)
Server's Private Key = Only key that opens lock

Browser:
1. Gets server's public key (lock)
2. Puts message in box
3. Locks box with public key
4. Sends locked box

Server:
1. Receives locked box
2. Uses private key to unlock
3. Reads message

Anyone in middle:
- Has locked box
- Has public key (lock)
- But CANNOT open! (no private key)
```

**Real Example**
```
Sending password "mypass123":

Step 1: Get server's public key
Public Key: "PUB-KEY-XYZ123"

Step 2: Encrypt password
Original: "mypass123"
Encrypted: "8k3#mF9@xL2$..."

Step 3: Send encrypted data
Browser → Internet → Server
Traveling: "8k3#mF9@xL2$..."

Hacker intercepts: "8k3#mF9@xL2$..."
Hacker tries decrypt: ❌ No private key!

Step 4: Server decrypts
Private Key: "PRIV-KEY-ABC789"
Decrypted: "mypass123" ✓
```

**Why Use Both Asymmetric and Symmetric?**

```
Asymmetric Encryption:
✓ Very secure
✗ Very SLOW (complex math)

Symmetric Encryption:
✓ Very fast
✗ How to share secret key securely?

Solution - Use Both:

Phase 1 (Handshake): Asymmetric
- Use public/private keys
- Securely exchange symmetric key
- Slow but only once

Phase 2 (Data): Symmetric
- Use shared symmetric key
- Encrypt all data
- Fast and secure!
```

---

## 5. HTTP Message Structure

### HTTP Request Structure

**Definition**: Format in which client sends data to server.

**Theory**
Every HTTP request follows specific structure. This standard ensures any server can understand requests from any client.

**Complete Structure**
```
[REQUEST LINE]
[HEADERS]
[BLANK LINE]
[BODY - optional]
```

**Raw Example**
```
POST /api/users HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0)
Content-Type: application/json
Content-Length: 52
Authorization: Bearer abc123

{"name":"Rahul","email":"rahul@gmail.com","age":25}
```

**Part 1: Request Line**

**Format**: `METHOD PATH VERSION`

**Example**: `POST /api/users HTTP/1.1`

**Parts**:
- **POST**: HTTP method (what to do)
- **/api/users**: Path (where)
- **HTTP/1.1**: Protocol version

**Real-World Analogy**
```
Courier Form:

Service: EXPRESS DELIVERY (Method)
Destination: Rahul's House, A-123 (Path)
Version: BlueDart 2.0 (Version)
```

---

**Part 2: Headers**

**Definition**: Key-value pairs with additional info.

**Format**: `Key: Value`

**Example**
```
Host: api.example.com
User-Agent: Mozilla/5.0
Content-Type: application/json
Authorization: Bearer token123
```

**Real-World Analogy**
```
Package Info (written outside):

Sender: Amit (User-Agent)
Recipient: Rahul (Host)
Type: Fragile (Content-Type)
ID: Aadhaar 1234 (Authorization)
Weight: 2 kg (Content-Length)
```
Info outside helps without opening package!

---

**Part 3: Blank Line**

**Purpose**: Separates headers from body.

**Technical**: Line with just `\r\n`

---

**Part 4: Body (Optional)**

**When present**:
- POST (creating)
- PUT (updating all)
- PATCH (updating partial)

**When absent**:
- GET (just fetching)
- DELETE (just deleting)

**Examples**
```
JSON Body:
{"name": "Rahul", "email": "rahul@gmail.com"}

Form Data:
name=Rahul&email=rahul@gmail.com

File Upload:
[binary file data]
```

---

### HTTP Response Structure

**Complete Structure**
```
[STATUS LINE]
[HEADERS]
[BLANK LINE]
[BODY]
```

**Raw Example**
```
HTTP/1.1 200 OK
Date: Sun, 08 Feb 2026 10:30:00 GMT
Server: nginx/1.18.0
Content-Type: application/json
Content-Length: 85
Set-Cookie: sessionId=xyz789

{"id":123,"name":"Rahul","email":"rahul@gmail.com"}
```

**Part 1: Status Line**

**Format**: `VERSION CODE TEXT`

**Example**: `HTTP/1.1 200 OK`

**Parts**:
- **HTTP/1.1**: Version
- **200**: Status code
- **OK**: Status text

**Examples**
```
HTTP/1.1 200 OK (Success)
HTTP/1.1 404 Not Found (Not found)
HTTP/1.1 500 Internal Server Error (Server error)
```

---

**Part 2: Response Headers**

**Example**
```
Content-Type: application/json (format)
Content-Length: 85 (size)
Set-Cookie: session=abc (store cookie)
Cache-Control: max-age=3600 (cache 1 hour)
```

---

**Part 3: Blank Line**

Separator between headers and body.

---

**Part 4: Response Body**

**Examples**

**JSON**:
```json
{
  "id": 123,
  "name": "Rahul",
  "email": "rahul@gmail.com"
}
```

**HTML**:
```html
<!DOCTYPE html>
<html>
<body><h1>Hello!</h1></body>
</html>
```

**Image**: `[binary image data]`

---

### Complete Request-Response Example

**Scenario**: Login to Facebook

**REQUEST**
```
POST /api/login HTTP/1.1
Host: facebook.com
User-Agent: Chrome/100.0
Content-Type: application/json
Content-Length: 58

{"email":"rahul@gmail.com","password":"mypass123"}
```

**RESPONSE**
```
HTTP/1.1 200 OK
Date: Sun, 08 Feb 2026 10:30:00 GMT
Content-Type: application/json
Set-Cookie: sessionId=xyz789; HttpOnly; Secure
Content-Length: 120

{"success":true,"user":{"id":123,"name":"Rahul"}}
```

**What Happened**:
1. Browser sent POST with email/password
2. Server verified credentials
3. Server created session, sent cookie
4. Server responded with success + user data
5. Browser stored cookie for future requests

---

## 6. HTTP Headers

### What Are Headers?

**Definition**: Metadata (information about data) sent with HTTP messages.

**Theory**
Headers are like labels on a package - they provide important information without opening the package. Both client and server use headers to communicate extra details.

**Format**: `HeaderName: HeaderValue`

**Real-World Analogy**

**Courier Package**:
```
OUTSIDE (Headers):
- Sender: Amit
- Recipient: Rahul
- Address: Mumbai 400001
- Fragile: Yes
- Weight: 2 kg

INSIDE (Body):
- Actual gift
```

**Why Important?**
1. Don't need to open to know details
2. Intermediate systems can process without reading body
3. Standard way to communicate metadata

---

### Types of Headers

#### 1. REQUEST HEADERS

**Definition**: Headers sent by client to server.

**Purpose**: Tell server about client's capabilities and preferences.

---

**A) Host**

**Definition**: Domain name of server.

**Format**: `Host: domain.com`

**Why needed**: One server can host multiple websites.

**Example**
```
GET /home HTTP/1.1
Host: instagram.com

Server knows: "Request for Instagram"
```

**Scenario**
```
Server IP: 157.240.22.35

This IP hosts:
- facebook.com
- instagram.com
- whatsapp.com

Request with Host: instagram.com
Server: "Route to Instagram ✓"
```

---

**B) User-Agent**

**Definition**: Identifies client (browser, OS, device).

**Format**: `User-Agent: Application/Version (Platform)`

**Why needed**: Server customizes response for different devices.

**Example**
```
User-Agent: Mozilla/5.0 (Windows NT 10.0) Chrome/100

Breakdown:
- Windows 10
- 64-bit
- Chrome 100
```

**Use Case**
```
Mobile:
User-Agent: iPhone iOS 15

Server sends:
- Mobile HTML
- Smaller images
- Touch interface

Desktop:
User-Agent: Windows Chrome

Server sends:
- Full desktop HTML
- High-res images
- Mouse interface
```

---

**C) Accept**

**Definition**: Tells what content types client can understand.

**Format**: `Accept: content-type`

**Examples**
```
Accept: application/json (want JSON)
Accept: text/html (want HTML)
Accept: image/png (want PNG)
Accept: */* (accept anything)
```

**Content Negotiation**
```
REQUEST 1:
Accept: application/json

RESPONSE:
Content-Type: application/json
{"id": 123, "name": "Rahul"}

---

REQUEST 2:
Accept: application/xml

RESPONSE:
Content-Type: application/xml
<user><id>123</id></user>
```

---

**D) Accept-Language**

**Definition**: Preferred language for response.

**Format**: `Accept-Language: language-code`

**Examples**
```
Accept-Language: hi-IN (Hindi - India)
Accept-Language: en-US (English - US)
Accept-Language: hi-IN, en-US (Hindi preferred)
```

**Example**
```
REQUEST:
Accept-Language: hi-IN

RESPONSE:
{"message": "नमस्ते!"}

---

REQUEST:
Accept-Language: en-US

RESPONSE:
{"message": "Hello!"}
```

---

**E) Accept-Encoding**

**Definition**: Compression formats client supports.

**Format**: `Accept-Encoding: algorithms`

**Examples**
```
Accept-Encoding: gzip
Accept-Encoding: gzip, deflate, br
```

**Example**
```
REQUEST:
Accept-Encoding: gzip

RESPONSE:
Content-Encoding: gzip
Content-Length: 3800 (compressed)

Original: 26 MB
Compressed: 3.8 MB
Saved: 85%! ✓
```

---

**F) Authorization**

**Definition**: Credentials to authenticate.

**Format**: `Authorization: Type credentials`

**Types**

**Bearer Token (JWT)**:
```
Authorization: Bearer eyJhbGc...
Most common in modern APIs
```

**Basic Auth**:
```
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
Format: Base64(username:password)
```

**API Key**:
```
Authorization: ApiKey sk_live_51H8...
```

**Example**
```
Without Authorization:
GET /api/profile

RESPONSE:
401 Unauthorized
{"error": "Login required"}

---

With Authorization:
GET /api/profile
Authorization: Bearer token123

RESPONSE:
200 OK
{"name": "Rahul"}
```

---

**G) Cookie**

**Definition**: Previously stored cookies sent to server.

**Format**: `Cookie: name=value; name2=value2`

**Example**
```
Cookie: sessionId=abc123; userId=456
```

**Flow**
```
Login:
POST /api/login

Response:
Set-Cookie: sessionId=abc123

Browser stores ✓

Later:
GET /api/profile
Cookie: sessionId=abc123

Server: "Session valid ✓"
```

---

**H) Referer**

**Definition**: URL of previous page.

**Format**: `Referer: https://previous-page.com`

**Example**
```
User searches "instagram" on Google
Clicks result

Request to Instagram:
Referer: https://google.com/search?q=instagram

Instagram: "User came from Google search"
(Analytics tracking)
```

---

**I) Content-Type** (in requests)

**Definition**: Format of request body.

**Examples**

**JSON**:
```
Content-Type: application/json
{"name": "Rahul"}
```

**Form**:
```
Content-Type: application/x-www-form-urlencoded
name=Rahul&email=rahul@gmail.com
```

**File Upload**:
```
Content-Type: multipart/form-data
[binary file data]
```

---

**J) Content-Length**

**Definition**: Size of body in bytes.

**Example**
```
POST /api/users
Content-Type: application/json
Content-Length: 52

{"name":"Rahul","email":"rahul@gmail.com"}

Server: "Read 52 bytes"
```

---

#### 2. RESPONSE HEADERS

**Definition**: Headers sent by server to client.

---

**A) Content-Type** (in response)

**Definition**: Format of response body.

**Examples**
```
Content-Type: application/json (JSON)
Content-Type: text/html (HTML)
Content-Type: image/png (PNG)
Content-Type: application/pdf (PDF)
Content-Type: video/mp4 (Video)
```

---

**B) Set-Cookie**

**Definition**: Instruct browser to store cookies.

**Format**: `Set-Cookie: name=value; attributes`

**Attributes**:
- **Path**: Where valid (`Path=/`)
- **Max-Age**: Validity in seconds
- **HttpOnly**: Not accessible via JavaScript
- **Secure**: Only over HTTPS
- **SameSite**: CSRF protection

**Example**
```
Set-Cookie: sessionId=abc123; Path=/; HttpOnly; Secure; Max-Age=86400

Browser stores for 24 hours
```

**Security**
```
Without HttpOnly:
Set-Cookie: sessionId=abc123

JavaScript:
document.cookie (Can steal! ❌)

---

With HttpOnly:
Set-Cookie: sessionId=abc123; HttpOnly

JavaScript:
document.cookie (Cannot access ✓)
```

---

**C) Cache-Control**

**Definition**: Caching rules.

**Directives**

**max-age**:
```
Cache-Control: max-age=3600
Cache for 1 hour
```

**no-cache**:
```
Cache-Control: no-cache
Cache but validate first
```

**no-store**:
```
Cache-Control: no-store
Don't cache (sensitive data)
```

**public**:
```
Cache-Control: public, max-age=86400
Anyone can cache (CDN, proxy)
```

**private**:
```
Cache-Control: private, max-age=3600
Only browser caches
```

---

**D) ETag**

**Definition**: Unique identifier for resource version (hash).

**Format**: `ETag: "hash-value"`

**Flow**
```
First Request:
ETag: "abc123"
{"data": "Hello"}

Browser caches ✓

Later (cache expired):
If-None-Match: "abc123"

If unchanged:
304 Not Modified (use cache)

If changed:
200 OK
ETag: "xyz789"
{"data": "Updated"}
```

---

**E) Server**

**Definition**: Server software name.

**Examples**
```
Server: nginx/1.18.0
Server: Apache/2.4.41
Server: cloudflare
```

---

#### 3. SECURITY HEADERS

**Purpose**: Protect against attacks.

---

**A) Strict-Transport-Security (HSTS)**

**Definition**: Force HTTPS.

**Format**: `Strict-Transport-Security: max-age=seconds`

**Example**
```
Strict-Transport-Security: max-age=31536000

Browser remembers 1 year:
"Always use HTTPS"

User types: http://facebook.com
Browser auto-converts: https://facebook.com
```

---

**B) Content-Security-Policy (CSP)**

**Definition**: Control resource loading.

**Example**
```
Content-Security-Policy: default-src 'self'

Only load from own domain

Attacker injects:
<script src="https://evil.com/malware.js"></script>

Browser: "BLOCKED! Not allowed" ✓
```

---

**C) X-Frame-Options**

**Definition**: Control iframe loading.

**Values**:
```
X-Frame-Options: DENY (no iframes)
X-Frame-Options: SAMEORIGIN (same domain only)
```

**Prevents clickjacking** ✓

---

**D) X-Content-Type-Options**

**Definition**: Prevent MIME sniffing.

**Example**
```
X-Content-Type-Options: nosniff
Content-Type: image/jpeg

Browser: "Treat as image only, don't execute"
(Prevents XSS) ✓
```

---

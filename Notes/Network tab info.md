---
tags:
  - dev-tools
  - api
  - front-end
  - browser
---

- [[#What it is|What it is]]
- [[#Common Values|Common Values]]
- [[#Example|Example]]
- [[#Why it matters|Why it matters]]
- [[#What it is|What it is]]
- [[#Example|Example]]
- [[#Components|Components]]
- [[#Used for|Used for]]
- [[#What it is|What it is]]
- [[#Example|Example]]
- [[#Examples|Examples]]
- [[#Notes|Notes]]
- [[#What it is|What it is]]
- [[#Common Ports|Common Ports]]
- [[#What it is|What it is]]
- [[#Status Code Categories|Status Code Categories]]
- [[#Common Status Codes|Common Status Codes]]
- [[#What it is|What it is]]
- [[#Steps|Steps]]
- [[#DevTools timing example|DevTools timing example]]
- [[#What it is|What it is]]
- [[#Examples|Examples]]
- [[#Why it matters|Why it matters]]
- [[#What it is|What it is]]
- [[#Referrer|Referrer]]
- [[#Common Policies|Common Policies]]
- [[#What it is|What it is]]
- [[#Difference from Resource Size|Difference from Resource Size]]
- [[#What it is|What it is]]
- [[#Differences|Differences]]
- [[#Debugging|Debugging]]
- [[#Performance|Performance]]
- [[#Security|Security]]
- [[#Backend Development|Backend Development]]

# HTTP Request Inspection (DevTools Network Tab Reference)

This note explains all the metadata fields commonly visible when inspecting a request in the **browser DevTools → Network tab**.

These values help debug:

- API calls
    
- Performance
    
- Security
    
- Caching
    
- Networking issues
    

---

# 1. Request Overview Section

This is the **top section** in DevTools showing general request information.

Example:

```
Request URL: https://api.example.com/users?id=12
Request Method: GET
Status Code: 200 OK
Remote Address: 104.21.34.12:443
Referrer Policy: strict-origin-when-cross-origin
```

---

# Scheme

### What it is

The **protocol used to communicate with the server**.

### Common Values

|Scheme|Meaning|
|---|---|
|`http`|HyperText Transfer Protocol|
|`https`|Secure HTTP (encrypted using TLS/SSL)|
|`ws`|WebSocket|
|`wss`|Secure WebSocket|
|`ftp`|File Transfer Protocol|

### Example

```
https://example.com/api
^^^^^
scheme
```

### Why it matters

HTTPS ensures:

- encryption
    
- data integrity
    
- authentication
    

Without HTTPS:

- data can be intercepted (MITM attacks)
    

---

# Host

### What it is

The **domain name or server hosting the resource**.

### Example

```
https://api.example.com/users
        ^^^^^^^^^^^^^^^
            host
```

### Components

```
api.example.com
│
subdomain
        │
     domain
             │
          TLD
```

### Used for

- DNS resolution
    
- routing request to correct server
    
- multi-tenant hosting
    

---

# Filename / Path

### What it is

The **specific resource being requested** on the server.

### Example

```
https://example.com/api/users
                      ^^^^^
                      filename / path
```

### Examples

|Path|Meaning|
|---|---|
|`/index.html`|webpage|
|`/api/users`|API endpoint|
|`/images/logo.png`|image|
|`/styles.css`|stylesheet|

### Notes

Servers route paths to:

- files
    
- API handlers
    
- controllers
    

---

# Address / Remote Address

### What it is

The **IP address of the server** that responded.

Example

```
104.21.34.12:443
```

Structure:

```
IP:PORT
```

Example:

```
192.168.1.10:3000
```

Meaning:

|Part|Meaning|
|---|---|
|IP|server machine|
|Port|application/service|

### Common Ports

|Port|Service|
|---|---|
|80|HTTP|
|443|HTTPS|
|3000|Node dev server|
|5432|PostgreSQL|

---

# Status Code

### What it is

The **server response status** indicating the result of the request.

Example

```
200 OK
```

### Status Code Categories

|Range|Meaning|
|---|---|
|1xx|Informational|
|2xx|Success|
|3xx|Redirect|
|4xx|Client Error|
|5xx|Server Error|

### Common Status Codes

|Code|Meaning|
|---|---|
|200|Success|
|201|Created|
|204|No Content|
|301|Permanent Redirect|
|302|Temporary Redirect|
|304|Not Modified (cache)|
|400|Bad Request|
|401|Unauthorized|
|403|Forbidden|
|404|Not Found|
|500|Server Error|
|502|Bad Gateway|
|503|Service Unavailable|

---

# DNS Resolution

### What it is

The process of converting:

```
domain name → IP address
```

Example

```
example.com → 93.184.216.34
```

### Steps

1. Browser cache
    
2. OS cache
    
3. DNS resolver
    
4. Authoritative DNS server
    

### DevTools timing example

```
DNS Lookup: 20 ms
```

Meaning:

Time taken to resolve domain → IP.

---

# Request Priority

### What it is

The **priority level the browser assigns to the request**.

Browsers prioritize resources to optimize loading.

### Examples

|Priority|Resource Type|
|---|---|
|Highest|HTML|
|High|CSS|
|Medium|JS|
|Low|images|
|Lowest|analytics|

### Why it matters

Critical resources load first.

Example:

```
HTML → CSS → JS → Images
```

---

# Referrer Policy

### What it is

Controls **how much referrer information is sent** when navigating.

### Referrer

Header containing the **previous page URL**.

Example:

```
Referer: https://google.com
```

### Common Policies

|Policy|Behavior|
|---|---|
|`no-referrer`|send nothing|
|`origin`|only domain|
|`strict-origin`|only origin if secure|
|`same-origin`|send only same site|
|`strict-origin-when-cross-origin`|modern default|

Example

```
strict-origin-when-cross-origin
```

Meaning:

- full referrer for same site
    
- only origin for cross-site
    

---

# Transferred

### What it is

The **actual data transferred over network**.

Example:

```
Transferred: 4.2 KB
```

### Difference from Resource Size

|Metric|Meaning|
|---|---|
|Transferred|compressed size|
|Resource Size|original size|

Example:

```
Resource: 20 KB
Transferred: 4 KB
```

Meaning compression used.

---

# Version

### What it is

The **HTTP protocol version used**.

Example

```
HTTP/1.1
HTTP/2
HTTP/3
```

### Differences

|Version|Feature|
|---|---|
|HTTP/1.1|sequential requests|
|HTTP/2|multiplexing|
|HTTP/3|QUIC (UDP based)|

---

# Request Headers

Headers sent **from client → server**.

They provide context about:

- browser
    
- accepted formats
    
- authentication
    
- cookies
    
- caching
    

Example

```
GET /api/users HTTP/1.1
Host: example.com
User-Agent: Chrome
Accept: application/json
```

---

# Common Request Headers

---

# Host

Specifies the domain name.

```
Host: api.example.com
```

Important for **virtual hosting**.

---

# User-Agent

Identifies the **browser and OS**.

Example

```
User-Agent: Mozilla/5.0 (Windows NT 10.0) Chrome/120
```

Used for:

- analytics
    
- browser compatibility
    

---

# Accept

Tells server which **content types the client accepts**.

Example

```
Accept: application/json
```

Other examples

```
Accept: text/html
Accept: image/webp
Accept: */*
```

---

# Accept-Encoding

Compression formats supported.

Example

```
Accept-Encoding: gzip, deflate, br
```

Meaning:

Server can send compressed response.

---

# Accept-Language

User preferred languages.

Example

```
Accept-Language: en-US,en;q=0.9
```

Used for:

- localization
    
- international sites
    

---

# Authorization

Used for **authentication**.

Example

```
Authorization: Bearer eyJhbGci...
```

Types

|Type|Example|
|---|---|
|Basic|base64 credentials|
|Bearer|JWT tokens|
|API Key|custom|

---

# Cookie

Cookies sent with request.

Example

```
Cookie: session=abc123
```

Used for:

- login sessions
    
- tracking
    
- personalization
    

---

# Content-Type

Specifies **format of request body**.

Examples

```
Content-Type: application/json
Content-Type: multipart/form-data
Content-Type: application/x-www-form-urlencoded
```

---

# Content-Length

Size of request body.

Example

```
Content-Length: 348
```

---

# Origin

Indicates the **origin making the request**.

Example

```
Origin: https://example.com
```

Important for:

- CORS security
    

---

# Referer

URL of previous page.

Example

```
Referer: https://google.com
```

Used for analytics.

---

# Response Headers

Headers sent **from server → client**.

They describe:

- response format
    
- caching
    
- security
    
- cookies
    

---

# Content-Type

Format of response body.

Example

```
Content-Type: application/json
```

Other examples

```
text/html
image/png
application/javascript
```

---

# Content-Length

Size of response body.

```
Content-Length: 2048
```

---

# Cache-Control

Controls caching behavior.

Example

```
Cache-Control: public, max-age=3600
```

Meaning

```
cache for 1 hour
```

---

# ETag

Unique identifier for cached resource.

Example

```
ETag: "a1b2c3d4"
```

Browser sends later:

```
If-None-Match
```

If unchanged → server returns:

```
304 Not Modified
```

---

# Last-Modified

Last update time.

Example

```
Last-Modified: Wed, 20 Feb 2024 10:20:30 GMT
```

Browser can use:

```
If-Modified-Since
```

---

# Set-Cookie

Server instructs browser to store cookie.

Example

```
Set-Cookie: sessionId=abc123; HttpOnly; Secure
```

Options:

|Flag|Meaning|
|---|---|
|HttpOnly|JS cannot access|
|Secure|HTTPS only|
|SameSite|CSRF protection|

---

# Access-Control-Allow-Origin

CORS header allowing cross-origin requests.

Example

```
Access-Control-Allow-Origin: *
```

Or

```
Access-Control-Allow-Origin: https://example.com
```

---

# Server

Identifies backend server.

Example

```
Server: nginx
Server: Apache
Server: Express
```

---

# Date

Timestamp when response generated.

Example

```
Date: Wed, 20 Feb 2024 10:20:30 GMT
```

---

# Connection

Controls connection behavior.

Example

```
Connection: keep-alive
```

Meaning:

Reuse TCP connection.

---

# DevTools Timing Breakdown

Network tab also shows request timing.

Example:

```
Queueing
DNS Lookup
Initial Connection
SSL
Request Sent
Waiting (TTFB)
Content Download
```

---

# Queueing

Time waiting in browser request queue.

---

# DNS Lookup

Time resolving domain → IP.

---

# Initial Connection

Time to establish TCP connection.

---

# SSL

Time for TLS handshake.

---

# Request Sent

Time to send request to server.

---

# Waiting (TTFB)

Time to first byte.

Server processing time.

---

# Content Download

Time to download response.

---

# Quick Full Request Example

```
GET /api/users HTTP/2
Host: api.example.com
User-Agent: Chrome
Accept: application/json
Authorization: Bearer token
```

Response:

```
HTTP/2 200 OK
Content-Type: application/json
Content-Length: 342
Cache-Control: max-age=3600
```

---

# Why Understanding Headers Matters

Headers are critical for:

### Debugging

API failures  
authentication issues

### Performance

caching  
compression

### Security

CORS  
cookies  
authentication

### Backend Development

API design  
rate limiting  
authorization

---

---
tags:
  - api
  - back-end
---

- [[#What is an HTTP Status Code?|What is an HTTP Status Code?]]
- [[#100 Continue|100 Continue]]
- [[#101 Switching Protocols|101 Switching Protocols]]
- [[#Successful request|Successful request]]
- [[#Validation error|Validation error]]
- [[#Unauthorized|Unauthorized]]
- [[#Server error|Server error]]
- [[#1 Use proper codes|1 Use proper codes]]
- [[#2 Separate error responses|2 Separate error responses]]
- [[#3 Use 422 for validation|3 Use 422 for validation]]

# HTTP API Status Codes – Complete Reference

## What is an HTTP Status Code?

An **HTTP status code** is a **3-digit number returned by a server** in response to a client's request.  
It indicates whether the request was **successful, failed, redirected, or invalid**.

Example:

```http
GET /api/users/1

Response:
200 OK
{
  "id": 1,
  "name": "John"
}
```

Structure:

```
HTTP/1.1 200 OK
```

|Part|Meaning|
|---|---|
|HTTP/1.1|Protocol version|
|200|Status code|
|OK|Status message|

---

# Status Code Categories

|Range|Category|Meaning|
|---|---|---|
|1xx|Informational|Request received, processing|
|2xx|Success|Request successful|
|3xx|Redirection|Client must take extra action|
|4xx|Client Error|Client made invalid request|
|5xx|Server Error|Server failed to process request|

---

# 1xx Informational Responses

These are **rare in APIs** but exist in HTTP protocol.

## 100 Continue

The server received request headers and client can send body.

Use case:

```
Large file uploads
```

Flow:

```
Client -> Send headers
Server -> 100 Continue
Client -> Send file
```

---

## 101 Switching Protocols

Server switches protocol.

Example:

```
HTTP → WebSocket
```

Example response:

```
101 Switching Protocols
Upgrade: websocket
```

Used in:

- WebSockets
    
- HTTP/2 upgrades
    

---

# 2xx Success Responses

These mean **request was successfully processed**.

---

# 200 OK

Most common success response.

Use when:

- GET request successful
    
- PUT update successful
    
- PATCH successful
    

Example:

```
GET /users/5
```

Response:

```json
200 OK
{
 "id": 5,
 "name": "Alice"
}
```

---

# 201 Created

Used when **a new resource is created**.

Example:

```
POST /users
```

Response:

```json
201 Created
{
 "id": 10,
 "name": "Bob"
}
```

Best practice:

Include resource location.

```
Location: /users/10
```

Use cases:

- Creating user
    
- Creating order
    
- Creating database entry
    

---

# 202 Accepted

Request accepted but **not processed yet**.

Example:

```
POST /send-email
```

Email will be sent via background job.

Response:

```
202 Accepted
```

Use cases:

- Background tasks
    
- Queue processing
    
- Large file processing
    

---

# 204 No Content

Request successful but **no response body**.

Use cases:

- Delete operation
    
- Update with no response
    

Example:

```
DELETE /users/10
```

Response:

```
204 No Content
```

---

# 3xx Redirection Responses

Client must **redirect to another URL**.

Rare in APIs but common in web.

---

# 301 Moved Permanently

Resource permanently moved.

Example:

```
/api/v1/users → /api/v2/users
```

---

# 302 Found

Temporary redirect.

Example:

```
User login redirect
```

---

# 304 Not Modified

Used in **caching**.

If resource not changed, server returns:

```
304 Not Modified
```

Browser uses cached version.

Used with:

```
ETag
If-Modified-Since
```

---

# 4xx Client Errors

Client made **invalid request**.

These are **very common in API development**.

---

# 400 Bad Request

Request is invalid.

Reasons:

- Invalid JSON
    
- Missing parameters
    
- Wrong data format
    

Example:

```json
400 Bad Request
{
 "error": "Invalid email format"
}
```

---

# 401 Unauthorized

User **not authenticated**.

Example:

```
Missing token
Invalid token
```

Response:

```json
401 Unauthorized
{
 "error": "Authentication required"
}
```

Common in:

- JWT auth
    
- OAuth
    

---

# 403 Forbidden

User authenticated but **not allowed**.

Example:

```
User tries to access admin route
```

Response:

```json
403 Forbidden
{
 "error": "Access denied"
}
```

Example scenario:

```
GET /admin/users
```

User role = `USER`

---

# 404 Not Found

Requested resource does not exist.

Example:

```
GET /users/999
```

Response:

```json
404 Not Found
{
 "error": "User not found"
}
```

One of the **most common API responses**.

---

# 405 Method Not Allowed

HTTP method not supported.

Example:

```
GET /users
```

But API only supports POST.

Response:

```
405 Method Not Allowed
```

---

# 409 Conflict

Conflict with existing data.

Example:

```
User already exists
```

Example:

```json
409 Conflict
{
 "error": "Email already registered"
}
```

Used for:

- Duplicate entries
    
- Version conflicts
    

---

# 410 Gone

Resource permanently removed.

Example:

```
Old API endpoints
Deleted resources
```

---

# 413 Payload Too Large

Request body too large.

Example:

```
Uploading 200MB file
```

Response:

```
413 Payload Too Large
```

---

# 415 Unsupported Media Type

Content type not supported.

Example:

```
Content-Type: text/xml
```

But API only accepts JSON.

Response:

```
415 Unsupported Media Type
```

---

# 422 Unprocessable Entity

Request valid but **business logic fails**.

Very common in modern APIs.

Example:

```json
422 Unprocessable Entity
{
 "error": "Password must contain special characters"
}
```

Difference:

|Code|Meaning|
|---|---|
|400|Request malformed|
|422|Request valid but validation fails|

---

# 429 Too Many Requests

Rate limit exceeded.

Example:

```
API allows 100 requests/min
```

Response:

```
429 Too Many Requests
Retry-After: 60
```

Used in:

- Public APIs
    
- Auth endpoints
    
- Payment APIs
    

---

# 5xx Server Errors

Server failed to process request.

These indicate **backend issues**.

---

# 500 Internal Server Error

Generic server error.

Example:

```
Database crashed
Unhandled exception
```

Response:

```json
500 Internal Server Error
{
 "error": "Something went wrong"
}
```

---

# 501 Not Implemented

Server does not support functionality.

Example:

```
PATCH endpoint not implemented
```

---

# 502 Bad Gateway

Server acting as gateway received invalid response.

Example:

```
API Gateway → Microservice failed
```

Common in:

- Reverse proxies
    
- Microservices
    

---

# 503 Service Unavailable

Server temporarily unavailable.

Example:

```
Maintenance mode
Server overloaded
```

Response:

```
503 Service Unavailable
Retry-After: 120
```

---

# 504 Gateway Timeout

Server waited too long for upstream server.

Example:

```
API gateway → microservice timed out
```

---

# Most Common API Codes (80% Use)

In real API development, these are used **most of the time**.

|Code|Meaning|Use|
|---|---|---|
|200|OK|Successful request|
|201|Created|Resource created|
|204|No Content|Delete success|
|400|Bad Request|Invalid input|
|401|Unauthorized|Not logged in|
|403|Forbidden|No permission|
|404|Not Found|Resource missing|
|409|Conflict|Duplicate resource|
|422|Validation error|Input validation|
|429|Rate limit|Too many requests|
|500|Internal error|Server crash|

---

# Example REST API Responses

## Successful request

```json
200 OK
{
 "data": {
   "id": 5,
   "name": "John"
 }
}
```

---

## Validation error

```json
422 Unprocessable Entity
{
 "errors": [
   {
     "field": "email",
     "message": "Invalid email format"
   }
 ]
}
```

---

## Unauthorized

```json
401 Unauthorized
{
 "error": "Token expired"
}
```

---

## Server error

```json
500 Internal Server Error
{
 "error": "Unexpected server error"
}
```

---

# Best Practices for API Status Codes

## 1 Use proper codes

Bad:

```
200 OK
{
 "error": "User not found"
}
```

Correct:

```
404 Not Found
```

---

## 2 Separate error responses

Structure errors clearly.

Example:

```json
{
 "error": {
   "code": "INVALID_EMAIL",
   "message": "Email format invalid"
 }
}
```

---

## 3 Use 422 for validation

Most modern APIs:

```
400 → malformed request
422 → validation errors
```

---

# Quick Decision Guide

|Situation|Status Code|
|---|---|
|GET success|200|
|POST create|201|
|DELETE success|204|
|Invalid input|400|
|Not logged in|401|
|No permission|403|
|Resource missing|404|
|Duplicate resource|409|
|Validation error|422|
|Too many requests|429|
|Server crash|500|

---

# Example Next.js API Handler

```ts
export async function GET(req: Request) {

 const user = await db.user.findUnique({
   where: { id: 5 }
 })

 if (!user) {
   return Response.json(
     { error: "User not found" },
     { status: 404 }
   )
 }

 return Response.json(user, { status: 200 })
}
```

---

# Summary

HTTP status codes help APIs communicate **result of requests clearly**.

Flow:

```
Client Request
      ↓
Server processes request
      ↓
Server returns status code
      ↓
Client decides what to do
```

---

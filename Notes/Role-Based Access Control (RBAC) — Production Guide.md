---
tags:
  - access-control
  - back-end
  - front-end
  - config
  - auth
---

- [[#🧠 What is RBAC?|🧠 What is RBAC?]]
	- [[#🧠 What is RBAC?#Simple Example|Simple Example]]
- [[#⚙️ Why RBAC?|⚙️ Why RBAC?]]
- [[#🧩 RBAC vs Permission-Based System|🧩 RBAC vs Permission-Based System]]
	- [[#🧩 RBAC vs Permission-Based System#Pure RBAC (Basic)|Pure RBAC (Basic)]]
	- [[#🧩 RBAC vs Permission-Based System#RBAC + Permissions (Professional Approach ✅)|RBAC + Permissions (Professional Approach ✅)]]
- [[#🏗️ Backend Implementation (Source of Truth)|🏗️ Backend Implementation (Source of Truth)]]
	- [[#🏗️ Backend Implementation (Source of Truth)#🗃️ Database Design|🗃️ Database Design]]
	- [[#🏗️ Backend Implementation (Source of Truth)#🔐 Authorization Middleware|🔐 Authorization Middleware]]
	- [[#🏗️ Backend Implementation (Source of Truth)#🚀 Usage|🚀 Usage]]
- [[#🔑 Authentication + Authorization Flow|🔑 Authentication + Authorization Flow]]
- [[#💾 Where to Store Roles & Permissions (Frontend)|💾 Where to Store Roles & Permissions (Frontend)]]
	- [[#💾 Where to Store Roles & Permissions (Frontend)#✅ Recommended|✅ Recommended]]
		- [[#✅ Recommended#1. In-Memory State (Best)|1. In-Memory State (Best)]]
		- [[#✅ Recommended#2. Inside JWT (Optional)|2. Inside JWT (Optional)]]
	- [[#💾 Where to Store Roles & Permissions (Frontend)#❌ Avoid|❌ Avoid]]
- [[#🖥️ Frontend Usage Patterns|🖥️ Frontend Usage Patterns]]
	- [[#🖥️ Frontend Usage Patterns#🔐 Route Protection|🔐 Route Protection]]
	- [[#🖥️ Frontend Usage Patterns#🧩 Component-Level Control|🧩 Component-Level Control]]
	- [[#🖥️ Frontend Usage Patterns#🧠 Custom Hook Pattern|🧠 Custom Hook Pattern]]
- [[#🔄 Handling Role & Permission Changes (VERY IMPORTANT)|🔄 Handling Role & Permission Changes (VERY IMPORTANT)]]
	- [[#🔄 Handling Role & Permission Changes (VERY IMPORTANT)#Problem|Problem]]
	- [[#🔄 Handling Role & Permission Changes (VERY IMPORTANT)#✅ Solutions|✅ Solutions]]
	- [[#🔄 Handling Role & Permission Changes (VERY IMPORTANT)#1. Short-Lived JWT + Refresh Token (Best Practice)|1. Short-Lived JWT + Refresh Token (Best Practice)]]
	- [[#🔄 Handling Role & Permission Changes (VERY IMPORTANT)#2. Fetch `/me` on App Load|2. Fetch `/me` on App Load]]
	- [[#🔄 Handling Role & Permission Changes (VERY IMPORTANT)#3. Polling (Simple but less efficient)|3. Polling (Simple but less efficient)]]
	- [[#🔄 Handling Role & Permission Changes (VERY IMPORTANT)#4. Real-Time Updates (Advanced)|4. Real-Time Updates (Advanced)]]
- [[#⚠️ Common Pitfalls & Fixes|⚠️ Common Pitfalls & Fixes]]
	- [[#⚠️ Common Pitfalls & Fixes#❌ 1. Trusting Frontend for Security|❌ 1. Trusting Frontend for Security]]
	- [[#⚠️ Common Pitfalls & Fixes#❌ 2. Stale Permissions|❌ 2. Stale Permissions]]
	- [[#⚠️ Common Pitfalls & Fixes#❌ 3. Overusing Roles|❌ 3. Overusing Roles]]
	- [[#⚠️ Common Pitfalls & Fixes#❌ 4. Hardcoding Logic|❌ 4. Hardcoding Logic]]
	- [[#⚠️ Common Pitfalls & Fixes#❌ 5. Storing Sensitive Data in LocalStorage|❌ 5. Storing Sensitive Data in LocalStorage]]
- [[#🧠 Advanced Patterns|🧠 Advanced Patterns]]
	- [[#🧠 Advanced Patterns#🔹 Attribute-Based Access Control (ABAC)|🔹 Attribute-Based Access Control (ABAC)]]
	- [[#🧠 Advanced Patterns#🔹 Policy-Based Access Control (PBAC)|🔹 Policy-Based Access Control (PBAC)]]
- [[#🏁 Recommended Architecture|🏁 Recommended Architecture]]
- [[#💡 Key Principles|💡 Key Principles]]


## 🧠 What is RBAC?

**Role-Based Access Control (RBAC)** is an authorization model where:

- Users are assigned **roles**
    
- Roles are assigned **permissions**
    
- Permissions define **what actions are allowed**
    

### Simple Example

```txt
Admin  → create_user, delete_user, view_reports
Editor → edit_content, publish_content
Viewer → view_content
```

---

## ⚙️ Why RBAC?

- Simplifies permission management
    
- Scales well with large teams
    
- Centralizes access control logic
    
- Makes audits and compliance easier
    

---

## 🧩 RBAC vs Permission-Based System

### Pure RBAC (Basic)

```ts
user = {
  id: "1",
  role: "admin"
}
```

### RBAC + Permissions (Professional Approach ✅)

```ts
user = {
  id: "1",
  roles: ["admin"],
  permissions: ["create_user", "delete_user"] // derived or override
}
```

👉 Roles = grouping  
👉 Permissions = actual control

---

## 🏗️ Backend Implementation (Source of Truth)

> ⚠️ Backend ALWAYS enforces access control

---

### 🗃️ Database Design

```txt
users
roles
permissions

user_roles (user_id, role_id)
role_permissions (role_id, permission_id)

(optional)
user_permissions (user_id, permission_id)
```

---

### 🔐 Authorization Middleware

```ts
function authorize(requiredPermission: string) {
  return (req, res, next) => {
    const user = req.user;

    if (!user.permissions.includes(requiredPermission)) {
      return res.status(403).json({ message: "Forbidden" });
    }

    next();
  };
}
```

---

### 🚀 Usage

```ts
app.post("/create-user", authorize("create_user"), handler);
```

---

## 🔑 Authentication + Authorization Flow

```txt
1. User logs in
2. Backend validates credentials
3. Backend returns:
   - Access Token (JWT)
   - Refresh Token
4. Frontend stores token
5. Frontend fetches user data (/me)
6. UI renders based on permissions
7. Backend validates every request
```

---

## 💾 Where to Store Roles & Permissions (Frontend)

### ✅ Recommended

#### 1. In-Memory State (Best)

- React Context / Zustand / Redux
    

```ts
const user = {
  roles: ["admin"],
  permissions: ["create_user"]
};
```

---

#### 2. Inside JWT (Optional)

```json
{
  "sub": "123",
  "roles": ["admin"]
}
```

👉 Use for UI hints only, NOT security

---

### ❌ Avoid

- LocalStorage (tamperable)
    
- Hardcoding roles in UI
    

---

## 🖥️ Frontend Usage Patterns

---

### 🔐 Route Protection

```ts
if (!user.permissions.includes("admin_dashboard")) {
  return <Unauthorized />;
}
```

---

### 🧩 Component-Level Control

```tsx
{user.permissions.includes("delete_user") && (
  <DeleteButton />
)}
```

---

### 🧠 Custom Hook Pattern

```ts
function usePermission(permission: string) {
  const { user } = useUser();

  return user?.permissions.includes(permission);
}
```

```tsx
const canEdit = usePermission("edit_post");
```

---

## 🔄 Handling Role & Permission Changes (VERY IMPORTANT)

### Problem

- User role updated in backend
    
- Frontend still using old permissions
    

---

### ✅ Solutions

---

### 1. Short-Lived JWT + Refresh Token (Best Practice)

- Access token expires quickly (5–15 min)
    
- Refresh token fetches updated permissions
    

```txt
Access Token → short-lived
Refresh Token → long-lived
```

👉 Ensures fresh permissions automatically

---

### 2. Fetch `/me` on App Load

```ts
GET /me
```

Response:

```json
{
  "roles": ["admin"],
  "permissions": ["create_user"]
}
```

👉 Store in global state

---

### 3. Polling (Simple but less efficient)

```ts
setInterval(() => fetchUser(), 5 * 60 * 1000);
```

---

### 4. Real-Time Updates (Advanced)

- WebSocket / Server-Sent Events (SSE)
    

```txt
Backend → pushes role updates → frontend updates state
```

👉 Used in:

- Admin dashboards
    
- Collaborative tools
    

---

## ⚠️ Common Pitfalls & Fixes

---

### ❌ 1. Trusting Frontend for Security

**Problem:**  
User can modify JS or API calls

**Fix:**  
✔ Always validate on backend

---

### ❌ 2. Stale Permissions

**Problem:**  
User role updated but UI still shows old access

**Fix:**  
✔ Short-lived tokens  
✔ Re-fetch `/me`

---

### ❌ 3. Overusing Roles

**Problem:**  
Too many roles → hard to manage

**Fix:**  
✔ Use fewer roles + granular permissions

---

### ❌ 4. Hardcoding Logic

```ts
if (user.role === "admin")
```

**Fix:**  
✔ Use permission-based checks

---

### ❌ 5. Storing Sensitive Data in LocalStorage

**Fix:**  
✔ Use HttpOnly cookies / in-memory state

---

## 🧠 Advanced Patterns

---

### 🔹 Attribute-Based Access Control (ABAC)

Instead of roles:

```ts
user.department === resource.department
```

👉 Dynamic and context-aware

---

### 🔹 Policy-Based Access Control (PBAC)

```json
{
  "action": "edit",
  "resource": "post",
  "condition": "user.id === post.ownerId"
}
```

👉 Used in AWS IAM, enterprise apps

---

## 🏁 Recommended Architecture

```txt
Backend:
- Manages roles & permissions
- Validates every request
- Issues tokens

Frontend:
- Stores permissions in memory
- Uses them for UI only

Flow:
Login → Token → /me → Store → Use in UI
```

---

## 💡 Key Principles

- Backend = authority
    
- Frontend = experience
    
- Permissions > Roles
    
- Always assume frontend is compromised
    

---

---
tags:
  - next
  - config
  - urls
---
- [[#📌 Context|📌 Context]]
- [[#`.env.development`|`.env.development`]]
- [[#🔹 `NEXT_PUBLIC_BASE_URL`|🔹 `NEXT_PUBLIC_BASE_URL`]]
	- [[#🔹 `NEXT_PUBLIC_BASE_URL`#What it is:|What it is:]]
	- [[#🔹 `NEXT_PUBLIC_BASE_URL`#Important:|Important:]]
	- [[#🔹 `NEXT_PUBLIC_BASE_URL`#What it typically represents:|What it typically represents:]]
	- [[#🔹 `NEXT_PUBLIC_BASE_URL`#Key Understanding:|Key Understanding:]]
- [[#🔹 `NEXT_PUBLIC_` Prefix|🔹 `NEXT_PUBLIC_` Prefix]]
- [[#🌍 Base URL = Domain|🌍 Base URL = Domain]]
- [[#📁 basePath = Sub-folder inside domain|📁 basePath = Sub-folder inside domain]]
	- [[#📁 basePath = Sub-folder inside domain#Effect on routes:|Effect on routes:]]
- [[#🧠 Mental Model|🧠 Mental Model]]
- [[#If You Want:|If You Want:]]
	- [[#If You Want:#✅ Option 1: Single App, Folder Routing|✅ Option 1: Single App, Folder Routing]]
	- [[#If You Want:#✅ Option 2: Multiple Apps + Reverse Proxy|✅ Option 2: Multiple Apps + Reverse Proxy]]
- [[#🔁 Redirect|🔁 Redirect]]
	- [[#🔁 Redirect#What Happens:|What Happens:]]
	- [[#🔁 Redirect#Important:|Important:]]
- [[#🔄 Rewrite|🔄 Rewrite]]
	- [[#🔄 Rewrite#Important:|Important:]]
- [[#🧠 Redirect vs Rewrite Summary|🧠 Redirect vs Rewrite Summary]]
	- [[#🧠 Redirect vs Rewrite Summary#Development:|Development:]]
	- [[#🧠 Redirect vs Rewrite Summary#Production:|Production:]]
	- [[#🧠 Redirect vs Rewrite Summary#globalNotFound|globalNotFound]]
	- [[#🧠 Redirect vs Rewrite Summary#serverActions.bodySizeLimit|serverActions.bodySizeLimit]]
- [[#1️⃣ Application Routing|1️⃣ Application Routing]]
- [[#2️⃣ Next.js Configuration|2️⃣ Next.js Configuration]]
- [[#3️⃣ Infrastructure Routing|3️⃣ Infrastructure Routing]]

## 📌 Context

This note explains in depth how configuration works in **Next.js**, especially around:

- `basePath`
    
- `BASE_URL`
    
- `assetPrefix`
    
- `redirects`
    
- `rewrites`
    
- `headers`
    
- Environment variables
    
- Hosting architecture implications
    

We will use the following config as reference:

```ts
import type { NextConfig } from "next";

const basePath = process.env.NEXT_PUBLIC_BASE_PATH || "/safephv_ri";

const config: NextConfig = {
  reactStrictMode: true,

  basePath: basePath,
  assetPrefix: process.env.NODE_ENV === "production" ? basePath : undefined,

  reactCompiler: true,

  async headers() {
    return [
      {
        source: "/:path*",
        headers: [
          {
            key: 'Access-Control-Allow-Origin',
            value: process.env.NODE_ENV === 'development' ? '*' : '',
          },
          {
            key: "Access-Control-Allow-Methods",
            value: "GET, POST, PUT, DELETE, OPTIONS",
          },
          {
            key: "Access-Control-Allow-Headers",
            value: "X-Requested-With, Content-Type, Authorization",
          },
        ],
      },
    ];
  },

  async redirects() {
    return [
      {
        source: "/",
        destination: basePath,
        permanent: true,
        basePath: false,
      },
    ];
  },

  async rewrites() {
    return [
      {
        source: "/api/:path((?!authentication).*)",
        destination: `${process.env.BACKEND_URL}/:path*`,
      },
    ];
  },

  experimental: {
    globalNotFound: true,
    serverActions: {
      bodySizeLimit: 104857600 * 5, // 500MB
    },
  },
};

export default config;
```

---

# 1️⃣ Environment Variables

## `.env.development`

```env
NEXT_PUBLIC_BASE_URL="http://localhost:3000"
NEXT_PUBLIC_BASE_PATH=/safephv_ri
```

---

## 🔹 `NEXT_PUBLIC_BASE_URL`

### What it is:

A **custom environment variable**.

### Important:

Next.js does **NOT** use this automatically.

It is only available because:

- You defined it.
    
- You use it in your code.
    

### What it typically represents:

The public domain of your app.

Example usage:

```ts
fetch(`${process.env.NEXT_PUBLIC_BASE_URL}/api/users`)
```

### Key Understanding:

|What It Is|What It Is Not|
|---|---|
|Public domain reference|Does NOT control server port|
|Used for absolute URLs|Does NOT control hosting|
|Used by your code|Not used by Next internally|

---

## 🔹 `NEXT_PUBLIC_` Prefix

Variables with `NEXT_PUBLIC_`:

- Available on client side
    
- Embedded at build time
    
- Accessible in browser
    

Without prefix:

- Only available server-side
    

---

# 2️⃣ Base URL vs Base Path

This is where confusion usually happens.

## 🌍 Base URL = Domain

Examples:

```
http://localhost:3000
https://example.com
```

Controlled by:

- `next dev`
    
- `next start`
    
- Hosting provider
    
- Reverse proxy
    
- Port configuration
    

NOT controlled by:

- `NEXT_PUBLIC_BASE_URL`
    

---

## 📁 basePath = Sub-folder inside domain

Configured in `next.config.js`:

```ts
basePath: "/safephv_ri"
```

If enabled:

```
http://localhost:3000/safephv_ri
```

Becomes your app root.

### Effect on routes:

|Without basePath|With basePath|
|---|---|
|`/about`|`/safephv_ri/about`|
|`/api/users`|`/safephv_ri/api/users`|

---

## 🧠 Mental Model

```
BASE URL + basePath = Full App Root
```

Example:

```
http://localhost:3000 + /safephv_ri
= http://localhost:3000/safephv_ri
```

---

# 3️⃣ Why Use basePath?

Use when:

- Deploying under subfolder
    
- Hosting behind reverse proxy
    
- Corporate infrastructure requires subdirectory
    
- Multiple apps share same domain
    

Do NOT use for organizing app features.

---

# 4️⃣ Can You Have Multiple basePaths?

❌ No.

`basePath` is:

- A single string
    
- Applied globally
    
- Compile-time configuration
    

One Next.js app = One basePath.

---

## If You Want:

```
example.com/store
example.com/blog
example.com/app
```

You have two options:

### ✅ Option 1: Single App, Folder Routing

```
app/
  store/
  blog/
  dashboard/
```

### ✅ Option 2: Multiple Apps + Reverse Proxy

Infrastructure routes:

|Path|App|
|---|---|
|`/store`|App 1|
|`/blog`|App 2|

Handled by:

- nginx
    
- load balancer
    
- cloud routing
    

This is infrastructure-level routing, not app-level routing.

---

# 5️⃣ assetPrefix

```ts
assetPrefix: process.env.NODE_ENV === "production" ? basePath : undefined
```

Controls where static assets load from.

Without basePath:

```
/_next/static/chunk.js
```

With basePath:

```
/safephv_ri/_next/static/chunk.js
```

Used for:

- Subfolder deployments
    
- CDN hosting
    
- Reverse proxy setups
    

It only affects:

- Static files
    
- JS bundles
    
- Images
    

---

# 6️⃣ Redirects vs Rewrites

This is critical.

---

## 🔁 Redirect

```ts
async redirects() {
  return [
    {
      source: "/",
      destination: basePath,
      permanent: true,
      basePath: false,
    },
  ];
}
```

### What Happens:

User visits:

```
http://localhost:3000/
```

Browser receives:

```
HTTP 301
```

Browser changes URL to:

```
http://localhost:3000/safephv_ri
```

### Important:

Redirect:

- Changes browser URL
    
- Visible in Network tab
    
- Real HTTP response
    

---

## 🔄 Rewrite

```ts
async rewrites() {
  return [
    {
      source: "/api/:path((?!authentication).*)",
      destination: `${process.env.BACKEND_URL}/:path*`,
    },
  ];
}
```

User calls:

```
/api/users
```

Internally becomes:

```
http://backend-url/users
```

But browser still shows:

```
/api/users
```

### Important:

Rewrite:

- Does NOT change browser URL
    
- Acts like internal proxy
    
- Used for backend forwarding
    

---

## 🧠 Redirect vs Rewrite Summary

|Feature|Redirect|Rewrite|
|---|---|---|
|Browser URL changes?|✅ Yes|❌ No|
|Visible to user?|✅ Yes|❌ No|
|Acts like proxy?|❌ No|✅ Yes|

---

# 7️⃣ Headers (CORS)

```ts
async headers() {
  return [
    {
      source: "/:path*",
      headers: [...]
    },
  ];
}
```

Applies headers to all routes.

Used here for CORS.

### Development:

```
Access-Control-Allow-Origin: *
```

### Production:

```
Access-Control-Allow-Origin: ""
```

Controls:

- Which domains can access your APIs
    
- Cross-origin requests
    
- Browser security rules
    

---

# 8️⃣ Experimental Features

```ts
experimental: {
  globalNotFound: true,
  serverActions: {
    bodySizeLimit: 104857600 * 5,
  },
}
```

### globalNotFound

Improves 404 handling.

### serverActions.bodySizeLimit

500MB limit.

Used for:

- File uploads
    
- Large request bodies
    
- Server Actions
    

Default is much smaller.

---

# 9️⃣ Hosting Architecture Mental Model

There are 3 layers:

## 1️⃣ Application Routing

Folders in `app/`

## 2️⃣ Next.js Configuration

- basePath
    
- redirects
    
- rewrites
    

## 3️⃣ Infrastructure Routing

- nginx
    
- cloud load balancer
    
- reverse proxy
    

Most confusion happens when mixing 2 and 3.

---

# 🔟 Common Misconceptions (Now Clarified)

|Statement|Truth|
|---|---|
|baseURL defines where Next runs|❌ No|
|basePath defines server port|❌ No|
|Redirects are invisible|❌ No|
|Rewrites change URL|❌ No|
|Multiple basePaths allowed|❌ No|
|One domain can host multiple apps|✅ Yes|

---

# 🎯 Final Mental Model

Think of your app like this:

```
[ Domain ]
     |
[ basePath ]
     |
[ Next.js Router ]
     |
[ Pages / API ]
```

And infrastructure sits above everything.

---

# 📌 Final Takeaway

- `BASE_URL` → informational, used by your code.
    
- `basePath` → mounting point for entire app.
    
- Redirect → visible URL change.
    
- Rewrite → invisible internal proxy.
    
- One app → one basePath.
    
- Multiple apps on one domain → infrastructure routing.
    

---
tags:
  - auth
  - front-end
  - next
---
- [[#1. What is NextAuth / Auth.js|1. What is NextAuth / Auth.js]]
- [[#Authentication vs Authorization|Authentication vs Authorization]]
- [[#Sessions|Sessions]]
	- [[#Sessions#JWT Session (default)|JWT Session (default)]]
	- [[#Sessions#Database Sessions|Database Sessions]]
	- [[#Sessions#Server Component|Server Component]]
	- [[#Sessions#Client Component|Client Component]]
- [[#Step 1 Generate Token|Step 1 Generate Token]]
- [[#Step 2 Send Email|Step 2 Send Email]]
- [[#Step 3 Reset Password|Step 3 Reset Password]]
	- [[#Step 3 Reset Password#Password hashing|Password hashing]]
	- [[#Step 3 Reset Password#Rate limiting login attempts|Rate limiting login attempts]]
	- [[#Step 3 Reset Password#Email verification|Email verification]]
	- [[#Step 3 Reset Password#CSRF protection|CSRF protection]]
	- [[#Step 3 Reset Password#HttpOnly cookies|HttpOnly cookies]]
	- [[#Step 3 Reset Password#1 Using credentials without hashing|1 Using credentials without hashing]]
	- [[#Step 3 Reset Password#2 Not validating input|2 Not validating input]]
	- [[#Step 3 Reset Password#3 Not protecting routes|3 Not protecting routes]]
	- [[#Step 3 Reset Password#4 Storing sensitive data in JWT|4 Storing sensitive data in JWT]]

# NextAuth (Auth.js v5) Complete Guide – Next.js Authentication (2026)

## 1. What is NextAuth / Auth.js

**NextAuth.js** (now called **Auth.js**) is an authentication library designed for **Next.js applications**.

It handles:

- Login
    
- Logout
    
- Sessions
    
- JWT tokens
    
- OAuth providers (Google, GitHub)
    
- Credentials login (email/password)
    
- Middleware auth protection
    
- CSRF protection
    
- Secure cookies
    
- Password reset flows
    

It abstracts most authentication complexity so developers do not manually handle session cookies, encryption, token validation, etc. ([Next.js](https://nextjs.org/learn/dashboard-app/adding-authentication?utm_source=chatgpt.com "App Router: Adding Authentication | Next.js"))

---

# 2. Important Concepts

Before implementing, understand these concepts.

## Authentication vs Authorization

Authentication = Who are you  
Authorization = What are you allowed to do

Example:

```
Authentication → login with email/password
Authorization → admin can delete users
```

---

## Sessions

Auth.js supports two session strategies.

### JWT Session (default)

Session stored in **encrypted cookie**

Pros

- No database required
    
- Fast
    
- Stateless
    

Cons

- Harder to revoke session
    

Example config:

```ts
session: {
  strategy: "jwt",
  maxAge: 30 * 24 * 60 * 60
}
```

---

### Database Sessions

Session stored in database

Pros

- Can revoke sessions
    
- Central control
    

Cons

- Requires DB queries
    

Auth.js supports both but **JWT is most common**. ([Next.js Launchpad](https://nextjslaunchpad.com/article/nextjs-authentication-authjs-v5-complete-guide-sessions-providers-route-protection?utm_source=chatgpt.com "Next.js Authentication with Auth.js v5: Sessions, Providers, and Route Protection | Next.js Launchpad"))

---

# 3. Modern Architecture (2026)

Typical stack:

```
Next.js 14+
Auth.js v5
Prisma
PostgreSQL
bcrypt
zod
server actions
```

Architecture pattern:

```
auth.config.ts   → edge safe config
auth.ts          → main auth setup
middleware.ts    → route protection
server actions   → login/register logic
db (Prisma)      → user storage
```

This separation helps middleware run at edge while database logic stays server-side. ([Next.js Launchpad](https://nextjslaunchpad.com/article/build-complete-auth-system-authjs-v5-registration-login-password-reset-nextjs?utm_source=chatgpt.com "Auth.js v5 Credentials Setup Guide (2026) | Next.js Launchpad"))

---

# 4. Install Dependencies

```
pnpm add next-auth@beta
pnpm add prisma @prisma/client
pnpm add bcryptjs
pnpm add zod
pnpm add nodemailer
```

---

# 5. Environment Variables

```
AUTH_SECRET=your-secret
DATABASE_URL=postgresql://...
NEXTAUTH_URL=http://localhost:3000
```

Generate secret:

```
openssl rand -base64 32
```

This encrypts cookies and JWT tokens. ([Next.js](https://nextjs.org/learn/dashboard-app/adding-authentication?utm_source=chatgpt.com "App Router: Adding Authentication | Next.js"))

---

# 6. Database Schema (Prisma)

Example schema.

```
model User {
  id            String   @id @default(uuid())
  name          String?
  email         String   @unique
  password      String
  emailVerified DateTime?
  role          String   @default("USER")
  createdAt     DateTime @default(now())
}

model PasswordResetToken {
  id        String   @id @default(uuid())
  email     String
  token     String
  expires   DateTime
}
```

Run migration:

```
npx prisma migrate dev
```

---

# 7. Auth Configuration

Create

```
/auth.config.ts
```

```ts
import type { NextAuthConfig } from "next-auth"

export const authConfig: NextAuthConfig = {
  pages: {
    signIn: "/login",
    error: "/auth/error"
  }
}
```

---

# 8. Main Auth Setup

Create

```
/auth.ts
```

```ts
import NextAuth from "next-auth"
import Credentials from "next-auth/providers/credentials"
import { PrismaAdapter } from "@auth/prisma-adapter"
import { db } from "@/lib/db"
import bcrypt from "bcryptjs"

export const { handlers, auth, signIn, signOut } = NextAuth({

  adapter: PrismaAdapter(db),

  session: {
    strategy: "jwt"
  },

  providers: [
    Credentials({

      name: "Credentials",

      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" }
      },

      async authorize(credentials) {

        const user = await db.user.findUnique({
          where: { email: credentials.email }
        })

        if (!user) return null

        const isValid = await bcrypt.compare(
          credentials.password,
          user.password
        )

        if (!isValid) return null

        return user
      }

    })
  ]
})
```

This does:

- verifies user credentials
    
- compares password
    
- returns user session
    

---

# 9. API Route Handler

Create:

```
app/api/auth/[...nextauth]/route.ts
```

```
import { handlers } from "@/auth"

export const { GET, POST } = handlers
```

This exposes the authentication endpoints.

---

# 10. Login Implementation

Example login form.

```ts
"use client"

import { signIn } from "next-auth/react"

async function handleLogin(email, password) {

  await signIn("credentials", {
    email,
    password,
    redirectTo: "/dashboard"
  })

}
```

---

# 11. Logout Implementation

```ts
import { signOut } from "next-auth/react"

<button onClick={() => signOut()}>
  Logout
</button>
```

---

# 12. Registration (Sign Up)

Auth.js does not create users automatically for credentials.

You create your own API or server action.

Example:

```ts
export async function register(data) {

  const hashedPassword = await bcrypt.hash(data.password, 10)

  await db.user.create({
    data: {
      email: data.email,
      password: hashedPassword
    }
  })

}
```

---

# 13. Session Access

### Server Component

Auth.js v5 introduced the **auth() function**.

```ts
import { auth } from "@/auth"

export default async function Page() {

  const session = await auth()

  if (!session) redirect("/login")

  return <div>Welcome {session.user.email}</div>
}
```

Older `getServerSession` is no longer required. ([authjs.cn](https://www.authjs.cn/getting-started/migrating-to-v5?utm_source=chatgpt.com "Migrating to v5"))

---

### Client Component

```
import { useSession } from "next-auth/react"

const { data: session } = useSession()
```

---

# 14. Route Protection (Middleware)

Create

```
middleware.ts
```

```ts
export { auth as middleware } from "@/auth"

export const config = {
  matcher: ["/dashboard/:path*"]
}
```

This ensures:

```
/dashboard/* requires login
```

---

# 15. Authorization (Roles)

Add role to JWT.

```ts
callbacks: {

 async jwt({ token, user }) {

   if(user){
     token.role = user.role
   }

   return token
 },

 async session({ session, token }) {

   session.user.role = token.role

   return session
 }

}
```

Then protect routes.

```ts
if(session.user.role !== "ADMIN"){
   redirect("/")
}
```

---

# 16. Forgot Password Flow

Flow:

```
User enters email
↓
Generate reset token
↓
Send email
↓
User clicks link
↓
Reset password
```

---

## Step 1 Generate Token

```ts
import crypto from "crypto"

const token = crypto.randomBytes(32).toString("hex")

await db.passwordResetToken.create({
  data:{
    email,
    token,
    expires: new Date(Date.now() + 3600000)
  }
})
```

---

## Step 2 Send Email

Use **nodemailer / resend**

Example:

```
resetLink =
http://localhost:3000/reset-password?token=xyz
```

---

## Step 3 Reset Password

Verify token.

```
const token = await db.passwordResetToken.findUnique({
  where: { token }
})
```

Update password.

```
await db.user.update({
  where:{ email: token.email },
  data:{
    password: hashedPassword
  }
})
```

Delete token.

---

# 17. Reset Password Page

```
async function resetPassword(password, token){

 await fetch("/api/reset-password",{
  method:"POST",
  body: JSON.stringify({password, token})
 })

}
```

---

# 18. JWT Customization

JWT callback runs on login.

```
callbacks: {

 async jwt({ token, user }) {

   if(user){
     token.id = user.id
   }

   return token
 }

}
```

Session callback exposes JWT fields.

```
callbacks: {

 async session({ session, token }) {

   session.user.id = token.id

   return session
 }

}
```

---

# 19. Protect API Routes

```
import { auth } from "@/auth"

export async function GET(){

 const session = await auth()

 if(!session){
   return Response.json(
     {error:"Unauthorized"},
     {status:401}
   )
 }

}
```

---

# 20. Secure Practices

Always implement:

### Password hashing

```
bcrypt
```

### Rate limiting login attempts

Prevent brute force.

### Email verification

Before allowing login.

### CSRF protection

Handled automatically by Auth.js.

### HttpOnly cookies

Auth.js uses secure cookies.

---

# 21. Full Authentication Flow

```
User registers
↓
Password hashed and saved
↓
User logs in
↓
Credentials verified
↓
JWT created
↓
JWT stored in cookie
↓
auth() retrieves session
↓
Middleware protects routes
↓
User logs out
↓
Session cookie removed
```

---

# 22. Example Folder Structure

```
app
 ├ api
 │  └ auth
 │    └ [...nextauth]
 │       └ route.ts
 │
 ├ login
 │  └ page.tsx
 │
 ├ register
 │  └ page.tsx
 │
 ├ dashboard
 │  └ page.tsx

lib
 └ db.ts

auth.ts
auth.config.ts
middleware.ts
```

---

# 23. OAuth Providers (Optional)

Add Google login.

```
import Google from "next-auth/providers/google"

providers:[
 Google({
   clientId: process.env.GOOGLE_ID,
   clientSecret: process.env.GOOGLE_SECRET
 })
]
```

---

# 24. Common Mistakes

### 1 Using credentials without hashing

Never store plain passwords.

---

### 2 Not validating input

Use **zod**

---

### 3 Not protecting routes

Always add middleware.

---

### 4 Storing sensitive data in JWT

JWT should contain only minimal info.

---

# 25. When NOT to Use NextAuth

Use custom auth when:

- You already have a backend auth server
    
- Using microservices
    
- Need custom token lifecycle
    

---

# 26. Production Improvements

Recommended additions:

```
Email verification
Rate limiting
2FA
Role based access
Audit logs
Session revocation
```

Many advanced examples implement these features. ([GitHub](https://github.com/zenWai/nextjs14-next-authv5-app-router?utm_source=chatgpt.com "GitHub - zenWai/nextjs14-next-authv5-app-router: Nextjs 14 w/ Next-Auth v5, Prisma, zod, react-hook-form | Forgot password | email verification | 2FA | User Roles | Rate Limiting"))

---

# 27. Mental Model (How NextAuth Works)

Think of it like this:

```
Login
 ↓
Provider validates user
 ↓
JWT created
 ↓
JWT stored in cookie
 ↓
auth() reads cookie
 ↓
session returned
```

---

# 28. Summary

Auth.js handles:

```
Login
Logout
Sessions
JWT
OAuth
Route protection
CSRF
Cookies
```

You implement:

```
User database
Registration
Password reset
Authorization rules
```


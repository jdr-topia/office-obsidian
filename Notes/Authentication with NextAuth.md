---
created: 2026-04-02
tags: [authentication, nextjs, next-auth]
related: [ [[Authentication and authorization MOC]], [[OAuth and social login]], [[Session vs JWT authentication]] ]
status: active
---

# Authentication with NextAuth

NextAuth.js (now **Auth.js**) is the de-facto authentication library for Next.js apps. It handles OAuth flows, session management, and token refresh automatically.

## Core Setup (App Router)

### 1. Install

```bash
npm install next-auth
```

### 2. Create the Route Handler

```ts
// app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth";
import Google from "next-auth/providers/google";
import CredentialsProvider from "next-auth/providers/credentials";

export const { GET, POST } = NextAuth({
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    CredentialsProvider({
      name: "Email",
      credentials: { email: {}, password: {} },
      async authorize(credentials) {
        const user = await verifyUser(credentials.email, credentials.password);
        return user ?? null;
      },
    }),
  ],
  session: { strategy: "jwt" },
  callbacks: {
    async jwt({ token, user }) {
      if (user) token.role = user.role; // Attach custom fields
      return token;
    },
    async session({ session, token }) {
      session.user.role = token.role as string; // Expose role to client
      return session;
    },
  },
});
```

### 3. Wrap App with SessionProvider

```tsx
// app/layout.tsx
import { SessionProvider } from "next-auth/react";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <SessionProvider>{children}</SessionProvider>
      </body>
    </html>
  );
}
```

---

## Reading Session in Components

```tsx
// Client Component
"use client";
import { useSession, signIn, signOut } from "next-auth/react";

export default function Navbar() {
  const { data: session, status } = useSession();

  if (status === "loading") return <p>Loading...</p>;
  if (!session) return <button onClick={() => signIn()}>Sign In</button>;

  return (
    <div>
      <p>Hello, {session.user?.name}</p>
      <button onClick={() => signOut()}>Sign Out</button>
    </div>
  );
}
```

---

## Server-Side Auth (RSC & Middleware)

```ts
// In a Server Component or Route Handler
import { auth } from "@/auth";

export default async function Page() {
  const session = await auth();
  if (!session) redirect("/login");
  return <Dashboard />;
}
```

```ts
// middleware.ts — Protects all /dashboard routes at the edge
export { auth as middleware } from "@/auth";

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

---

## Related Topics
- [[Authentication and authorization MOC]]
- [[OAuth and social login]]
- [[Session vs JWT authentication]]
- [[Protected routes]]

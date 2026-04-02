---
created: 2026-04-02
tags: [authentication, oauth, security]
related: [ [[Authentication and authorization MOC]], [[Session vs JWT authentication]] ]
status: active
---

# OAuth and social login

OAuth 2.0 is an authorization framework that enables a third-party application (your app) to obtain limited access to a user's account on an external service (Google, GitHub, etc.) without needing their password.

## The Core Idea: Delegated Authentication

Instead of your app handling passwords, you say: *"I trust Google to verify who this person is."*

---

## OAuth 2.0 Flow (Authorization Code Flow)

This is the most secure and recommended flow for web apps.

```
1. User clicks "Sign in with Google"
2. Your app redirects user to Google's Authorization Server
   (with client_id, redirect_uri, scope, state)
3. User approves the permissions on Google's consent screen
4. Google redirects back to your redirect_uri with an `authorization_code`
5. Your SERVER exchanges the code for an `access_token` (+ `refresh_token`)
   using your secret `client_secret`
6. Your server uses the access_token to fetch the user's profile from Google
7. Your app creates/updates the user in YOUR database and issues its own session
```

The `client_secret` is **never exposed to the browser**. This is why the code exchange must happen server-side.

---

## Key Concepts

### Scopes
Defines what data you are requesting access to.
- `openid profile email` — Standard OIDC scopes to get basic user info.

### State Parameter
A random, unique value you generate and send with the initial redirect. Google returns it, and you verify it matches. This prevents **CSRF attacks** on the OAuth flow.

### OpenID Connect (OIDC)
An identity layer built on top of OAuth 2.0. It adds the concept of an **ID Token** (a JWT containing the user's info — name, email, avatar), simplifying the "fetch user profile" step.

---

## Next.js Implementation

Use **NextAuth.js** (now Auth.js) which handles the entire server-side flow for you.

```ts
// app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth";
import GoogleProvider from "next-auth/providers/google";

export const { GET, POST } = NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
});
```

---

## Related Topics
- [[Authentication and authorization MOC]]
- [[Session vs JWT authentication]]
- [[Authentication with NextAuth]]
- [[CSRF protection]]

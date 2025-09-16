# 🔐 Authentication & Authorization

## Overview

Masterji implements a robust authentication system supporting multiple authentication providers and role-based access control (RBAC). The system is built using NextAuth.js with a custom adapter for Prisma.

## 📋 Authentication Flow

### 1. User Authentication

#### Supported Authentication Methods:
1. **GitHub OAuth**
   - OAuth 2.0 flow with GitHub
   - Requires GitHub OAuth App credentials
   - Handles user profile synchronization

2. **Email/Passwordless**
   - Magic link authentication
   - One-time password (OTP) via email
   - Secure session management

### 2. Session Management

- **JWT-based sessions** for stateless authentication
- Secure HTTP-only cookies for session storage
- Automatic session refresh and validation
- Session timeout and expiry handling

## 🔑 Authorization

### User Roles

1. **Admin**
   - Full system access
   - Can manage all courses and users
   - Access to admin dashboard

2. **Instructor**
   - Can create and manage their own courses
   - Can view and manage their students' progress
   - Limited access to admin features

3. **Student**
   - Can enroll in courses
   - Access to purchased courses
   - Track learning progress

### Protected Routes

- `/admin/**` - Admin-only routes
- `/dashboard` - Authenticated user dashboard
- `/api/admin/**` - Admin API endpoints

## 🔐 Security Measures

1. **Password Security**
   - No password storage (OAuth only or passwordless)
   - Secure password reset flow
   - Rate limiting on authentication attempts

2. **Session Security**
   - JWT signing with strong secret
   - Short-lived access tokens
   - Secure cookie attributes (HttpOnly, Secure, SameSite)

3. **CSRF Protection**
   - Built-in CSRF protection
   - State parameter in OAuth flow

## 🛠️ Implementation Details

### Authentication Configuration

```typescript
// lib/auth.ts
export const {
  handlers: { GET, POST },
  auth,
  signIn,
  signOut,
} = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
    EmailProvider({
      server: {
        host: process.env.EMAIL_SERVER_HOST,
        port: process.env.EMAIL_SERVER_PORT,
        auth: {
          user: process.env.EMAIL_SERVER_USER,
          pass: process.env.EMAIL_SERVER_PASSWORD,
        },
      },
      from: process.env.EMAIL_FROM,
    }),
  ],
  session: {
    strategy: "jwt",
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
  callbacks: {
    async session({ session, token }) {
      if (token) {
        session.user.id = token.id;
        session.user.role = token.role;
      }
      return session;
    },
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },
  },
});
```

### Protecting API Routes

```typescript
// app/api/protected/route.ts
import { auth } from "@/lib/auth";

export const GET = auth((req) => {
  if (!req.auth) {
    return new Response("Unauthorized", { status: 401 });
  }
  
  // Check user role
  if (req.auth.user.role !== "ADMIN") {
    return new Response("Forbidden", { status: 403 });
  }
  
  // Handle request
  return Response.json({ data: "Protected data" });
});
```

### Protecting Pages

```typescript
// app/dashboard/page.tsx
import { auth } from "@/lib/auth";

export default async function DashboardPage() {
  const session = await auth();
  
  if (!session) {
    redirect("/login");
  }
  
  return (
    <div>
      <h1>Welcome, {session.user.name}!</h1>
      {/* Dashboard content */}
    </div>
  );
}
```

## 🔄 Social Login Integration

### GitHub OAuth Setup

1. Create a new OAuth App on GitHub
2. Set the Homepage URL to your application URL
3. Set the Authorization callback URL to `[your-app-url]/api/auth/callback/github`
4. Add the Client ID and Client Secret to your environment variables

### Email Provider Setup

1. Configure your email provider (e.g., Resend, SendGrid)
2. Add the SMTP credentials to your environment variables
3. Customize the email templates as needed

## 🔒 Role-Based Access Control (RBAC)

### Middleware Protection

```typescript
// middleware.ts
import { withAuth } from "next-auth/middleware";

export default withAuth({
  callbacks: {
    authorized: ({ token }) => {
      // Only allow access if user is authenticated and has admin role
      return token?.role === "ADMIN";
    },
  },
});

// Protect admin routes
export const config = {
  matcher: ["/admin/:path*"],
};
```

### Component-Level Protection

```tsx
// components/ProtectedComponent.tsx
"use client";

import { useSession } from "next-auth/react";

export default function ProtectedComponent() {
  const { data: session, status } = useSession();
  
  if (status === "loading") {
    return <div>Loading...</div>;
  }
  
  if (!session) {
    return <div>Please sign in to view this content</div>;
  }
  
  if (session.user.role !== "ADMIN") {
    return <div>You don't have permission to view this content</div>;
  }
  
  return (
    <div>
      <h2>Admin Dashboard</h2>
      {/* Admin content */}
    </div>
  );
}
```

## 🔄 Session Management

### Custom Session Handling

```typescript
// lib/auth-utils.ts
import { auth } from "./auth";

export async function getCurrentUser() {
  const session = await auth();
  return session?.user;
}

export async function requireAdmin() {
  const user = await getCurrentUser();
  if (!user || user.role !== "ADMIN") {
    throw new Error("Unauthorized");
  }
  return user;
}
```

### Force Sign Out

```typescript
// app/actions/auth.ts
"use server";

import { signOut } from "@/lib/auth";

export async function signOutAction() {
  await signOut({ redirectTo: "/login" });
}
```

## 🚨 Security Best Practices

1. **Environment Variables**
   - Never commit sensitive data to version control
   - Use strong, unique secrets for JWT signing
   - Rotate secrets regularly

2. **Rate Limiting**
   - Implement rate limiting on authentication endpoints
   - Use services like Upstash or Redis for distributed rate limiting

3. **Monitoring**
   - Log authentication attempts
   - Set up alerts for suspicious activities
   - Monitor for brute force attempts

4. **Compliance**
   - Follow OWASP authentication guidelines
   - Implement proper CORS policies
   - Regular security audits

## 🔗 Related Documentation

- [NextAuth.js Documentation](https://next-auth.js.org/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Prisma Adapter for NextAuth.js](https://authjs.dev/reference/adapter/prisma)

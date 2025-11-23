# Migrating from Clerk to WorkOS

## Overview
This guide walks through replacing Clerk (B2C/B2B authentication) with WorkOS (Enterprise B2B authentication) for next-forge applications.

**Estimated Time:** 4-6 hours
**Difficulty:** Medium
**Best For:** B2B SaaS targeting enterprise customers

---

## When to Use WorkOS vs Clerk

### Use WorkOS if:
- ✅ Building B2B SaaS for enterprise customers
- ✅ Need SAML/SSO (included in free tier)
- ✅ Need Directory Sync (SCIM)
- ✅ Want full control over auth UI
- ✅ Prioritize enterprise features over ease-of-use

### Keep Clerk if:
- ✅ Building B2C application
- ✅ Want pre-built UI components
- ✅ Need social logins (Google, GitHub, etc.)
- ✅ Smaller user base (< 10K MAUs)
- ✅ Want faster implementation

---

## Feature Comparison

| Feature | Clerk | WorkOS | Winner |
|---------|-------|--------|--------|
| **SSO (SAML, OIDC)** | Enterprise plan (~$100/mo) | ✅ Free tier (3 connections) | WorkOS |
| **Directory Sync** | Enterprise plan | ✅ Included | WorkOS |
| **Pre-built UI** | ✅ Excellent | ❌ Headless (DIY) | Clerk |
| **Social Logins** | ✅ Built-in | ⚠️ Via OAuth | Clerk |
| **Organizations** | ✅ Built-in | ✅ Built-in | Tie |
| **User Management UI** | ✅ Dashboard | ⚠️ API-based | Clerk |
| **Free Tier** | 10,000 MAUs | Unlimited users, 3 SSO | WorkOS (better) |
| **Pricing (Paid)** | $25/mo + $0.02/MAU | $125/mo (includes SSO) | Clerk (cheaper for < 5K users) |
| **Setup Time** | 30 minutes | 2-3 hours | Clerk |

---

## What Changes

### Before (Clerk)
```
apps/app/
  ├── Uses Clerk React components: <SignIn>, <SignUp>, <UserButton>
  ├── Uses Clerk hooks: useUser(), useAuth()
  └── Middleware: clerkMiddleware()

packages/auth/
  └── Clerk configuration and helpers

Environment:
  - NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
  - CLERK_SECRET_KEY
  - CLERK_WEBHOOK_SECRET
```

### After (WorkOS)
```
apps/app/
  ├── Custom auth UI (you build it)
  ├── WorkOS SDK: getAuthorizationUrl(), authenticate()
  └── Custom middleware for session management

packages/auth/
  └── WorkOS configuration and session helpers

Environment:
  - WORKOS_API_KEY
  - WORKOS_CLIENT_ID
  - NEXT_PUBLIC_WORKOS_CLIENT_ID
```

---

## Phase 1: Setup WorkOS (30 min)

### Step 1: Create WorkOS Account

1. Sign up at https://workos.com
2. Create new organization
3. Note your API credentials:
   - API Key
   - Client ID

**Acceptance Criteria:**
- ✅ WorkOS account created
- ✅ API Key visible in dashboard
- ✅ Client ID visible in dashboard

---

### Step 2: Install WorkOS SDK

```bash
cd apps/app
npm install @workos-inc/node
```

Update `apps/app/package.json`:
```json
{
  "dependencies": {
    "@workos-inc/node": "^7.0.0"
  }
}
```

---

### Step 3: Configure Environment Variables

**Remove (Clerk):**
```bash
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
CLERK_SECRET_KEY
CLERK_WEBHOOK_SECRET
```

**Add (WorkOS):**
```bash
# In apps/app/.env.local
WORKOS_API_KEY=sk_live_...
WORKOS_CLIENT_ID=client_...
NEXT_PUBLIC_WORKOS_CLIENT_ID=client_...
WORKOS_REDIRECT_URI=http://localhost:3000/auth/callback
```

**Production:**
```bash
WORKOS_REDIRECT_URI=https://app.yourdomain.com/auth/callback
```

---

## Phase 2: Build Auth UI (2-3 hours)

### Step 4: Create Sign-In Page

Unlike Clerk's `<SignIn />` component, you must build your own UI.

**Create `apps/app/app/sign-in/page.tsx`:**

```typescript
'use client';

import { useState } from 'react';
import { Button } from '@repo/design-system/components/button';
import { Input } from '@repo/design-system/components/input';

export default function SignInPage() {
  const [email, setEmail] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSignIn = async () => {
    setLoading(true);

    // For email/password authentication
    const res = await fetch('/api/auth/sign-in', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    });

    const { url } = await res.json();

    // Redirect to WorkOS authorization
    window.location.href = url;
  };

  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="w-full max-w-md p-8 space-y-6">
        <h1 className="text-2xl font-bold">Sign In</h1>

        <Input
          type="email"
          placeholder="Enter your email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />

        <Button onClick={handleSignIn} disabled={loading}>
          {loading ? 'Loading...' : 'Sign In'}
        </Button>

        {/* SSO Options */}
        <div className="space-y-2">
          <p className="text-sm text-gray-600">Or sign in with SSO:</p>
          <Button variant="outline" onClick={() => handleSSOSignIn('google')}>
            Sign in with Google SSO
          </Button>
        </div>
      </div>
    </div>
  );
}
```

**Create `apps/app/app/api/auth/sign-in/route.ts`:**

```typescript
import { WorkOS } from '@workos-inc/node';
import { NextResponse } from 'next/server';

const workos = new WorkOS(process.env.WORKOS_API_KEY!);

export async function POST(req: Request) {
  const { email } = await req.json();

  // Get authorization URL
  const authorizationUrl = workos.userManagement.getAuthorizationUrl({
    provider: 'authkit', // or specific provider like 'google-oauth'
    clientId: process.env.WORKOS_CLIENT_ID!,
    redirectUri: process.env.WORKOS_REDIRECT_URI!,
    state: '', // Optional state for CSRF protection
  });

  return NextResponse.json({ url: authorizationUrl });
}
```

---

### Step 5: Create Auth Callback Handler

**Create `apps/app/app/auth/callback/route.ts`:**

```typescript
import { WorkOS } from '@workos-inc/node';
import { NextResponse } from 'next/server';
import { cookies } from 'next/headers';

const workos = new WorkOS(process.env.WORKOS_API_KEY!);

export async function GET(req: Request) {
  const url = new URL(req.url);
  const code = url.searchParams.get('code');

  if (!code) {
    return NextResponse.redirect('/sign-in?error=no_code');
  }

  try {
    // Exchange code for user profile
    const { user, accessToken, refreshToken } = await workos.userManagement.authenticateWithCode({
      code,
      clientId: process.env.WORKOS_CLIENT_ID!,
    });

    // Create session (you manage this!)
    const sessionId = await createSession(user.id, accessToken, refreshToken);

    // Set session cookie
    cookies().set('session', sessionId, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 60 * 60 * 24 * 7, // 7 days
    });

    // Redirect to dashboard
    return NextResponse.redirect(new URL('/dashboard', req.url));
  } catch (error) {
    console.error('Auth error:', error);
    return NextResponse.redirect('/sign-in?error=auth_failed');
  }
}

// Session management (you implement this)
async function createSession(userId: string, accessToken: string, refreshToken: string) {
  // Store in your database or Redis
  // Return session ID
  return 'session_id';
}
```

---

### Step 6: Build Session Management

**Unlike Clerk, WorkOS doesn't manage sessions - you do!**

**Create `packages/auth/session.ts`:**

```typescript
import { cookies } from 'next/headers';

interface Session {
  userId: string;
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
}

// Store sessions in database or Redis
const sessions = new Map<string, Session>();

export async function createSession(data: Session): Promise<string> {
  const sessionId = crypto.randomUUID();
  sessions.set(sessionId, data);
  return sessionId;
}

export async function getSession(): Promise<Session | null> {
  const sessionId = cookies().get('session')?.value;
  if (!sessionId) return null;

  const session = sessions.get(sessionId);
  if (!session) return null;

  // Check if expired
  if (Date.now() > session.expiresAt) {
    sessions.delete(sessionId);
    return null;
  }

  return session;
}

export async function destroySession(sessionId: string): Promise<void> {
  sessions.delete(sessionId);
  cookies().delete('session');
}
```

---

### Step 7: Create Middleware for Auth Protection

**Update `apps/app/middleware.ts`:**

```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(req: NextRequest) {
  const session = req.cookies.get('session');

  // Protected routes
  const protectedPaths = ['/dashboard', '/settings', '/profile'];
  const isProtectedPath = protectedPaths.some(path => req.nextUrl.pathname.startsWith(path));

  if (isProtectedPath && !session) {
    return NextResponse.redirect(new URL('/sign-in', req.url));
  }

  // Redirect authenticated users away from auth pages
  const authPaths = ['/sign-in', '/sign-up'];
  const isAuthPath = authPaths.some(path => req.nextUrl.pathname.startsWith(path));

  if (isAuthPath && session) {
    return NextResponse.redirect(new URL('/dashboard', req.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

---

## Phase 3: Implement SSO (1-2 hours)

### Step 8: Configure SSO Connections

**In WorkOS Dashboard:**

1. Navigate to "Connections"
2. Click "Add Connection"
3. Select provider (Google, Microsoft, Okta, etc.)
4. Configure according to provider's requirements
5. Note the connection ID

**SSO Sign-In Flow:**

```typescript
// In sign-in page, add SSO button
const handleSSOSignIn = async (provider: string) => {
  const res = await fetch('/api/auth/sso', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ provider }),
  });

  const { url } = await res.json();
  window.location.href = url;
};

// API route for SSO
export async function POST(req: Request) {
  const { provider } = await req.json();

  const authorizationUrl = workos.sso.getAuthorizationUrl({
    provider, // 'google-oauth', 'microsoft-oauth', etc.
    clientId: process.env.WORKOS_CLIENT_ID!,
    redirectUri: process.env.WORKOS_REDIRECT_URI!,
  });

  return NextResponse.json({ url: authorizationUrl });
}
```

---

## Phase 4: User Migration (2-3 hours)

### Step 9: Export Users from Clerk

```bash
# Use Clerk API to export users
curl https://api.clerk.com/v1/users \
  -H "Authorization: Bearer ${CLERK_SECRET_KEY}" \
  > clerk-users.json
```

**Export script:**
```javascript
// scripts/export-clerk-users.js
const fetch = require('node-fetch');
const fs = require('fs');

async function exportUsers() {
  const response = await fetch('https://api.clerk.com/v1/users', {
    headers: {
      'Authorization': `Bearer ${process.env.CLERK_SECRET_KEY}`,
    },
  });

  const users = await response.json();
  fs.writeFileSync('clerk-users.json', JSON.stringify(users, null, 2));
}

exportUsers();
```

---

### Step 10: Import Users to WorkOS

```javascript
// scripts/import-to-workos.js
const { WorkOS } = require('@workos-inc/node');
const fs = require('fs');

const workos = new WorkOS(process.env.WORKOS_API_KEY);

async function importUsers() {
  const clerkUsers = JSON.parse(fs.readFileSync('clerk-users.json'));

  for (const user of clerkUsers) {
    await workos.userManagement.createUser({
      email: user.email_addresses[0].email_address,
      firstName: user.first_name,
      lastName: user.last_name,
    });
  }
}

importUsers();
```

**Important:** Users will need to reset passwords (WorkOS can't import encrypted passwords)

---

## Phase 5: Replace Clerk Components (1-2 hours)

### Step 11: Replace Clerk Hooks

**Before (Clerk):**
```typescript
import { useUser, useAuth } from '@clerk/nextjs';

export default function Dashboard() {
  const { user } = useUser();
  const { signOut } = useAuth();

  return (
    <div>
      <p>Welcome, {user?.firstName}!</p>
      <button onClick={() => signOut()}>Sign Out</button>
    </div>
  );
}
```

**After (WorkOS):**
```typescript
'use client';

import { useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';

export default function Dashboard() {
  const [user, setUser] = useState<any>(null);
  const router = useRouter();

  useEffect(() => {
    fetch('/api/auth/me')
      .then(res => res.json())
      .then(setUser);
  }, []);

  const handleSignOut = async () => {
    await fetch('/api/auth/sign-out', { method: 'POST' });
    router.push('/sign-in');
  };

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <p>Welcome, {user.firstName}!</p>
      <button onClick={handleSignOut}>Sign Out</button>
    </div>
  );
}

// API route for /api/auth/me
export async function GET() {
  const session = await getSession();
  if (!session) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });

  const user = await workos.userManagement.getUser(session.userId);
  return NextResponse.json(user);
}
```

---

### Step 12: Replace Organization Management

**Clerk Organizations → WorkOS Organizations**

```typescript
// Create organization
const org = await workos.organizations.createOrganization({
  name: 'Acme Corp',
  domains: ['acme.com'],
});

// Add user to organization
await workos.organizations.createOrganizationMembership({
  organizationId: org.id,
  userId: user.id,
  roleSlug: 'admin',
});

// List user's organizations
const orgs = await workos.organizations.listOrganizations({
  userId: user.id,
});
```

---

## Phase 6: Webhooks (30 min)

### Step 13: Configure WorkOS Webhooks

**In WorkOS Dashboard:**
1. Navigate to Webhooks
2. Add endpoint: `https://yourapp.com/api/webhooks/workos`
3. Subscribe to events:
   - `user.created`
   - `user.updated`
   - `user.deleted`
4. Copy webhook secret

**Create webhook handler:**
```typescript
// apps/app/app/api/webhooks/workos/route.ts
import { WorkOS } from '@workos-inc/node';
import { NextResponse } from 'next/server';

const workos = new WorkOS(process.env.WORKOS_API_KEY!);

export async function POST(req: Request) {
  const payload = await req.text();
  const signature = req.headers.get('workos-signature');

  try {
    const webhook = workos.webhooks.constructEvent({
      payload,
      signature: signature!,
      secret: process.env.WORKOS_WEBHOOK_SECRET!,
    });

    switch (webhook.event) {
      case 'user.created':
        // Handle user creation
        break;
      case 'user.updated':
        // Handle user update
        break;
      case 'user.deleted':
        // Handle user deletion
        break;
    }

    return NextResponse.json({ received: true });
  } catch (error) {
    return NextResponse.json({ error: 'Webhook signature verification failed' }, { status: 400 });
  }
}
```

---

## Phase 7: Cleanup (30 min)

### Step 14: Remove Clerk Dependencies

```bash
# Remove packages
pnpm remove @clerk/nextjs

# Delete Clerk-specific files
rm -rf apps/app/app/(unauthenticated)/sign-in/[[...sign-in]]
rm -rf apps/app/app/(unauthenticated)/sign-up/[[...sign-up]]

# Update package.json scripts
# Remove any Clerk-specific scripts
```

### Step 15: Update Environment Variables

**Remove from Vercel:**
- ❌ `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
- ❌ `CLERK_SECRET_KEY`
- ❌ `CLERK_WEBHOOK_SECRET`

**Add to Vercel:**
- ✅ `WORKOS_API_KEY`
- ✅ `WORKOS_CLIENT_ID`
- ✅ `NEXT_PUBLIC_WORKOS_CLIENT_ID`
- ✅ `WORKOS_REDIRECT_URI`
- ✅ `WORKOS_WEBHOOK_SECRET`

---

## Testing Checklist

- ✅ Sign-in flow works
- ✅ Sign-up flow works (if applicable)
- ✅ SSO login works
- ✅ Session persists across page reloads
- ✅ Sign-out works
- ✅ Middleware protects routes correctly
- ✅ User data displays correctly
- ✅ Webhooks fire and process correctly
- ✅ Organizations work (if using)
- ✅ No Clerk references in code

---

## Cost Comparison

### Clerk Pricing
- Free: 10,000 MAUs
- Pro: $25/mo + $0.02/MAU
- Enterprise SSO: +$99/mo

**Example (10K users with SSO):**
- Pro: $25 + (10,000 * $0.02) = $225/mo
- Enterprise: $225 + $99 = $324/mo

### WorkOS Pricing
- Free: Unlimited users, 3 SSO connections
- Growth: $125/mo (includes SSO, Directory Sync)

**Example (10K users with SSO):**
- Free tier: $0 (if ≤3 SSO connections)
- Growth: $125/mo

**Savings:** $199-324/mo with WorkOS

---

## Common Pitfalls

1. **Session Management:** WorkOS is headless - you must handle sessions yourself
2. **UI Components:** No pre-built components - build everything
3. **User Migration:** Can't import passwords - users must reset
4. **Social Logins:** Requires more setup than Clerk
5. **Local Development:** Need to configure localhost redirects

---

## Rollback Plan

If migration fails:

1. Keep Clerk packages installed during transition
2. Use feature flag to toggle between Clerk/WorkOS
3. Run both authentication systems in parallel
4. Don't delete Clerk users until fully migrated
5. Test thoroughly in staging first

---

## Support Resources

- **WorkOS Docs:** https://workos.com/docs
- **WorkOS SDKs:** https://workos.com/docs/sdks
- **SSO Setup:** https://workos.com/docs/sso
- **User Management:** https://workos.com/docs/user-management

---

## Recommendation

**Migrate to WorkOS if:**
- Building B2B SaaS for enterprises
- Need SSO without paying extra
- Comfortable building custom auth UI
- Want more control over auth flow

**Keep Clerk if:**
- Building B2C or small B2B
- Want faster implementation
- Prefer pre-built components
- Don't need enterprise SSO

For most startups: **Start with Clerk, migrate to WorkOS when you have enterprise customers.**

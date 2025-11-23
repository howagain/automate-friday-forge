# Migrating from Prisma + Neon to Convex

## Overview
This guide walks through replacing Prisma (ORM) + Neon (PostgreSQL) + Custom API routes with Convex, which provides database, backend functions, and real-time subscriptions in one service.

**Estimated Time:** 8-12 hours for typical application
**Difficulty:** Medium-Hard
**Prerequisites:**
- Familiarity with TypeScript
- Understanding of current Prisma schema
- Convex account (sign up at convex.dev)

---

## What Changes

### Before (Prisma + Neon)
```
apps/app/ (Frontend)
  └── Uses Prisma Client to query database

packages/database/
  ├── prisma/schema.prisma (Schema definition)
  └── Generated Prisma Client

apps/api/ (Backend API)
  └── Custom API routes for business logic

Neon PostgreSQL (Cloud database)
```

### After (Convex)
```
apps/app/ (Frontend)
  └── Uses Convex React hooks (useQuery, useMutation)

convex/ (Backend + Database)
  ├── schema.ts (Schema definition)
  ├── queries.ts (Read operations)
  ├── mutations.ts (Write operations)
  └── actions.ts (External API calls)

Convex Cloud (Database + Backend runtime)

❌ REMOVED: packages/database/, apps/api/
```

---

## Phase 1: Setup Convex (30 min)

### Step 1: Install Convex

```bash
# In project root
npm install convex

# Initialize Convex
npx convex dev

# This will:
# 1. Create convex/ directory
# 2. Create convex.json config
# 3. Prompt you to login/create account
# 4. Create a new Convex project
```

**Acceptance Criteria:**
- ✅ `convex/` directory created
- ✅ `convex.json` exists with deployment URL
- ✅ `.env.local` has `CONVEX_DEPLOYMENT` variable
- ✅ Convex dashboard shows your project

---

### Step 2: Configure Next.js for Convex

Update `apps/app/next.config.ts`:

```typescript
import { withConvex } from "@convex-dev/next";

const config = {
  // ... existing config
};

export default withConvex(config);
```

Update `apps/app/app/layout.tsx`:

```typescript
import { ConvexProvider, ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <ConvexProvider client={convex}>
          {children}
        </ConvexProvider>
      </body>
    </html>
  );
}
```

**Acceptance Criteria:**
- ✅ Convex provider wraps application
- ✅ No TypeScript errors
- ✅ `NEXT_PUBLIC_CONVEX_URL` in environment variables

---

## Phase 2: Migrate Database Schema (2-3 hours)

### Step 3: Convert Prisma Schema to Convex

**Prisma Example:**
```prisma
// packages/database/prisma/schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  posts     Post[]
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
}
```

**Convex Equivalent:**
```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    email: v.string(),
    name: v.optional(v.string()),
    createdAt: v.number(), // Unix timestamp
  }).index("by_email", ["email"]),

  posts: defineTable({
    title: v.string(),
    content: v.optional(v.string()),
    published: v.boolean(),
    authorId: v.id("users"), // Foreign key to users table
    createdAt: v.number(),
  }).index("by_author", ["authorId"]),
});
```

### Key Differences:

| Feature | Prisma | Convex |
|---------|--------|--------|
| **Primary Key** | `@id @default(uuid())` | Auto-generated `_id` |
| **Foreign Keys** | `@relation` | `v.id("tableName")` |
| **Timestamps** | `DateTime` | `v.number()` (Unix ms) |
| **Optional Fields** | `?` | `v.optional()` |
| **Unique Constraints** | `@unique` | `.index()` |
| **Enums** | `enum` | `v.union()` or `v.string()` |

### Migration Checklist:

- ✅ Create `convex/schema.ts`
- ✅ Convert all Prisma models to Convex tables
- ✅ Add indexes for frequently queried fields
- ✅ Update optional fields to use `v.optional()`
- ✅ Convert relations to `v.id()` references
- ✅ Deploy schema: `npx convex dev` (auto-deploys)

---

## Phase 3: Migrate Queries (3-4 hours)

### Step 4: Convert Prisma Queries to Convex Queries

**Example: Get all users**

**Before (Prisma):**
```typescript
// In API route or server component
import { prisma } from '@repo/database';

const users = await prisma.user.findMany({
  where: { published: true },
  include: { posts: true },
  orderBy: { createdAt: 'desc' },
});
```

**After (Convex):**
```typescript
// convex/users.ts
import { query } from "./_generated/server";
import { v } from "convex/values";

export const getUsers = query({
  args: {},
  handler: async (ctx) => {
    const users = await ctx.db.query("users").collect();
    return users;
  },
});

// In React component:
import { useQuery } from "convex/react";
import { api } from "../convex/_generated/api";

const users = useQuery(api.users.getUsers);
```

---

### Common Query Patterns

#### 1. Find by ID
**Prisma:**
```typescript
const user = await prisma.user.findUnique({
  where: { id: userId }
});
```

**Convex:**
```typescript
export const getUser = query({
  args: { id: v.id("users") },
  handler: async (ctx, args) => {
    return await ctx.db.get(args.id);
  },
});
```

---

#### 2. Find with Filter
**Prisma:**
```typescript
const posts = await prisma.post.findMany({
  where: { authorId: userId, published: true }
});
```

**Convex:**
```typescript
export const getPublishedPostsByAuthor = query({
  args: { authorId: v.id("users") },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("posts")
      .withIndex("by_author", (q) => q.eq("authorId", args.authorId))
      .filter((q) => q.eq(q.field("published"), true))
      .collect();
  },
});
```

---

#### 3. Relations (Join)
**Prisma:**
```typescript
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: { posts: true }
});
```

**Convex:**
```typescript
export const getUserWithPosts = query({
  args: { userId: v.id("users") },
  handler: async (ctx, args) => {
    const user = await ctx.db.get(args.userId);
    if (!user) return null;

    const posts = await ctx.db
      .query("posts")
      .withIndex("by_author", (q) => q.eq("authorId", args.userId))
      .collect();

    return { ...user, posts };
  },
});
```

---

## Phase 4: Migrate Mutations (2-3 hours)

### Step 5: Convert Prisma Writes to Convex Mutations

**Example: Create user**

**Before (Prisma):**
```typescript
const newUser = await prisma.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
  },
});
```

**After (Convex):**
```typescript
// convex/users.ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const createUser = mutation({
  args: {
    email: v.string(),
    name: v.optional(v.string()),
  },
  handler: async (ctx, args) => {
    const userId = await ctx.db.insert("users", {
      email: args.email,
      name: args.name,
      createdAt: Date.now(),
    });
    return userId;
  },
});

// In React component:
import { useMutation } from "convex/react";
import { api } from "../convex/_generated/api";

const createUser = useMutation(api.users.createUser);

// Call it:
await createUser({ email: 'user@example.com', name: 'John Doe' });
```

---

### Common Mutation Patterns

#### 1. Update
**Prisma:**
```typescript
await prisma.user.update({
  where: { id: userId },
  data: { name: 'Jane Doe' }
});
```

**Convex:**
```typescript
export const updateUser = mutation({
  args: {
    id: v.id("users"),
    name: v.string(),
  },
  handler: async (ctx, args) => {
    await ctx.db.patch(args.id, { name: args.name });
  },
});
```

---

#### 2. Delete
**Prisma:**
```typescript
await prisma.user.delete({
  where: { id: userId }
});
```

**Convex:**
```typescript
export const deleteUser = mutation({
  args: { id: v.id("users") },
  handler: async (ctx, args) => {
    await ctx.db.delete(args.id);
  },
});
```

---

## Phase 5: Migrate API Routes to Convex Actions (2-3 hours)

### Step 6: Replace Next.js API Routes

**Use Case:** External API calls, sending emails, processing payments

**Before (Next.js API Route):**
```typescript
// apps/api/app/webhooks/stripe/route.ts
export async function POST(req: Request) {
  const body = await req.json();

  // Process Stripe webhook
  const session = body.data.object;

  // Update database
  await prisma.subscription.create({
    data: {
      userId: session.metadata.userId,
      stripeSessionId: session.id,
    },
  });

  return Response.json({ received: true });
}
```

**After (Convex Action):**
```typescript
// convex/stripe.ts
import { action } from "./_generated/server";
import { v } from "convex/values";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export const handleStripeWebhook = action({
  args: { sessionId: v.string() },
  handler: async (ctx, args) => {
    // Verify webhook signature here

    const session = await stripe.checkout.sessions.retrieve(args.sessionId);

    // Call internal mutation to update database
    await ctx.runMutation(api.subscriptions.create, {
      userId: session.metadata!.userId,
      stripeSessionId: session.id,
    });

    return { received: true };
  },
});
```

**Key Differences:**
- ✅ Actions can call external APIs (queries/mutations cannot)
- ✅ Actions can call mutations using `ctx.runMutation()`
- ✅ Actions run in Convex cloud (not your Next.js server)

---

## Phase 6: Add Real-time with Subscriptions (1 hour)

### Step 7: Replace Liveblocks with Convex Subscriptions

**Before (Liveblocks):**
```typescript
// Lots of setup code for Liveblocks...
```

**After (Convex - FREE!):**
```typescript
// convex/messages.ts
export const getMessages = query({
  args: { roomId: v.string() },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_room", (q) => q.eq("roomId", args.roomId))
      .order("desc")
      .collect();
  },
});

// In React component - automatically updates!
const messages = useQuery(api.messages.getMessages, { roomId: "room-1" });
```

**That's it!** Convex handles real-time automatically - no extra setup needed.

---

## Phase 7: Data Migration (2-4 hours)

### Step 8: Migrate Existing Data from Neon to Convex

**Option A: Export/Import (Recommended for small datasets)**

```bash
# 1. Export from Neon
npx prisma db pull
npx prisma generate
node scripts/export-data.js

# 2. Import to Convex
node scripts/import-to-convex.js
```

**Example export script:**
```javascript
// scripts/export-data.js
const { PrismaClient } = require('@prisma/client');
const fs = require('fs');

const prisma = new PrismaClient();

async function exportData() {
  const users = await prisma.user.findMany();
  const posts = await prisma.post.findMany();

  fs.writeFileSync('data/users.json', JSON.stringify(users, null, 2));
  fs.writeFileSync('data/posts.json', JSON.stringify(posts, null, 2));
}

exportData();
```

**Example import script:**
```javascript
// scripts/import-to-convex.js
const { ConvexHttpClient } = require("convex/browser");
const fs = require('fs');

const client = new ConvexHttpClient(process.env.CONVEX_URL);

async function importData() {
  const users = JSON.parse(fs.readFileSync('data/users.json'));
  const posts = JSON.parse(fs.readFileSync('data/posts.json'));

  // Import users
  for (const user of users) {
    await client.mutation(api.users.createUser, {
      email: user.email,
      name: user.name,
    });
  }

  // Import posts
  for (const post of posts) {
    await client.mutation(api.posts.createPost, {
      title: post.title,
      content: post.content,
      authorId: post.authorId, // Map old ID to new Convex ID
    });
  }
}

importData();
```

**Option B: Dual-write (For production migrations)**

1. Keep both Prisma and Convex temporarily
2. Write to both databases
3. Verify data consistency
4. Switch reads to Convex
5. Remove Prisma

---

## Phase 8: Cleanup (30 min)

### Step 9: Remove Old Code

```bash
# Remove packages
pnpm remove prisma @prisma/client

# Delete directories
rm -rf packages/database
rm -rf apps/api

# Update package.json scripts
# Remove: "migrate", "db:push", etc.
```

### Step 10: Update Environment Variables

**Remove:**
- ❌ `DATABASE_URL`
- ❌ Any Neon-specific variables

**Add:**
- ✅ `CONVEX_DEPLOYMENT` (auto-added by `npx convex dev`)
- ✅ `NEXT_PUBLIC_CONVEX_URL` (auto-added)

---

## Testing Checklist

Before deploying:

- ✅ All queries return expected data
- ✅ All mutations work correctly
- ✅ Real-time updates working
- ✅ Authentication integrated with Convex
- ✅ File uploads working (if applicable)
- ✅ Data migrated successfully
- ✅ No references to Prisma in codebase
- ✅ Local development works (`npx convex dev`)
- ✅ Production deployment works

---

## Common Pitfalls

### 1. Timestamp Format
**Issue:** Prisma uses `DateTime`, Convex uses Unix milliseconds
**Solution:**
```typescript
// In Convex
createdAt: Date.now()

// Display in React
new Date(createdAt).toLocaleDateString()
```

### 2. Primary Keys
**Issue:** Prisma UUIDs vs Convex auto-generated IDs
**Solution:** Map old IDs during migration, use Convex IDs going forward

### 3. Relations
**Issue:** Prisma auto-handles relations, Convex requires manual joins
**Solution:** Create query functions that fetch related data

### 4. Transactions
**Issue:** Prisma transactions not directly translatable
**Solution:** Use Convex mutations (atomic by default) or `ctx.runMutation()` chains

---

## Deployment

### Development
```bash
npx convex dev
```

### Production
```bash
npx convex deploy
```

Environment variables in Vercel:
- `CONVEX_DEPLOYMENT=prod:your-deployment`
- `NEXT_PUBLIC_CONVEX_URL=https://your-deployment.convex.cloud`

---

## Performance Comparison

| Operation | Prisma + Neon | Convex | Winner |
|-----------|---------------|--------|--------|
| Simple query | ~50-100ms | ~20-50ms | ✅ Convex |
| Complex query | ~100-300ms | ~50-150ms | ✅ Convex |
| Write | ~50-100ms | ~30-80ms | ✅ Convex |
| Real-time | Need Liveblocks (~$49/mo) | Included | ✅ Convex |
| Cold start | N/A (database) | ~100ms | Prisma |

---

## Cost Comparison

| Tier | Prisma + Neon | Convex |
|------|---------------|--------|
| **Free** | 0.5 GB storage, 10 projects | 1M function calls/month, generous storage |
| **Paid** | $19/mo + compute | $25/mo |
| **Scale** | Pay per compute | $25/mo fixed until high scale |

---

## Support Resources

- **Convex Docs:** https://docs.convex.dev
- **Convex Discord:** https://convex.dev/community
- **Migration Guide:** https://docs.convex.dev/database/migrations
- **Schema Design:** https://docs.convex.dev/database/schemas

---

## Rollback Plan

If migration fails:

1. Keep `packages/database` until fully migrated
2. Use feature flags to toggle between Prisma and Convex
3. Run both in parallel during transition
4. Have database backup before migration
5. Test in staging environment first

---

## Next Steps

1. Review this guide with your team
2. Set up Convex account
3. Migrate schema first (test with empty database)
4. Migrate one feature at a time
5. Test thoroughly
6. Migrate data last
7. Remove Prisma/Neon dependencies

**Remember:** This is a significant architectural change. Take your time, test thoroughly, and migrate incrementally!

# Component Swap Matrix

## Overview
This document evaluates each service in the next-forge template and identifies swap candidates. Focus areas: Convex (database) and WorkOS (authentication).

---

## Priority Swaps (High Impact)

### 🔥 1. Database: Prisma + Neon → Convex

| Aspect | Current (Prisma + Neon) | Alternative (Convex) | Advantage |
|--------|-------------------------|----------------------|-----------|
| **Type** | PostgreSQL ORM + Serverless Postgres | Real-time database + backend | Convex = all-in-one |
| **Schema** | Prisma schema language | TypeScript-based schema | Native TypeScript |
| **Queries** | Prisma Client (SQL-like) | Convex queries (TypeScript functions) | Type-safe, no SQL |
| **Real-time** | Requires separate service (Liveblocks) | Built-in subscriptions | Simpler stack |
| **Backend Logic** | Separate API routes | Convex functions (queries/mutations/actions) | Co-located with data |
| **Migrations** | Manual Prisma migrations | Automatic schema migrations | Less manual work |
| **Local Dev** | Need local Postgres OR cloud DB | Works offline with local backend | Better DX |
| **Cost (Free Tier)** | Neon: 0.5GB | Convex: Generous free tier | Similar |
| **Cost (Paid)** | Neon: $19/mo + compute | Convex: $25/mo | Comparable |
| **Scaling** | Manual connection pooling | Auto-scales | Easier |
| **Learning Curve** | Prisma = moderate, SQL knowledge helps | Convex = steeper initially | Trade-off |

**Difficulty:** ⚠️ **Medium-Hard** (requires rewriting data layer)
**Worth It:** ✅ **YES** - Simplifies stack significantly, eliminates API layer
**Effort:** 8-12 hours for typical app
**Blockers:**
- Need to rewrite all Prisma queries as Convex functions
- Need to migrate schema
- Lose some PostgreSQL-specific features

**Recommendation:** **DO THIS SWAP** - Convex eliminates the need for separate API routes and provides real-time out of the box.

---

### 🔥 2. Authentication: Clerk → WorkOS

| Aspect | Current (Clerk) | Alternative (WorkOS) | Advantage |
|--------|-----------------|----------------------|-----------|
| **Focus** | B2C + B2B | B2B Enterprise | WorkOS better for SaaS |
| **SSO** | Add-on ($99/mo) | Included in free tier | Cost savings |
| **User Management** | Built-in UI components | Headless (build your own) | More control vs faster setup |
| **Organizations** | Included | Included | Tie |
| **Directory Sync** | Enterprise plan | Included | WorkOS wins |
| **MFA** | Included | Included | Tie |
| **Pricing (Free)** | 10,000 MAUs | Unlimited users, 3 SSO connections | WorkOS better value |
| **Pricing (Paid)** | $25/mo + $0.02/MAU | $125/mo | Clerk cheaper for small scale |
| **DX** | Excellent (drop-in components) | Good (more manual setup) | Clerk easier |
| **Customization** | Limited | Full control | WorkOS more flexible |

**Difficulty:** ⚠️ **Medium** (need to rebuild auth UI)
**Worth It:** ✅ **YES IF** - Building B2B SaaS with enterprise customers
**Worth It:** ❌ **NO IF** - Building B2C or small B2B
**Effort:** 4-6 hours
**Blockers:**
- No pre-built UI components (need to build login/signup)
- Migration of existing users

**Recommendation:** **CONDITIONAL** - If targeting enterprise B2B SaaS, WorkOS is better. Otherwise, keep Clerk.

---

## Medium Priority Swaps

### 3. Real-time Collaboration: Liveblocks → (Eliminated by Convex)

| Aspect | Current | With Convex | Advantage |
|--------|---------|-------------|-----------|
| **Service** | Liveblocks ($0-49/mo) | Convex built-in subscriptions | No extra service |
| **Features** | Rich collaboration primitives | Basic subscriptions | Liveblocks has more features |
| **Use Case** | Complex collaboration (cursors, presence) | Simple real-time updates | Depends on needs |

**Difficulty:** ⚠️ **Medium** (if using complex features)
**Worth It:** ✅ **YES** - If you're swapping to Convex AND only need simple real-time
**Worth It:** ❌ **NO** - If you need advanced collaboration features (keep Liveblocks)
**Effort:** 2-4 hours
**Recommendation:** **EVALUATE AFTER CONVEX SWAP** - Convex may cover your real-time needs.

---

### 4. Email: Resend → Alternative (SendGrid, Postmark, AWS SES)

| Aspect | Resend | SendGrid | Postmark | AWS SES |
|--------|--------|----------|----------|---------|
| **Free Tier** | 100/day | 100/day | 100/mo | 62K/mo (with EC2) |
| **Paid** | $20/mo (50K) | $20/mo (40K) | $15/mo (10K) | Pay-per-email |
| **DX** | Excellent | Good | Excellent | Complex |
| **Templates** | React Email (built-in) | Handlebars | Templates | DIY |

**Difficulty:** ✅ **Easy** (just swap API)
**Worth It:** ❌ **NO** - Resend is already excellent
**Effort:** 1-2 hours
**Recommendation:** **KEEP RESEND** - Best DX, fair pricing

---

### 5. Payments: Stripe → Alternative (Paddle, Lemon Squeezy)

| Aspect | Stripe | Paddle | Lemon Squeezy |
|--------|--------|--------|---------------|
| **Fees** | 2.9% + 30¢ | 5% + 50¢ | 5% + 50¢ |
| **Merchant of Record** | No (you are) | Yes | Yes |
| **Tax Handling** | DIY / Stripe Tax (extra) | Included | Included |
| **DX** | Excellent | Good | Good |

**Difficulty:** ⚠️ **Hard** (complete rewrite)
**Worth It:** ✅ **YES IF** - You want merchant-of-record (no tax headaches)
**Worth It:** ❌ **NO IF** - You're OK handling taxes
**Effort:** 12-16 hours
**Recommendation:** **KEEP STRIPE** - Unless tax handling is a major concern

---

## Low Priority Swaps (Not Worth It)

### 6. Error Tracking: Sentry → Alternatives

**Alternatives:** LogRocket, Rollbar, Bugsnag
**Recommendation:** **KEEP SENTRY** - Industry standard, excellent DX

### 7. Analytics: PostHog + Google Analytics → Alternatives

**Alternatives:** Mixpanel, Amplitude, Heap
**Recommendation:** **KEEP BOTH** - PostHog is open-source friendly, GA is ubiquitous

### 8. CMS: BaseHub → Alternatives

**Alternatives:** Contentful, Sanity, Payload CMS, Strapi
**Recommendation:** **EVALUATE** - BaseHub is new, consider more established CMSs

### 9. Logging: BetterStack → Alternatives

**Alternatives:** Datadog, New Relic, Papertrail
**Recommendation:** **KEEP BETTERSTACK** - Good value, simple setup

### 10. Security: Arcjet → Alternatives

**Alternatives:** Cloudflare, Upstash Rate Limit
**Recommendation:** **KEEP ARCJET** - Purpose-built for Next.js

### 11. Webhooks: Svix → Alternatives

**Alternatives:** Hookdeck, DIY
**Recommendation:** **KEEP SVIX** - Best webhook infrastructure

### 12. Notifications: Knock → Alternatives

**Alternatives:** Novu (open-source), DIY
**Recommendation:** **EVALUATE** - Novu is open-source alternative worth considering

---

## Swap Recommendations Summary

| Component | Current | Recommended Swap | Priority | Effort | Rationale |
|-----------|---------|------------------|----------|--------|-----------|
| **Database + Backend** | Prisma + Neon | ✅ Convex | 🔥 **HIGH** | 8-12h | Simplifies stack, adds real-time, better DX |
| **Authentication** | Clerk | ⚠️ WorkOS (conditional) | 🟡 **MEDIUM** | 4-6h | Only if B2B enterprise focus |
| **Real-time** | Liveblocks | ✅ Remove (use Convex) | 🟡 **MEDIUM** | 2-4h | If Convex covers your needs |
| **Payments** | Stripe | ❌ Keep | 🟢 **LOW** | - | Stripe is best |
| **Email** | Resend | ❌ Keep | 🟢 **LOW** | - | Resend is excellent |
| **Monitoring** | Sentry, BetterStack, PostHog | ❌ Keep | 🟢 **LOW** | - | Good stack |
| **CMS** | BaseHub | ⚠️ Consider Sanity/Contentful | 🟡 **MEDIUM** | 4-6h | More established options |
| **Notifications** | Knock | ⚠️ Consider Novu | 🟢 **LOW** | 3-5h | Open-source alternative |

---

## Recommended Swap Path

### Phase 1: Foundation (Do First)
1. **Convex Migration** (8-12 hours)
   - Migrate database schema
   - Rewrite Prisma queries as Convex functions
   - Replace API routes with Convex actions
   - Remove packages: `@repo/database`, Neon dependency
   - Remove apps/api (Convex replaces it)

2. **Liveblocks Removal** (2-4 hours)
   - Replace with Convex subscriptions
   - Update real-time components
   - Remove `@repo/collaboration`

**Result:** Simplified stack, eliminated 3 services (Neon, Prisma, Liveblocks), removed entire `apps/api` app

### Phase 2: Authentication (If Needed)
3. **WorkOS Migration** (4-6 hours) - **ONLY IF B2B ENTERPRISE**
   - Build custom auth UI
   - Migrate users
   - Update auth logic
   - Replace `@repo/auth`

**Result:** Better B2B features, included SSO

### Phase 3: Refinements (Optional)
4. **CMS Swap** (4-6 hours) - **If BaseHub doesn't meet needs**
5. **Notifications** (3-5 hours) - **If want open-source (Novu)**

---

## New Architecture After Swaps

### Before (14 services)
```
Frontend (Next.js)
├── Auth: Clerk
├── Database: Prisma + Neon
├── API: Custom Express/Next.js routes
├── Real-time: Liveblocks
├── Payments: Stripe
├── Email: Resend
├── CMS: BaseHub
├── Monitoring: Sentry, BetterStack, PostHog, GA
├── Security: Arcjet
├── Webhooks: Svix
└── Notifications: Knock
```

### After Recommended Swaps (11 services, simpler)
```
Frontend (Next.js)
├── Backend + DB + Real-time: Convex (replaces 3 services!)
├── Auth: WorkOS (or keep Clerk if B2C)
├── Payments: Stripe
├── Email: Resend
├── CMS: BaseHub (or Sanity/Contentful)
├── Monitoring: Sentry, BetterStack, PostHog, GA
├── Security: Arcjet
├── Webhooks: Svix
└── Notifications: Knock (or Novu)
```

**Services eliminated:** 3-4
**Code complexity:** Significantly reduced (no API layer)
**Development speed:** Faster (Convex is more integrated)

---

## Cost Comparison

### Current Stack (Free Tiers)
- Vercel: $0
- Neon: $0 (0.5GB limit)
- Clerk: $0 (10K MAU)
- Liveblocks: $0 (100 MAU)
- Others: $0
- **Total: $0/month** (with limits)

### After Swaps (Free Tiers)
- Vercel: $0
- Convex: $0 (generous limits)
- WorkOS: $0 (better limits than Clerk)
- Others: $0
- **Total: $0/month** (better limits)

### Current Stack (Paid, Typical SaaS)
- Vercel Pro: $20
- Neon: $19
- Clerk: $25
- Liveblocks: $49
- Others: ~$50
- **Total: ~$163/month**

### After Swaps (Paid)
- Vercel Pro: $20
- Convex: $25
- WorkOS: $125 (if enterprise features needed)
- Others: ~$50
- **Total: ~$95-220/month** (depending on auth choice)

**Savings:** $40-70/month if keeping Clerk, or similar cost with better enterprise features if using WorkOS

---

## Migration Risks

### High Risk
- ❌ **Data migration** (Neon → Convex): Must migrate all production data carefully
- ❌ **User migration** (Clerk → WorkOS): Risk of losing users during transition
- ❌ **Downtime**: API layer removal requires careful deployment

### Medium Risk
- ⚠️ **Learning curve**: Team must learn Convex/WorkOS
- ⚠️ **Testing**: Need comprehensive testing after migration
- ⚠️ **Dependencies**: Some packages may depend on Prisma/Clerk

### Low Risk
- ✅ **Email/CMS swaps**: Easy to swap, minimal dependencies
- ✅ **Monitoring swaps**: Can run in parallel during transition

---

## Decision Framework

### Should I swap to Convex?
**YES if:**
- ✅ You want simpler architecture
- ✅ You need real-time features
- ✅ You're comfortable with TypeScript
- ✅ You haven't launched yet (easier migration)

**NO if:**
- ❌ You need PostgreSQL-specific features
- ❌ Team is deeply experienced with Prisma
- ❌ You have complex SQL queries
- ❌ Already in production with large user base

### Should I swap to WorkOS?
**YES if:**
- ✅ Building B2B SaaS
- ✅ Need enterprise SSO
- ✅ Need directory sync
- ✅ Want full control over auth UI

**NO if:**
- ❌ Building B2C
- ❌ Want pre-built auth UI
- ❌ Small team/startup (Clerk is easier)

---

## Next Steps

1. **Review this matrix** with your team
2. **Decide on swaps** based on your specific needs
3. **Read detailed migration guides:**
   - `SWAP-CONVEX.md` - Complete Convex migration guide
   - `SWAP-WORKOS.md` - Complete WorkOS migration guide
4. **Use skills** to automate the swap process
5. **Test thoroughly** before deploying

---

## Questions to Ask Before Swapping

1. **What problem am I solving?** (Don't swap just to swap)
2. **What's the effort vs benefit?** (Is it worth the time?)
3. **Can I test in isolation?** (Swap one component at a time)
4. **What's my rollback plan?** (If something goes wrong)
5. **Does my team have the expertise?** (Learning curve acceptable?)

Remember: **The best code is code that works.** Don't over-optimize prematurely.

# Fiverr Freelancer: Testing & Deployment Instructions

## Mission
Test the Creative Launch application locally, deploy to Vercel, configure production webhooks, and document the entire process. Estimated time: 4-5 hours.

## Your Background
- Experienced with Node.js and Next.js applications
- Familiar with Convex (though this project uses Prisma/Neon for now)
- Comfortable with command-line tools and deployment workflows

## Prerequisites
- Node.js 20+ installed
- pnpm installed (`npm install -g pnpm`)
- Git access to repository: `https://github.com/howagain/creative-launch`
- Access to environment files (created by AI Coding Agent)
- Vercel account credentials

---

## Phase 4: Local Development Testing (90 minutes)

### Step 1: Clone and Setup (15 min)

```bash
# Clone the repository
git clone https://github.com/howagain/creative-launch.git
cd creative-launch

# Verify environment files exist
ls -la apps/api/.env.local
ls -la apps/app/.env.local
ls -la apps/web/.env.local
ls -la packages/database/.env
ls -la packages/cms/.env.local

# If any are missing, STOP and notify the team
```

**Acceptance Criteria:**
- ✅ Repository cloned successfully
- ✅ All 5 `.env` files present
- ✅ Files are not empty (check with `cat apps/app/.env.local`)

---

### Step 2: Install Dependencies (10 min)

```bash
# Install all dependencies
pnpm install
```

**Acceptance Criteria:**
- ✅ No errors during installation
- ✅ `node_modules/` directory created
- ✅ Output shows "X packages installed"

**Common Issues:**
- If you see peer dependency warnings, that's OK
- If install fails, try: `pnpm store prune && pnpm install`

---

### Step 3: Database Migration (10 min)

```bash
# Run database migration
pnpm migrate

# This runs:
# - prisma format (formats schema)
# - prisma generate (generates Prisma Client)
# - prisma db push (creates tables in Neon)
```

**Acceptance Criteria:**
- ✅ Output shows "🚀 Your database is now in sync with your Prisma schema"
- ✅ No errors about connection refused
- ✅ Tables created in Neon database

**Verification:**
```bash
# Open Prisma Studio to see the database
pnpm --filter database studio
```
- Navigate to http://localhost:5555
- Take screenshot showing tables (even if empty)

---

### Step 4: Start Development Servers (15 min)

```bash
# Start all apps in development mode
pnpm dev
```

This starts 3 servers:
- **web** (marketing site): http://localhost:3001
- **app** (main application): http://localhost:3000
- **api** (API server): http://localhost:3002

**Acceptance Criteria:**
- ✅ All 3 apps show "ready in XXXms"
- ✅ No compilation errors in terminal
- ✅ Ports 3000, 3001, 3002 are accessible

**Common Issues:**
- **Port already in use:** Kill the process (`lsof -ti:3000 | xargs kill`) or use different ports
- **Module not found:** Re-run `pnpm install`
- **Database connection error:** Check `DATABASE_URL` in `.env` files

---

### Step 5: Test Web App (5 min)

```bash
# In browser, visit: http://localhost:3001
```

**What to Check:**
- ✅ Page loads without errors
- ✅ No console errors in browser DevTools
- ✅ Navigation works (click links)
- ✅ Styles load correctly (Tailwind CSS)

**Acceptance Criteria:**
- Screenshot of homepage showing next-forge branding
- Browser console shows 0 errors

---

### Step 6: Test Main App & Authentication (20 min)

```bash
# In browser, visit: http://localhost:3000
```

**Steps:**
1. You should be redirected to sign-in page
2. Click "Sign up"
3. Create test account with email: `test@example.com`
4. Complete Clerk sign-up flow
5. Should land on authenticated dashboard

**Acceptance Criteria:**
- ✅ Clerk sign-up widget loads
- ✅ Can create account successfully
- ✅ Redirected to dashboard after sign-in
- ✅ User appears in Clerk dashboard (check clerk.com)
- ✅ User appears in database (check Prisma Studio)

**Screenshots Needed:**
1. Clerk sign-in page
2. Authenticated dashboard
3. Clerk dashboard showing new user
4. Prisma Studio showing user record

---

### Step 7: Test Stripe Payment Flow (15 min)

**Prerequisites:** Stripe test mode enabled

**Steps:**
1. In the app, navigate to pricing/checkout page
2. Click "Subscribe" or "Buy" button
3. Enter Stripe test card: `4242 4242 4242 4242`
4. Expiry: Any future date (e.g., 12/34)
5. CVC: Any 3 digits (e.g., 123)
6. Complete payment

**Acceptance Criteria:**
- ✅ Stripe checkout loads
- ✅ Test payment succeeds
- ✅ Success page displays
- ✅ Event appears in Stripe dashboard (stripe.com → Events)
- ✅ No errors in terminal or browser console

**Screenshots Needed:**
1. Stripe checkout page
2. Payment success page
3. Stripe dashboard showing payment event

---

### Step 8: Test API Health Check (5 min)

```bash
# Test API endpoint
curl http://localhost:3002/health
```

**Expected Response:**
```json
{"status":"ok"}
```

**Acceptance Criteria:**
- ✅ API returns 200 OK
- ✅ Response is valid JSON
- ✅ Status is "ok"

---

### Step 9: Check Logs & Monitoring (10 min)

**Check BetterStack:**
- Visit betterstack.com dashboard
- Verify logs are being received
- Send test log entry

**Check PostHog:**
- Visit posthog.com dashboard
- Verify events are being tracked
- Check that page views appear

**Check Sentry (if configured):**
- Visit sentry.io dashboard
- Trigger a test error in the app
- Verify error appears in Sentry

**Acceptance Criteria:**
- ✅ Logs appear in BetterStack
- ✅ Events appear in PostHog
- ✅ All monitoring services receiving data

**Screenshots Needed:**
1. BetterStack logs
2. PostHog events
3. Sentry dashboard (if applicable)

---

### Step 10: Record Working Demo (15 min)

**Create a screen recording (2-3 minutes) showing:**
1. All 3 servers running in terminal
2. Homepage (localhost:3001)
3. Sign-in flow (localhost:3000)
4. Authenticated dashboard
5. Test payment with Stripe
6. Database with user records

**Tools:** Loom, QuickTime, OBS, or any screen recorder

**Acceptance Criteria:**
- ✅ Video clearly shows all working features
- ✅ No errors visible in console
- ✅ Upload to Google Drive/YouTube
- ✅ Share link with team

---

## Phase 7: Deployment to Vercel (60 minutes)

### Step 11: Verify Code is Pushed to GitHub (5 min)

```bash
# Check current status
git status

# Should show: "nothing to commit, working tree clean"
# If you made any changes, commit them:
git add .
git commit -m "Add environment configuration"
git push
```

**Acceptance Criteria:**
- ✅ Latest code pushed to GitHub
- ✅ GitHub repository shows all files
- ✅ No uncommitted changes locally

---

### Step 12: Link Vercel Project (10 min)

```bash
# Login to Vercel CLI (if not already logged in)
vercel login

# Link the project
vercel link

# When prompted:
# "Set up and deploy?" → Y
# "Which scope?" → [select your account]
# "Link to existing project?" → N
# "Project name?" → creative-launch
# "In which directory?" → ./ (current directory)
```

**Acceptance Criteria:**
- ✅ `.vercel/` directory created
- ✅ `.vercel/project.json` contains project ID
- ✅ Vercel dashboard shows "creative-launch" project

---

### Step 13: Configure Environment Variables in Vercel (20 min)

**Option A: Via Dashboard (Recommended)**

1. Go to https://vercel.com/dashboard
2. Select "creative-launch" project
3. Go to Settings → Environment Variables
4. Add each variable from all `.env.local` files

**Variables to add (combine from all env files):**

```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
CLERK_SECRET_KEY
CLERK_WEBHOOK_SECRET
DATABASE_URL
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY
STRIPE_SECRET_KEY
STRIPE_WEBHOOK_SECRET
RESEND_TOKEN
RESEND_FROM
SENTRY_DSN
BETTERSTACK_API_KEY
BETTERSTACK_URL
NEXT_PUBLIC_POSTHOG_KEY
NEXT_PUBLIC_POSTHOG_HOST
NEXT_PUBLIC_GA_MEASUREMENT_ID
ARCJET_KEY
BASEHUB_TOKEN
LIVEBLOCKS_SECRET
NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY
KNOCK_API_KEY
KNOCK_SECRET_API_KEY
NEXT_PUBLIC_KNOCK_API_KEY
KNOCK_FEED_CHANNEL_ID
NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID
SVIX_TOKEN
FLAGS_SECRET
```

**Important:** For production URLs, use:
```
NEXT_PUBLIC_APP_URL=https://creative-launch.vercel.app
NEXT_PUBLIC_WEB_URL=https://creative-launch-web.vercel.app
NEXT_PUBLIC_API_URL=https://creative-launch-api.vercel.app
```

**Option B: Via CLI (Advanced)**

```bash
# Example for one variable:
vercel env add CLERK_SECRET_KEY production

# Paste the value when prompted
```

**Acceptance Criteria:**
- ✅ All 25+ environment variables added
- ✅ Each variable set for "Production", "Preview", and "Development"
- ✅ Screenshot showing environment variables page (blur secrets)

---

### Step 14: Deploy to Production (10 min)

```bash
# Deploy to production
vercel --prod

# This will:
# 1. Build all apps
# 2. Run tests
# 3. Deploy to production
# 4. Provide production URLs
```

**Acceptance Criteria:**
- ✅ Build completes without errors
- ✅ Deployment succeeds
- ✅ You receive 3 production URLs:
  - Web: `https://creative-launch.vercel.app` (or similar)
  - App: `https://creative-launch-app.vercel.app`
  - API: `https://creative-launch-api.vercel.app`

**Common Issues:**
- **Build fails:** Check build logs in Vercel dashboard
- **Environment variables missing:** Re-add via dashboard
- **Timeout:** Increase build timeout in Vercel settings

---

### Step 15: Test Production Deployment (10 min)

**Visit each production URL:**

1. **Web:** https://creative-launch.vercel.app
   - ✅ Homepage loads
   - ✅ No console errors
   - ✅ Styles correct

2. **App:** https://creative-launch-app.vercel.app
   - ✅ Sign-in page loads
   - ✅ Create new user (use different email than local testing)
   - ✅ User appears in Clerk dashboard
   - ✅ User appears in database

3. **API:** https://creative-launch-api.vercel.app/health
   - ✅ Returns `{"status":"ok"}`

**Acceptance Criteria:**
- ✅ All 3 apps accessible via HTTPS
- ✅ SSL certificates valid (lock icon in browser)
- ✅ No mixed content warnings
- ✅ Authentication works in production

**Screenshots Needed:**
1. Web homepage (production)
2. App dashboard (production, logged in)
3. API health check response

---

### Step 16: Configure Custom Domains (Optional, 15 min)

**If custom domains are provided:**

1. Go to Vercel dashboard → Domains
2. Add domains:
   - `creative-launch.com` → web app
   - `app.creative-launch.com` → main app
   - `api.creative-launch.com` → API
3. Follow DNS configuration instructions
4. Wait for SSL provisioning (5-10 min)

**Acceptance Criteria:**
- ✅ Domains added in Vercel
- ✅ DNS records configured
- ✅ SSL certificates issued
- ✅ Domains resolve correctly

---

## Phase 8: Production Webhooks Configuration (30 minutes)

### Step 17: Update Stripe Webhooks (10 min)

1. Go to https://dashboard.stripe.com/webhooks
2. Click existing webhook OR create new one
3. Update endpoint URL to: `https://creative-launch-api.vercel.app/api/webhooks/payments`
4. Ensure these events are selected:
   - `checkout.session.completed`
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_succeeded`
   - `invoice.payment_failed`
5. Copy webhook signing secret (starts with `whsec_`)
6. Update `STRIPE_WEBHOOK_SECRET` in Vercel environment variables
7. Re-deploy: `vercel --prod`

**Acceptance Criteria:**
- ✅ Webhook endpoint updated
- ✅ Test webhook sent successfully
- ✅ Webhook appears in Stripe dashboard with "Success" status

**Screenshot:** Stripe webhook configuration showing production URL

---

### Step 18: Update Clerk Production Instance (10 min)

1. Go to https://dashboard.clerk.com
2. Switch to production instance (toggle in top-left)
3. Navigate to Webhooks → Update endpoint
4. URL: `https://creative-launch-api.vercel.app/api/webhooks/auth`
5. Ensure events selected: `user.created`, `user.updated`, `user.deleted`
6. Copy production webhook secret
7. Update `CLERK_WEBHOOK_SECRET` in Vercel
8. Re-deploy: `vercel --prod`

**Acceptance Criteria:**
- ✅ Production webhook configured
- ✅ Test webhook successful
- ✅ Webhook shows in Clerk dashboard

**Screenshot:** Clerk webhook configuration

---

### Step 19: Configure Svix Webhook Endpoints (10 min)

1. Go to Svix dashboard
2. Add production endpoints
3. Configure retry logic and timeouts
4. Test webhook delivery

**Acceptance Criteria:**
- ✅ Endpoints configured
- ✅ Test delivery succeeds

---

### Step 20: End-to-End Production Test (30 min)

**Complete this workflow in production:**

1. Visit `https://creative-launch-app.vercel.app`
2. Sign up with NEW email address: `prod-test@example.com`
3. Complete authentication
4. Navigate to pricing page
5. Make test payment with Stripe (use test card)
6. Verify:
   - ✅ User created in Clerk
   - ✅ User record in database
   - ✅ Clerk webhook received (check logs)
   - ✅ Payment successful in Stripe
   - ✅ Stripe webhook received (check logs)
   - ✅ Events in PostHog
   - ✅ Logs in BetterStack

**Acceptance Criteria:**
- ✅ Complete user signup flow works
- ✅ Payment flow works end-to-end
- ✅ All webhooks fire successfully
- ✅ Data synced across all services

**Record this as video (2-3 min) showing:**
- Sign-up process
- Payment flow
- Webhook logs
- Database records
- Monitoring dashboards

---

## Phase 9: Documentation (90 minutes)

### Step 21: Write VANILLA-DEPLOYMENT.md (30 min)

Create `.claude/docs/VANILLA-DEPLOYMENT.md`:

```markdown
# Vanilla next-forge Deployment Guide

## Overview
This guide documents deploying the unmodified next-forge template to production.

## Prerequisites
- Node.js 20+
- pnpm installed
- 14 service accounts created
- Environment variables configured

## Step-by-Step Process

### 1. Project Setup
[Document what you did in Steps 1-3]

### 2. Local Development
[Document what you did in Steps 4-10]

### 3. Vercel Deployment
[Document what you did in Steps 11-16]

### 4. Webhook Configuration
[Document what you did in Steps 17-19]

### 5. Production Testing
[Document what you did in Step 20]

## Time Estimate
- Setup: 30 minutes
- Local testing: 90 minutes
- Deployment: 60 minutes
- Webhooks: 30 minutes
- Testing: 30 minutes
- **Total: 4 hours**

## Common Issues
[List any problems you encountered and how you solved them]
```

---

### Step 22: Create SERVICE-COSTS.md (20 min)

Document monthly costs for each service:

```markdown
# Service Cost Breakdown

| Service | Free Tier | Paid Tier | What We Use | Monthly Cost |
|---------|-----------|-----------|-------------|--------------|
| Vercel | ✅ Hobby | Pro $20/mo | Free tier | $0 |
| Neon | ✅ 0.5GB | Pro $19/mo | Free tier | $0 |
| Clerk | ✅ 10K MAU | Pro $25/mo | Free tier | $0 |
| Stripe | Pay-as-you-go | - | 2.9% + 30¢ | Variable |
| Resend | ✅ 100/day | Pro $20/mo | Free tier | $0 |
...

**Total estimated monthly cost:** $0-50 (depends on usage)
```

---

### Step 23: Write TROUBLESHOOTING-DEPLOYMENT.md (40 min)

Document every error you encountered and how you fixed it:

```markdown
# Deployment Troubleshooting

## Build Errors

### Error: "Module not found: '@repo/database'"
**Cause:** Dependencies not installed correctly
**Solution:**
```bash
pnpm install
pnpm build
```

### Error: "Database connection refused"
**Cause:** DATABASE_URL incorrect or Neon database not accessible
**Solution:**
1. Verify DATABASE_URL in .env
2. Check Neon dashboard for connection string
3. Ensure IP allowlist includes 0.0.0.0/0

[Continue for EVERY error encountered...]
```

---

## Final Deliverables

Submit to client:

### 1. Documentation
- ✅ `VANILLA-DEPLOYMENT.md` - Complete deployment guide
- ✅ `SERVICE-COSTS.md` - Cost breakdown
- ✅ `TROUBLESHOOTING-DEPLOYMENT.md` - Error solutions
- ✅ `.claude/docs/ENV-VAULT.md` - Environment variables (already created)

### 2. Videos
- ✅ Local development demo (2-3 min)
- ✅ Production deployment walkthrough (2-3 min)
- ✅ End-to-end production test (2-3 min)

### 3. Screenshots
Organized in Google Drive folder:
- **Local Testing (10+ screenshots)**
  - Dev servers running
  - Homepage, app, API
  - Clerk sign-in
  - Stripe checkout
  - Prisma Studio with data
  - Monitoring dashboards

- **Production (10+ screenshots)**
  - Vercel deployment success
  - Production URLs live
  - Prod authentication working
  - Webhook configurations
  - Monitoring in production

### 4. Production URLs
- Web: https://creative-launch.vercel.app
- App: https://creative-launch-app.vercel.app
- API: https://creative-launch-api.vercel.app/health

### 5. Access Credentials
Document handed off:
- Vercel project access
- Production test account credentials
- Any new API keys generated during deployment

---

## Success Criteria

Before marking complete, verify:

- ✅ Local development works completely
- ✅ All environment variables configured
- ✅ Production deployment successful
- ✅ All 3 apps accessible via HTTPS
- ✅ Authentication works in production
- ✅ Payment flow works in production
- ✅ Webhooks configured and tested
- ✅ Monitoring services receiving data
- ✅ 3 videos recorded and uploaded
- ✅ 20+ screenshots organized
- ✅ 3 documentation files written
- ✅ No critical errors in production

---

## Communication

Throughout this process, update the client on:
1. When you start each phase
2. Any blockers or issues
3. When you complete each major milestone
4. Questions about requirements

Use this format:
```
**Update: [Phase Name]**
Status: ✅ Complete / 🚧 In Progress / ❌ Blocked
Time spent: X hours
Issues: [Any problems]
Next: [What's next]
```

---

## Estimated Timeline

| Phase | Time | Dependencies |
|-------|------|--------------|
| Local Setup (Steps 1-10) | 90 min | Environment files ready |
| Deployment (Steps 11-16) | 60 min | Local testing complete |
| Webhooks (Steps 17-20) | 60 min | Deployment complete |
| Documentation (Steps 21-23) | 90 min | All phases complete |
| **Total** | **5 hours** | - |

## Questions?

If you get stuck, contact:
- **Technical issues:** [Claude Code Agent]
- **Service access:** [Client]
- **Requirements clarification:** [Client]

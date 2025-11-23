# AI Web Agent: Service Account Setup Instructions

## Mission
Create accounts for 14 third-party services and collect API keys/credentials. This should take approximately 90 minutes with all tasks running in parallel (open 14 browser tabs).

## Prerequisites
- Email address: `[USER_WILL_PROVIDE]`
- Password manager access (to store credentials)
- Credit card for paid services (some free tiers available)
- Phone number for 2FA verification

## Output Format
Create a Google Doc with this table structure:

| Service | Dashboard URL | API Keys | Screenshots | Status |
|---------|---------------|----------|-------------|---------|
| Vercel | URL | Keys | Link | ✅ |
| ... | ... | ... | ... | ... |

---

## Service Setup Instructions

### Tier 1: Core Infrastructure (Start First - Critical Path)

#### 1. Vercel (Hosting Platform)
**URL:** https://vercel.com/signup
- **Account Type:** Free tier is fine initially
- **Setup Steps:**
  1. Sign up with provided email
  2. Verify email
  3. Skip team creation (personal account OK)
  4. Install Vercel CLI: Note that installation will be done by contractor
  5. Login via CLI: `vercel login`
  6. Run: `vercel whoami` to get username
- **Credentials to Collect:**
  - Account email
  - Account username (from `vercel whoami`)
  - Team ID (if created)
- **Screenshot:** Dashboard homepage showing account name
- **Verification:** `vercel whoami` returns your username

---

#### 2. Neon PostgreSQL (Database)
**URL:** https://neon.tech/
- **Account Type:** Free tier (0.5 GB storage, 10 projects)
- **Setup Steps:**
  1. Sign up with GitHub or email
  2. Create new project named "creative-launch"
  3. Select region (choose closest to your users)
  4. Copy connection string
- **Credentials to Collect:**
  - `DATABASE_URL` (connection string with password)
  - Project ID
  - Dashboard URL
- **Screenshot:** Project dashboard showing connection details
- **Verification:** Connection string starts with `postgresql://`

---

#### 3. Clerk (Authentication)
**URL:** https://clerk.com/
- **Account Type:** Free tier (10,000 MAUs)
- **Setup Steps:**
  1. Sign up with email
  2. Create new application: "Creative Launch"
  3. Select "Next.js" as framework
  4. Copy publishable key and secret key
  5. Navigate to Webhooks → Add endpoint
  6. Set endpoint URL: `[WILL_BE_UPDATED_AFTER_DEPLOYMENT]/api/webhooks/auth`
  7. Subscribe to events: `user.created`, `user.updated`, `user.deleted`
  8. Copy webhook secret
- **Credentials to Collect:**
  - `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` (starts with `pk_test_` or `pk_live_`)
  - `CLERK_SECRET_KEY` (starts with `sk_test_` or `sk_live_`)
  - `CLERK_WEBHOOK_SECRET` (starts with `whsec_`)
- **Screenshot:** API Keys page + Webhook configuration
- **Verification:** Keys start with correct prefixes

---

### Tier 2: Essential Features (Can Do in Parallel)

#### 4. Stripe (Payments)
**URL:** https://stripe.com/
- **Account Type:** Free (pay-as-you-go)
- **Setup Steps:**
  1. Sign up with email
  2. Complete business profile (use personal info for testing)
  3. Navigate to Developers → API Keys
  4. Toggle "Test mode" ON
  5. Copy "Publishable key" and "Secret key"
  6. Install Stripe CLI (optional for local testing)
  7. Webhooks → Add endpoint
  8. URL: `[WILL_BE_UPDATED_AFTER_DEPLOYMENT]/api/webhooks/payments`
  9. Events: Select all payment events
  10. Copy webhook signing secret
- **Credentials to Collect:**
  - `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` (starts with `pk_test_`)
  - `STRIPE_SECRET_KEY` (starts with `sk_test_`)
  - `STRIPE_WEBHOOK_SECRET` (starts with `whsec_`)
- **Screenshot:** API keys page + Webhook details
- **Verification:** Test with card 4242 4242 4242 4242

---

#### 5. Resend (Email Sending)
**URL:** https://resend.com/
- **Account Type:** Free tier (100 emails/day)
- **Setup Steps:**
  1. Sign up with email
  2. Verify your domain OR use `onboarding@resend.dev` for testing
  3. Generate API key
  4. Note "from" email address
- **Credentials to Collect:**
  - `RESEND_TOKEN` (API key starting with `re_`)
  - `RESEND_FROM` (email address like `noreply@yourdomain.com`)
- **Screenshot:** API Keys page
- **Verification:** Send test email via dashboard

---

#### 6. Sentry (Error Tracking)
**URL:** https://sentry.io/
- **Account Type:** Free tier (5,000 events/month)
- **Setup Steps:**
  1. Sign up with email
  2. Create new project: "Creative Launch"
  3. Select platform: "Next.js"
  4. Copy DSN
  5. Skip wizard setup (we'll configure manually)
- **Credentials to Collect:**
  - `SENTRY_DSN` (looks like `https://...@o....ingest.sentry.io/...`)
  - `SENTRY_ORG` (organization slug)
  - `SENTRY_PROJECT` (project slug)
- **Screenshot:** Project settings → Client Keys (DSN)
- **Verification:** DSN contains `@sentry.io`

---

#### 7. BetterStack (Logging & Uptime Monitoring)
**URL:** https://betterstack.com/
- **Account Type:** Free tier
- **Setup Steps:**
  1. Sign up with email
  2. Create new source for logs
  3. Select "HTTP" as source type
  4. Copy source token
  5. Note the ingest URL
- **Credentials to Collect:**
  - `BETTERSTACK_API_KEY` (source token)
  - `BETTERSTACK_URL` (ingest URL)
- **Screenshot:** Source details page
- **Verification:** Send test log entry

---

### Tier 3: Analytics & Monitoring

#### 8. PostHog (Product Analytics)
**URL:** https://posthog.com/
- **Account Type:** Free tier (1M events/month)
- **Setup Steps:**
  1. Sign up with email
  2. Create new project: "Creative Launch"
  3. Copy project API key
  4. Note instance URL (US or EU)
- **Credentials to Collect:**
  - `NEXT_PUBLIC_POSTHOG_KEY` (project API key)
  - `NEXT_PUBLIC_POSTHOG_HOST` (usually `https://app.posthog.com` or `https://eu.posthog.com`)
- **Screenshot:** Project settings → API keys
- **Verification:** Key visible in settings

---

#### 9. Google Analytics 4
**URL:** https://analytics.google.com/
- **Account Type:** Free
- **Setup Steps:**
  1. Sign in with Google account
  2. Create new account: "Creative Launch"
  3. Create new property: "Creative Launch Web"
  4. Select industry category & timezone
  5. Copy Measurement ID
- **Credentials to Collect:**
  - `NEXT_PUBLIC_GA_MEASUREMENT_ID` (starts with `G-`)
- **Screenshot:** Property settings showing Measurement ID
- **Verification:** Measurement ID starts with `G-`

---

### Tier 4: Advanced Features

#### 10. Arcjet (Security & Rate Limiting)
**URL:** https://arcjet.com/
- **Account Type:** Free tier
- **Setup Steps:**
  1. Sign up with email or GitHub
  2. Create new site: "Creative Launch"
  3. Copy API key
- **Credentials to Collect:**
  - `ARCJET_KEY` (starts with `ajkey_`)
- **Screenshot:** API keys page
- **Verification:** Key starts with `ajkey_`

---

#### 11. BaseHub (Headless CMS)
**URL:** https://basehub.com/
- **Account Type:** Free tier
- **Setup Steps:**
  1. Sign up with email
  2. Create new repository: "creative-launch"
  3. Generate API token
- **Credentials to Collect:**
  - `BASEHUB_TOKEN` (API token)
- **Screenshot:** Settings → API tokens
- **Verification:** Token generated successfully

---

#### 12. Liveblocks (Real-time Collaboration)
**URL:** https://liveblocks.io/
- **Account Type:** Free tier (100 MAUs)
- **Setup Steps:**
  1. Sign up with email
  2. Create new project: "Creative Launch"
  3. Copy secret key
  4. Copy public key
- **Credentials to Collect:**
  - `LIVEBLOCKS_SECRET` (starts with `sk_`)
  - `NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY` (starts with `pk_`)
- **Screenshot:** API keys page
- **Verification:** Keys start with correct prefixes

---

#### 13. Knock (Notifications)
**URL:** https://knock.app/
- **Account Type:** Free tier (10,000 notifications/month)
- **Setup Steps:**
  1. Sign up with email
  2. Create environment: "Development"
  3. Copy API keys (secret, public)
  4. Create feed channel
  5. Copy feed channel ID
- **Credentials to Collect:**
  - `KNOCK_API_KEY` (publishable key)
  - `KNOCK_SECRET_API_KEY` (secret key)
  - `NEXT_PUBLIC_KNOCK_API_KEY` (public key)
  - `NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID` (feed channel ID)
  - `KNOCK_FEED_CHANNEL_ID` (feed channel ID)
- **Screenshot:** API keys page + Channels page
- **Verification:** 5 credentials collected

---

#### 14. Svix (Webhook Infrastructure)
**URL:** https://svix.com/
- **Account Type:** Free tier (50,000 messages/month)
- **Setup Steps:**
  1. Sign up with email
  2. Create new application: "Creative Launch"
  3. Copy authentication token
- **Credentials to Collect:**
  - `SVIX_TOKEN` (authentication token)
- **Screenshot:** Settings → Authentication
- **Verification:** Token visible in settings

---

## Final Deliverables

### Google Doc Table
Complete the table with ALL 14 services including:
- ✅ Status checkmarks
- Dashboard URLs
- All API keys (formatted as code blocks)
- Links to screenshots

### Screenshots Folder
Upload to Google Drive folder with these screenshots:
1. `vercel-dashboard.png`
2. `neon-connection.png`
3. `clerk-api-keys.png`
4. `clerk-webhooks.png`
5. `stripe-api-keys.png`
6. `stripe-webhooks.png`
7. `resend-api-keys.png`
8. `sentry-dsn.png`
9. `betterstack-source.png`
10. `posthog-keys.png`
11. `google-analytics-measurement-id.png`
12. `arcjet-key.png`
13. `basehub-token.png`
14. `liveblocks-keys.png`
15. `knock-keys.png`
16. `knock-feed-channel.png`
17. `svix-token.png`

### Credentials JSON (For Next Step)
Create a JSON file with all credentials:

```json
{
  "vercel": {
    "username": "...",
    "teamId": "..."
  },
  "database": {
    "DATABASE_URL": "postgresql://..."
  },
  "clerk": {
    "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY": "pk_test_...",
    "CLERK_SECRET_KEY": "sk_test_...",
    "CLERK_WEBHOOK_SECRET": "whsec_..."
  },
  "stripe": {
    "NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY": "pk_test_...",
    "STRIPE_SECRET_KEY": "sk_test_...",
    "STRIPE_WEBHOOK_SECRET": "whsec_..."
  },
  "resend": {
    "RESEND_TOKEN": "re_...",
    "RESEND_FROM": "noreply@..."
  },
  "sentry": {
    "SENTRY_DSN": "https://...",
    "SENTRY_ORG": "...",
    "SENTRY_PROJECT": "..."
  },
  "betterstack": {
    "BETTERSTACK_API_KEY": "...",
    "BETTERSTACK_URL": "..."
  },
  "posthog": {
    "NEXT_PUBLIC_POSTHOG_KEY": "...",
    "NEXT_PUBLIC_POSTHOG_HOST": "https://app.posthog.com"
  },
  "googleAnalytics": {
    "NEXT_PUBLIC_GA_MEASUREMENT_ID": "G-..."
  },
  "arcjet": {
    "ARCJET_KEY": "ajkey_..."
  },
  "basehub": {
    "BASEHUB_TOKEN": "..."
  },
  "liveblocks": {
    "LIVEBLOCKS_SECRET": "sk_...",
    "NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY": "pk_..."
  },
  "knock": {
    "KNOCK_API_KEY": "...",
    "KNOCK_SECRET_API_KEY": "...",
    "NEXT_PUBLIC_KNOCK_API_KEY": "...",
    "NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID": "...",
    "KNOCK_FEED_CHANNEL_ID": "..."
  },
  "svix": {
    "SVIX_TOKEN": "..."
  }
}
```

## Time Estimate
- **Tier 1 (Critical):** 30 minutes
- **Tier 2 (Essential):** 30 minutes
- **Tier 3 (Analytics):** 15 minutes
- **Tier 4 (Advanced):** 15 minutes
- **Total:** ~90 minutes wall time (if done in parallel)

## Common Issues

### Email Verification Delays
- Check spam folder
- Use email alias if needed
- Some services may take 5-10 minutes to send verification

### Credit Card Required
These services may require credit card even for free tier:
- Stripe (always)
- Vercel (for production domains)
- Sentry (sometimes)

### 2FA Requirements
- Have phone ready for SMS codes
- Use authenticator app if preferred

### Webhook URLs Unknown
- For Clerk, Stripe, Svix webhooks: Use placeholder URL initially
- Will be updated after Vercel deployment
- Note in doc: "UPDATE AFTER DEPLOYMENT"

## Success Criteria
- ✅ All 14 services have accounts created
- ✅ All API keys collected and formatted
- ✅ 17+ screenshots uploaded
- ✅ JSON credentials file created
- ✅ Google Doc table 100% complete
- ✅ No "pending" or "failed" statuses

## Hand-off to Next Agent
Once complete, share the Google Doc link and credentials JSON file with the AI Coding Agent for environment file generation.

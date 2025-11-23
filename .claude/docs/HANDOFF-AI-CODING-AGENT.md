# AI Coding Agent: Environment File Generation Instructions

## Mission
Generate all 6 `.env.local` files from the API keys collected by the AI Web Agent. Create a validation script to verify all required variables are set. Estimated time: 20-30 minutes.

## Prerequisites
- Access to credentials JSON file from AI Web Agent
- Repository cloned locally
- Understanding of which files need which variables

## Input
You'll receive a `credentials.json` file with this structure:

```json
{
  "vercel": { "username": "...", "teamId": "..." },
  "database": { "DATABASE_URL": "postgresql://..." },
  "clerk": { "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY": "pk_test_...", ... },
  ...
}
```

## Output Files to Create

### 1. `apps/api/.env.local`
### 2. `apps/app/.env.local`
### 3. `apps/web/.env.local`
### 4. `packages/database/.env`
### 5. `packages/cms/.env.local`
### 6. `packages/internationalization/.env.local`

---

## Task 1: Create Environment File Generator Script

Create `scripts/generate-env-files.js`:

```javascript
const fs = require('fs');
const path = require('path');

// Read credentials from JSON file
const credentials = JSON.parse(fs.readFileSync('./credentials.json', 'utf8'));

// Helper to write env file
function writeEnvFile(filePath, variables) {
  const content = Object.entries(variables)
    .map(([key, value]) => `${key}="${value}"`)
    .join('\n');

  fs.writeFileSync(filePath, content + '\n');
  console.log(`✅ Created: ${filePath}`);
}

// 1. apps/api/.env.local
writeEnvFile('apps/api/.env.local', {
  // Auth
  CLERK_SECRET_KEY: credentials.clerk.CLERK_SECRET_KEY,
  CLERK_WEBHOOK_SECRET: credentials.clerk.CLERK_WEBHOOK_SECRET,

  // Payments
  STRIPE_SECRET_KEY: credentials.stripe.STRIPE_SECRET_KEY,
  STRIPE_WEBHOOK_SECRET: credentials.stripe.STRIPE_WEBHOOK_SECRET,

  // Email
  RESEND_TOKEN: credentials.resend.RESEND_TOKEN,
  RESEND_FROM: credentials.resend.RESEND_FROM,

  // Monitoring
  BETTERSTACK_API_KEY: credentials.betterstack.BETTERSTACK_API_KEY,
  BETTERSTACK_URL: credentials.betterstack.BETTERSTACK_URL,

  // Security
  ARCJET_KEY: credentials.arcjet.ARCJET_KEY,

  // URLs (localhost for development)
  NEXT_PUBLIC_APP_URL: 'http://localhost:3000',
  NEXT_PUBLIC_WEB_URL: 'http://localhost:3001',
  NEXT_PUBLIC_API_URL: 'http://localhost:3002',
});

// 2. apps/app/.env.local
writeEnvFile('apps/app/.env.local', {
  // Auth
  NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: credentials.clerk.NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY,
  CLERK_SECRET_KEY: credentials.clerk.CLERK_SECRET_KEY,
  CLERK_WEBHOOK_SECRET: credentials.clerk.CLERK_WEBHOOK_SECRET,

  // Database
  DATABASE_URL: credentials.database.DATABASE_URL,

  // Payments
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: credentials.stripe.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY,
  STRIPE_SECRET_KEY: credentials.stripe.STRIPE_SECRET_KEY,
  STRIPE_WEBHOOK_SECRET: credentials.stripe.STRIPE_WEBHOOK_SECRET,

  // Email
  RESEND_TOKEN: credentials.resend.RESEND_TOKEN,
  RESEND_FROM: credentials.resend.RESEND_FROM,

  // Monitoring
  BETTERSTACK_API_KEY: credentials.betterstack.BETTERSTACK_API_KEY,
  BETTERSTACK_URL: credentials.betterstack.BETTERSTACK_URL,
  NEXT_PUBLIC_POSTHOG_KEY: credentials.posthog.NEXT_PUBLIC_POSTHOG_KEY,
  NEXT_PUBLIC_POSTHOG_HOST: credentials.posthog.NEXT_PUBLIC_POSTHOG_HOST,
  NEXT_PUBLIC_GA_MEASUREMENT_ID: credentials.googleAnalytics.NEXT_PUBLIC_GA_MEASUREMENT_ID,

  // Security
  ARCJET_KEY: credentials.arcjet.ARCJET_KEY,

  // Feature Flags
  FLAGS_SECRET: 'your-feature-flags-secret',  // Generate this

  // Real-time
  LIVEBLOCKS_SECRET: credentials.liveblocks.LIVEBLOCKS_SECRET,
  NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY: credentials.liveblocks.NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY,

  // CMS
  BASEHUB_TOKEN: credentials.basehub.BASEHUB_TOKEN,

  // Webhooks
  SVIX_TOKEN: credentials.svix.SVIX_TOKEN,

  // Notifications
  KNOCK_API_KEY: credentials.knock.KNOCK_API_KEY,
  KNOCK_FEED_CHANNEL_ID: credentials.knock.KNOCK_FEED_CHANNEL_ID,
  KNOCK_SECRET_API_KEY: credentials.knock.KNOCK_SECRET_API_KEY,
  NEXT_PUBLIC_KNOCK_API_KEY: credentials.knock.NEXT_PUBLIC_KNOCK_API_KEY,
  NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID: credentials.knock.NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID,

  // URLs
  NEXT_PUBLIC_APP_URL: 'http://localhost:3000',
  NEXT_PUBLIC_WEB_URL: 'http://localhost:3001',
  NEXT_PUBLIC_DOCS_URL: 'http://localhost:3004',
});

// 3. apps/web/.env.local
writeEnvFile('apps/web/.env.local', {
  // CMS
  BASEHUB_TOKEN: credentials.basehub.BASEHUB_TOKEN,

  // Email
  RESEND_TOKEN: credentials.resend.RESEND_TOKEN,
  RESEND_FROM: credentials.resend.RESEND_FROM,

  // Analytics
  NEXT_PUBLIC_GA_MEASUREMENT_ID: credentials.googleAnalytics.NEXT_PUBLIC_GA_MEASUREMENT_ID,

  // Security
  ARCJET_KEY: credentials.arcjet.ARCJET_KEY,

  // URLs
  NEXT_PUBLIC_APP_URL: 'http://localhost:3000',
  NEXT_PUBLIC_WEB_URL: 'http://localhost:3001',
});

// 4. packages/database/.env
writeEnvFile('packages/database/.env', {
  DATABASE_URL: credentials.database.DATABASE_URL,
});

// 5. packages/cms/.env.local
writeEnvFile('packages/cms/.env.local', {
  BASEHUB_TOKEN: credentials.basehub.BASEHUB_TOKEN,
});

// 6. packages/internationalization/.env.local
// This file is typically empty or has optional localization keys
writeEnvFile('packages/internationalization/.env.local', {
  // Add if using paid translation service
  // TRANSLATION_API_KEY: ''
});

console.log('\n✅ All environment files created successfully!');
console.log('\nNext steps:');
console.log('1. Run: node scripts/validate-env.js');
console.log('2. Run: pnpm install');
console.log('3. Run: pnpm migrate');
console.log('4. Run: pnpm dev');
```

---

## Task 2: Create Environment Validation Script

Create `scripts/validate-env.js`:

```javascript
const fs = require('fs');
const path = require('path');

// Define required variables for each file
const requirements = {
  'apps/api/.env.local': [
    'CLERK_SECRET_KEY',
    'CLERK_WEBHOOK_SECRET',
    'STRIPE_SECRET_KEY',
    'STRIPE_WEBHOOK_SECRET',
    'RESEND_TOKEN',
    'RESEND_FROM',
    'BETTERSTACK_API_KEY',
    'BETTERSTACK_URL',
    'ARCJET_KEY',
    'NEXT_PUBLIC_APP_URL',
    'NEXT_PUBLIC_WEB_URL',
    'NEXT_PUBLIC_API_URL',
  ],
  'apps/app/.env.local': [
    'NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY',
    'CLERK_SECRET_KEY',
    'DATABASE_URL',
    'NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY',
    'STRIPE_SECRET_KEY',
    'RESEND_TOKEN',
    'NEXT_PUBLIC_POSTHOG_KEY',
    'NEXT_PUBLIC_GA_MEASUREMENT_ID',
    'ARCJET_KEY',
    'LIVEBLOCKS_SECRET',
    'BASEHUB_TOKEN',
    'SVIX_TOKEN',
    'KNOCK_API_KEY',
    'NEXT_PUBLIC_KNOCK_API_KEY',
    'NEXT_PUBLIC_APP_URL',
  ],
  'apps/web/.env.local': [
    'BASEHUB_TOKEN',
    'RESEND_TOKEN',
    'NEXT_PUBLIC_GA_MEASUREMENT_ID',
    'ARCJET_KEY',
    'NEXT_PUBLIC_APP_URL',
    'NEXT_PUBLIC_WEB_URL',
  ],
  'packages/database/.env': [
    'DATABASE_URL',
  ],
  'packages/cms/.env.local': [
    'BASEHUB_TOKEN',
  ],
};

function parseEnvFile(filePath) {
  if (!fs.existsSync(filePath)) {
    return null;
  }

  const content = fs.readFileSync(filePath, 'utf8');
  const vars = {};

  content.split('\n').forEach(line => {
    const match = line.match(/^([^=]+)=(.*)$/);
    if (match) {
      vars[match[1].trim()] = match[2].trim();
    }
  });

  return vars;
}

let hasErrors = false;

console.log('🔍 Validating environment files...\n');

Object.entries(requirements).forEach(([filePath, requiredVars]) => {
  const fullPath = path.join(process.cwd(), filePath);
  const vars = parseEnvFile(fullPath);

  if (!vars) {
    console.log(`❌ ${filePath} - FILE NOT FOUND`);
    hasErrors = true;
    return;
  }

  const missing = requiredVars.filter(v => !vars[v] || vars[v] === '""' || vars[v] === "''");

  if (missing.length === 0) {
    console.log(`✅ ${filePath} - All ${requiredVars.length} variables set`);
  } else {
    console.log(`❌ ${filePath} - Missing ${missing.length} variables:`);
    missing.forEach(v => console.log(`   - ${v}`));
    hasErrors = true;
  }
});

if (hasErrors) {
  console.log('\n❌ Environment validation FAILED');
  console.log('Please add the missing variables and run this script again.');
  process.exit(1);
} else {
  console.log('\n✅ All environment files validated successfully!');
  console.log('\nYou can now run:');
  console.log('  pnpm install');
  console.log('  pnpm migrate');
  console.log('  pnpm dev');
}
```

---

## Task 3: Create .env.vault.md Documentation

Create `.claude/docs/ENV-VAULT.md`:

```markdown
# Environment Variables Vault

## Overview
This document explains all environment variables used in the Creative Launch project.

| Variable | Service | Purpose | Example | Required |
|----------|---------|---------|---------|----------|
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk | Auth (client-side) | `pk_test_...` | ✅ |
| `CLERK_SECRET_KEY` | Clerk | Auth (server-side) | `sk_test_...` | ✅ |
| `CLERK_WEBHOOK_SECRET` | Clerk | Verify webhooks | `whsec_...` | ✅ |
| `DATABASE_URL` | Neon | PostgreSQL connection | `postgresql://...` | ✅ |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe | Payments (client) | `pk_test_...` | ✅ |
| `STRIPE_SECRET_KEY` | Stripe | Payments (server) | `sk_test_...` | ✅ |
| `STRIPE_WEBHOOK_SECRET` | Stripe | Verify webhooks | `whsec_...` | ✅ |
| `RESEND_TOKEN` | Resend | Send emails | `re_...` | ✅ |
| `RESEND_FROM` | Resend | From address | `noreply@domain.com` | ✅ |
| `SENTRY_DSN` | Sentry | Error tracking | `https://...@sentry.io/...` | ⚠️ Optional |
| `BETTERSTACK_API_KEY` | BetterStack | Logging | Token | ✅ |
| `BETTERSTACK_URL` | BetterStack | Log ingest URL | `https://...` | ✅ |
| `NEXT_PUBLIC_POSTHOG_KEY` | PostHog | Analytics | Project key | ✅ |
| `NEXT_PUBLIC_POSTHOG_HOST` | PostHog | Instance URL | `https://app.posthog.com` | ✅ |
| `NEXT_PUBLIC_GA_MEASUREMENT_ID` | Google Analytics | Web analytics | `G-...` | ✅ |
| `ARCJET_KEY` | Arcjet | Security/rate limiting | `ajkey_...` | ✅ |
| `BASEHUB_TOKEN` | BaseHub | CMS content | Token | ✅ |
| `LIVEBLOCKS_SECRET` | Liveblocks | Real-time (server) | `sk_...` | ✅ |
| `NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY` | Liveblocks | Real-time (client) | `pk_...` | ✅ |
| `KNOCK_API_KEY` | Knock | Notifications | Key | ✅ |
| `KNOCK_SECRET_API_KEY` | Knock | Notifications (server) | Secret | ✅ |
| `NEXT_PUBLIC_KNOCK_API_KEY` | Knock | Notifications (client) | Public key | ✅ |
| `KNOCK_FEED_CHANNEL_ID` | Knock | Feed channel | Channel ID | ✅ |
| `NEXT_PUBLIC_KNOCK_FEED_CHANNEL_ID` | Knock | Feed channel (client) | Channel ID | ✅ |
| `SVIX_TOKEN` | Svix | Webhook delivery | Token | ✅ |
| `FLAGS_SECRET` | Vercel | Feature flags | Generated secret | ⚠️ Optional |
| `NEXT_PUBLIC_APP_URL` | - | App base URL | `http://localhost:3000` | ✅ |
| `NEXT_PUBLIC_WEB_URL` | - | Web base URL | `http://localhost:3001` | ✅ |
| `NEXT_PUBLIC_API_URL` | - | API base URL | `http://localhost:3002` | ⚠️ Optional |
| `NEXT_PUBLIC_DOCS_URL` | - | Docs base URL | `http://localhost:3004` | ⚠️ Optional |
| `VERCEL_PROJECT_PRODUCTION_URL` | Vercel | Auto-set by Vercel | `project.vercel.app` | Auto |

## URL Configuration

### Development (Localhost)
```
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_WEB_URL=http://localhost:3001
NEXT_PUBLIC_API_URL=http://localhost:3002
```

### Production (After Deployment)
```
NEXT_PUBLIC_APP_URL=https://app.creative-launch.com
NEXT_PUBLIC_WEB_URL=https://creative-launch.com
NEXT_PUBLIC_API_URL=https://api.creative-launch.com
```

## Security Notes
- ✅ Never commit `.env.local` or `.env` files to git
- ✅ Keys starting with `NEXT_PUBLIC_` are exposed to client-side code
- ✅ Server-only keys (without `NEXT_PUBLIC_`) are safe
- ✅ Rotate webhook secrets if compromised
- ✅ Use test mode keys for development
```

---

## Task 4: Create .gitignore Verification

Verify that `.gitignore` includes:

```
.env
.env.local
.env*.local
.env.vault
```

If missing, add these lines to `.gitignore`.

---

## Execution Steps

1. **Run the generator:**
   ```bash
   node scripts/generate-env-files.js
   ```
   Expected output:
   ```
   ✅ Created: apps/api/.env.local
   ✅ Created: apps/app/.env.local
   ✅ Created: apps/web/.env.local
   ✅ Created: packages/database/.env
   ✅ Created: packages/cms/.env.local
   ✅ Created: packages/internationalization/.env.local

   ✅ All environment files created successfully!
   ```

2. **Validate the files:**
   ```bash
   node scripts/validate-env.js
   ```
   Expected output:
   ```
   🔍 Validating environment files...

   ✅ apps/api/.env.local - All 12 variables set
   ✅ apps/app/.env.local - All 24 variables set
   ✅ apps/web/.env.local - All 6 variables set
   ✅ packages/database/.env - All 1 variables set
   ✅ packages/cms/.env.local - All 1 variables set

   ✅ All environment files validated successfully!
   ```

3. **Verify .gitignore:**
   ```bash
   cat .gitignore | grep "\.env"
   ```
   Should show `.env` patterns

4. **Create documentation:**
   - Create `.claude/docs/ENV-VAULT.md` with the table above

---

## Deliverables

- ✅ `scripts/generate-env-files.js` - Generator script
- ✅ `scripts/validate-env.js` - Validation script
- ✅ 6 `.env.local` / `.env` files created and populated
- ✅ `.claude/docs/ENV-VAULT.md` - Documentation
- ✅ `.gitignore` verified to exclude env files
- ✅ Validation script passes with 0 errors

## Success Criteria

```bash
# This command should succeed:
node scripts/validate-env.js

# Output should be:
✅ All environment files validated successfully!
```

## Hand-off to Next Agent (Fiverr Freelancer)

Once validation passes, notify the user that environment setup is complete and the Fiverr Freelancer can proceed with:
1. Installing dependencies (`pnpm install`)
2. Running migrations (`pnpm migrate`)
3. Starting dev servers (`pnpm dev`)
4. Testing the application locally

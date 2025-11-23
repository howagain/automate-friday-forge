# Automate Friday Forge - Template Documentation

This directory contains reusable guides and automation tools for deploying next-forge projects.

## Quick Start

When starting a new project from this template:

1. **Copy template to new project directory**
2. **Follow handoff guides in order** (see below)
3. **Evaluate component swaps** based on your needs
4. **Document project-specific setup** in your project's `.claude/` directory

---

## Documentation Structure

### 📋 Deployment Guides (Use in Order)

These guides walk you through deploying a vanilla next-forge project:

1. **HANDOFF-AI-WEB-AGENT.md**
   → Create accounts for all 14 third-party services
   → Time: ~90 minutes (parallelizable)
   → Output: API keys in Google Doc

2. **HANDOFF-AI-CODING-AGENT.md**
   → Generate all `.env` files from API keys
   → Create validation scripts
   → Time: ~20 minutes

3. **HANDOFF-FIVERR-FREELANCER.md**
   → Test locally, deploy to Vercel, configure webhooks
   → Document the process
   → Time: ~4-5 hours

**Total deployment time:** ~6-7 hours (vanilla template, zero customization)

---

### 🔄 Component Swap Guides

After deploying the vanilla template, evaluate these swaps:

- **COMPONENT-SWAP-MATRIX.md** - Overview of all possible swaps with recommendations
- **SWAP-CONVEX.md** - Migrate from Prisma/Neon to Convex (High Priority)
- **SWAP-WORKOS.md** - Migrate from Clerk to WorkOS (Conditional)

**Recommended swap order:**
1. Convex (simplifies stack significantly)
2. WorkOS (only if building B2B enterprise SaaS)
3. Others (evaluate based on needs)

---

## Skills Directory

Automation skills for Claude Code will be added here to streamline:
- Project initialization
- Service account setup
- Environment configuration
- Component swaps
- Deployment verification

---

## Commands Directory

Slash commands for common operations:
- `/deploy-new-project [name]` - Initialize and deploy new project
- `/swap-to-convex` - Guide through Convex migration
- `/swap-to-workos` - Guide through WorkOS migration

---

## Project-Specific Documentation

**Do NOT store project-specific information in this template directory.**

Instead, each project should have its own `.claude/` directory with:
- Project-specific setup notes
- Custom environment variable documentation
- Project-specific troubleshooting
- Team access credentials

**Example project structure:**
```
my-project/
├── .claude/
│   ├── docs/
│   │   ├── PROJECT-SETUP.md (specific to this project)
│   │   └── TEAM-ACCESS.md (credentials, access info)
│   └── skills/ (project-specific skills)
└── ... (project files)
```

---

## Contributing Back to Template

When you discover improvements or solve problems while setting up a project:

1. **Document the solution** in your project's `.claude/docs/`
2. **Generalize it** (remove project-specific details)
3. **Create PR** to this template repository
4. **Update relevant guides** with the improvement

This creates a feedback loop where each project deployment improves the template for future projects.

---

## Service Costs Reference

| Service | Free Tier | When to Upgrade | Cost |
|---------|-----------|-----------------|------|
| Vercel | Hobby (personal projects) | Production apps | $20/mo |
| Neon | 0.5GB, 10 projects | > 0.5GB data | $19/mo |
| Clerk | 10K MAUs | > 10K users | $25/mo |
| Stripe | Pay-as-you-go | - | 2.9% + 30¢ |
| Convex (swap) | 1M calls/mo | High traffic | $25/mo |
| WorkOS (swap) | 3 SSO, unlimited users | > 3 SSO connections | $125/mo |

**Vanilla stack (free tier):** $0/month
**Vanilla stack (paid):** ~$160/month
**After recommended swaps (paid):** ~$95-220/month

---

## Troubleshooting

### Common Issues During Setup

**Build fails on Vercel:**
- Check all environment variables are set
- Verify DATABASE_URL is accessible from Vercel
- Check build logs for specific errors

**Local dev server won't start:**
- Run `pnpm install` to ensure dependencies are installed
- Check all `.env.local` files exist
- Verify ports 3000, 3001, 3002 are available

**Database connection fails:**
- Verify DATABASE_URL format is correct
- Check Neon dashboard - database should be "Active"
- Ensure IP allowlist includes 0.0.0.0/0 for Vercel

**Webhooks not firing:**
- Verify webhook URLs point to production (not localhost)
- Check webhook signing secrets are correct
- Review webhook logs in service dashboards

---

## Quick Decision Tree

### Should I use this template?

✅ **YES if:**
- Building a SaaS application
- Need authentication, payments, database
- Want production-ready starter
- Comfortable with Next.js/React

❌ **NO if:**
- Simple static site (use plain Next.js)
- Different framework (Vue, Svelte, etc.)
- Not using TypeScript
- Want minimal dependencies

### Should I swap to Convex?

✅ **YES if:**
- Want simpler architecture
- Need real-time features
- Haven't launched yet (easier migration)
- Comfortable with TypeScript

### Should I swap to WorkOS?

✅ **YES if:**
- Building B2B enterprise SaaS
- Need SSO/SAML
- Want full control over auth UI

❌ **NO if:**
- Building B2C
- Want pre-built auth components
- Small startup (Clerk is easier)

---

## Support

- **Template Issues:** https://github.com/vercel/next-forge/issues
- **Next.js Docs:** https://nextjs.org/docs
- **next-forge Docs:** https://www.next-forge.com/docs

---

## Version

Template Version: next-forge 5.2.1
Documentation Last Updated: 2025-11-23

---

## Next Steps

1. Copy this template to your new project directory
2. Run through deployment guides (HANDOFF-*.md files)
3. Evaluate component swaps (COMPONENT-SWAP-MATRIX.md)
4. Document your specific setup in your project's `.claude/` directory
5. Consider contributing improvements back to this template

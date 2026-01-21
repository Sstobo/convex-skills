# Part 12: Development & Deployment

### Development Commands

```bash
# Start full-stack development
pnpm dev

# Start frontend only (port 3000)
pnpm dev:web

# Start Convex backend only
pnpm dev:convex

# Type check
pnpm lint

# Format code
pnpm format
```

### Convex CLI Commands

```bash
# Development mode (watches for changes)
npx convex dev

# Deploy to production
npx convex deploy

# Set environment variable
npx convex env set KEY value

# List environment variables
npx convex env list

# Import data
npx convex import --table tableName data.jsonl

# Export data
npx convex export --table tableName > data.jsonl

# Run a function
npx convex run module:function '{"arg": "value"}'
```

### Vercel Deployment

**vercel.json:**

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "buildCommand": "npx convex deploy --cmd 'npm run build'"
}
```

**Environment Variables (Vercel Dashboard):**

```
CONVEX_DEPLOY_KEY=prod:...
VITE_CONVEX_URL=https://your-deployment.convex.cloud
VITE_CONVEX_SITE_URL=https://your-deployment.convex.site
VITE_SITE_URL=https://your-domain.com
```

### Production Checklist

- [ ] Set `BETTER_AUTH_SECRET` in Convex env
- [ ] Set `SITE_URL` to production domain
- [ ] Configure custom domain in Convex dashboard
- [ ] Enable rate limiting for public mutations
- [ ] Set up error monitoring (Sentry, etc.)
- [ ] Configure backup schedule

---


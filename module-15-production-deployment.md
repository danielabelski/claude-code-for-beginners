<div align="center">

![Module 15](https://img.shields.io/badge/Module_15-a8502f?style=for-the-badge&labelColor=221c17)
![Time](https://img.shields.io/badge/⏱_60_min-3b3029?style=for-the-badge&labelColor=221c17)
![Difficulty](https://img.shields.io/badge/Advanced-a8502f?style=for-the-badge&labelColor=221c17)

# Production Deployment

**Deploy your applications to production with Claude Code's assistance**

[← Previous](module-14-api-integration.md) · [🏠 Home](README.md)

</div>

---

## What You'll Learn

The full arc of getting your app out into the world - deployment fundamentals and platform choices (Vercel, Heroku, VPS), Docker containerization, CI/CD pipelines, environment configuration, monitoring, logging, security, and handling database migrations in production without losing sleep. Plus the shortcut for when you don't need a server at all: publishing a live page straight from your session as an artifact.

---

## Lesson 1: Deployment Fundamentals

### What is Deployment?

Deployment means making your application available on the internet so other people can use it. That's it. Everything else in this module is about doing that *well*.

Your app behaves differently on your laptop than on a production server, and understanding the gap is what separates a working demo from a reliable product.

**Development** is your local playground - you're the only user, debug mode is on, you're on test data, and breaking things is fine. **Production** is the real thing - real users, real data, no debug info exposed, and it has to stay reliable and fast. Treat them differently.

---

### Deployment Checklist

Before you deploy, run through this with Claude Code:

```
Help me prepare this application for production:
1. Check all environment variables are documented
2. Verify no secrets in code
3. Ensure error handling doesn't leak sensitive info
4. Add proper logging
5. Set up health check endpoint
6. Configure CORS properly
7. Add rate limiting
8. Set up HTTPS
9. Minify/bundle code
10. Run all tests
```

---

## Lesson 2: Deploying to Vercel

### What is Vercel?

Vercel is a platform tuned for frontend frameworks - Next.js, React, Vue, Svelte, static sites, serverless functions. If you're building with Next.js especially, Vercel is the path of least resistance.

---

### Deploying with Claude Code

```
Help me deploy this Next.js application to Vercel:
1. Install Vercel CLI
2. Configure vercel.json
3. Set up environment variables
4. Deploy to production
5. Set up custom domain
```

Claude Code walks you through the CLI commands:

```bash
# Install Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy
vercel --prod
```

And writes a `vercel.json` for you:

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "env": {
    "DATABASE_URL": "@database-url",
    "API_KEY": "@api-key"
  }
}
```

---

## Lesson 3: Deploying to Heroku

### What is Heroku?

Heroku is aimed at backend and full-stack apps - Node.js APIs, Python/Django, Ruby on Rails, that sort of thing. It hides most of the server management, which is a relief when you're starting out.

---

### Deploying with Claude Code

```
Help me deploy this Express API to Heroku:
1. Create Procfile
2. Set up heroku.yml if needed
3. Configure environment variables
4. Add PostgreSQL database
5. Set up deployment
6. Configure logging
```

Claude Code creates your Procfile:

```
web: node dist/server.js
```

Then guides you through the deploy commands:

```bash
# Install Heroku CLI first, then:

# Login
heroku login

# Create app
heroku create my-app-name

# Add PostgreSQL
heroku addons:create heroku-postgresql:essential-0

# Set environment variables
heroku config:set NODE_ENV=production
heroku config:set API_KEY=your-key

# Deploy
git push heroku main

# View logs
heroku logs --tail
```

---

## Lesson 4: Deploying to a VPS (Virtual Private Server)

### What is a VPS?

A VPS is your own server - DigitalOcean, Linode, AWS EC2. You get full control over everything, which is both the upside and the downside. Great for custom configs, running several apps, or just learning how servers actually work. Fair warning: more setup than a managed platform.

---

### Setting Up VPS Deployment

```
Help me deploy this Node.js app to an Ubuntu VPS:
1. Set up the server (Node.js, PM2, Nginx)
2. Configure firewall
3. Set up SSL with Let's Encrypt
4. Configure Nginx reverse proxy
5. Set up PM2 for process management
6. Configure automatic restarts
7. Set up deployment script
```

Claude Code generates a deployment guide. On the server:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Nginx
sudo apt install -y nginx

# Configure firewall
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

Nginx configuration:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

PM2 ecosystem file:

```javascript
module.exports = {
  apps: [{
    name: 'my-app',
    script: './dist/server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
};
```

---

## Lesson 5: Docker Deployment

### Why Docker?

Docker packages your app with everything it needs to run - runtime, dependencies, system libraries, all of it. So it runs the same on your laptop, in CI, and in production. No more "works on my machine." It also makes scaling and isolation much simpler.

---

### Creating Docker Configuration

```
Create Docker configuration for this Node.js API:
1. Write Dockerfile
2. Create .dockerignore
3. Write docker-compose.yml for local development
4. Add production docker-compose
5. Include health checks
```

**Dockerfile:**
```dockerfile
# Build stage
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist

# Security: run as non-root
USER node

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**.dockerignore:**
```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
.vscode
coverage
.DS_Store
```

**docker-compose.yml:**
```yaml
# docker-compose.yml (Compose V2 - no version key needed)

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## Lesson 6: CI/CD Pipelines

### What is CI/CD?

CI is Continuous Integration - automatically testing your code whenever changes get pushed. CD is Continuous Deployment - automatically shipping code that passes those tests. Together they catch bugs early and let you deploy faster without the manual grind of doing it by hand every time.

---

### GitHub Actions for CI/CD

```
Create a GitHub Actions workflow that:
1. Runs on every push to main
2. Runs all tests
3. Runs linter
4. Builds the application
5. Deploys to production if tests pass
6. Sends notifications on failure
```

**.github/workflows/deploy.yml:**
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          ./scripts/deploy.sh
```

---

## Lesson 7: Environment Configuration

### Managing Environment Variables

This matters more than it sounds. Misconfigured environment variables are behind a surprising share of production incidents - a missing database URL, a wrong API key, a secret accidentally committed to git.

```
Help me set up proper environment configuration:
1. Create .env.example template
2. Document all environment variables
3. Set up different configs for dev/staging/prod
4. Add validation for required variables
5. Secure sensitive values
```

**config/env.ts:**
```typescript
import dotenv from 'dotenv';

dotenv.config();

interface Config {
  nodeEnv: string;
  port: number;
  databaseUrl: string;
  jwtSecret: string;
  apiKey: string;
}

function validateEnv(): Config {
  const required = [
    'DATABASE_URL',
    'JWT_SECRET',
    'API_KEY'
  ];

  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }

  return {
    nodeEnv: process.env.NODE_ENV || 'development',
    port: parseInt(process.env.PORT || '3000'),
    databaseUrl: process.env.DATABASE_URL!,
    jwtSecret: process.env.JWT_SECRET!,
    apiKey: process.env.API_KEY!
  };
}

export const config = validateEnv();
```

The idea: validate early. If a required variable is missing, the app should crash immediately at startup with a clear message - not fail mysteriously ten minutes later when some request tries to hit the database.

---

## Lesson 8: Monitoring and Logging

### Setting Up Production Logging

Once your app is live, you can't just `console.log` and squint at your terminal. You need structured, leveled, persistent logging.

```
Add production-grade logging:
1. Use structured logging (JSON format)
2. Different log levels (error, warn, info, debug)
3. Log rotation
4. Send errors to monitoring service
5. Don't log sensitive data
```

Using Winston:

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    // Write all logs to console
    new winston.transports.Console({
      format: winston.format.simple()
    }),
    // Write errors to error.log
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error'
    }),
    // Write all logs to combined.log
    new winston.transports.File({
      filename: 'logs/combined.log'
    })
  ]
});

export default logger;
```

---

### Health Checks

Health check endpoints tell your infrastructure whether your app is actually working - not just running, but ready to handle requests.

```
Add health check endpoints:
1. /health - basic health check
2. /health/ready - readiness check (dependencies)
3. /health/live - liveness check
```

```typescript
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

app.get('/health/ready', async (req, res) => {
  try {
    // Check database
    await db.query('SELECT 1');

    res.json({
      status: 'ready',
      checks: {
        database: 'ok'
      }
    });
  } catch (error) {
    res.status(503).json({
      status: 'not ready',
      checks: {
        database: 'failed'
      }
    });
  }
});
```

---

## Lesson 9: Database Migrations in Production

### Safe Database Migrations

This one can bite. Schema changes on a live database with real data are among the riskiest parts of any deployment. Always have a rollback plan, and always test on staging first.

```
Create a safe database migration strategy:
1. Version all schema changes
2. Write up and down migrations
3. Test migrations on staging first
4. Backup before migrating
5. Have rollback plan
```

Using a migration tool:

```typescript
// migrations/001_create_users_table.ts
export async function up(db) {
  await db.schema.createTable('users', (table) => {
    table.increments('id').primary();
    table.string('email').notNullable().unique();
    table.string('password_hash').notNullable();
    table.timestamps(true, true);
  });
}

export async function down(db) {
  await db.schema.dropTable('users');
}
```

Running migrations:

```bash
# Backup database first!
pg_dump mydb > backup_$(date +%Y%m%d).sql

# Run migrations
npm run migrate:latest

# If something goes wrong, rollback
npm run migrate:rollback
```

---

## Lesson 10: Share Without Deploying - Artifacts

### Not Everything Needs a Server

Everything above is for software people will actually *use*. But a lot of what you want to show someone isn't an app - it's a dashboard of last week's errors, a PR walkthrough, three layout options side by side, a migration checklist. Spinning up Vercel for that is overkill.

For those, Claude Code can publish an **artifact**: a live web page on claude.ai, built from whatever your session can reach - your code, your data, your connected tools. No deploy, no hosting, no backend. Just ask:

```text
Make an artifact that walks through this PR with the diff annotated inline.
```

```text
Build a dashboard artifact of last week's deploy failures by service and keep it updated as you investigate.
```

Claude writes the page, publishes it, prints the URL, and opens it in your browser.

### How It Behaves

- **Private by default.** A new artifact is visible only to you. Share it from the **Share** button in the page header - within your organization on Team/Enterprise, or as a public link.
- **Updates in place.** Ask Claude to change it and it republishes to the *same* URL. Anyone with the page open sees the new version. Every publish is kept as a version.
- **Findable later.** `/artifacts` lists everything you've published, even after `/clear` or in a new session. Pass Claude an artifact's URL to update it from a different session.
- **Comments (Team/Enterprise).** Teammates can comment on a shared page, and Claude can read the threads and make the changes they ask for.

### Good Uses

```text
Make an artifact with four different layouts for the settings panel, as a grid with a one-line tradeoff under each.
```

```text
Turn this migration plan into a checklist artifact. Check items off as you complete them.
```

```text
Make a triage board artifact with each open issue as a draggable card across Now, Next, Later, Cut. Add a "Copy as prompt" button so I can paste the final order back here.
```

That last one is a neat trick: the page becomes a tiny editor, and the result flows back into your session.

There are two shortcuts for common formats: `/slides <brief>` makes a presentation you can edit and export, and `/design <brief>` drafts a visual mockup on a canvas you can tweak by hand.

### What It Isn't

An artifact is **one self-contained page with no backend**. It can't have multiple routes, log users in, or store a database of its own. External requests are locked down too - scripts only from a few public CDNs, no outside images. If you're building something people log into, that's the rest of this module.

Also worth knowing: artifacts need a Pro, Max, Team, or Enterprise plan and a session signed in with `/login` - they don't work when you're running on an API key. And a styled page costs more tokens than the same content as terminal text, so ask for one when seeing beats reading, not by default.

---

## Hands-On Practice

### Exercise 1: Deploy Full-Stack App

**Task:** Deploy a complete application

```
Take the project from Module 9 and:
1. Prepare it for production
2. Set up environment configuration
3. Add health check endpoints
4. Create Docker configuration
5. Deploy to your choice of platform
6. Set up monitoring
7. Test in production
```

---

### Exercise 2: CI/CD Pipeline

**Task:** Set up automated deployment

```
Create a CI/CD pipeline that:
1. Runs tests on every PR
2. Deploys to staging on merge to develop
3. Deploys to production on merge to main
4. Sends notifications on failure
5. Includes database migrations
```

---

### Exercise 3: Zero-Downtime Deployment

**Task:** Implement rolling deployment

```
Set up deployment strategy that:
1. Keeps old version running during deploy
2. Gradually shifts traffic to new version
3. Monitors for errors
4. Automatically rolls back if issues detected
5. Ensures zero downtime
```

---

## Module 15 Checklist

That wraps the final module. Make sure you can:

- [ ] Prepare applications for production
- [ ] Deploy to various platforms
- [ ] Use Docker for containerization
- [ ] Set up CI/CD pipelines
- [ ] Configure environments properly
- [ ] Implement monitoring and logging
- [ ] Handle database migrations safely
- [ ] Follow security best practices
- [ ] Know when an artifact beats a real deployment, and publish one

---

## Production Deployment Best Practices

**Before every deployment:**
- Run all tests
- Review changes
- Backup the database
- Have a rollback plan
- Deploy during low-traffic times when possible
- Monitor closely after deployment
- Use hooks to automate pre-deployment checks (linting, tests, security scans)

**Security** -- these aren't optional:
- Use HTTPS everywhere
- Keep dependencies updated
- Never expose error details to users
- Use security headers
- Rate limit your APIs
- Validate all input

**Performance:**
- Minify and bundle assets
- Enable compression
- Use a CDN for static files
- Cache appropriately
- Monitor performance over time

---

## Automating Deployment Workflows

As your deployment process matures, codify it in a CLAUDE.md and use hooks to enforce it:

```markdown
# CLAUDE.md (deployment section)
## Deployment Checklist
- All tests must pass before deploying
- Run security audit: npm audit
- Check for environment variable documentation
- Database migrations must be tested on staging first
- Tag releases with semantic versioning
```

You can also set up a hook that automatically runs linting after Claude Code edits files, catching issues before they ever reach your deployment pipeline:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "npm run lint --quiet 2>/dev/null || true"
      }]
    }]
  }
}
```

This kind of automation is what separates a hobby project from a production-grade workflow. We covered hooks in detail in Module 12 - go back and review if you want to set this up for your deployments.

---

## Wrapping Up the Course

You made it through the entire Claude Code for Beginners course. You can now build applications with Claude Code, write clean code, test and debug it, and deploy to production on solid practices.

What comes next is simple: build things. Real projects - not just tutorials - are where all of this clicks into place. Contribute to open source if that's your thing. Share what you learned. And keep going.

> **Keep going.** The [Advanced Modules](https://payhip.com/b/8E107) pick up right where this course ends -- production Kubernetes deployments, multi-agent systems, enterprise integration, and performance optimization. The [Real Projects Pack](https://payhip.com/b/dFXWO) gives you 14 complete project templates to practice on. [Bundle both and save $10.](https://payhip.com/b/S8nU1)

---

*Course complete. Go build something.*

---

<div align="center">

[← Previous Module](module-14-api-integration.md) · [🏠 Home](README.md) · [🏆 Course Complete!]

</div>

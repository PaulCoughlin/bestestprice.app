# 🚀 Development Environment Setup Guide

Complete guide for setting up your development environment for **Bestest Price** with Vercel, PostgreSQL, and Next.js.

---

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Development Setup](#local-development-setup)
3. [Vercel Account Setup](#vercel-account-setup)
4. [Database Setup (Vercel Postgres)](#database-setup)
5. [Email Service Setup (Resend)](#email-service-setup)
6. [Project Configuration](#project-configuration)
7. [Running Locally](#running-locally)
8. [Deployment to Vercel](#deployment-to-vercel)
9. [Testing & Debugging](#testing--debugging)
10. [Common Issues & Solutions](#common-issues--solutions)

---

## Prerequisites

### Required Software

1. **Node.js 18+** (LTS version recommended)
   ```bash
   # Check your version
   node --version

   # Download from: https://nodejs.org/
   ```

2. **Git** (for version control)
   ```bash
   # Check if installed
   git --version

   # Download from: https://git-scm.com/
   ```

3. **VS Code** (recommended) or your preferred editor
   - Download from: https://code.visualstudio.com/
   - Recommended extensions:
     - ES7+ React/Redux/React-Native snippets
     - Tailwind CSS IntelliSense
     - Prettier - Code formatter
     - ESLint

### Required Accounts

1. **GitHub Account** (you already have this!)
   - Your repo: https://github.com/PaulCoughlin/bestestprice.app

2. **Vercel Account** (free tier is perfect for development)
   - Sign up: https://vercel.com/signup
   - **Important**: Sign up with your GitHub account for seamless integration

3. **Resend Account** (for sending emails)
   - Sign up: https://resend.com/signup
   - Free tier: 100 emails/day (perfect for development)

---

## Local Development Setup

### Step 1: Clone Your Repository

```bash
# Clone the repository
git clone https://github.com/PaulCoughlin/bestestprice.app.git

# Navigate into the directory
cd bestestprice.app
```

### Step 2: Create Next.js Project Structure

Since we only have documentation so far, we need to initialize the Next.js project:

```bash
# Initialize Next.js with TypeScript, Tailwind, and App Router
npx create-next-app@latest app --typescript --tailwind --app --no-src-dir --import-alias "@/*"

# This creates an 'app' directory with the Next.js project
```

**Important**: We'll create the app in a subdirectory to keep our documentation files at the root.

### Step 3: Install Dependencies

```bash
# Navigate to the app directory
cd app

# Install all required dependencies
npm install @vercel/postgres bcrypt jsonwebtoken resend puppeteer cheerio axios recharts lucide-react next-themes

# Install Radix UI components
npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu @radix-ui/react-select @radix-ui/react-toast

# Install utility libraries
npm install tailwind-merge clsx class-variance-authority zod date-fns uuid

# Install dev dependencies
npm install -D @types/bcrypt @types/jsonwebtoken @types/uuid
```

### Step 4: Initialize Shadcn UI

```bash
# Still in the app directory
npx shadcn-ui@latest init

# When prompted, choose:
# - Style: Default
# - Base color: Slate
# - CSS variables: Yes

# Add commonly used components
npx shadcn-ui@latest add button input dialog dropdown-menu card badge select toast skeleton
```

---

## Vercel Account Setup

### Step 1: Create Vercel Account

1. Go to https://vercel.com/signup
2. Click "Continue with GitHub"
3. Authorize Vercel to access your GitHub account
4. Complete the signup process

### Step 2: Install Vercel CLI

The Vercel CLI allows you to develop and deploy from your terminal:

```bash
# Install Vercel CLI globally
npm install -g vercel

# Login to Vercel
vercel login

# Follow the prompts to authenticate
```

### Step 3: Link Your Project to Vercel

```bash
# From your app directory
cd app

# Link to Vercel (creates a new project)
vercel link

# When prompted:
# - Set up and deploy? No (we'll do this manually)
# - Link to existing project? No
# - What's your project's name? bestestprice-app (or your choice)
# - In which directory is your code located? ./ (current directory)
```

This creates a `.vercel` directory with your project configuration.

---

## Database Setup

### Step 1: Create Vercel Postgres Database

1. **Via Vercel Dashboard** (easiest for first time):
   - Go to https://vercel.com/dashboard
   - Click on your project ("bestestprice-app")
   - Go to the "Storage" tab
   - Click "Create Database"
   - Choose "Postgres"
   - Choose a region close to you (e.g., "US East" if you're in North America)
   - Click "Create"

2. **Via Vercel CLI** (alternative):
   ```bash
   vercel env add POSTGRES_URL production
   # This will guide you through creating a database
   ```

### Step 2: Get Database Connection Strings

1. In Vercel Dashboard → Your Project → Storage → Postgres Database
2. Click on your database
3. Go to the ".env.local" tab
4. Copy all the environment variables (you'll need these):
   - `POSTGRES_URL`
   - `POSTGRES_PRISMA_URL`
   - `POSTGRES_URL_NON_POOLING`
   - `POSTGRES_USER`
   - `POSTGRES_HOST`
   - `POSTGRES_PASSWORD`
   - `POSTGRES_DATABASE`

### Step 3: Create Database Tables

We'll create the tables using the Vercel Postgres dashboard:

1. In your database view, click on the "Query" tab
2. Copy and paste the SQL schema from `PROJECT.md` (lines 82-182)
3. Run the queries one by one or all at once
4. Verify tables are created in the "Data" tab

**Pro Tip**: Save the schema as `schema.sql` in your project for future reference.

---

## Email Service Setup

### Step 1: Create Resend Account

1. Go to https://resend.com/signup
2. Sign up with your email
3. Verify your email address

### Step 2: Get API Key

1. In Resend Dashboard: https://resend.com/api-keys
2. Click "Create API Key"
3. Name it: "Bestest Price Development"
4. Select permissions: "Sending access"
5. Click "Create"
6. **Copy the API key immediately** (you can't see it again!)

### Step 3: Verify Your Domain (Optional for MVP)

For development, you can use Resend's test email feature:
- From: `onboarding@resend.dev`
- To: Your email address

For production, you'll need to verify your domain:
1. Add your domain in Resend
2. Add DNS records to your domain provider
3. Wait for verification

---

## Project Configuration

### Step 1: Create Environment Variables File

Create `.env.local` in your `app` directory:

```bash
cd app
touch .env.local
```

### Step 2: Add Environment Variables

Open `.env.local` and add all the following:

```env
# ============================================
# DATABASE (from Vercel Postgres)
# ============================================
POSTGRES_URL="paste_from_vercel"
POSTGRES_PRISMA_URL="paste_from_vercel"
POSTGRES_URL_NON_POOLING="paste_from_vercel"
POSTGRES_USER="paste_from_vercel"
POSTGRES_HOST="paste_from_vercel"
POSTGRES_PASSWORD="paste_from_vercel"
POSTGRES_DATABASE="paste_from_vercel"

# ============================================
# AUTHENTICATION
# ============================================
JWT_SECRET="your-super-secret-jwt-key-at-least-32-characters-long"
JWT_EXPIRES_IN="7d"

# ============================================
# EMAIL (Resend)
# ============================================
RESEND_API_KEY="re_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
EMAIL_FROM="onboarding@resend.dev"
EMAIL_FROM_NAME="Bestest Price"

# ============================================
# APPLICATION
# ============================================
NEXT_PUBLIC_APP_URL="http://localhost:3000"
NEXT_PUBLIC_APP_NAME="Bestest Price Tracker"

# ============================================
# CRON PROTECTION
# ============================================
CRON_SECRET="generate-a-random-secret-string-here"

# ============================================
# SCRAPER CONFIGURATION
# ============================================
SCRAPER_TIMEOUT=30000
SCRAPER_MAX_RETRIES=3
```

### Step 3: Generate Secure Secrets

Use these commands to generate secure secrets:

```bash
# Generate JWT_SECRET (32 characters)
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Generate CRON_SECRET
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Copy these into your `.env.local` file.

### Step 4: Add .env.local to .gitignore

Make sure `.env.local` is in your `.gitignore` (it should be by default with Next.js):

```bash
# Check if it's there
cat .gitignore | grep .env.local

# If not, add it
echo ".env.local" >> .gitignore
```

---

## Running Locally

### Step 1: Start Development Server

```bash
# From the app directory
cd app

# Start the dev server
npm run dev
```

You should see:
```
✓ Ready in 2.5s
○ Local: http://localhost:3000
```

### Step 2: Open in Browser

Open http://localhost:3000 in your browser.

You should see the default Next.js welcome page (we'll build our app in the next phases).

### Step 3: Verify Hot Reload

1. Edit `app/page.tsx`
2. Save the file
3. Browser should automatically refresh with changes

---

## Deployment to Vercel

### Deploy via Git Push (Recommended)

Vercel automatically deploys when you push to GitHub:

```bash
# Make changes to your code
git add .
git commit -m "Your commit message"
git push origin main

# Vercel will automatically:
# 1. Detect the push
# 2. Build your project
# 3. Deploy to production
# 4. Give you a URL (e.g., bestestprice-app.vercel.app)
```

### Deploy via Vercel CLI (For Testing)

```bash
# Deploy to preview environment
vercel

# Deploy to production
vercel --prod
```

### First Deployment Setup

When you deploy for the first time:

1. **Add Environment Variables to Vercel**:
   - Go to Vercel Dashboard → Your Project → Settings → Environment Variables
   - Add all variables from your `.env.local`
   - Make sure to add them for all environments (Production, Preview, Development)

2. **Configure Build Settings** (usually automatic):
   - Build Command: `npm run build`
   - Output Directory: `.next`
   - Install Command: `npm install`
   - Framework Preset: Next.js

---

## Testing & Debugging

### Testing Database Connection

Create a test API route to verify your database connection:

```typescript
// app/api/test-db/route.ts
import { sql } from '@vercel/postgres';

export async function GET() {
  try {
    const result = await sql`SELECT NOW()`;
    return Response.json({
      success: true,
      timestamp: result.rows[0].now
    });
  } catch (error) {
    return Response.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

Test it: http://localhost:3000/api/test-db

### Testing Email Service

Create a test API route for Resend:

```typescript
// app/api/test-email/route.ts
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

export async function POST(request: Request) {
  const { to } = await request.json();

  try {
    const data = await resend.emails.send({
      from: process.env.EMAIL_FROM!,
      to: [to],
      subject: 'Test Email from Bestest Price',
      html: '<p>If you received this, your email setup is working!</p>'
    });

    return Response.json({ success: true, id: data.id });
  } catch (error) {
    return Response.json({ success: false, error: error.message }, { status: 500 });
  }
}
```

Test with curl:
```bash
curl -X POST http://localhost:3000/api/test-email \
  -H "Content-Type: application/json" \
  -d '{"to":"your-email@example.com"}'
```

### Viewing Logs

**Local Development**:
- Console logs appear in your terminal
- `console.log()` statements show up immediately

**Vercel Production**:
- View logs in Vercel Dashboard → Your Project → Logs
- Or use CLI: `vercel logs`

### Using Vercel Dev

For better local development that mimics production:

```bash
# Use Vercel dev instead of npm run dev
vercel dev

# This gives you:
# - Access to Vercel environment variables
# - Edge functions locally
# - Better environment parity
```

---

## Common Issues & Solutions

### Issue: "Module not found" errors

**Solution**:
```bash
# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Issue: Environment variables not working

**Solutions**:
1. Restart your dev server after changing `.env.local`
2. Verify variables don't have quotes around values
3. Use `NEXT_PUBLIC_` prefix for client-side variables
4. Check for typos in variable names

### Issue: Database connection fails

**Solutions**:
1. Verify your IP is allowed (Vercel Postgres allows all by default)
2. Check your connection string is correct
3. Ensure you're using the correct URL for your use case:
   - `POSTGRES_URL`: General purpose with pooling
   - `POSTGRES_URL_NON_POOLING`: Direct connections (e.g., migrations)
   - `POSTGRES_PRISMA_URL`: If using Prisma ORM

### Issue: Puppeteer not working on Vercel

**Solution**:
Vercel has Chrome built-in for Edge Functions. Use this config:

```typescript
// Use this in your scraper
const browser = await puppeteer.launch({
  headless: true,
  args: ['--no-sandbox', '--disable-setuid-sandbox']
});
```

For Vercel, you may need to use `chrome-aws-lambda` instead:
```bash
npm install chrome-aws-lambda
```

### Issue: "Can't resolve 'fs'" error

**Solution**:
This happens with Puppeteer. Add to `next.config.js`:

```javascript
module.exports = {
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
        net: false,
        tls: false,
      };
    }
    return config;
  },
};
```

### Issue: Build fails on Vercel

**Solutions**:
1. Check build logs in Vercel Dashboard
2. Ensure all environment variables are set
3. Try building locally first: `npm run build`
4. Check for TypeScript errors: `npx tsc --noEmit`

---

## Development Workflow

### Daily Development Flow

```bash
# 1. Pull latest changes
git pull origin main

# 2. Create a new branch for your feature
git checkout -b feature/your-feature-name

# 3. Start development server
cd app
npm run dev

# 4. Make your changes and test

# 5. Commit and push
git add .
git commit -m "Description of changes (v0.1.x)"
git push origin feature/your-feature-name

# 6. Create PR on GitHub and merge to main

# 7. Vercel automatically deploys main branch
```

### Version Control Best Practices

1. **Always update version numbers** in commits:
   - README.md
   - index.html
   - Future: package.json

2. **Use meaningful commit messages**:
   ```
   ✅ Good: "Add user authentication with JWT (v0.2.0)"
   ❌ Bad: "updates"
   ```

3. **Test locally before pushing**:
   ```bash
   npm run build  # Verify build works
   npm run lint   # Check for linting errors
   ```

---

## Useful Commands Cheatsheet

```bash
# Development
npm run dev              # Start dev server
npm run build            # Build for production
npm run start            # Start production server
npm run lint             # Run ESLint

# Vercel
vercel                   # Deploy to preview
vercel --prod            # Deploy to production
vercel env ls            # List environment variables
vercel logs              # View production logs
vercel dev               # Run with Vercel environment

# Git
git status               # Check status
git add .                # Stage all changes
git commit -m "message"  # Commit changes
git push                 # Push to GitHub
git pull                 # Pull latest changes

# Database (example with psql if you want direct access)
psql $POSTGRES_URL       # Connect to database directly
```

---

## Next Steps

Now that your environment is set up:

1. ✅ You have Node.js, Git, and VS Code installed
2. ✅ Your Vercel account is connected to GitHub
3. ✅ Database is created and ready
4. ✅ Email service is configured
5. ✅ Local development environment is running

**You're ready to start Phase 1: Authentication System!**

See `PROJECT.md` for the complete implementation roadmap.

---

## Additional Resources

### Documentation
- **Next.js**: https://nextjs.org/docs
- **Vercel**: https://vercel.com/docs
- **Vercel Postgres**: https://vercel.com/docs/storage/vercel-postgres
- **Resend**: https://resend.com/docs
- **Tailwind CSS**: https://tailwindcss.com/docs
- **Shadcn UI**: https://ui.shadcn.com

### Community & Support
- **Next.js Discord**: https://nextjs.org/discord
- **Vercel Discord**: https://vercel.com/discord
- **Stack Overflow**: Tag questions with `nextjs`, `vercel`, `postgresql`

### Tools
- **Postman**: https://www.postman.com/ (API testing)
- **TablePlus**: https://tableplus.com/ (Database GUI)
- **DevTools**: Built into Chrome/Firefox (debugging)

---

**Happy Coding! 🚀**

If you run into any issues not covered here, check the Vercel documentation or feel free to ask!

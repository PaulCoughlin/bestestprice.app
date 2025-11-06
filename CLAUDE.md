# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bestest Price** is a SaaS competitor price tracking application currently in the planning/design phase. The project is set up as a LocalWP environment but is being designed to be built with Next.js 14, TypeScript, and PostgreSQL (transitioning away from WordPress/MySQL).

**Current Status**: The codebase contains detailed project specifications and a landing page, but the actual application has not been implemented yet.

## Architecture & Tech Stack

### Planned Technology Stack
- **Frontend**: Next.js 14 (App Router), React, TypeScript
- **Styling**: Tailwind CSS, Shadcn UI components, Lucide icons
- **Backend**: Next.js API Routes, Vercel Serverless Functions
- **Database**: PostgreSQL (Vercel Postgres) - NOT MySQL/WordPress
- **Authentication**: Custom JWT-based auth with httpOnly cookies, bcrypt password hashing
- **Email**: Resend API for transactional emails
- **Web Scraping**: Puppeteer (dynamic sites), Cheerio (static HTML)
- **Charts**: Recharts library for price history visualization
- **Theming**: next-themes for dark/light mode
- **Deployment**: Vercel with cron jobs for scheduled price checks

### Key Design Decisions

1. **Database**: Uses PostgreSQL with nested category structure (parent_id self-reference)
2. **Authentication**: Custom JWT implementation, NOT NextAuth or third-party auth
3. **Scraping Strategy**: Puppeteer for JS-heavy sites, Cheerio fallback for static HTML
4. **Price Extraction**: Custom parser handling £, $, € with various number formats (1,234.56 vs 1.234,56)
5. **Architecture**: Multi-user SaaS with complete data isolation per user

## Project Structure (Planned)

```
src/
├── app/                          # Next.js App Router
│   ├── (auth)/                  # Auth pages (login, register, verify, password reset)
│   ├── (dashboard)/             # Main app pages (protected routes)
│   ├── api/                     # API routes
│   │   ├── auth/               # Authentication endpoints
│   │   ├── prices/             # Price tracking CRUD
│   │   ├── categories/         # Category management
│   │   ├── scraper/            # Scraping utilities (proxy, test-selector)
│   │   └── cron/               # Scheduled jobs
│   └── globals.css
├── components/
│   ├── ui/                     # Shadcn components
│   ├── dashboard/              # Dashboard-specific (PriceList, CategoryTree, charts)
│   ├── modals/                 # Modal dialogs
│   ├── auth/                   # Auth forms
│   └── layout/                 # Layout components (Navbar, Sidebar, ThemeToggle)
├── lib/
│   ├── db.ts                   # PostgreSQL connection pool
│   ├── auth.ts                 # JWT session management
│   ├── email.ts                # Resend email service
│   ├── scraper.ts              # Puppeteer/Cheerio logic
│   └── price-extractor.ts      # Currency/price parsing
├── hooks/                       # Custom React hooks
└── types/                       # TypeScript definitions
```

## Database Schema

### Core Tables
- **users**: User accounts with email verification, theme preferences, check time/timezone
- **categories**: Nested folder structure (parent_id, color, icon) for organizing prices
- **tracked_prices**: Price configurations (URL, CSS selector, current price, status)
- **price_history**: Historical price records for trend analysis
- **notifications**: Email notification log (price changes, errors)
- **password_resets**: Password reset token management

**Key Relationships**:
- Categories have self-referential parent_id for nesting
- Tracked prices belong to users and optionally to categories
- All user data is isolated (DELETE CASCADE on user_id)

See [project-outline.md](project-outline.md) lines 82-175 for complete schema.

## Core Features & Workflows

### 1. Visual Element Selection
Users select price elements visually from competitor websites:
- Server-side proxy loads external page via Puppeteer
- JavaScript injection highlights elements on hover
- Click captures CSS selector automatically
- Selector validation ensures reliability
- See [project-outline.md](project-outline.md) lines 466-510

### 2. Price Scraping Engine
Robust scraping with error handling:
- Auto-detects if site requires Puppeteer or can use Cheerio
- 30-second timeout, 3 retry attempts
- User-agent rotation
- Extracts currency (£, $, €) and numeric value
- Handles formats: 1,234.56 / 1.234,56 / 1234.56
- See [project-outline.md](project-outline.md) lines 806-853

### 3. Scheduled Price Checks
Hourly cron job checks prices based on user timezone:
- Vercel Cron hits `/api/cron/daily-check` endpoint
- Protected with CRON_SECRET token
- Queries users needing checks (±30 min window)
- Processes each user's active tracked prices
- Logs to price_history and sends notifications on changes
- See [project-outline.md](project-outline.md) lines 866-920

### 4. Nested Category Organization
Unlimited depth folder structure:
- Categories have parent_id (self-reference)
- Custom pastel colors and Lucide icons
- Expand/collapse tree view
- Organize by company, service type, or both
- Supports drag-and-drop sorting (future)

### 5. Price History Charts
Interactive Recharts visualizations:
- Line/Area charts with gradient fills
- Tooltips with formatted data
- Date range filtering
- Multi-price comparison overlays
- Statistics: lowest, highest, average, trend
- See [project-outline.md](project-outline.md) lines 924-997

## Development Workflow

### Starting Fresh (Not Yet Built)
This project is **not yet implemented**. To begin development:

1. **Create Next.js project**:
   ```bash
   npx create-next-app@latest competitor-price-tracker --typescript --tailwind --app
   cd competitor-price-tracker
   ```

2. **Install dependencies**:
   ```bash
   npm install @vercel/postgres bcrypt jsonwebtoken resend puppeteer cheerio axios recharts lucide-react next-themes
   npm install -D @types/bcrypt @types/jsonwebtoken
   ```

3. **Set up Shadcn UI**:
   ```bash
   npx shadcn-ui@latest init
   npx shadcn-ui@latest add button input dialog dropdown-menu card badge select toast
   ```

4. **Configure environment variables** (`.env.local`):
   ```env
   POSTGRES_URL=...
   JWT_SECRET=...
   RESEND_API_KEY=...
   CRON_SECRET=...
   NEXT_PUBLIC_APP_URL=http://localhost:3000
   ```

5. **Create database schema**: Use SQL from [project-outline.md](project-outline.md) lines 82-175

### Development Commands (Once Built)
```bash
npm run dev          # Start dev server (http://localhost:3000)
npm run build        # Production build
npm run start        # Start production server
npm run lint         # Run ESLint
npm run type-check   # TypeScript type checking
```

### Testing Selectors
```bash
# Test CSS selector on a URL
curl -X POST http://localhost:3000/api/scraper/test-selector \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "selector": ".price"}'
```

## Implementation Phases

The project is designed to be built in 9 phases (see [project-outline.md](project-outline.md)):

1. **Phase 1** (2-3 days): Project setup, authentication system
2. **Phase 2** (2-3 days): UI layout, dark/light mode, settings
3. **Phase 3** (2-3 days): Category management (nested structure)
4. **Phase 4** (3-4 days): Price tracking CRUD operations
5. **Phase 5** (3-4 days): Visual element selector (proxy + highlighting)
6. **Phase 6** (3-4 days): Scraping engine (Puppeteer/Cheerio)
7. **Phase 7** (3-4 days): Scheduled checks & email notifications
8. **Phase 8** (3-4 days): Price history charts & analytics
9. **Phase 9** (2-3 days): Polish, testing, security review

**Total Timeline**: 3-4 weeks

## Security Considerations

- **Passwords**: bcrypt with cost factor 10, minimum 8 characters
- **Sessions**: JWT in httpOnly cookies, SameSite strict
- **SQL Injection**: Always use parameterized queries with PostgreSQL
- **XSS**: Sanitize user input, validate URLs before proxying
- **CSRF**: SameSite cookies, verify Origin header
- **Rate Limiting**:
  - Login: 5 attempts per 15 minutes
  - Registration: 3 accounts per hour per IP
  - API: 100 requests per minute per user
- **Cron Protection**: Verify CRON_SECRET token on `/api/cron/*` endpoints

## Design System

### Colors
**Pastel Category Colors**:
- Soft Pink: #FFE5EC
- Soft Blue: #E0F2FE
- Soft Green: #DCFCE7
- Soft Purple: #F3E8FF
- Soft Yellow: #FEF3C7
- Soft Orange: #FFEDD5

**Semantic Colors**:
- Primary: Blue-600 (actions, links)
- Success: Green-600 (price drops)
- Error: Red-600 (price increases)
- Warning: Amber-600 (alerts)

**Theme Support**: Full dark/light mode with system preference detection

### Component Conventions
- Use Shadcn UI components from `components/ui/`
- All icons from Lucide React
- Loading states: Skeleton loaders, spinners
- Error states: Toast notifications
- Forms: Validation with helpful error messages

## Important Files

- [project-outline.md](project-outline.md) - Complete technical specification (1336 lines)
- [README.md](README.md) - User-facing documentation and setup instructions
- [index.html](index.html) - Landing page (temporary, will be replaced with Next.js)

## Common Patterns

### Authentication Middleware
All dashboard routes require JWT validation:
```typescript
// Example pattern (to be implemented)
const user = await verifyJWT(request);
if (!user) return Response.json({ error: 'Unauthorized' }, { status: 401 });
```

### Database Queries
Use connection pooling, parameterized queries:
```typescript
// Example pattern
import { sql } from '@vercel/postgres';
const result = await sql`SELECT * FROM users WHERE email = ${email}`;
```

### Price Extraction
Currency detection + number parsing:
```typescript
// See project-outline.md lines 806-853 for full implementation
const { currency, price, raw } = extractPrice(text);
```

### Scraping Decision Tree
```typescript
// Determine scraping strategy
const isJavaScriptHeavy = await detectJavaScript(url);
const content = isJavaScriptHeavy
  ? await scrapePuppeteer(url, selector)
  : await scrapeCheerio(url, selector);
```

## Email Notifications

**Templates** (HTML + plain text):
- **Price Change**: Old → New, percentage change, link to dashboard
- **Scraper Error**: Error message, link to edit price configuration

**Service**: Resend API (requires verified domain for production)

**Format**: See [project-outline.md](project-outline.md) lines 602-620 for template example

## Cron Job Configuration

**Vercel Setup** (vercel.json):
```json
{
  "crons": [
    {
      "path": "/api/cron/daily-check",
      "schedule": "0 * * * *"
    }
  ]
}
```

**Protection**: All cron endpoints must verify CRON_SECRET header

## Environment Context

**Current Setup**: LocalWP (Local by Flywheel) development environment
- Located at: `D:\LocalWP\sites\bestestpriceapp\app\public`
- Database: MySQL (local), username: root, password: root
- **Note**: This is for LOCAL DEVELOPMENT ONLY. Production will use Vercel + PostgreSQL

**Migration Path**: This LocalWP setup is temporary. The actual application will be built separately as a standalone Next.js project deployed to Vercel.

## Reference Documentation

All detailed specifications are in [project-outline.md](project-outline.md):
- Database schema: lines 82-175
- Project structure: lines 180-286
- Implementation phases: lines 289-741
- Authentication flow: lines 746-788
- Scraping strategy: lines 790-853
- Cron job logic: lines 856-920
- Chart implementation: lines 922-997
- Environment variables: lines 1001-1038
- Dependencies: lines 1042-1090
- Security considerations: lines 1093-1130

## Key Principles

1. **Multi-user isolation**: All queries must filter by user_id
2. **Security first**: Never skip authentication, always validate input
3. **Error handling**: Graceful failures with user-friendly messages
4. **Performance**: Use connection pooling, caching, pagination
5. **Accessibility**: Keyboard navigation, screen reader support, ARIA labels
6. **Responsive**: Mobile-first design, test on real devices
7. **Type safety**: Leverage TypeScript, avoid 'any'
8. **Testing**: Validate scrapers on multiple page loads (selectors can break)

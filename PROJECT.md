# Competitor Price Tracking SaaS - Project Outline

## Overview
A sleek multi-user SaaS tool for tracking competitor prices with live updates. Users can visually select price elements from websites and receive notifications when prices change.

---

## Core Features

### User Management
- Multi-user system with isolated data per user
- Email/password authentication with verification
- Password reset functionality
- User preferences (check time, timezone)
- **Dark mode & Light mode** with user preference storage

### Price Tracking
- Visual DOM element selection (server-side proxy for MVP)
- Nested folder/category structure (organize by company, service type, or both)
- Configurable check frequency per user (default: daily at 23:50 UTC)
- Currency detection (£ and $ symbols)
- Status indicators (active, error, paused)

### Notifications
- Email alerts on price changes (old → new, percentage change)
- Email alerts on scraper errors
- Notification history log

### Visualization
- **Interactive price history charts** (using Recharts library)
  - Line chart with gradient fills
  - Area chart option
  - Tooltip showing exact prices and dates
  - Zoom and pan capabilities
  - Multiple price comparison on same chart
  - Percentage change indicators
- Detailed history table view
- Price trend indicators (up/down arrows, percentage badges)

### Organization
- Nested categories with unlimited depth
- Custom colors (pastel palette) and icons for categories
- Drag-and-drop sorting (future enhancement)
- Expandable/collapsible folder views
- Visual hierarchy with indentation

---

## Tech Stack

### Frontend
- **Next.js 14** (App Router)
- **Tailwind CSS** for styling
- **Shadcn UI** components (consistent, modern design)
- **Lucide Icons** for all icons
- **Recharts** for advanced data visualization
- **next-themes** for dark/light mode management

### Backend
- **Next.js API Routes**
- **PostgreSQL** database (Vercel Postgres)
- **Puppeteer** for JavaScript-heavy sites
- **Cheerio** for static HTML parsing
- **Resend** for transactional email

### Authentication
- Custom auth implementation
- **bcrypt** for password hashing
- **JWT** tokens in httpOnly cookies
- Email verification flow

### Deployment
- **Vercel** hosting
- PostgreSQL database (Vercel Postgres)
- Serverless functions
- Vercel Cron for scheduled tasks

---

## Database Schema

```sql
-- Users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  verified BOOLEAN DEFAULT FALSE,
  verification_token VARCHAR(255),
  check_time TIME DEFAULT '23:50:00',
  timezone VARCHAR(50) DEFAULT 'UTC',
  theme VARCHAR(10) DEFAULT 'system', -- 'light', 'dark', 'system'
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Password reset tokens
CREATE TABLE password_resets (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL,
  token VARCHAR(255) NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  used BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
CREATE INDEX idx_token ON password_resets(token);
CREATE INDEX idx_expires ON password_resets(expires_at);

-- Categories (nested structure)
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL,
  parent_id INT NULL,
  name VARCHAR(255) NOT NULL,
  color VARCHAR(7) DEFAULT '#E0E0E0', -- Hex color code
  icon VARCHAR(50) DEFAULT 'folder', -- Lucide icon name
  sort_order INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE CASCADE
);
CREATE INDEX idx_user_parent ON categories(user_id, parent_id);

-- Status enum type
CREATE TYPE price_status AS ENUM ('active', 'error', 'paused');

-- Tracked prices
CREATE TABLE tracked_prices (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL,
  category_id INT NULL,
  company_name VARCHAR(255) NOT NULL,
  service_name VARCHAR(255) NOT NULL,
  url TEXT NOT NULL,
  css_selector TEXT NOT NULL,
  current_price DECIMAL(10, 2) NULL,
  currency_symbol VARCHAR(3) NULL,
  last_checked TIMESTAMP NULL,
  status price_status DEFAULT 'active',
  error_message TEXT NULL,
  sort_order INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL
);
CREATE INDEX idx_user_status ON tracked_prices(user_id, status);
CREATE INDEX idx_last_checked ON tracked_prices(last_checked);

-- Price history
CREATE TABLE price_history (
  id SERIAL PRIMARY KEY,
  tracked_price_id INT NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  currency_symbol VARCHAR(3) NULL,
  checked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (tracked_price_id) REFERENCES tracked_prices(id) ON DELETE CASCADE
);
CREATE INDEX idx_tracked_price_date ON price_history(tracked_price_id, checked_at);

-- Notification type enum
CREATE TYPE notification_type AS ENUM ('price_change', 'error');

-- Email notifications log
CREATE TABLE notifications (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL,
  tracked_price_id INT NOT NULL,
  type notification_type NOT NULL,
  old_price DECIMAL(10, 2) NULL,
  new_price DECIMAL(10, 2) NULL,
  percentage_change DECIMAL(5, 2) NULL,
  error_message TEXT NULL,
  sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (tracked_price_id) REFERENCES tracked_prices(id) ON DELETE CASCADE
);
CREATE INDEX idx_user_sent ON notifications(user_id, sent_at);
```

---

## Project Structure

```
competitor-price-tracker/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   ├── register/
│   │   │   │   └── page.tsx
│   │   │   ├── verify/
│   │   │   │   └── page.tsx
│   │   │   ├── forgot-password/
│   │   │   │   └── page.tsx
│   │   │   └── reset-password/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx (main dashboard)
│   │   │   └── settings/
│   │   │       └── page.tsx
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   │   ├── register/route.ts
│   │   │   │   ├── login/route.ts
│   │   │   │   ├── verify/route.ts
│   │   │   │   ├── logout/route.ts
│   │   │   │   ├── forgot-password/route.ts
│   │   │   │   └── reset-password/route.ts
│   │   │   ├── user/
│   │   │   │   ├── profile/route.ts
│   │   │   │   └── preferences/route.ts
│   │   │   ├── categories/
│   │   │   │   ├── route.ts (list, create)
│   │   │   │   └── [id]/route.ts (update, delete)
│   │   │   ├── prices/
│   │   │   │   ├── route.ts (list, create)
│   │   │   │   └── [id]/
│   │   │   │       ├── route.ts (get, update, delete)
│   │   │   │       └── history/route.ts
│   │   │   ├── scraper/
│   │   │   │   ├── proxy/route.ts (load page via proxy)
│   │   │   │   ├── test-selector/route.ts
│   │   │   │   └── check-prices/route.ts
│   │   │   └── cron/
│   │   │       └── daily-check/route.ts
│   │   ├── providers.tsx (ThemeProvider wrapper)
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/ (shadcn components)
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── dropdown-menu.tsx
│   │   │   ├── card.tsx
│   │   │   ├── badge.tsx
│   │   │   ├── select.tsx
│   │   │   ├── toast.tsx
│   │   │   └── ... (other shadcn components)
│   │   ├── dashboard/
│   │   │   ├── PriceList.tsx
│   │   │   ├── CategoryTree.tsx
│   │   │   ├── PriceCard.tsx
│   │   │   ├── PriceHistoryChart.tsx (Recharts implementation)
│   │   │   ├── PriceTrendBadge.tsx
│   │   │   └── EmptyState.tsx
│   │   ├── modals/
│   │   │   ├── AddPriceModal.tsx
│   │   │   ├── EditPriceModal.tsx
│   │   │   ├── AddCategoryModal.tsx
│   │   │   ├── EditCategoryModal.tsx
│   │   │   ├── ElementSelector.tsx
│   │   │   └── PriceHistoryModal.tsx
│   │   ├── auth/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── ForgotPasswordForm.tsx
│   │   └── layout/
│   │       ├── Navbar.tsx
│   │       ├── Sidebar.tsx
│   │       ├── ThemeToggle.tsx
│   │       └── UserMenu.tsx
│   ├── lib/
│   │   ├── db.ts (PostgreSQL connection via @vercel/postgres)
│   │   ├── auth.ts (session management, JWT)
│   │   ├── email.ts (Resend email service)
│   │   ├── scraper.ts (Puppeteer/Cheerio logic)
│   │   ├── price-extractor.ts (parse price from text)
│   │   ├── password.ts (bcrypt utilities)
│   │   └── utils.ts (general utilities)
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useCategories.ts
│   │   ├── usePrices.ts
│   │   └── useTheme.ts
│   └── types/
│       └── index.ts
├── public/
├── .env.local
├── next.config.js
├── tailwind.config.js
├── tsconfig.json
├── components.json (shadcn config)
└── package.json
```

---

## Implementation Phases

### Phase 1: Project Setup & Authentication
**Duration: 2-3 days**

1. Initialize Next.js project with TypeScript
2. Install and configure dependencies:
   - Tailwind CSS
   - Shadcn UI
   - @vercel/postgres
   - bcrypt
   - jsonwebtoken
   - Resend
   - next-themes
3. Set up PostgreSQL database with schema
4. Create database connection utility with connection pooling
5. Implement authentication system:
   - Registration with email verification
   - Login/logout with JWT sessions
   - Password reset flow
   - Protected route middleware
6. Build auth UI:
   - Login page
   - Registration page
   - Verification page
   - Password reset pages

**Deliverables:**
- Working authentication system
- Database structure created
- Protected routes working

---

### Phase 2: Core UI & Theme System
**Duration: 2-3 days**

1. Set up dark/light mode with next-themes
2. Create base layout components:
   - Dashboard layout with sidebar
   - Top navbar (title, settings icon, theme toggle, user menu)
   - Responsive sidebar (collapsible on mobile)
3. Implement theme toggle component
4. Create empty state UI
5. Build settings page:
   - Profile information
   - Password change
   - Check time & timezone selectors
   - Theme preference
6. Style with Tailwind and test both themes

**Design System:**
- **Light Mode Colors:**
  - Background: White (#FFFFFF)
  - Secondary background: Gray-50 (#F9FAFB)
  - Text: Gray-900 (#111827)
  - Borders: Gray-200 (#E5E7EB)
  
- **Dark Mode Colors:**
  - Background: Gray-900 (#111827)
  - Secondary background: Gray-800 (#1F2937)
  - Text: Gray-50 (#F9FAFB)
  - Borders: Gray-700 (#374151)

- **Accent Colors:**
  - Primary (buttons): Blue-600 (light) / Blue-500 (dark)
  - Success: Green-600 (light) / Green-500 (dark)
  - Error: Red-600 (light) / Red-500 (dark)
  - Warning: Amber-600 (light) / Amber-500 (dark)

- **Pastel Colors (for categories):**
  - Soft Pink: #FFE5EC
  - Soft Blue: #E0F2FE
  - Soft Green: #DCFCE7
  - Soft Purple: #F3E8FF
  - Soft Yellow: #FEF3C7
  - Soft Orange: #FFEDD5

**Deliverables:**
- Complete layout system
- Working dark/light mode
- Settings page functional
- Consistent design system

---

### Phase 3: Category Management
**Duration: 2-3 days**

1. Create category API routes:
   - GET /api/categories (list with nested structure)
   - POST /api/categories (create new)
   - PUT /api/categories/[id] (update)
   - DELETE /api/categories/[id] (delete with children handling)
2. Build category tree component:
   - Nested display with indentation
   - Expand/collapse functionality
   - Show icon and color
   - Item count badges
3. Create category modals:
   - Add category modal (with parent selector)
   - Edit category modal
   - Color picker (pastel palette)
   - Icon picker (filtered Lucide icons)
4. Implement drag-and-drop sorting (optional for MVP)

**Category Structure Example:**
```
📁 Voice APIs (blue)
  └─ 📁 Text-to-Speech (pink)
      ├─ 🏢 Competitor A
      └─ 🏢 Competitor B
📁 Video Tools (green)
  └─ 🏢 Competitor C
```

**Deliverables:**
- Working category CRUD
- Visual category tree
- Color and icon customization

---

### Phase 4: Price Tracking - Basic CRUD
**Duration: 3-4 days**

1. Create price API routes:
   - GET /api/prices (list all for user)
   - POST /api/prices (create new tracked price)
   - GET /api/prices/[id] (get single price with history)
   - PUT /api/prices/[id] (update)
   - DELETE /api/prices/[id] (delete)
2. Build price card component:
   - Company name & service name
   - Current price display (with currency)
   - Status badge (active/error/paused)
   - Last checked timestamp
   - Quick actions (edit, delete, view history)
   - Trend indicator (up/down arrow with percentage)
3. Create price list view:
   - Display within category structure
   - Sort options
   - Filter by status
   - Search functionality
4. Build add/edit price modal (manual input for now):
   - Company name input
   - Service name input
   - URL input
   - CSS selector input (textarea)
   - Category selector (nested dropdown)
5. Implement price extraction function:
   - Detect currency symbols (£, $)
   - Extract numeric value
   - Handle various formats (1,234.56, 1234.56, etc.)

**Price Card Design:**
```
┌─────────────────────────────────┐
│ 🏢 Competitor A                  │
│ Basic Plan                    ⚡ │
│                                  │
│ $99.00  ↑ 10%                   │
│ Last checked: 2 hours ago        │
│                                  │
│ [View History] [Edit] [Delete]   │
└─────────────────────────────────┘
```

**Deliverables:**
- Price CRUD operations
- Price card component
- Add/edit functionality
- Price extraction logic

---

### Phase 5: Element Selector (Visual Selection)
**Duration: 3-4 days**

1. Create proxy API route:
   - GET /api/scraper/proxy?url=... 
   - Fetch external page via Puppeteer/Axios
   - Inject highlighting JavaScript
   - Return modified HTML
2. Build element selector component:
   - URL input field
   - Embedded iframe/container for proxied page
   - JavaScript injection for hover highlighting
   - Click handler to capture CSS selector
   - Selector display and validation
3. Implement CSS selector generation:
   - Generate unique selector path
   - Test selector reliability
   - Fallback strategies (multiple selectors)
4. Create test selector API route:
   - POST /api/scraper/test-selector
   - Verify selector works on current page
   - Extract and return value
5. Integrate with add price modal:
   - Replace manual selector input with visual picker
   - Show preview of selected element
   - Allow manual adjustment if needed

**Element Selection Flow:**
```
1. User clicks "Add Price"
2. Modal opens with URL input
3. User enters competitor URL
4. Page loads in iframe with highlighting
5. User hovers over elements (highlights yellow)
6. User clicks on price element
7. Selector captured and validated
8. Preview shows extracted price
9. User confirms and saves
```

**Deliverables:**
- Working proxy system
- Visual element selector
- Selector generation and validation
- Integration with add price flow

---

### Phase 6: Scraping Engine
**Duration: 3-4 days**

1. Build core scraper utility:
   - Puppeteer setup for dynamic sites
   - Cheerio fallback for static HTML
   - User-agent rotation
   - Error handling and retries (3 attempts)
   - 30-second timeout
2. Implement price extraction:
   - Use CSS selector to find element
   - Extract text content
   - Parse currency and number
   - Validate extracted data
3. Create manual check API route:
   - POST /api/scraper/check-prices
   - Allow manual trigger for specific price
   - Update current_price in database
   - Log to price_history
4. Build price comparison logic:
   - Compare new price with current_price
   - Calculate percentage change
   - Determine if notification needed
5. Handle error cases:
   - Selector not found
   - Page timeout
   - Invalid price format
   - Network errors
   - Update status to 'error'
   - Store error message

**Scraper Configuration:**
```javascript
{
  timeout: 30000,
  retries: 3,
  userAgents: [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ...',
    // ... more user agents
  ],
  puppeteerOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
}
```

**Deliverables:**
- Robust scraping engine
- Price extraction and parsing
- Error handling
- Manual check functionality

---

### Phase 7: Scheduled Checks & Notifications
**Duration: 3-4 days**

1. Create cron API route:
   - POST /api/cron/daily-check
   - Protected with secret token
   - Query users needing checks based on timezone + check_time
   - Process each user's tracked prices
2. Implement scheduling logic:
   - Calculate next check time per user
   - Handle timezone conversions
   - Batch processing (avoid memory issues)
3. Build email notification system:
   - Set up Resend API
   - Create email templates (HTML + plain text):
     - Price change notification
     - Error notification
   - Include relevant details:
     - Company & service name
     - Old price → New price
     - Percentage change
     - Link to page
     - Link to dashboard
4. Log notifications:
   - Insert to notifications table
   - Track sent emails
   - Prevent duplicate notifications
5. Set up Vercel Cron job:
   - Configure in vercel.json
   - Runs every hour
   - Hits /api/cron/daily-check endpoint
   - Protected with CRON_SECRET

**Email Template (Price Change):**
```
Subject: Price Alert: [Company] - [Service] changed from $X to $Y

Hi [User Name],

We detected a price change for [Company] - [Service]:

Old Price: $99.00
New Price: $89.00
Change: -10% 🎉

View details: [Link to dashboard]
View page: [Link to competitor page]

---
Price Tracker
Unsubscribe | Settings
```

**Deliverables:**
- Scheduled check system
- Email notifications working
- Cron job configured
- Notification logging

---

### Phase 8: Price History & Advanced Visualization
**Duration: 3-4 days**

1. Create price history API route:
   - GET /api/prices/[id]/history
   - Return chronological price data
   - Support date range filtering
2. Build interactive chart component with Recharts:
   - Line chart with gradient fill
   - Area chart option toggle
   - Multiple line overlay (compare prices)
   - Responsive design
   - Tooltips with formatted data
   - Zoom and pan controls
   - Date range selector
   - Export to image option
3. Create price history modal:
   - Chart display at top
   - Detailed history table below
   - Filters (date range, show only changes)
   - Statistics summary:
     - Lowest price
     - Highest price
     - Average price
     - Current trend
     - Volatility score
4. Implement trend indicators:
   - Calculate trend direction
   - Show percentage change badges
   - Color coding (green down, red up, gray stable)
   - Mini sparkline charts on price cards
5. Add comparison features:
   - Select multiple prices to compare
   - Overlay on same chart
   - Normalize scales option

**Chart Features:**
- **Line Chart:** Clean lines with gradient fill below
- **Area Chart:** Stacked or overlaid areas
- **Tooltips:** Show exact price, date, and percentage change
- **Interactions:** Click data point to see details, zoom time range
- **Responsive:** Mobile-friendly touch interactions
- **Export:** Download as PNG or CSV

**Price History Table:**
```
| Date & Time          | Price   | Change | Status  |
|---------------------|---------|--------|---------|
| Nov 6, 2024 14:30   | $89.00  | -10%   | Success |
| Nov 5, 2024 14:30   | $99.00  | +10%   | Success |
| Nov 4, 2024 14:30   | $90.00  | -      | Success |
| Nov 3, 2024 14:30   | Error   | -      | Failed  |
```

**Deliverables:**
- Interactive price history charts
- Detailed history view
- Trend analysis
- Comparison features

---

### Phase 9: Polish & Testing
**Duration: 2-3 days**

1. Implement loading states:
   - Skeleton loaders for data fetching
   - Spinner for actions
   - Progress indicators for bulk operations
2. Enhance error handling:
   - Toast notifications for user actions
   - Friendly error messages
   - Retry mechanisms
   - Error boundaries
3. Add validation:
   - Form validation with helpful messages
   - URL validation
   - CSS selector validation
   - Input sanitization
4. Security review:
   - SQL injection prevention (parameterized queries)
   - XSS protection
   - CSRF tokens
   - Rate limiting on API routes
   - Secure password requirements
5. Responsive design:
   - Mobile-first approach
   - Tablet breakpoints
   - Desktop optimization
   - Touch-friendly interactions
6. Performance optimization:
   - Database query optimization
   - API response caching
   - Image optimization
   - Code splitting
7. Accessibility:
   - Keyboard navigation
   - Screen reader support
   - ARIA labels
   - Focus management
8. Testing:
   - User flow testing
   - Cross-browser testing
   - Mobile device testing
   - Email delivery testing

**Deliverables:**
- Polished, production-ready application
- Comprehensive error handling
- Responsive across devices
- Security hardened
- Performance optimized

---

## Key Technical Details

### Authentication Flow

**Registration:**
```
1. User submits email, name, password
2. Validate input, check email uniqueness
3. Hash password with bcrypt
4. Generate verification token (UUID)
5. Insert user into database (verified=false)
6. Send verification email
7. Return success message
```

**Login:**
```
1. User submits email, password
2. Query user from database
3. Verify password with bcrypt
4. Check if user is verified
5. Generate JWT token
6. Set httpOnly cookie
7. Return user data
```

**Session Management:**
```javascript
// JWT Payload
{
  userId: 123,
  email: 'user@example.com',
  iat: 1699280400,
  exp: 1699885200 // 7 days
}

// Cookie Options
{
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
}
```

### Scraping Strategy

**Decision Tree:**
```
Is the site JavaScript-heavy?
├─ Yes → Use Puppeteer
│   ├─ Launch headless browser
│   ├─ Navigate to URL
│   ├─ Wait for element
│   └─ Extract content
└─ No → Use Cheerio
    ├─ Fetch HTML with Axios
    ├─ Parse with Cheerio
    └─ Query with CSS selector
```

**Price Extraction Function:**
```javascript
function extractPrice(text) {
  // Remove whitespace
  text = text.trim();
  
  // Look for currency symbols
  const currencyMatch = text.match(/[£$€]/);
  const currency = currencyMatch ? currencyMatch[0] : null;
  
  // Extract number (handles: 1,234.56 or 1234.56 or 1.234,56)
  const cleaned = text.replace(/[^\d.,]/g, '');
  
  // Determine decimal separator
  let price;
  if (cleaned.includes(',') && cleaned.includes('.')) {
    // Has both - last one is decimal
    const lastComma = cleaned.lastIndexOf(',');
    const lastPeriod = cleaned.lastIndexOf('.');
    if (lastPeriod > lastComma) {
      // Format: 1.234,56 → 1234.56
      price = parseFloat(cleaned.replace(/\./g, '').replace(',', '.'));
    } else {
      // Format: 1,234.56 → 1234.56
      price = parseFloat(cleaned.replace(/,/g, ''));
    }
  } else if (cleaned.includes(',')) {
    // Only comma - could be thousands or decimal
    const parts = cleaned.split(',');
    if (parts.length === 2 && parts[1].length === 2) {
      // Likely decimal: 99,99 → 99.99
      price = parseFloat(cleaned.replace(',', '.'));
    } else {
      // Likely thousands: 1,234 → 1234
      price = parseFloat(cleaned.replace(/,/g, ''));
    }
  } else {
    // Only period or no separator
    price = parseFloat(cleaned);
  }
  
  return {
    currency,
    price: isNaN(price) ? null : price,
    raw: text
  };
}
```

### Cron Job Implementation

**Vercel Cron Setup (vercel.json):**
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

**API Route Logic:**
```javascript
// /api/cron/daily-check/route.ts

export async function POST(req) {
  // 1. Verify cron secret
  const token = req.headers.get('authorization');
  if (token !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  // 2. Get current time in UTC
  const now = new Date();
  
  // 3. Query users whose check time is now (±30 min window)
  const users = await getUsersNeedingCheck(now);
  
  // 4. For each user, get their active tracked prices
  for (const user of users) {
    const prices = await getActiveTrackedPrices(user.id);
    
    // 5. Check each price
    for (const price of prices) {
      try {
        const result = await scrapePrice(price.url, price.css_selector);
        
        if (result.success) {
          // Compare with current price
          const changed = result.price !== price.current_price;
          
          // Update database
          await updatePrice(price.id, result.price, result.currency);
          
          // Log to history
          await logPriceHistory(price.id, result.price, result.currency);
          
          // Send notification if changed
          if (changed) {
            await sendPriceChangeEmail(user, price, result);
          }
        } else {
          // Update status to error
          await updatePriceStatus(price.id, 'error', result.error);
          
          // Send error notification
          await sendErrorEmail(user, price, result.error);
        }
      } catch (error) {
        console.error(`Failed to check price ${price.id}:`, error);
      }
    }
  }
  
  return Response.json({ success: true, checked: users.length });
}
```

### Chart Implementation with Recharts

```javascript
import {
  LineChart,
  Line,
  AreaChart,
  Area,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer
} from 'recharts';

function PriceHistoryChart({ data, currency = '$' }) {
  // Format data for Recharts
  const chartData = data.map(item => ({
    date: new Date(item.checked_at).toLocaleDateString(),
    price: parseFloat(item.price),
    timestamp: item.checked_at
  }));
  
  // Custom tooltip
  const CustomTooltip = ({ active, payload }) => {
    if (active && payload && payload.length) {
      const data = payload[0].payload;
      return (
        <div className="bg-white dark:bg-gray-800 p-3 rounded shadow-lg border">
          <p className="font-semibold">
            {currency}{data.price.toFixed(2)}
          </p>
          <p className="text-sm text-gray-600 dark:text-gray-400">
            {new Date(data.timestamp).toLocaleString()}
          </p>
        </div>
      );
    }
    return null;
  };
  
  return (
    <ResponsiveContainer width="100%" height={400}>
      <AreaChart data={chartData}>
        <defs>
          <linearGradient id="colorPrice" x1="0" y1="0" x2="0" y2="1">
            <stop offset="5%" stopColor="#3b82f6" stopOpacity={0.3}/>
            <stop offset="95%" stopColor="#3b82f6" stopOpacity={0}/>
          </linearGradient>
        </defs>
        <CartesianGrid strokeDasharray="3 3" opacity={0.3} />
        <XAxis 
          dataKey="date" 
          tick={{ fontSize: 12 }}
          stroke="currentColor"
        />
        <YAxis 
          tick={{ fontSize: 12 }}
          stroke="currentColor"
          tickFormatter={(value) => `${currency}${value}`}
        />
        <Tooltip content={<CustomTooltip />} />
        <Area
          type="monotone"
          dataKey="price"
          stroke="#3b82f6"
          strokeWidth={2}
          fillOpacity={1}
          fill="url(#colorPrice)"
        />
      </AreaChart>
    </ResponsiveContainer>
  );
}
```

---

## Environment Variables

```env
# Database (provided by Vercel Postgres)
POSTGRES_URL=your_postgres_url
POSTGRES_PRISMA_URL=your_prisma_url
POSTGRES_URL_NON_POOLING=your_non_pooling_url

# Authentication
JWT_SECRET=your-super-secret-jwt-key-min-32-chars
JWT_EXPIRES_IN=7d

# Email (Resend)
RESEND_API_KEY=your_resend_api_key
EMAIL_FROM=noreply@yourdomain.com
EMAIL_FROM_NAME=Price Tracker

# Application
NEXT_PUBLIC_APP_URL=https://yourapp.com
NEXT_PUBLIC_APP_NAME=Competitor Price Tracker

# Cron Protection
CRON_SECRET=random-secret-string-for-cron-authentication

# Scraper Configuration
SCRAPER_TIMEOUT=30000
SCRAPER_MAX_RETRIES=3
```

---

## Package Dependencies

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.0.0",

    "@vercel/postgres": "^0.5.0",
    "bcrypt": "^5.1.1",
    "jsonwebtoken": "^9.0.2",
    "resend": "^2.0.0",

    "puppeteer": "^21.5.0",
    "cheerio": "^1.0.0-rc.12",
    "axios": "^1.6.0",

    "recharts": "^2.10.0",
    "lucide-react": "^0.292.0",
    "next-themes": "^0.2.1",

    "@radix-ui/react-dialog": "^1.0.5",
    "@radix-ui/react-dropdown-menu": "^2.0.6",
    "@radix-ui/react-select": "^2.0.0",
    "@radix-ui/react-toast": "^1.1.5",

    "tailwindcss": "^3.3.0",
    "tailwind-merge": "^2.0.0",
    "clsx": "^2.0.0",
    "class-variance-authority": "^0.7.0",

    "zod": "^3.22.4",
    "date-fns": "^2.30.0",
    "uuid": "^9.0.1"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "@types/react": "^18.2.0",
    "@types/bcrypt": "^5.0.2",
    "@types/jsonwebtoken": "^9.0.5",
    "@types/uuid": "^9.0.7",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.31"
  }
}
```

---

## Security Considerations

### Password Security
- Minimum 8 characters, require mix of uppercase, lowercase, numbers
- Hash with bcrypt (cost factor: 10)
- Never log or expose passwords
- Rate limit login attempts (5 attempts per 15 minutes)

### SQL Injection Prevention
- Always use parameterized queries
- Never concatenate user input into queries
- Use parameterized queries with @vercel/postgres

### XSS Protection
- Sanitize all user input
- Use Next.js automatic escaping
- Set proper Content-Security-Policy headers
- Validate and sanitize URLs before proxying

### CSRF Protection
- Use SameSite cookies
- Implement CSRF tokens for state-changing operations
- Verify Origin header on sensitive endpoints

### Rate Limiting
- Login: 5 attempts per 15 minutes per IP
- Registration: 3 accounts per hour per IP
- Password reset: 3 requests per hour per email
- API calls: 100 requests per minute per user
- Scraping: Configurable per price item

### API Security
- Protect cron endpoints with secret tokens
- Validate JWT on all protected routes
- Use httpOnly cookies for sessions
- Implement proper CORS policies

---

## Performance Optimization

### Database
- Index frequently queried columns
- Use connection pooling
- Implement query caching
- Paginate large result sets
- Archive old price history data (optional)

### Frontend
- Code splitting with Next.js dynamic imports
- Lazy load charts and heavy components
- Optimize images with Next.js Image component
- Use React.memo for expensive renders
- Debounce search and filter inputs

### Scraping
- Queue system for bulk checks (optional)
- Parallel processing with limits
- Cache scraped pages (short duration)
- Respect robots.txt
- Implement exponential backoff on retries

---

## Future Enhancements (Post-MVP)

### Chrome Extension
- Visual element selector that works on any page
- One-click price tracking
- Direct communication with web app
- Sync settings and data

### Advanced Features
- Multi-currency support
- Price drop alerts with thresholds
- Competitor comparison dashboard
- Export reports (PDF, CSV)
- API access for integrations
- Team collaboration features
- Webhooks for price changes
- Mobile app (React Native)

### Analytics
- Price trend predictions
- Market analysis
- Competitor insights
- Custom reports
- Data visualization dashboard

### Integrations
- Slack notifications
- Discord webhooks
- Zapier integration
- Google Sheets export
- Calendar reminders

---

## Deployment Checklist

### Pre-Deployment
- [ ] Environment variables configured in Vercel
- [ ] Vercel Postgres database created and migrated
- [ ] Resend API key configured
- [ ] Domain email verified in Resend
- [ ] SSL certificate (automatic with Vercel)
- [ ] Vercel Cron configured in vercel.json
- [ ] Rate limiting tested
- [ ] Error tracking configured (optional: Sentry)

### Testing
- [ ] Registration flow
- [ ] Email verification
- [ ] Login/logout
- [ ] Password reset
- [ ] Add/edit/delete categories
- [ ] Add/edit/delete prices
- [ ] Element selector works
- [ ] Price scraping accurate
- [ ] Email notifications sent
- [ ] Charts render correctly
- [ ] Dark/light mode works
- [ ] Mobile responsive
- [ ] Cross-browser compatibility

### Go-Live
- [ ] Deploy to Vercel
- [ ] Verify Vercel Cron running
- [ ] Test end-to-end user flow
- [ ] Monitor error logs
- [ ] Check email deliverability
- [ ] Performance testing
- [ ] Security audit

### Post-Launch
- [ ] Monitor server resources
- [ ] Track user feedback
- [ ] Log error patterns
- [ ] Optimize slow queries
- [ ] Plan next features

---

## Support & Maintenance

### Monitoring
- Server uptime and response times
- Database performance
- Scraping success/failure rates
- Email delivery rates
- Error logs and patterns
- User growth metrics

### Regular Tasks
- Review and update user-agents
- Check for broken scrapers
- Update dependencies
- Backup database
- Archive old data
- Review security patches

### User Support
- Help with selector creation
- Troubleshoot scraping issues
- Investigate email delivery
- Handle data export requests
- Process account deletions (GDPR)

---

## Contact & Resources

### Documentation
- Next.js: https://nextjs.org/docs
- Vercel Postgres: https://vercel.com/docs/storage/vercel-postgres
- Shadcn UI: https://ui.shadcn.com
- Tailwind: https://tailwindcss.com/docs
- Recharts: https://recharts.org
- Puppeteer: https://pptr.dev
- Resend: https://resend.com/docs

### Tools
- Vercel Dashboard for database management
- Postman for API testing
- Chrome DevTools for selector testing
- Resend dashboard for email testing

---

## Project Timeline

**Total Estimated Time: 3-4 weeks**

- Week 1: Phases 1-3 (Setup, Auth, UI, Categories)
- Week 2: Phases 4-5 (Price CRUD, Element Selector)
- Week 3: Phases 6-7 (Scraping, Scheduling, Notifications)
- Week 4: Phases 8-9 (Visualization, Polish, Testing)

**Buffer:** Add 20% time for unexpected issues and refinements.

---

## Success Metrics

### MVP Launch Criteria
- [ ] Users can register and verify email
- [ ] Users can add prices with visual selector
- [ ] Prices are organized in nested categories
- [ ] Daily checks run automatically
- [ ] Email notifications work
- [ ] Charts display price history
- [ ] Dark/light mode functional
- [ ] Mobile responsive
- [ ] No major bugs

### Post-Launch Goals
- Achieve 90%+ scraping success rate
- Email deliverability >95%
- Page load time <2 seconds
- Zero security incidents
- Positive user feedback
- Stable performance under load

---

## Notes for Development

1. **Start Simple:** Build and test each phase before moving on
2. **Test Early:** Don't wait until the end to test core features
3. **Security First:** Never compromise on auth and data security
4. **User Experience:** Prioritize clear error messages and loading states
5. **Mobile Matters:** Test on real devices, not just browser DevTools
6. **Email Testing:** Use Mailtrap in development before going live
7. **Selector Reliability:** Test selectors on multiple page loads
8. **Performance:** Monitor database queries and optimize early
9. **Dark Mode:** Test all components in both themes
10. **Backup Plan:** Regular database backups from day one

---

**End of Project Outline**

Ready for development with Claude Code! 🚀

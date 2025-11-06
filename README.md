# 🎯 Competitor Price Tracker

**Version: 0.1.0** | Last Updated: 2025-11-06

A modern SaaS platform for tracking competitor pricing with automated monitoring, visual element selection, and intelligent notifications.

![Next.js](https://img.shields.io/badge/Next.js-14-black)
![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Latest-blue)
![Vercel](https://img.shields.io/badge/Deployed%20on-Vercel-black)

---

## ✨ Features

- **Visual Element Selection** - Point-and-click price selection from any website
- **Automated Monitoring** - Daily price checks with configurable schedules
- **Smart Notifications** - Email alerts for price changes and errors
- **Nested Organization** - Categorize prices by company, service, or custom folders
- **Price History & Analytics** - Interactive charts showing price trends over time
- **Multi-Currency Support** - Handles £ and $ with intelligent parsing
- **Dark/Light Mode** - Beautiful themes with seamless switching
- **Multi-User SaaS** - Secure authentication with isolated user data

---

## 🚀 Tech Stack

### Core
- **Next.js 14** - React framework with App Router
- **TypeScript** - Type-safe development
- **Vercel** - Serverless deployment platform

### Database & Backend
- **Vercel Postgres** - Managed PostgreSQL database
- **Vercel Cron** - Scheduled price checks
- **JWT** - Secure session management

### UI & Styling
- **Tailwind CSS** - Utility-first styling
- **Shadcn UI** - High-quality React components
- **Lucide Icons** - Beautiful icon library
- **Recharts** - Interactive data visualization
- **next-themes** - Theme management

### Web Scraping
- **Puppeteer** - Headless browser for dynamic sites
- **Cheerio** - HTML parsing for static sites
- **Axios** - HTTP client

### Email
- **Resend** - Transactional email service

---

## 📋 Prerequisites

- Node.js 18+ installed
- Vercel account (free tier available)
- GitHub account for deployment
- Custom domain for email sending (optional for MVP)

---

## 🛠️ Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/competitor-price-tracker.git
cd competitor-price-tracker
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Set Up Environment Variables

Create a `.env.local` file in the root directory:

```env
# Database (provided by Vercel Postgres)
POSTGRES_URL=your_postgres_url
POSTGRES_PRISMA_URL=your_prisma_url
POSTGRES_URL_NON_POOLING=your_non_pooling_url

# Authentication
JWT_SECRET=your-super-secret-jwt-key-min-32-characters
JWT_EXPIRES_IN=7d

# Email (Resend)
RESEND_API_KEY=your_resend_api_key
EMAIL_FROM=noreply@yourdomain.com
EMAIL_FROM_NAME=Price Tracker

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME=Competitor Price Tracker

# Cron Protection
CRON_SECRET=your-random-secret-string

# Scraper Configuration
SCRAPER_TIMEOUT=30000
SCRAPER_MAX_RETRIES=3
```

### 4. Set Up Database

Create the database schema using the SQL in `docs/schema.sql` or run migrations:

```bash
npm run db:migrate
```

### 5. Run Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 📁 Project Structure

```
competitor-price-tracker/
├── src/
│   ├── app/                    # Next.js App Router pages
│   │   ├── (auth)/            # Authentication pages
│   │   ├── (dashboard)/       # Main application pages
│   │   └── api/               # API routes
│   ├── components/            # React components
│   │   ├── ui/               # Shadcn UI components
│   │   ├── dashboard/        # Dashboard-specific components
│   │   ├── modals/           # Modal dialogs
│   │   └── layout/           # Layout components
│   ├── lib/                  # Utility functions
│   │   ├── db.ts            # Database connection
│   │   ├── auth.ts          # Authentication utilities
│   │   ├── email.ts         # Email service
│   │   └── scraper.ts       # Web scraping logic
│   └── types/               # TypeScript type definitions
├── public/                  # Static assets
├── docs/                    # Documentation
└── vercel.json             # Vercel configuration
```

---

## 🚢 Deployment

### Deploy to Vercel

1. Push your code to GitHub
2. Import repository in Vercel dashboard
3. Add Vercel Postgres database to project
4. Configure environment variables
5. Deploy!

Vercel will automatically:
- Detect Next.js configuration
- Build and deploy on every push to main
- Provision SSL certificates
- Set up serverless functions

### Configure Cron Jobs

Add to `vercel.json`:

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

---

## 🔐 Security

- Passwords hashed with bcrypt (cost factor: 10)
- JWT tokens in httpOnly cookies
- CSRF protection with SameSite cookies
- SQL injection prevention via parameterized queries
- XSS protection with input sanitization
- Rate limiting on authentication endpoints
- Secure session management

---

## 📊 Database Schema

### Core Tables
- `users` - User accounts and preferences
- `categories` - Nested folder organization
- `tracked_prices` - Price tracking configuration
- `price_history` - Historical price data
- `notifications` - Email notification log
- `password_resets` - Password reset tokens

See full schema in `docs/schema.sql`

---

## 🎨 Design System

### Colors
- **Primary**: Blue-600 (actions, links)
- **Success**: Green-600 (price drops, success states)
- **Error**: Red-600 (price increases, error states)
- **Warning**: Amber-600 (warnings, alerts)

### Pastel Colors (Categories)
- Soft Pink (#FFE5EC)
- Soft Blue (#E0F2FE)
- Soft Green (#DCFCE7)
- Soft Purple (#F3E8FF)
- Soft Yellow (#FEF3C7)
- Soft Orange (#FFEDD5)

### Themes
- Dark mode (default) - Gray-900 background
- Light mode - White background
- System preference detection

---

## 🧪 Testing

```bash
# Run unit tests
npm test

# Run E2E tests
npm run test:e2e

# Type checking
npm run type-check

# Linting
npm run lint
```

---

## 📝 API Documentation

### Authentication Endpoints
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `POST /api/auth/verify` - Email verification
- `POST /api/auth/forgot-password` - Request password reset
- `POST /api/auth/reset-password` - Reset password

### Price Tracking Endpoints
- `GET /api/prices` - List all tracked prices
- `POST /api/prices` - Create new tracked price
- `GET /api/prices/[id]` - Get single price
- `PUT /api/prices/[id]` - Update price
- `DELETE /api/prices/[id]` - Delete price
- `GET /api/prices/[id]/history` - Get price history

### Category Endpoints
- `GET /api/categories` - List all categories
- `POST /api/categories` - Create category
- `PUT /api/categories/[id]` - Update category
- `DELETE /api/categories/[id]` - Delete category

### Scraper Endpoints
- `POST /api/scraper/proxy` - Load page via proxy
- `POST /api/scraper/test-selector` - Test CSS selector
- `POST /api/scraper/check-prices` - Manual price check

### Cron Endpoints
- `POST /api/cron/daily-check` - Scheduled price checks (protected)

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 🆘 Support

For issues, questions, or feature requests:
- Open an issue on GitHub
- Email: support@yourcompany.com
- Documentation: See `/docs` folder

---

## 🗺️ Roadmap

### Phase 1 (MVP) ✅
- User authentication
- Visual element selection
- Price tracking and history
- Email notifications
- Dark/light mode

### Phase 2 (Coming Soon)
- Chrome extension
- Advanced analytics
- Multi-currency support
- Price drop thresholds
- Export reports (PDF, CSV)

### Phase 3 (Future)
- Team collaboration
- API access
- Webhooks
- Mobile app
- Competitor insights dashboard

---

## 🙏 Acknowledgments

- [Next.js](https://nextjs.org/) - The React framework
- [Vercel](https://vercel.com/) - Deployment platform
- [Shadcn UI](https://ui.shadcn.com/) - UI components
- [Tailwind CSS](https://tailwindcss.com/) - Styling
- [Recharts](https://recharts.org/) - Data visualization

---

## 📌 Version History

### Version 0.1.0 (2025-11-06)
- Initial documentation setup
- Project specification complete
- Architecture defined (PostgreSQL + Vercel + Resend)
- Database schema designed
- Implementation phases planned

---

**Built with ❤️ using Next.js and Vercel**

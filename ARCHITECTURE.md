# LCEO System Architecture

## Overview

This document describes the architecture of the LCEO web application.

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER DEVICES                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐           │
│  │ Desktop │  │ Tablet  │  │ Mobile  │  │  USSD   │           │
│  │ Browser │  │ Browser │  │ Browser │  │  Phone  │           │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘           │
└───────┼───────────┼───────────┼───────────┼──────────────────┘
        │           │           │           │
        └───────────┴───────────┴───────────┘
                    │
                    │ HTTPS
                    ▼
        ┌───────────────────────┐
        │   CDN / Web Server    │
        │   (Vercel/Netlify)    │
        └───────────┬───────────┘
                    │
                    │ Static Assets
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                     FRONTEND APPLICATION                         │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    React Application                       │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │ │
│  │  │ Public Pages │  │  Dashboards  │  │    Shared    │   │ │
│  │  │              │  │              │  │  Components  │   │ │
│  │  │ • Home       │  │ • Admin      │  │ • Header     │   │ │
│  │  │ • About      │  │ • Donor      │  │ • Footer     │   │ │
│  │  │ • Programs   │  │ • Beneficiary│  │ • Chatbot    │   │ │
│  │  │ • Impact     │  │              │  │ • Forms      │   │ │
│  │  │ • Donate     │  │              │  │ • Charts     │   │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │ │
│  │                                                           │ │
│  │  ┌──────────────────────────────────────────────────┐   │ │
│  │  │           State Management                        │   │ │
│  │  │  • AuthContext (User authentication)              │   │ │
│  │  │  • TranslationContext (i18n EN/FR/RW)            │   │ │
│  │  │  • Local State (useState, useReducer)            │   │ │
│  │  └──────────────────────────────────────────────────┘   │ │
│  │                                                           │ │
│  │  ┌──────────────────────────────────────────────────┐   │ │
│  │  │           API Service Layer                       │   │ │
│  │  │  /src/services/api.ts                            │   │ │
│  │  │  • authAPI • usersAPI • beneficiariesAPI         │   │ │
│  │  │  • programsAPI • donationsAPI • adminAPI         │   │ │
│  │  │  • JWT Token Management                          │   │ │
│  │  │  • Error Handling                                │   │ │
│  │  └──────────────────────────────────────────────────┘   │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          │ REST API (JSON)
                          │ JWT Authentication
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                      BACKEND API SERVER                          │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    API Endpoints                           │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │ │
│  │  │  Auth    │  │  Users   │  │  Data    │  │  Admin   │ │ │
│  │  │          │  │          │  │          │  │          │ │ │
│  │  │ • Login  │  │ • Profile│  │ • CRUD   │  │ • Stats  │ │ │
│  │  │ • Logout │  │ • Update │  │ • List   │  │ • Reports│ │ │
│  │  │ • Verify │  │ • Roles  │  │ • Filter │  │ • Import │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │ │
│  │                                                           │ │
│  │  ┌──────────────────────────────────────────────────┐   │ │
│  │  │         Middleware                                │   │ │
│  │  │  • Authentication (JWT)                           │   │ │
│  │  │  • Authorization (RBAC)                           │   │ │
│  │  │  • Validation                                     │   │ │
│  │  │  • Rate Limiting                                  │   │ │
│  │  │  • CORS                                           │   │ │
│  │  │  • Logging                                        │   │ │
│  │  └──────────────────────────────────────────────────┘   │ │
│  │                                                           │ │
│  │  ┌──────────────────────────────────────────────────┐   │ │
│  │  │         Business Logic                            │   │ │
│  │  │  • User Management                                │   │ │
│  │  │  • Program Management                             │   │ │
│  │  │  • Tracking & Goals                               │   │ │
│  │  │  • Donations Processing                           │   │ │
│  │  │  • Impact Calculations                            │   │ │
│  │  │  • Notifications                                  │   │ │
│  │  └──────────────────────────────────────────────────┘   │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          │ Database Queries
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATABASE LAYER                             │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              PostgreSQL Database                           │ │
│  │                                                            │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │ │
│  │  │  Users   │  │Programs  │  │Donations │  │  Logs    │ │ │
│  │  │  Staff   │  │Projects  │  │Recurring │  │Metrics   │ │ │
│  │  │Benefic's │  │Stories   │  │Payments  │  │Activity  │ │ │
│  │  │ Donors   │  │          │  │          │  │          │ │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │ │
│  │                                                            │ │
│  │  Main Tables:                                              │ │
│  │  • users, beneficiaries, donors, staff                     │ │
│  │  • programs, projects, impact_metrics                      │ │
│  │  • donations, recurring_donations                          │ │
│  │  • weekly_trackings, goals                                 │ │
│  │  • stories, notifications                                  │ │
│  │  • activity_logs, beneficiary_documents                    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   External   │  │   Payment    │  │     SMS      │
│  Services    │  │  Gateways    │  │   Service    │
│              │  │              │  │              │
│ • Kobo       │  │ • Stripe     │  │ • Twilio     │
│ • Email      │  │ • MTN MoMo   │  │ • Africa's   │
│ • Storage    │  │ • PayPal     │  │   Talking    │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## Component Architecture

### Frontend Components Hierarchy

```
App
├── AuthProvider
│   └── TranslationProvider
│       ├── Header
│       │   ├── Navigation
│       │   ├── Language Selector
│       │   └── Login Dropdown
│       │
│       ├── Main Content
│       │   ├── Public Pages
│       │   │   ├── HomePage
│       │   │   ├── AboutPage
│       │   │   ├── HowWeWorkPage
│       │   │   ├── ImpactPage
│       │   │   ├── DonatePage
│       │   │   └── ExplorePage
│       │   │       ├── VirtualTour
│       │   │       ├── DayInLife
│       │   │       ├── BeforeAfterGallery
│       │   │       ├── ImpactCounter
│       │   │       └── SmartTools
│       │   │
│       │   ├── Login Pages
│       │   │   ├── BeneficiaryLogin
│       │   │   ├── DonorLogin
│       │   │   └── AdminLogin
│       │   │
│       │   └── Dashboard Pages
│       │       ├── BeneficiaryDashboard
│       │       │   ├── ProgressTracker
│       │       │   ├── GoalsManager
│       │       │   ├── ReportSubmission
│       │       │   └── Charts
│       │       │
│       │       ├── DonorDashboard
│       │       │   ├── DonationHistory
│       │       │   ├── RecurringDonations
│       │       │   ├── ImpactView
│       │       │   └── Receipts
│       │       │
│       │       └── AdminDashboard
│       │           ├── Statistics
│       │           ├── BeneficiaryManagement
│       │           ├── ProgramManagement
│       │           ├── ContentManagement
│       │           ├── Reports
│       │           └── Analytics
│       │
│       ├── Footer
│       │   ├── Social Links
│       │   ├── Quick Links
│       │   └── Newsletter
│       │
│       ├── Chatbot
│       │   └── MultiLanguageBot
│       │
│       └── Toaster (Notifications)
```

---

## Data Flow

### Authentication Flow

```
1. User submits login form
   └─> LoginPage component

2. Form calls AuthContext.login()
   └─> AuthContext

3. AuthContext calls authAPI.login()
   └─> /src/services/api.ts

4. API makes POST request to backend
   └─> POST /auth/login

5. Backend validates credentials
   └─> Returns { user, token }

6. API service returns response
   └─> AuthContext receives data

7. AuthContext saves user & token
   └─> localStorage & state

8. User is redirected to dashboard
   └─> Based on user_type
```

### Data Fetching Flow

```
1. Component mounts
   └─> useEffect hook

2. Calls API function
   └─> e.g., programsAPI.list()

3. API service makes request
   └─> GET /programs
   └─> Includes Authorization header

4. Backend validates token
   └─> Processes request

5. Backend returns data
   └─> JSON response

6. API service receives response
   └─> Type-checked against interfaces

7. Component updates state
   └─> useState/useReducer

8. UI re-renders
   └─> Displays data
```

### Donation Flow

```
User (Donor Dashboard)
  │
  ├─> Clicks "Donate Now"
  │     └─> DonatePage component
  │
  ├─> Fills donation form
  │     • Amount
  │     • Currency
  │     • Program/Project
  │     • Payment method
  │
  ├─> Submits form
  │     └─> donationsAPI.create()
  │           └─> POST /donations
  │
  ├─> Backend processes
  │     • Creates donation record
  │     • Initiates payment
  │     • Returns transaction details
  │
  ├─> Payment gateway
  │     • Stripe / MTN MoMo
  │     • Processes payment
  │     • Sends webhook
  │
  ├─> Backend receives webhook
  │     • Updates payment status
  │     • Sends receipt
  │
  └─> UI shows confirmation
        • Success message
        • Receipt download
        • Impact preview
```

---

## Technology Stack

### Frontend
```
┌─────────────────────────────────────┐
│          React 18.3.1               │
├─────────────────────────────────────┤
│ Language: TypeScript                │
│ Build Tool: Vite 6.3.5              │
│ CSS: Tailwind CSS v4               │
│ UI Components: Radix UI             │
│ Icons: Lucide React                 │
│ Charts: Recharts                    │
│ Animations: Motion (Framer Motion)  │
│ Forms: React Hook Form              │
│ Notifications: Sonner               │
│ Date Handling: date-fns             │
└─────────────────────────────────────┘
```

### Backend (Your Stack)
```
┌─────────────────────────────────────┐
│          Node.js / Express          │
├─────────────────────────────────────┤
│ Database: PostgreSQL                │
│ ORM: [Your choice]                  │
│ Authentication: JWT                 │
│ Validation: [Your choice]           │
│ File Storage: [Your choice]         │
│ Payment: Stripe, MTN MoMo          │
│ SMS: Twilio, Africa's Talking      │
│ Email: [Your choice]                │
└─────────────────────────────────────┘
```

---

## Security Architecture

### Frontend Security

```
┌─────────────────────────────────────┐
│      Browser Security               │
├─────────────────────────────────────┤
│ • HTTPS Only                        │
│ • Content Security Policy           │
│ • XSS Protection (React escaping)   │
│ • No eval() usage                   │
│ • Secure token storage              │
│ • Input validation                  │
└─────────────────────────────────────┘
```

### Backend Security

```
┌─────────────────────────────────────┐
│      API Security                   │
├─────────────────────────────────────┤
│ • JWT Authentication                │
│ • Role-Based Access Control         │
│ • Rate Limiting                     │
│ • CORS Configuration                │
│ • SQL Injection Prevention          │
│ • Input Sanitization                │
│ • CSRF Protection                   │
│ • Password Hashing (bcrypt)         │
│ • Secure Headers                    │
└─────────────────────────────────────┘
```

### Data Protection

```
User Credentials
  └─> HTTPS → Backend
       └─> Hashed Password (bcrypt)
            └─> Database (Encrypted at rest)

JWT Token
  └─> localStorage (XSS protected)
       └─> HTTP Header (Bearer token)
            └─> Backend validation

Sensitive Data
  └─> Encrypted in transit (HTTPS)
       └─> Encrypted at rest (DB encryption)
            └─> Access control (RBAC)
```

---

## Deployment Architecture

### Development Environment

```
Developer Machine
  ├─> Frontend: http://localhost:5173
  │    └─> Vite Dev Server
  │
  └─> Backend: http://localhost:3000/api
       └─> Express Server
       └─> PostgreSQL Local
```

### Production Environment

```
                     ┌──────────────┐
                     │   Cloudflare │
                     │   DNS / CDN  │
                     └──────┬───────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
    ┌──────────────────┐       ┌──────────────────┐
    │   Frontend       │       │    Backend       │
    │   (Vercel/       │       │    (Heroku/      │
    │   Netlify)       │       │    AWS/DigitalO) │
    │                  │       │                  │
    │ • Static Assets  │       │ • API Server     │
    │ • React App      │       │ • PostgreSQL     │
    │ • Auto Deploy    │       │ • Redis Cache    │
    │ • Global CDN     │       │ • File Storage   │
    └──────────────────┘       └────────┬─────────┘
                                        │
                            ┌───────────┴──────────┐
                            │                      │
                            ▼                      ▼
                  ┌──────────────┐      ┌──────────────┐
                  │  Payment     │      │   External   │
                  │  Providers   │      │   Services   │
                  │              │      │              │
                  │ • Stripe     │      │ • Email      │
                  │ • MTN MoMo   │      │ • SMS        │
                  │ • PayPal     │      │ • Storage    │
                  └──────────────┘      └──────────────┘
```

---

## Performance Optimization

### Frontend

- **Code Splitting**: Dynamic imports for routes
- **Lazy Loading**: Components loaded on demand
- **Image Optimization**: Unsplash CDN
- **Caching**: Service Worker (PWA)
- **Minification**: Production builds
- **Tree Shaking**: Remove unused code
- **CDN**: Static assets served globally

### Backend

- **Database Indexing**: Fast queries
- **Query Optimization**: Efficient joins
- **Caching**: Redis for frequent data
- **Pagination**: Large datasets
- **Connection Pooling**: Database connections
- **Load Balancing**: Multiple instances
- **Rate Limiting**: Prevent abuse

---

## Monitoring & Logging

### Frontend Monitoring

```
User Actions
  └─> Analytics (Google Analytics)
  └─> Error Tracking (Sentry)
  └─> Performance (Web Vitals)
  └─> User Feedback (Hotjar)
```

### Backend Monitoring

```
API Requests
  └─> Request Logging (Winston/Morgan)
  └─> Error Tracking (Sentry)
  └─> Performance (New Relic/DataDog)
  └─> Uptime Monitoring (Pingdom)
  └─> Database Monitoring (pgAnalyze)
```

---

## Scalability Considerations

### Horizontal Scaling

- Multiple frontend instances (CDN)
- Multiple backend instances (Load balancer)
- Database read replicas
- Caching layer (Redis)
- Queue system (Bull/RabbitMQ)

### Vertical Scaling

- Increase server resources
- Optimize database queries
- Upgrade infrastructure

---

## Disaster Recovery

### Backup Strategy

- **Database**: Daily automated backups
- **Files**: S3/Cloud Storage with versioning
- **Code**: Git version control
- **Configuration**: Environment variables backup

### Recovery Plan

1. Identify issue
2. Switch to backup instance
3. Restore from backup
4. Verify data integrity
5. Resume normal operations

---

## Future Architecture

### Phase 2 Enhancements

- Real-time updates (WebSockets)
- Mobile app (React Native)
- Offline-first (PWA)
- Microservices architecture
- GraphQL API
- Kubernetes deployment

---

**Last Updated:** January 28, 2026

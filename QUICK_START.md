# LCEO Frontend - Quick Start Guide

## 🚀 Getting Started

### Prerequisites
- Node.js 16+ installed
- Backend API running (or use demo mode)

### Installation

```bash
# Install dependencies
npm install

# Copy environment template
cp .env.example .env

# Update .env with your backend URL
# REACT_APP_API_URL=http://localhost:3000/api

# Start development server
npm run dev
```

The app will open at `http://localhost:5173`

---

## 🎯 Demo Mode (Current Setup)

The app currently runs in **demo mode** with mock authentication:

### Login Credentials

**Admin Portal:**
- Email: `admin@lceo.org`
- Password: `admin123`

**Donor Portal:**
- Email: `donor@lceo.org`
- Password: `donor123`

**Beneficiary Portal:**
- Email: `beneficiary@lceo.org`
- Password: `beneficiary123`
- OR Phone: `0788123456` / `beneficiary123`

### Available Pages
- **Public**: Home, About, How We Work, Strategic Direction, Impact, Resources, Explore, Get Involved, Donate
- **Dashboards**: Admin, Donor, Beneficiary (login required)

---

## 🔧 Connecting to Your Backend

### Step 1: Configure Environment

```bash
# .env file
REACT_APP_API_URL=https://your-api-url.com/api
```

### Step 2: Enable Real Authentication

In `/src/app/components/AuthContext.tsx`, find line 75 and uncomment this block:

```typescript
const response = await authAPI.login({
  ...(isPhone ? { phone: identifier } : { email: identifier }),
  password,
});

const newUser: User = {
  id: response.user.id,
  email: response.user.email,
  phone: response.user.phone,
  user_type: response.user.user_type as UserRole,
  name: '', // Will be populated from beneficiary/donor/staff table
  language: response.user.language,
};

setUser(newUser);
localStorage.setItem('lceo_user', JSON.stringify(newUser));
localStorage.setItem('lceo_auth_token', response.token);
return true;
```

Then **remove** the demo credentials block (lines 66-74).

### Step 3: Update Dashboard Components

Replace mock data with API calls. Example for Admin Dashboard:

```typescript
import { adminAPI, programsAPI, beneficiariesAPI } from '@/services/api';

useEffect(() => {
  const loadData = async () => {
    try {
      const stats = await adminAPI.getDashboardStats();
      const programs = await programsAPI.list();
      const beneficiaries = await beneficiariesAPI.list();
      
      // Update your state...
      setDashboardStats(stats);
      setPrograms(programs);
      setBeneficiaries(beneficiaries);
    } catch (error) {
      console.error('Failed to load dashboard:', error);
      toast.error('Failed to load dashboard data');
    }
  };
  loadData();
}, []);
```

---

## 📁 Project Structure

```
src/
├── app/
│   ├── components/
│   │   ├── pages/           # Page components
│   │   │   ├── HomePage.tsx
│   │   │   ├── AdminDashboard.tsx
│   │   │   ├── BeneficiaryDashboard.tsx
│   │   │   ├── DonorDashboard.tsx
│   │   │   └── ...
│   │   ├── ui/              # Reusable UI components
│   │   ├── AuthContext.tsx  # Authentication state
│   │   ├── TranslationContext.tsx
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── ...
│   └── App.tsx              # Main app component
├── services/
│   └── api.ts               # API service layer
├── types/
│   └── backend.ts           # TypeScript type definitions
└── styles/
    ├── theme.css            # Design tokens & colors
    └── index.css            # Global styles
```

---

## 🎨 Brand Colors

```css
Primary:   #5bc3ab (Teal/Mint Green)
Secondary: #e9cfa4 (Warm Beige)
Accent:    #FF6B35 (Orange)
Success:   #28A745 (Green)
```

All colors are defined in `/src/styles/theme.css` as CSS variables.

---

## 📚 Key Files to Know

### `/src/types/backend.ts`
All TypeScript interfaces matching your backend database schema. Use these types when:
- Creating new components
- Handling API responses
- Form submissions

### `/src/services/api.ts`
All API functions ready to use. Examples:

```typescript
// Get all programs
const programs = await programsAPI.list();

// Create a donation
const donation = await donationsAPI.create({
  amount: 100,
  currency: Currency.USD,
  donation_type: DonationType.ONE_TIME,
  // ... other fields
});

// Submit weekly tracking
const tracking = await beneficiariesAPI.submitTracking(beneficiaryId, {
  week_ending: '2024-01-28',
  attendance: AttendanceStatus.PRESENT,
  // ... other fields
});
```

### `/src/app/components/AuthContext.tsx`
Authentication state management. Access current user:

```typescript
const { user, isAuthenticated, logout } = useAuth();

// Check user type
if (user?.user_type === 'admin') {
  // Admin-only code
}
```

---

## 🌍 Multi-Language Support

The app supports English, French, and Kinyarwanda:

```typescript
import { useTranslation } from '@/app/components/TranslationContext';

const { t, language, setLanguage } = useTranslation();

// Use translations
<p>{t('welcome')}</p>

// Change language
setLanguage('rw'); // en, fr, or rw
```

---

## 🔍 Available API Endpoints

All endpoints are defined in `/src/services/api.ts`:

### Authentication
```typescript
authAPI.login({ email, password })
authAPI.register({ email, phone, password, user_type })
authAPI.logout()
authAPI.forgotPassword(email)
authAPI.resetPassword(token, password)
```

### Users
```typescript
usersAPI.getProfile()
usersAPI.updateProfile(data)
usersAPI.getBeneficiaries()
usersAPI.getDonors()
```

### Beneficiaries
```typescript
beneficiariesAPI.list({ status?, program_id? })
beneficiariesAPI.getById(id)
beneficiariesAPI.create(data)
beneficiariesAPI.update(id, data)
beneficiariesAPI.submitTracking(id, data)
beneficiariesAPI.getGoals(id)
```

### Programs
```typescript
programsAPI.list()
programsAPI.getById(id)
programsAPI.create(data)
programsAPI.update(id, data)
programsAPI.getImpact(id)
```

### Donations
```typescript
donationsAPI.create(data)
donationsAPI.list()
donationsAPI.createRecurring(data)
donationsAPI.cancelRecurring(id)
```

### Admin
```typescript
adminAPI.getDashboardStats()
adminAPI.getAnalytics({ start_date, end_date })
adminAPI.getDonationReport({ start_date, end_date })
adminAPI.importBeneficiaries(file)
adminAPI.syncKobo()
```

Full list in `/src/services/api.ts`

---

## 🧪 Testing

```bash
# Run tests (if configured)
npm test

# Build for production
npm run build

# Preview production build
npm run preview
```

---

## 📖 Additional Documentation

- **Full Backend Integration Guide**: `/BACKEND_INTEGRATION.md`
- **Recent Changes**: `/CHANGELOG.md`
- **Type Definitions**: `/src/types/backend.ts`
- **API Service**: `/src/services/api.ts`

---

## 🐛 Common Issues

### "Cannot connect to backend"
- Check `.env` has correct `REACT_APP_API_URL`
- Ensure backend is running
- Check CORS settings on backend

### "Authentication not working"
- Verify backend returns correct JWT token format
- Check token is stored in `localStorage` as `lceo_auth_token`
- Ensure backend validates token correctly

### "Types don't match"
- Compare `/src/types/backend.ts` with your backend schema
- Update types if schema changed
- Rebuild frontend: `npm run build`

---

## 🆘 Need Help?

1. Check `/BACKEND_INTEGRATION.md` for detailed integration steps
2. Review type definitions in `/src/types/backend.ts`
3. Look at API examples in `/src/services/api.ts`
4. Check demo components for usage patterns

---

## 🎉 You're Ready!

The frontend is fully aligned with your backend structure. Just:
1. Set your API URL
2. Connect the dashboards to real data
3. Test and deploy

Good luck! 🚀

# Backend Integration Guide

## Overview

This frontend is now fully aligned with the LCEO backend structure. All TypeScript types match the backend database schema, and the API service layer is ready for integration.

## Updated Features

### 1. **Color Scheme**
- Primary: `#5bc3ab` (teal/mint green)
- Secondary: `#e9cfa4` (warm beige)
- Updated throughout all components and theme

### 2. **User Roles**
The system now supports only 3 user types (matching backend):
- **Admin**: Full system access, content management, reports
- **Donor**: Donation management, impact tracking
- **Beneficiary**: Progress reporting, goal tracking

**Removed roles**: Editor and Mentor (as they are not in your backend schema)

### 3. **Type Definitions**
All backend types are defined in `/src/types/backend.ts`:
- User, Beneficiary, Donor, Staff
- Programs, Projects, Donations
- Weekly Tracking, Goals, Stories
- All enums matching your backend

### 4. **API Service Layer**
Complete API service in `/src/services/api.ts` with functions for:

#### Authentication
```typescript
authAPI.login({ email, password })
authAPI.register({ email, phone, password, user_type })
authAPI.logout()
authAPI.forgotPassword(email)
authAPI.resetPassword(token, password)
authAPI.verifyPhone(phone, code)
authAPI.verifyEmail(email, code)
```

#### Users
```typescript
usersAPI.getProfile()
usersAPI.updateProfile(data)
usersAPI.getBeneficiaries()
usersAPI.getDonors()
usersAPI.getStaff()
```

#### Beneficiaries
```typescript
beneficiariesAPI.list({ status?, program_id? })
beneficiariesAPI.getById(id)
beneficiariesAPI.create(data)
beneficiariesAPI.update(id, data)
beneficiariesAPI.getProgress(id)
beneficiariesAPI.submitTracking(id, data)
beneficiariesAPI.getGoals(id)
beneficiariesAPI.createGoal(id, data)
```

#### Programs & Projects
```typescript
programsAPI.list()
programsAPI.getById(id)
programsAPI.create(data)
programsAPI.update(id, data)
programsAPI.getProjects(id)
programsAPI.getBeneficiaries(id)
programsAPI.getImpact(id)

projectsAPI.list()
projectsAPI.getById(id)
projectsAPI.getDonations(id)
```

#### Donations
```typescript
donationsAPI.create(data)
donationsAPI.list()
donationsAPI.getById(id)
donationsAPI.cancel(id)
donationsAPI.listRecurring()
donationsAPI.createRecurring(data)
donationsAPI.updateRecurring(id, data)
donationsAPI.cancelRecurring(id)
```

#### Content
```typescript
contentAPI.listStories({ is_featured?, is_published? })
contentAPI.getStory(id)
contentAPI.createStory(data)
contentAPI.updateStory(id, data)
```

#### Admin
```typescript
adminAPI.getDashboardStats()
adminAPI.getAnalytics({ start_date?, end_date? })
adminAPI.getDonationReport({ start_date, end_date })
adminAPI.getBeneficiaryReport({ program_id? })
adminAPI.getProgramReport({ program_id? })
adminAPI.importBeneficiaries(file)
adminAPI.importDonations(file)
adminAPI.syncKobo()
adminAPI.getActivityLogs({ user_id?, entity_type? })
adminAPI.getSystemHealth()
```

## How to Integrate with Backend

### Step 1: Configure API Base URL

Update the API base URL in `/src/services/api.ts`:

```typescript
const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3000/api';
```

Or set it in your `.env` file:

```
REACT_APP_API_URL=https://your-backend-url.com/api
```

### Step 2: Enable Real Authentication

In `/src/app/components/AuthContext.tsx`, uncomment the real API integration:

```typescript
// Replace this section:
const credentials = DEMO_CREDENTIALS[role];
// ... demo logic

// With this (already in the file, just uncomment):
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
```

### Step 3: Update Dashboard Components

The dashboard components currently use mock data. Update them to use real API calls:

**Example for BeneficiaryDashboard:**

```typescript
import { beneficiariesAPI } from '@/services/api';

// In component:
useEffect(() => {
  const loadData = async () => {
    try {
      const trackings = await beneficiariesAPI.getTrackings(user.id);
      const goals = await beneficiariesAPI.getGoals(user.id);
      setTrackings(trackings);
      setGoals(goals);
    } catch (error) {
      console.error('Failed to load data:', error);
    }
  };
  loadData();
}, [user.id]);
```

**Example for AdminDashboard:**

```typescript
import { adminAPI, programsAPI, beneficiariesAPI } from '@/services/api';

useEffect(() => {
  const loadDashboard = async () => {
    try {
      const stats = await adminAPI.getDashboardStats();
      const programs = await programsAPI.list();
      const beneficiaries = await beneficiariesAPI.list();
      // Update state...
    } catch (error) {
      console.error('Failed to load dashboard:', error);
    }
  };
  loadDashboard();
}, []);
```

### Step 4: Handle Token Refresh

The API service includes a `refreshToken` function. Implement automatic token refresh:

```typescript
// In api.ts, add interceptor:
async function apiRequest<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
  // ... existing code
  
  if (!response.ok) {
    if (response.status === 401) {
      // Try to refresh token
      try {
        const { token } = await authAPI.refreshToken();
        localStorage.setItem('lceo_auth_token', token);
        // Retry the request
        return apiRequest(endpoint, options);
      } catch {
        // Redirect to login
        window.location.href = '/';
      }
    }
    // ... rest of error handling
  }
}
```

## Demo Credentials (Remove in Production)

Current demo credentials for testing:

**Beneficiary:**
- Email: beneficiary@lceo.org
- Phone: 0788123456
- Password: beneficiary123

**Donor:**
- Email: donor@lceo.org
- Password: donor123

**Admin:**
- Email: admin@lceo.org
- Password: admin123

## Data Structure Examples

### Creating a Beneficiary
```typescript
const newBeneficiary = await beneficiariesAPI.create({
  full_name: "Marie Uwase",
  date_of_birth: "1998-05-15",
  location: {
    province: "Kigali City",
    district: "Gasabo",
    sector: "Kimironko"
  },
  program_id: "program-uuid-here",
  status: BeneficiaryStatus.ACTIVE,
  enrollment_date: "2024-01-15",
  start_capital: 50000,
  current_capital: 50000,
  business_type: "Tailoring",
  tracking_frequency: TrackingFrequency.WEEKLY,
  profile_completion: 80,
  requires_special_attention: false
});
```

### Submitting Weekly Tracking
```typescript
const tracking = await beneficiariesAPI.submitTracking(beneficiaryId, {
  week_ending: "2024-01-28",
  attendance: AttendanceStatus.PRESENT,
  task_given: "Develop marketing plan for business",
  task_completion_status: TaskStatus.COMPLETED,
  income_this_week: 25000,
  expenses_this_week: 15000,
  current_capital: 60000,
  sales_data: {
    products_sold: 12,
    revenue: 25000,
    customers_served: 8
  },
  challenges: "Limited access to raw materials",
  solutions_implemented: "Found new supplier with better prices",
  notes: "Business is growing steadily",
  next_week_plan: {
    goals: ["Increase production by 20%", "Reach 5 new customers"]
  }
});
```

### Creating a Donation
```typescript
const donation = await donationsAPI.create({
  amount: 100,
  currency: Currency.USD,
  donation_type: DonationType.ONE_TIME,
  program_id: "program-uuid",
  payment_method: PaymentMethod.CARD,
  payment_details: {
    card_last4: "4242",
    card_brand: "Visa"
  },
  is_anonymous: false,
  donor_message: "Keep up the great work!"
});
```

### Creating a Program
```typescript
const program = await programsAPI.create({
  name: {
    en: "Youth Entrepreneurship Program",
    rw: "Gahunda y'ubucuruzi bw'urubyiruko"
  },
  description: {
    en: "Empowering young women through business training",
    rw: "Gushyira imbaraga abakobwa bato binyuze mu mahugurwa y'ubucuruzi"
  },
  category: ProgramCategory.ENTREPRENEURSHIP,
  sdg_alignment: [
    {
      goal_number: 1,
      goal_name: "No Poverty",
      targets: ["1.2", "1.4"]
    },
    {
      goal_number: 5,
      goal_name: "Gender Equality",
      targets: ["5.5", "5.a"]
    }
  ],
  kpi_targets: [
    {
      metric_name: "Beneficiaries enrolled",
      target_value: 100,
      current_value: 45,
      unit: "people"
    }
  ],
  start_date: "2024-01-01",
  status: ProgramStatus.ACTIVE,
  budget: 50000,
  funds_allocated: 30000,
  funds_utilized: 15000,
  sort_order: 1
});
```

## CORS Configuration

Ensure your backend allows requests from the frontend domain:

```typescript
// Backend CORS config example
app.use(cors({
  origin: ['http://localhost:5173', 'https://your-frontend-domain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

## Error Handling

All API calls should be wrapped in try-catch blocks:

```typescript
try {
  const data = await beneficiariesAPI.list();
  setData(data);
} catch (error) {
  console.error('API Error:', error);
  toast.error('Failed to load data. Please try again.');
}
```

## Testing Checklist

- [ ] API base URL configured correctly
- [ ] Authentication flow working (login/logout)
- [ ] Token storage and retrieval working
- [ ] Protected routes redirect to login when unauthenticated
- [ ] All CRUD operations tested
- [ ] File uploads working (beneficiary import, donation import)
- [ ] Real-time data updates reflected in UI
- [ ] Error handling and user feedback working
- [ ] CORS configured on backend
- [ ] Production environment variables set

## Notes

1. **Phone Number Format**: Ensure phone numbers match your backend validation (currently using Rwandan format: 07XXXXXXXX)

2. **Multilingual Content**: All user-facing content (programs, stories, notifications) uses the `MultilingualText` type with `en`, `rw`, and `fr` fields

3. **Currency Handling**: The system supports RWF, USD, and EUR with exchange rate tracking

4. **File Uploads**: Document uploads and imports use FormData - ensure your backend handles multipart/form-data

5. **Date Formats**: All dates should be in ISO 8601 format (YYYY-MM-DD)

6. **JSON Fields**: Location, permissions, metadata, etc. are stored as JSON objects in the database

## Support

For questions about backend integration, please refer to:
- Type definitions: `/src/types/backend.ts`
- API service: `/src/services/api.ts`
- Auth context: `/src/app/components/AuthContext.tsx`

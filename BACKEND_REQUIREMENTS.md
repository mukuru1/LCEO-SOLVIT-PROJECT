# Backend API Requirements for LCEO Frontend

## Overview

This document specifies the exact response formats your backend should return to work seamlessly with the frontend.

---

## Authentication Endpoints

### POST `/auth/login`

**Request:**
```json
{
  "email": "user@example.com",  // OR "phone": "0788123456"
  "password": "password123"
}
```

**Response (Success - 200):**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "phone": "0788123456",
    "user_type": "admin" | "donor" | "beneficiary",
    "language": "en" | "rw",
    "is_verified": true,
    "is_active": true,
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z",
    "last_login_at": "2024-01-28T08:15:00Z"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (Error - 401):**
```json
{
  "message": "Invalid credentials"
}
```

### POST `/auth/register`

**Request:**
```json
{
  "email": "new@example.com",    // Optional (required if no phone)
  "phone": "0788123456",          // Optional (required if no email)
  "password": "password123",
  "user_type": "beneficiary" | "donor" | "admin"
}
```

**Response (Success - 201):**
```json
{
  "user": {
    "id": "uuid",
    "email": "new@example.com",
    "phone": "0788123456",
    "user_type": "beneficiary",
    "language": "en",
    "is_verified": false,
    "is_active": true,
    "created_at": "2024-01-28T10:30:00Z"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### POST `/auth/logout`

**Request:** (Headers only, needs Authorization token)

**Response (Success - 200):**
```json
{
  "message": "Logged out successfully"
}
```

### POST `/auth/refresh-token`

**Request:** (Headers only, needs Authorization token)

**Response (Success - 200):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## User Endpoints

### GET `/users/profile`

**Response (Success - 200):**
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "phone": "0788123456",
  "user_type": "beneficiary",
  "language": "en",
  "is_verified": true,
  "is_active": true,
  "profile": {
    // For beneficiary:
    "full_name": "Sarah Uwase",
    "date_of_birth": "1998-05-15",
    "location": {
      "province": "Kigali City",
      "district": "Gasabo",
      "sector": "Kimironko"
    },
    "program_id": "uuid",
    "status": "active",
    "enrollment_date": "2024-01-15",
    "business_type": "Tailoring",
    "current_capital": 75000,
    "profile_completion": 85
    
    // For donor:
    // "full_name": "John Smith",
    // "country": "USA",
    // "total_donated": 500,
    // etc.
    
    // For admin (staff):
    // "full_name": "Admin User",
    // "role": "super_admin",
    // "department": "Operations"
    // etc.
  }
}
```

### PUT `/users/profile`

**Request:**
```json
{
  "email": "newemail@example.com",  // Optional
  "language": "rw",                  // Optional
  // Other profile fields based on user type
}
```

**Response (Success - 200):**
Same as GET `/users/profile`

---

## Beneficiary Endpoints

### GET `/beneficiaries`

**Query Parameters:**
- `status` (optional): `active` | `graduated` | `inactive`
- `program_id` (optional): UUID of program

**Response (Success - 200):**
```json
[
  {
    "id": "uuid",
    "user_id": "uuid",
    "full_name": "Sarah Uwase",
    "date_of_birth": "1998-05-15",
    "location": {
      "province": "Kigali City",
      "district": "Gasabo",
      "sector": "Kimironko"
    },
    "program_id": "uuid",
    "status": "active",
    "enrollment_date": "2024-01-15",
    "start_capital": 50000,
    "current_capital": 75000,
    "business_type": "Tailoring",
    "tracking_frequency": "weekly",
    "last_tracking_date": "2024-01-21",
    "next_tracking_date": "2024-01-28",
    "profile_completion": 85,
    "requires_special_attention": false,
    "created_at": "2024-01-15T10:00:00Z",
    "updated_at": "2024-01-25T14:30:00Z"
  }
]
```

### GET `/beneficiaries/:id`

**Response (Success - 200):**
Single beneficiary object (same structure as array item above)

### POST `/beneficiaries/:id/trackings`

**Request:**
```json
{
  "week_ending": "2024-01-28",
  "attendance": "present" | "absent" | "late",
  "task_given": "Develop marketing plan",
  "task_completion_status": "completed" | "in_progress" | "not_done",
  "income_this_week": 25000,
  "expenses_this_week": 15000,
  "current_capital": 60000,
  "sales_data": {
    "products_sold": 12,
    "revenue": 25000,
    "customers_served": 8
  },
  "challenges": "Limited raw materials",
  "solutions_implemented": "Found new supplier",
  "notes": "Business growing steadily",
  "next_week_plan": {
    "goals": ["Increase production by 20%"],
    "activities": ["Visit new supplier", "Update pricing"]
  }
}
```

**Response (Success - 201):**
```json
{
  "id": "uuid",
  "beneficiary_id": "uuid",
  "week_ending": "2024-01-28",
  "attendance": "present",
  // ... all submitted fields
  "submitted_at": "2024-01-28T15:30:00Z",
  "submitted_by": "uuid",
  "is_offline_sync": false
}
```

### GET `/beneficiaries/:id/trackings`

**Response (Success - 200):**
```json
[
  {
    "id": "uuid",
    "week_ending": "2024-01-28",
    "attendance": "present",
    "task_given": "...",
    "task_completion_status": "completed",
    "income_this_week": 25000,
    "expenses_this_week": 15000,
    "current_capital": 60000,
    "submitted_at": "2024-01-28T15:30:00Z"
  }
]
```

### GET `/beneficiaries/:id/goals`

**Response (Success - 200):**
```json
[
  {
    "id": "uuid",
    "beneficiary_id": "uuid",
    "description": "Increase monthly revenue to 200,000 RWF",
    "type": "financial" | "business" | "education" | "personal" | "skills",
    "target_amount": 200000,
    "current_progress": 150000,
    "target_date": "2024-06-30",
    "status": "in_progress" | "not_started" | "achieved" | "abandoned",
    "milestones": [
      {
        "description": "Reach 150,000 RWF",
        "target_date": "2024-03-31",
        "completed": true,
        "completed_date": "2024-03-15"
      }
    ],
    "created_at": "2024-01-15T10:00:00Z"
  }
]
```

---

## Program Endpoints

### GET `/programs`

**Response (Success - 200):**
```json
[
  {
    "id": "uuid",
    "name": {
      "en": "Youth Entrepreneurship Program",
      "rw": "Gahunda y'ubucuruzi bw'urubyiruko",
      "fr": "Programme d'entrepreneuriat des jeunes"
    },
    "description": {
      "en": "Empowering young women through business training",
      "rw": "Gushyira imbaraga abakobwa bato...",
      "fr": "Autonomiser les jeunes femmes..."
    },
    "category": "entrepreneurship" | "education" | "health" | "cross_cutting",
    "sdg_alignment": [
      {
        "goal_number": 1,
        "goal_name": "No Poverty",
        "targets": ["1.2", "1.4"]
      }
    ],
    "kpi_targets": [
      {
        "metric_name": "Beneficiaries enrolled",
        "target_value": 100,
        "current_value": 45,
        "unit": "people"
      }
    ],
    "start_date": "2024-01-01",
    "end_date": null,
    "status": "active" | "planning" | "completed" | "archived",
    "budget": 50000,
    "funds_allocated": 30000,
    "funds_utilized": 15000,
    "cover_image": "https://...",
    "created_at": "2024-01-01T00:00:00Z"
  }
]
```

### GET `/programs/:id/impact`

**Response (Success - 200):**
```json
{
  "program_id": "uuid",
  "metrics": [
    {
      "metric_name": "Total Beneficiaries",
      "metric_value": 156,
      "measurement_unit": "people",
      "period": "annual",
      "period_date": "2024-01-01"
    },
    {
      "metric_name": "Businesses Started",
      "metric_value": 45,
      "measurement_unit": "businesses",
      "period": "annual",
      "period_date": "2024-01-01"
    }
  ]
}
```

---

## Donation Endpoints

### POST `/donations`

**Request:**
```json
{
  "amount": 100,
  "currency": "USD" | "RWF" | "EUR",
  "donation_type": "one_time" | "monthly" | "quarterly" | "yearly",
  "program_id": "uuid",        // Optional
  "project_id": "uuid",        // Optional
  "payment_method": "card" | "mobile_money" | "bank_transfer" | "paypal",
  "is_anonymous": false,
  "donor_message": "Keep up the great work!"
}
```

**Response (Success - 201):**
```json
{
  "id": "uuid",
  "donor_id": "uuid",
  "amount": 100,
  "currency": "USD",
  "local_amount": 125000,      // In RWF
  "exchange_rate": 1250,
  "donation_type": "one_time",
  "payment_status": "pending",
  "transaction_id": "txn_abc123",
  "payment_details": {
    "card_last4": "4242",
    "card_brand": "Visa"
  },
  "created_at": "2024-01-28T10:00:00Z"
}
```

### GET `/donations`

**Response (Success - 200):**
```json
[
  {
    "id": "uuid",
    "amount": 100,
    "currency": "USD",
    "donation_type": "one_time",
    "program_id": "uuid",
    "payment_status": "completed",
    "receipt_sent": true,
    "receipt_number": "RCT-2024-001",
    "created_at": "2024-01-28T10:00:00Z"
  }
]
```

### POST `/donations/recurring`

**Request:**
```json
{
  "amount": 50,
  "currency": "USD",
  "frequency": "monthly" | "quarterly" | "yearly",
  "program_id": "uuid",        // Optional
  "payment_method_id": "pm_abc123"  // Stripe/payment provider ID
}
```

**Response (Success - 201):**
```json
{
  "id": "uuid",
  "donor_id": "uuid",
  "amount": 50,
  "currency": "USD",
  "frequency": "monthly",
  "status": "active",
  "next_charge_date": "2024-02-28",
  "subscription_id": "sub_abc123",
  "created_at": "2024-01-28T10:00:00Z"
}
```

---

## Admin Dashboard Endpoints

### GET `/admin/dashboard/stats`

**Response (Success - 200):**
```json
{
  "total_beneficiaries": 156,
  "active_beneficiaries": 142,
  "total_donors": 89,
  "active_programs": 4,
  "total_donations_amount": 125000,
  "total_donations_count": 234,
  "this_month_donations": 12500,
  "new_beneficiaries_this_month": 12,
  "businesses_launched": 45,
  "success_stories": 28
}
```

### GET `/admin/dashboard/analytics`

**Query Parameters:**
- `start_date`: YYYY-MM-DD
- `end_date`: YYYY-MM-DD

**Response (Success - 200):**
```json
{
  "donations_by_month": [
    { "month": "2024-01", "amount": 15000, "count": 45 },
    { "month": "2024-02", "amount": 18000, "count": 52 }
  ],
  "beneficiaries_by_program": [
    { "program_name": "Entrepreneurship", "count": 85 },
    { "program_name": "Education", "count": 71 }
  ],
  "capital_growth": [
    { "month": "2024-01", "average": 65000 },
    { "month": "2024-02", "average": 72000 }
  ]
}
```

### GET `/admin/reports/donations`

**Query Parameters:**
- `start_date`: YYYY-MM-DD
- `end_date`: YYYY-MM-DD

**Response (Success - 200):**
```json
{
  "total_amount": 125000,
  "total_count": 234,
  "by_currency": {
    "USD": 50000,
    "RWF": 75000000,
    "EUR": 0
  },
  "by_type": {
    "one_time": 180,
    "monthly": 45,
    "quarterly": 9
  },
  "top_donors": [
    {
      "donor_name": "John Smith",
      "total_donated": 5000,
      "donation_count": 12
    }
  ],
  "donations": [
    {
      "id": "uuid",
      "date": "2024-01-28",
      "donor_name": "John Smith",
      "amount": 100,
      "currency": "USD",
      "program": "Entrepreneurship"
    }
  ]
}
```

---

## Content Endpoints

### GET `/content/stories`

**Query Parameters:**
- `is_featured` (optional): true | false
- `is_published` (optional): true | false

**Response (Success - 200):**
```json
[
  {
    "id": "uuid",
    "title": {
      "en": "From Vulnerability to Victory",
      "rw": "Kuva mu ntege nke kugera ku ntsinzi"
    },
    "content": {
      "en": "Sarah's journey began...",
      "rw": "Urugendo rwa Sarah rwatangiye..."
    },
    "author_name": "Sarah Uwase",
    "author_role": "beneficiary" | "donor" | "staff" | "partner" | "volunteer",
    "author_photo": "https://...",
    "program_id": "uuid",
    "beneficiary_id": "uuid",
    "media": [
      {
        "type": "image",
        "url": "https://...",
        "caption": "Sarah at her tailoring shop"
      }
    ],
    "is_featured": true,
    "is_published": true,
    "published_date": "2024-01-15",
    "language": "en",
    "view_count": 256,
    "share_count": 42,
    "created_at": "2024-01-15T10:00:00Z"
  }
]
```

---

## Error Response Format

All errors should follow this format:

**4xx/5xx Errors:**
```json
{
  "message": "Error message here",
  "errors": {  // Optional, for validation errors
    "field_name": ["Error 1", "Error 2"]
  }
}
```

---

## Headers

All authenticated requests should include:

```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

---

## Date/Time Formats

- All dates: ISO 8601 format (`YYYY-MM-DD`)
- All timestamps: ISO 8601 with timezone (`2024-01-28T10:30:00Z`)

---

## Multilingual Content

Fields marked with `MultilingualText` should be objects with language keys:

```json
{
  "en": "English text",
  "rw": "Kinyarwanda text",
  "fr": "French text"
}
```

At minimum, provide English (`en`). Other languages are optional but recommended.

---

## Pagination (Optional but Recommended)

For large datasets, consider pagination:

**Response:**
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 156,
    "total_pages": 8
  }
}
```

Frontend can adapt to handle both paginated and non-paginated responses.

---

## CORS Configuration

Your backend must allow:

```javascript
Access-Control-Allow-Origin: http://localhost:5173 (dev) or your production domain
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

---

## Testing Checklist

- [ ] Login returns user object and JWT token
- [ ] Token is validated on protected routes
- [ ] All enum values match frontend types
- [ ] Date formats are ISO 8601
- [ ] Multilingual fields have at least `en` key
- [ ] Error responses include clear messages
- [ ] CORS headers are configured
- [ ] File uploads work with multipart/form-data
- [ ] Pagination works (if implemented)

---

## Quick Reference

**User Types:** `admin`, `donor`, `beneficiary`

**Program Categories:** `education`, `entrepreneurship`, `health`, `cross_cutting`

**Beneficiary Status:** `active`, `graduated`, `inactive`

**Payment Methods:** `card`, `mobile_money`, `bank_transfer`, `paypal`

**Currencies:** `USD`, `RWF`, `EUR`

See `/src/types/backend.ts` for complete enum definitions.

# API Integration Testing Checklist

Use this checklist to verify your backend API is working correctly with the LCEO frontend.

---

## ✅ Authentication & Authorization

### Login
- [ ] Can login with email and password
- [ ] Can login with phone and password (for beneficiaries)
- [ ] Returns JWT token in response
- [ ] Returns user object with correct user_type
- [ ] Token is stored in localStorage as `lceo_auth_token`
- [ ] Invalid credentials return 401 error
- [ ] Error messages are clear and helpful

### Registration
- [ ] Can register with email
- [ ] Can register with phone number
- [ ] Can specify user_type (admin/donor/beneficiary)
- [ ] Duplicate email/phone returns appropriate error
- [ ] Password validation works
- [ ] Returns JWT token and user object

### Token Management
- [ ] Token is included in Authorization header on protected routes
- [ ] Invalid/expired token returns 401
- [ ] Refresh token endpoint works
- [ ] Token refresh happens automatically before expiration

### Logout
- [ ] Logout endpoint invalidates token
- [ ] Frontend clears localStorage on logout
- [ ] User is redirected to home page

---

## 👤 User Management

### Profile
- [ ] GET /users/profile returns current user data
- [ ] Profile includes role-specific data (beneficiary/donor/staff profile)
- [ ] PUT /users/profile updates user information
- [ ] Language preference is saved and applied

### User Lists (Admin Only)
- [ ] GET /users/beneficiaries returns all beneficiaries
- [ ] GET /users/donors returns all donors
- [ ] GET /users/staff returns all staff
- [ ] Non-admin users get 403 error

---

## 👩‍💼 Beneficiary Management

### List & View
- [ ] GET /beneficiaries returns all beneficiaries
- [ ] Can filter by status (active/graduated/inactive)
- [ ] Can filter by program_id
- [ ] GET /beneficiaries/:id returns single beneficiary
- [ ] All required fields are present in response
- [ ] Location data is properly formatted (JSON object)

### Create & Update
- [ ] POST /beneficiaries creates new beneficiary
- [ ] All required fields are validated
- [ ] PUT /beneficiaries/:id updates beneficiary
- [ ] Only admins can create/update beneficiaries
- [ ] Validation errors return clear messages

### Progress Tracking
- [ ] GET /beneficiaries/:id/trackings returns tracking history
- [ ] POST /beneficiaries/:id/trackings submits new tracking
- [ ] Week ending date is validated
- [ ] Attendance status is enum (present/absent/late)
- [ ] Task completion status is enum
- [ ] Sales data is saved as JSON object
- [ ] Next week plan is saved as JSON object

### Goals
- [ ] GET /beneficiaries/:id/goals returns all goals
- [ ] POST /beneficiaries/:id/goals creates new goal
- [ ] PUT /beneficiaries/:id/goals/:goalId updates goal
- [ ] Goal type is enum (financial/business/education/personal/skills)
- [ ] Goal status is enum (not_started/in_progress/achieved/abandoned)
- [ ] Milestones are saved as JSON array

---

## 📚 Programs & Projects

### Programs
- [ ] GET /programs returns all programs
- [ ] GET /programs/:id returns single program
- [ ] Program names are multilingual (en/rw/fr)
- [ ] Program descriptions are multilingual
- [ ] SDG alignment is JSON array
- [ ] KPI targets are JSON array
- [ ] Category is enum (education/entrepreneurship/health/cross_cutting)
- [ ] Status is enum (planning/active/completed/archived)
- [ ] POST /programs creates new program (admin only)
- [ ] PUT /programs/:id updates program (admin only)

### Program Relationships
- [ ] GET /programs/:id/projects returns program's projects
- [ ] GET /programs/:id/beneficiaries returns program's beneficiaries
- [ ] GET /programs/:id/impact returns impact metrics

### Projects
- [ ] GET /projects returns all projects
- [ ] GET /projects/:id returns single project
- [ ] Project names and descriptions are multilingual
- [ ] Timeline is JSON object
- [ ] Location is JSON object
- [ ] Impact metrics are JSON object
- [ ] Gallery is JSON array of image URLs

---

## 💰 Donations

### One-Time Donations
- [ ] POST /donations creates new donation
- [ ] Amount and currency are required
- [ ] Exchange rate is calculated and saved
- [ ] Transaction ID is unique
- [ ] Payment status is tracked (pending/completed/failed/refunded)
- [ ] Payment method is enum (card/mobile_money/bank_transfer/paypal)
- [ ] Donation type is enum (one_time/monthly/quarterly/yearly)
- [ ] Can specify program_id or project_id
- [ ] Anonymous donations work
- [ ] Donor message is saved

### Recurring Donations
- [ ] POST /donations/recurring creates subscription
- [ ] Frequency is enum (monthly/quarterly/yearly)
- [ ] Next charge date is calculated correctly
- [ ] Status is enum (active/paused/cancelled)
- [ ] GET /donations/recurring returns user's recurring donations
- [ ] PUT /donations/recurring/:id updates subscription
- [ ] DELETE /donations/recurring/:id cancels subscription

### Donation History
- [ ] GET /donations returns donor's donation history
- [ ] Donations are sorted by date (newest first)
- [ ] Receipt information is included
- [ ] Related program/project information is included

---

## 📊 Admin Dashboard

### Statistics
- [ ] GET /admin/dashboard/stats returns overview statistics
- [ ] Total beneficiaries count is accurate
- [ ] Total donors count is accurate
- [ ] Total donations amount is calculated correctly
- [ ] This month's data is filtered correctly
- [ ] Active programs count is correct

### Analytics
- [ ] GET /admin/dashboard/analytics accepts date range
- [ ] Returns donations by month
- [ ] Returns beneficiaries by program
- [ ] Returns capital growth data
- [ ] Data is aggregated correctly

### Reports
- [ ] GET /admin/reports/donations generates donation report
- [ ] Can filter by date range
- [ ] Includes totals by currency
- [ ] Includes breakdown by donation type
- [ ] Includes top donors list
- [ ] GET /admin/reports/beneficiaries generates beneficiary report
- [ ] Can filter by program
- [ ] GET /admin/reports/programs generates program report

### Data Import
- [ ] POST /admin/import/beneficiaries accepts Excel file
- [ ] File is validated before processing
- [ ] Returns success/failure report
- [ ] POST /admin/import/donations accepts CSV file
- [ ] Handles data validation errors

### External Sync
- [ ] POST /admin/sync/kobo syncs with Kobo Toolbox
- [ ] Returns sync status and count
- [ ] Handles API errors gracefully

### Activity Logs
- [ ] GET /admin/activity-logs returns user activity
- [ ] Can filter by user_id
- [ ] Can filter by entity_type
- [ ] Includes old and new values for changes
- [ ] Includes IP address and user agent

---

## 📝 Content Management

### Stories
- [ ] GET /content/stories returns all stories
- [ ] Can filter by is_featured
- [ ] Can filter by is_published
- [ ] Story title and content are multilingual
- [ ] Media is JSON array with type and URL
- [ ] Author role is enum
- [ ] View count and share count are tracked
- [ ] POST /content/stories creates new story (admin only)
- [ ] PUT /content/stories/:id updates story (admin only)

---

## 🔒 Security

### Authorization
- [ ] Protected routes require valid JWT token
- [ ] Admin-only routes check user role
- [ ] Users can only access their own data (donors/beneficiaries)
- [ ] Admins can access all data
- [ ] Proper HTTP status codes (401, 403, 404, etc.)

### Data Validation
- [ ] Email format is validated
- [ ] Phone number format is validated
- [ ] Required fields are enforced
- [ ] Enum values are validated
- [ ] Date formats are validated
- [ ] Amount/number fields accept decimals
- [ ] SQL injection prevention is in place
- [ ] XSS prevention is in place

### CORS
- [ ] Frontend domain is allowed
- [ ] Credentials are allowed
- [ ] Proper headers are set
- [ ] OPTIONS requests are handled

---

## 📱 Phone Number Support (Beneficiaries)

### Login
- [ ] Can login with phone number instead of email
- [ ] Phone number format is validated (Rwandan: 07XXXXXXXX)
- [ ] Phone number is stored in E.164 format or local format

### Registration
- [ ] Can register with phone number only (no email required)
- [ ] Phone number uniqueness is enforced
- [ ] SMS verification code is sent (if implemented)

### Verification
- [ ] POST /auth/verify-phone validates phone number
- [ ] Verification code is validated
- [ ] Phone is marked as verified

---

## 🌍 Multi-Language Support

### Content
- [ ] All user-facing content supports en/rw/fr
- [ ] At minimum, English (en) is provided
- [ ] Missing translations fall back to English
- [ ] Program names are multilingual
- [ ] Program descriptions are multilingual
- [ ] Story titles and content are multilingual
- [ ] Notification messages are multilingual

---

## 📊 Data Formats

### Dates
- [ ] All dates are in ISO 8601 format (YYYY-MM-DD)
- [ ] All timestamps include timezone (YYYY-MM-DDTHH:mm:ssZ)
- [ ] Date validation rejects invalid dates

### Numbers
- [ ] Currency amounts support decimals (2 decimal places)
- [ ] Exchange rates support 4 decimal places
- [ ] Percentages are stored as integers (0-100)

### JSON Fields
- [ ] Location fields are valid JSON objects
- [ ] Multilingual text fields are valid JSON objects
- [ ] Sales data is valid JSON
- [ ] Timeline is valid JSON
- [ ] Milestones are valid JSON arrays

---

## 🚀 Performance

### Response Times
- [ ] GET requests respond in < 500ms
- [ ] POST requests respond in < 1s
- [ ] Large lists use pagination
- [ ] Database queries are optimized
- [ ] Indexes are in place for commonly queried fields

### Caching
- [ ] Static content has cache headers
- [ ] Frequently accessed data is cached
- [ ] Cache invalidation works correctly

---

## 🐛 Error Handling

### Error Responses
- [ ] All errors return JSON with "message" field
- [ ] Validation errors include "errors" object with field names
- [ ] HTTP status codes are correct
- [ ] Error messages are user-friendly
- [ ] Stack traces are not exposed in production

### Edge Cases
- [ ] Empty lists return [] not null
- [ ] Missing optional fields return null
- [ ] Invalid IDs return 404
- [ ] Duplicate data returns 409
- [ ] Rate limiting is implemented

---

## 📦 File Uploads

### Document Upload
- [ ] Accepts PDF, JPG, PNG files
- [ ] File size is limited (max 5MB recommended)
- [ ] Files are validated before storage
- [ ] File URLs are returned in response
- [ ] Files are accessible via returned URL

### Import Files
- [ ] Excel/CSV files are validated
- [ ] Row-by-row validation is performed
- [ ] Partial success is handled gracefully
- [ ] Import results include success/failure counts

---

## ✨ Nice to Have

### Optional Features
- [ ] Email notifications are sent
- [ ] SMS notifications are sent (for phone users)
- [ ] Receipt emails are sent for donations
- [ ] Tracking reminders are sent to beneficiaries
- [ ] Real-time updates with WebSockets
- [ ] API rate limiting
- [ ] API versioning (v1, v2, etc.)
- [ ] API documentation (Swagger/OpenAPI)

---

## 🧪 Testing Tools

### Recommended Tools:
- **Postman**: API testing and collection management
- **Insomnia**: Alternative to Postman
- **curl**: Command-line testing
- **Jest/Mocha**: Automated backend tests
- **Cypress**: End-to-end frontend tests

### Test Data:
- Create test users for each role
- Create test programs and beneficiaries
- Create test donations with different statuses
- Test with edge cases (empty data, maximum values, etc.)

---

## 📞 Support

If you encounter issues:
1. Check error messages in browser console
2. Check network tab in browser DevTools
3. Verify API endpoint URL is correct
4. Verify JWT token is being sent
5. Check backend logs for errors
6. Refer to `/BACKEND_REQUIREMENTS.md` for expected response formats

---

## ✅ Final Checklist

Before going live:
- [ ] All endpoints tested and working
- [ ] Security measures in place
- [ ] Error handling is comprehensive
- [ ] Performance is acceptable
- [ ] Documentation is complete
- [ ] Demo credentials are removed
- [ ] Production environment variables set
- [ ] CORS configured for production domain
- [ ] SSL/HTTPS enabled
- [ ] Database backups configured
- [ ] Monitoring and logging in place

---

**Last Updated:** January 28, 2026
**Frontend Version:** Aligned with LCEO Backend Schema v1

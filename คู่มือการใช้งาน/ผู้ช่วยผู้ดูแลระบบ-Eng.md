# SRI-KETGUIDE User Manual for Staff Members

**Role:** Staff (Administrative Assistant)  
**Primary Responsibility:** Review and approve payment slips submitted by sponsors (advertisers)

⚠️ **Note:** The primary responsibility of Staff members is to review and approve payment slips only. Other management functions (such as user management, package management) are exclusively handled by Administrators.

---

## Table of Contents
1. [System Access](#1-system-access)
2. [Staff Panel Overview](#2-staff-panel-overview)
3. [Payment Slips (Primary Function)](#3-payment-slips-primary-function)
4. [Viewing and Reviewing Slips](#4-viewing-and-reviewing-slips)
5. [Approving Payment Slips](#5-approving-payment-slips)
6. [Rejecting Payment Slips](#6-rejecting-payment-slips)
7. [Filtering and Searching Slips](#7-filtering-and-searching-slips)
8. [Staff Permissions and Limitations](#8-staff-permissions-and-limitations)
9. [Frequently Asked Questions (FAQ)](#9-frequently-asked-questions-faq)
10. [Warnings and Precautions](#10-warnings-and-precautions)

---

## 1. System Access

### 1.1 Login Screen

```
+-------------------------------------------------------------------+
|                                                                   |
|                     SRI-KETGUIDE                                  |
|          Intelligent Tourism Promotion System                     |
|                                                                   |
|  +----------------------------------------------------------+     |
|  |                                                          |     |
|  |   Email: [____________________________]                  |     |
|  |                                                          |     |
|  |   Password: [____________________________]               |     |
|  |                                                          |     |
|  |              [     Log In     ]                          |     |
|  |                                                          |     |
|  |   [ ] Remember me                                         |     |
|  |                                                          |     |
|  |   Forgot password?                                       |     |
|  |                                                          |     |
|  +----------------------------------------------------------+     |
|                                                                   |
|  URL: https://frontend.sri-ketguide.com/#/login                |
|                                                                   |
+-------------------------------------------------------------------+
```

### 1.2 Login Procedure

1. Open a web browser and navigate to the URL: `https://frontend.sri-ketguide.com/#/login`
2. Enter your login credentials:
   - **Email**: Enter the email address provided by the administrator
   - **Password**: Enter the password you received
3. Click the **"Log In"** button or press Enter
4. Optionally, select **"Remember me"** to save your login session

### 1.3 Staff Account Information

**Staff Account:**
- Email: `staff@example.com`
- Password: `1234`
- Role: `staff`
- Responsibility: Review and approve payment slips

⚠️ **Note:** Since the Staff role primarily focuses on payment slip approval, this manual emphasizes that functionality.

### 1.4 Login Security Precautions

⚠️ **Security Warnings:**
- Do not share your login credentials with others
- Log out when you have finished your work session
- Do not select "Remember me" when using a public computer
- If you forget your password, contact the system administrator

---

## 2. Staff Panel Overview

### 2.1 Staff Panel Interface

```
+-------------------------------------------------------------------+
|  ┌───────────────────────────────────────────────────────────┐  |
|  │  SRI-KETGUIDE                           [Search] [Logout]  │  |
|  └───────────────────────────────────────────────────────────┘  |
|                                                                   |
|  ┌───────────────────────────────────────────────────────────┐  |
|  │  [Chat]  [Staff Panel▼]  [Sponsor Panel]  [Admin Panel]   │  |
|  └───────────────────────────────────────────────────────────┘  |
|                                                                   |
|  ┌───────────────────────────────────────────────────────────┐  |
|  │  ╔═══════════════════════════════════════════════════════╗ │  |
|  │  ║  👨‍💼  Staff Panel                                  ║ │  |
|  │  ║     Review and approve payment slips                    ║ │  |
|  │  ╚═══════════════════════════════════════════════════════╝ │  |
|  └───────────────────────────────────────────────────────────┘  |
|                                                                   |
|  ┌───────────────────────────────────────────────────────────┐  |
|  │  💰 Payment Slips                                         │  |
|  │     [All▼] [Pending (3)] [Approved] [Rejected]            │  |
|  └───────────────────────────────────────────────────────────┘  |
|                                                                   |
+-------------------------------------------------------------------+
```

### 2.2 Staff Panel Components

The main Staff Panel interface consists of:

1. **Navigation Bar**
   - System logo and name
   - Menu items: Chat, Staff Panel, Sponsor Panel, Admin Panel
   - Search field
   - Logout button

2. **Content Area**
   - Displays payment slips awaiting review
   - Slip table with action buttons (View, Approve, Reject)

3. **Filter Controls**
   - Filter by status: All, Pending Review, Approved, Rejected

### 2.3 Difference Between Staff and Admin Roles

⚠️ **Staff Permissions:**
- ✅ View all payment slips
- ✅ Approve payment slips
- ✅ Reject payment slips
- ❌ User management (Admin only)
- ❌ Package management (Admin only)
- ❌ Business management (Admin only - Staff may have limited access in some systems)

---

## 3. Payment Slips (Primary Function)

### 3.1 Payment Slip Interface

```
+-------------------------------------------------------------------+
|  💰 Payment Slips     [All▼] [Pending] [Approved] [Rejected]│
|  ┌─────────────────────────────────────────────────────────┐     |
|  | ID | Business | Package | Amount | Date | Status | Actions|     |
|  ├──────┼────────┼─────────┼───────┼───────┼───────┼───────│     |
|  | #1a | Babaray  | Premium | 999฿ | Jan 1 | ⏳Pending|[View][✓][✗]|     |
|  | #1b | Better   |Standard | 599฿ | Jan 2 | ✅Approved| [View]      |     |
|  | #1c | Sweet    | Free    | 0฿   | Jan 3 | ❌Rejected| [View]      |     |
|  └──────┴────────┴─────────┴───────┴───────┴───────┴───────┘     |
+-------------------------------------------------------------------+
```

### 3.2 Importance of Payment Slip Verification

⚠️ **Primary Responsibilities of Staff:**
- Verify the accuracy of payment slips
- Verify that the payment amount matches the selected package
- Verify the clarity of the slip image
- Approve or reject slips fairly and consistently

---

## 4. Viewing and Reviewing Slips

### 4.1 Viewing Payment Slips

**Procedure:**

1. Locate the payment slip you wish to view
2. Click the **"View"** button (🖼️ icon)
3. The system will display the payment slip image:

```
+-------------------------------------------------------------------+
|                    Payment Slip Image                              |
|  ┌─────────────────────────────────────────────────────────┐     |
|  |                                                         |     |
|  |                  [Large Slip Image]                     |     |
|  |                                                         |     |
|  |  Slip ID: #1a2b3c4d                                    |     |
|  |  Business: Babaray Hotel                                |     |
|  |  Package: Premium                                       |     |
|  |  Amount: 999 THB                                        |     |
|  |  Date: January 1, 2025                                  |     |
|  |                                                         |     |
|  |              [Close]                                    |     |
|  |                                                         |     |
|  └─────────────────────────────────────────────────────────┘     |
+-------------------------------------------------------------------+
```

**API Endpoint:**
- `GET /api/payment-slips/{slipId}`
- Image path: `/uploads/payment-slips/{filename}`

---

## 5. Approving Payment Slips

**Procedure:**

1. Locate a slip with **"Pending Review"** status (⏳)
2. Click the **"Approve"** button (✓ icon)
3. The system will display a confirmation dialog:

```
+-------------------------------------------------------------------+
|                    Confirm Approval                                |
|                                                                   |
|  Do you want to approve slip #1a2b3c4d?                          |
|                                                                   |
|  Business: Babaray Hotel                                         |
|  Package: Premium                                                |
|  Amount: 999 THB                                                 |
|                                                                   |
|           [Cancel]              [Approve]                         |
+-------------------------------------------------------------------+
```

4. Click **"Approve"** to confirm

**API Endpoint:**
- `POST /api/payment-slips/{slipId}/approve`
- Requires Authorization Header and Role: `staff` or `admin`

**Results:**
- The slip status will change to **"Approved"** (✅)
- The business will receive the package according to the payment
- The system will automatically create or update the Business Package

---

## 6. Rejecting Payment Slips

**Procedure:**

1. Locate a slip with **"Pending Review"** status
2. Click the **"Reject"** button (✗ icon)
3. Enter the rejection reason (optional):

```
+-------------------------------------------------------------------+
|                    Reject Payment Slip                            |
|                                                                   |
|  Do you want to reject slip #1a2b3c4d?                           |
|                                                                   |
|  Rejection reason (optional):                                    |
|  [_________________________________________]                     |
|  [_________________________________________]                     |
|                                                                   |
|           [Cancel]              [Reject]                          |
+-------------------------------------------------------------------+
```

4. Click **"Reject"** to confirm

**API Endpoint:**
- `POST /api/payment-slips/{slipId}/reject`
- Requires Authorization Header and Role: `staff` or `admin`

**Results:**
- The slip status will change to **"Rejected"** (❌)
- The business will receive a notification (if notification system is enabled)
- The business can upload a new slip

---

## 7. Filtering and Searching Slips

### 7.1 Filtering Payment Slips

**Procedure:**

1. Click the **"All"** dropdown menu in the top right corner
2. Select the desired status:
   - **All**: Display all slips
   - **Pending Review**: Display only slips awaiting approval
   - **Approved**: Display only approved slips
   - **Rejected**: Display only rejected slips

**API Endpoint:**
- `GET /api/payment-slips?status={status}`

⚠️ **Warning:**
- Review slips carefully before approval
- Verify that:
  - The payment amount matches the selected package
  - The transfer date and time are correct
  - The business information is accurate
  - The slip image is clear and readable

---

## 8. Staff Permissions and Limitations

### 8.1 What Staff Can Do

✅ **Authorized Actions:**
- View all payment slips
- View details of individual slips
- Approve payment slips
- Reject payment slips
- Filter slips by status

### 8.2 What Staff Cannot Do

❌ **Unauthorized Actions:**
- ❌ User management (create, edit, delete, suspend) - Admin only
- ❌ Package management (create, edit, delete) - Admin only
- ❌ Business management (in some cases) - Refer to actual system permissions
- ❌ System configuration

### 8.3 Requesting Additional Features

If Staff members need access to other management functions:
1. Contact an Administrator to request additional permissions
2. Request that an Administrator perform the management task
3. Request an upgrade to Admin role (subject to system configuration)

⚠️ **Note:** Other management functions should be handled exclusively by Administrators for security and proper system control.

---

## 9. Frequently Asked Questions (FAQ)

### 9.1 Questions About Staff Permissions

**Q: Can Staff members manage users or packages?**
A: ❌ **No.** Staff members can only approve and reject payment slips. User and package management are Administrator responsibilities.

**Q: I need additional permissions to manage businesses. What should I do?**
A: Contact an Administrator to:
- Request additional permissions for your Staff account
- Request an upgrade to Administrator role
- Request that an Administrator perform the management task

### 9.2 Questions About Approving Slips

**Q: Can an approved slip be canceled?**
A: Approved slips cannot be canceled directly. Contact an Administrator to handle the situation if an error occurs.

**Q: I see an irregular slip. What should I do?**
A: 
1. Reject the slip and record a clear reason
2. If uncertain, contact an Administrator for guidance
3. Do not approve slips if you have doubts

**Q: What should I verify before approving a slip?**
A: Verify:
- ✅ The payment amount matches the selected package (Premium: 999 THB, Standard: 599 THB, Free: 0 THB)
- ✅ The transfer date and time are correct
- ✅ The business information matches the slip (business name, owner)
- ✅ The slip image is clear and readable

**Q: What can advertisers do if their slip is rejected?**
A: Advertisers can upload a new slip, but they should ensure the new slip is correct before submission.

### 9.3 Questions About System Usage

**Q: I don't see any slips in the system. What should I do?**
A: 
1. Check the filter settings - a status filter may be applied
2. Try refreshing the web page
3. Contact an Administrator to verify if there are pending slips for review

**Q: I made a mistake approving/rejecting a slip. What should I do?**
A: Contact an Administrator immediately to:
- Request correction of the slip status
- Explain the situation to find a solution

---

## 10. Related Commands and Paths

### 10.1 API Endpoints for Staff

⚠️ **Note:** Staff members have access only to the APIs listed below.

#### 10.1.1 Payment Slips - Primary Function

```
GET    /api/payment-slips                 - Retrieve all slips (Staff/Admin/Sponsor)
GET    /api/payment-slips/:id             - Retrieve slip details (Staff/Admin)
POST   /api/payment-slips/:id/approve     - Approve slip (Staff/Admin) ⭐
POST   /api/payment-slips/:id/reject      - Reject slip (Staff/Admin) ⭐
```

**Example Request (Approve):**
```bash
POST /api/payment-slips/123/approve
Authorization: Bearer {accessToken}
```

**Example Request (Reject):**
```bash
POST /api/payment-slips/123/reject
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "reason": "Payment amount does not match package"
}
```

#### 10.1.2 APIs Not Available to Staff

❌ **Users** - Admin only
❌ **Packages** - Admin only
❌ **Transactions** - Depends on system configuration

⚠️ **To access these APIs, contact an Administrator**

### 10.2 Related File Paths

**Frontend:**
- `/home/vit/llm-chat-frontend/src/templates/staff.html` - Staff Panel page
- `/home/vit/llm-chat-frontend/src/js/staff.js` - JavaScript file for Staff Panel
- `/home/vit/llm-chat-frontend/src/js/api.js` - API Client file

**Backend:**
- `/home/vit/llm-chat-backend/src/routes/payment-slip.routes.js` - Routes for payment slips (primary function)
- `/home/vit/llm-chat-backend/src/controllers/payment-slip.controller.js` - Controller for slips

**Database:**
- Database: `llmchat_db`
- Related tables:
  - `payment_slips` - Payment slips
  - `business_packages` - Business-package relationships (updated after approval)

⚠️ **Note:** Other tables (e.g., `users`, `packages`, `businesses`) are managed by Administrators.

### 10.3 Related Docker Commands

```bash
# View API logs
docker logs vit-llm-chat-api-1 -f

# View Database logs
docker logs vit-llm-chat-database-1 -f

# Access API container
docker exec -it vit-llm-chat-api-1 bash

# Access Database container
docker exec -it vit-llm-chat-database-1 psql -U postgres -d llmchat_db

# Restart API service
docker restart vit-llm-chat-api-1
```

### 10.4 Related SQL Commands for Staff

**Commands that Staff may need:**

```sql
-- View pending slips (primary function)
SELECT id, business_id, status, amount, created_at 
FROM payment_slips 
WHERE status = 'pending'
ORDER BY created_at ASC;

-- View approved slips
SELECT id, business_id, status, amount, created_at, approved_at
FROM payment_slips 
WHERE status = 'approved'
ORDER BY approved_at DESC
LIMIT 50;

-- View rejected slips
SELECT id, business_id, status, amount, created_at, rejected_at
FROM payment_slips 
WHERE status = 'rejected'
ORDER BY rejected_at DESC
LIMIT 50;

-- Count slips by status
SELECT status, COUNT(*) as count 
FROM payment_slips 
GROUP BY status;
```

⚠️ **Note:** 
- Staff should primarily use the Web Interface
- These SQL commands are for data verification only
- Do not modify data in the database directly (use the Web Interface)

---

## 11. Warnings and Precautions

### 11.1 Payment Slip Verification

⚠️ **Warnings:**
1. **Review slips carefully**: 
   - Payment amount must match the package
   - Date and time must be correct
   - Business information must match the slip
2. **Approve with caution**: Approval will immediately grant the package to the business
3. **Record rejection reasons**: Record clear reasons for reference

### 11.2 Security

⚠️ **Warnings:**
1. **Log out when finished**: Always log out after completing your work session
2. **Do not share login credentials**: Never share your email and password with others
3. **Use strong passwords**: Change your password periodically

### 11.3 Common Issues and Troubleshooting

#### Issue: Unable to log in
**Resolution:**
1. Verify email (`staff@example.com`) and password (`1234`)
2. Check if the account has been suspended (contact Administrator)
3. Clear browser cache and cookies
4. Contact an Administrator

#### Issue: Unable to approve slip
**Resolution:**
1. Verify that you have Staff permissions
2. Verify that the slip is in "Pending Review" status
3. Verify that the slip has not already been approved
4. Contact an Administrator if the issue persists

#### Issue: Slips not displaying
**Resolution:**
1. Check the filter settings - a status filter may be applied
2. Try refreshing the web page
3. Verify that no date filter is applied
4. Contact an Administrator if slips still do not appear

---

## 12. Support and Assistance

### 12.1 Contact Administrator

If you encounter problems or need assistance:
- **Email**: `admin@example.com`
- **Phone**: [Contact information]

### 12.2 Frequently Asked Questions (FAQ)

**Q: Can Staff members manage users or packages?**
A: ❌ No. Staff members can only approve and reject payment slips. User and package management are Administrator responsibilities.

**Q: Can an approved slip be canceled?**
A: Approved slips cannot be canceled directly. Contact an Administrator to handle the situation.

**Q: I see an irregular slip. What should I do?**
A: Reject the slip and record a clear reason. If uncertain, contact an Administrator for guidance.

**Q: Can Staff members edit business information?**
A: This depends on system configuration, but generally Staff should not edit business information. Contact an Administrator.

**Q: I need additional permissions. What should I do?**
A: Contact an Administrator to request additional permissions or request an upgrade to Administrator role.

---

## Summary

This manual covers the use of the SRI-KETGUIDE system for Staff members, whose primary responsibilities include:

✅ **Review and approve payment slips** - Primary function  
✅ **Reject incorrect slips** - When errors are found  
✅ **View and filter slips** - For work efficiency  

❌ **Cannot manage:**
- ❌ Users - Admin only
- ❌ Packages - Admin only
- ❌ System configuration

For access to additional features, please contact an Administrator.

**Staff Account:**
- Email: `staff@example.com`
- Password: `1234`
- Role: `staff`

---

**Last Updated:** October 2025  
**Version:** 2.0 (Updated to match actual Staff permissions)  
**URL:** https://frontend.sri-ketguide.com  
**Role:** Staff (Administrative Assistant - Payment Slip Approval)


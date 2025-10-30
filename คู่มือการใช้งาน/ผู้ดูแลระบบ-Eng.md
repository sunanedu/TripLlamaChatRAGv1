# SRI-KETGUIDE User Manual for Administrators

**User Account:** `admin@example.com`  
**Password:** `1234`  
**URL:** `https://frontend.sri-ketguide.com/#/admin`

⚠️ **Note:** This manual is for **Administrators only**. Administrators have full access to manage the entire system, including all features that Staff have, and can manage users, packages, and system configuration.

---

## Table of Contents
1. [System Access](#1-system-access)
2. [Admin Panel Overview](#2-admin-panel-overview)
3. [User Management (Admin Only)](#3-user-management-admin-only)
4. [Package Management (Admin Only)](#4-package-management-admin-only)
5. [Business Management](#5-business-management)
6. [Payment Slips](#6-payment-slips)
7. [Business Image Management](#7-business-image-management)
8. [System Configuration](#8-system-configuration)
9. [Creating Embeddings](#9-creating-embeddings)
10. [Data Backup and Recovery](#10-data-backup-and-recovery)
11. [Transaction Management](#11-transaction-management)
12. [Related Commands and Paths](#12-related-commands-and-paths)
13. [Warnings and Precautions for Administrators](#13-warnings-and-precautions-for-administrators)

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
|  +----------------------------------------------------------+     |
|                                                                   |
|  URL: https://frontend.sri-ketguide.com/#/login                |
|                                                                   |
+-------------------------------------------------------------------+
```

### 1.2 Login Information for Admin

- **Email:** `admin@example.com`
- **Password:** `1234`
- **Role:** `admin` (Full privileges)

⚠️ **Security Warnings for Administrators:**
- ⚠️ **Never share login credentials with anyone**
- ⚠️ **Use strong passwords and change them periodically**
- ⚠️ **Always log out after completing your work session**
- ⚠️ **Do not remember passwords on public computers**
- ⚠️ **Regularly check login logs**

---

## 2. Admin Panel Overview

### 2.1 Admin Panel Interface

```
+-------------------------------------------------------------------+
|  ┌───────────────────────────────────────────────────────────┐  |
|  │  SRI-KETGUIDE           [Chat] [Staff] [Sponsor] [Admin]  │  |
|  └───────────────────────────────────────────────────────────┘  |
|                                                                   |
|  ┌───────────────────────────────────────────────────────────┐  |
|  │  📋 Management Tabs:                                       │  |
|  │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────┐ │  |
|  │  │ 🏪 Manage  │ │ 👥 Manage  │ │ 📦 Manage  │ │ 💰 Slips│ │  |
|  │  │Business(40)│ │ Users (15) │ │Packages (3)│ │  (12)  │ │  |
|  │  └────────────┘ └────────────┘ └────────────┘ └────────┘ │  |
|  └───────────────────────────────────────────────────────────┘  |
|                                                                   |
|  ┌───────────────────────────────────────────────────────────┐  |
|  │  🛡️  Admin Panel                                          │  |
|  │     Manage entire system                                   │  |
|  └───────────────────────────────────────────────────────────┘  |
|                                                                   |
+-------------------------------------------------------------------+
```

### 2.2 Difference Between Admin and Staff

**Admin has additional privileges:**

✅ **User Management** - Create/edit/delete/suspend all users  
✅ **Package Management** - Create/edit/delete packages  
✅ **Full System Management** - Access all features  
✅ **View Logs and Statistics** - Monitor system usage  
✅ **System Configuration** - Adjust system settings  

**Staff can:**
✅ Manage businesses  
✅ Manage payment slips  
❌ User management (no permission)  
❌ Package management (no permission)  

---

## 3. User Management (Admin Only)

### 3.1 User Management Interface

```
+-------------------------------------------------------------------+
|  👥 User Management                    [+ Add User]                 |
|  ┌─────────────────────────────────────────────────────────┐     |
|  │ Email         │ Name │ Role  │ Status │ Actions        │     |
|  ├───────────────┼──────┼───────┼────────┼────────────│     |
|  │ admin@...     │ Admin│ admin │ ✅     │ [Edit] [Delete]│     |
|  │ staff@...     │ Staff│ staff │ ✅     │ [Edit] [Delete]│     |
|  │ sponsor@...   │ Sponsor│ sponsor│ ✅ │ [Edit] [Delete]│     |
|  │ member@...    │ Member│ member│ ✅   │ [Edit] [Delete]│     |
|  │ guest@...     │ Guest│ guest │ ✅    │ [Edit] [Delete]│     |
|  └───────────────┴──────┴───────┴────────┴────────────┘     |
+-------------------------------------------------------------------+
```

### 3.2 User Management Features

#### 3.2.1 Adding Users

**Procedure:**

1. Click the **"+ Add User"** button
2. Fill in the form:

```
+-------------------------------------------------------------------+
|                  Add New User                                      |
|  ┌─────────────────────────────────────────────────────────┐     |
|  │                                                         |     |
|  │  Email *:                                               |     |
|  │  [_________________________________________]             |     |
|  │                                                         |     |
|  │  Password *:                                            |     |
|  │  [_________________________________________]             |     |
|  │  (Minimum 6 characters)                                 |     |
|  │                                                         |     |
|  │  Confirm Password *:                                    |     |
|  │  [_________________________________________]             |     |
|  │                                                         |     |
|  │  First Name:                                            |     |
|  │  [_________________________________________]             |     |
|  │                                                         |     |
|  │  Last Name:                                             |     |
|  │  [_________________________________________]             |     |
|  │                                                         |     |
|  │  Role *:                                                |     |
|  │  [Select Role▼]                                          |     |
|  │    - admin                                              |     |
|  │    - staff                                              |     |
|  │    - sponsor                                            |     |
|  │    - member_pro                                         |     |
|  │    - member                                             |     |
|  │    - guest                                              |     |
|  │                                                         |     |
|  │  [Cancel]              [Save]                            |     |
|  │                                                         |     |
|  └─────────────────────────────────────────────────────────┘     |
+-------------------------------------------------------------------+
```

**API Endpoint:**
- `POST /api/users`
- Requires Authorization Header and Role: `admin` only

**Example Request:**
```json
{
  "email": "newuser@example.com",
  "password": "securepassword123",
  "role": "member",
  "first_name": "John",
  "last_name": "Doe"
}
```

⚠️ **Warnings:**
- Ensure email is unique
- Use strong passwords (minimum 6 characters)
- Select appropriate role for user's duties

#### 3.2.2 Editing Users

**Procedure:**

1. Find the user you want to edit
2. Click the **"Edit"** button (✏️ icon)
3. Edit information:

**Editable Information:**
- Email
- Password (if you want to change)
- First/Last Name
- Role
- Status (suspend/activate)

**API Endpoint:**
- `PUT /api/users/{userId}`

⚠️ **Important Warnings:**
- ⚠️ **Do not change user roles unnecessarily**
- ⚠️ **Changing to Admin role requires extreme caution**
- ⚠️ **Verify requirements before changing roles**
- ⚠️ **Primary Admin should not be deleted or suspended**

#### 3.2.3 Suspending/Activating Accounts

**Procedure:**

1. Find the user you want to suspend/activate
2. Click the **"Edit"** button
3. Change status **"Suspend/Activate"**:

```
+-------------------------------------------------------------------+
|              Edit User                                             |
|  ┌─────────────────────────────────────────────────────────┐     |
|  │                                                         |     |
|  │  Email: user@example.com                                |     |
|  │                                                         |     |
|  │  Status:                                                |     |
|  │  [○] Active                                             |     |
|  │  [●] Suspended                                          |     |
|  │                                                         |     |
|  │  [Cancel]              [Save]                            |     |
|  │                                                         |     |
|  └─────────────────────────────────────────────────────────┘     |
+-------------------------------------------------------------------+
```

**API Endpoint:**
- `PUT /api/users/{userId}/status`

**Request:**
```json
{
  "is_suspended": true  // or false
}
```

**Results:**
- Suspended users cannot log in
- User data and businesses remain
- Can be activated again

⚠️ **Warnings:**
- ⚠️ **Primary Admin cannot suspend themselves**
- ⚠️ **Suspension may affect businesses linked to that account**
- ⚠️ **Notify users before suspending accounts**

#### 3.2.4 Deleting Users

**Procedure:**

1. Find the user you want to delete
2. Click the **"Delete"** button (🗑️ icon)
3. System displays confirmation message:

```
+-------------------------------------------------------------------+
|                    ⚠️ Confirm Deletion                             |
|                                                                   |
|  Do you want to delete user "user@example.com"?                   |
|                                                                   |
|  ⚠️ This action will:                                            |
|  • Delete all user data                                          |
|  • Delete usage history                                          |
|  • Businesses linked to this account will have no owner          |
|                                                                   |
|  This action cannot be undone!                                    |
|                                                                   |
|           [Cancel]              [Delete]                          |
+-------------------------------------------------------------------+
```

4. Click **"Delete"** to confirm

**API Endpoint:**
- `DELETE /api/users/{userId}`

⚠️ **Critical Warnings:**
- ⚠️⚠️⚠️ **Do not delete primary Admin**
- ⚠️⚠️ **User deletion cannot be recovered**
- ⚠️ **Backup data before deletion**
- ⚠️ **Verify user has no important businesses before deletion**

#### 3.2.5 Viewing User Details

**Procedure:**

1. Find the user you want to view details for
2. Click on name or email (if there's a link) or "View Details" button
3. View information:

**Information Displayed:**
- Email
- First/Last Name
- Role
- Status
- Account creation date
- Number of related businesses
- Usage history (if available)

**API Endpoint:**
- `GET /api/users/{userId}`

---

## 4. Package Management (Admin Only)

### 4.1 Package Management Interface

```
+-------------------------------------------------------------------+
|  📦 Package Management                    [+ Add Package]         |
|  ┌─────────────────────────────────────────────────────────┐     |
|  │ Package Name │ Price │ Rate/Min │ Status │ Actions     │     |
|  ├──────────────┼───────┼───────────┼───────┼────────────│     |
|  │ Premium      │ 999฿  │ Unlimited│ ✅     │ [Edit] [Del]│     |
|  │ Standard     │ 599฿  │ Unlimited│ ✅     │ [Edit] [Del]│     |
|  │ Free         │ 0฿    │ Unlimited│ ✅     │ [Edit] [Del]│     |
|  └──────────────┴───────┴───────────┴───────┴────────────┘     |
+-------------------------------------------------------------------+
```

### 4.2 Package Management Features

#### 4.2.1 Adding Packages

**Procedure:**

1. Click the **"+ Add Package"** button
2. Fill in information:

```
+-------------------------------------------------------------------+
|                  Add New Package                                   |
|  ┌─────────────────────────────────────────────────────────┐     |
|  │                                                         |     |
|  │  Package Name *:                                        |     |
|  │  [_________________________________________]             |     |
|  │  (e.g.: Premium, Standard, Free)                         |     |
|  │                                                         |     |
|  │  Price (THB) *:                                         |     |
|  │  [_________________________________________]             |     |
|  │  (e.g.: 999, 599, 0)                                    |     |
|  │                                                         |     |
|  │  Requests per minute:                                   |     |
|  │  [_________________________________________]             |     |
|  │  (Leave empty for "unlimited")                           |     |
|  │                                                         |     |
|  │  Description:                                           |     |
|  │  [_________________________________________]             |     |
|  │  [_________________________________________]             |     |
|  │                                                         |     |
|  │  [Cancel]              [Save]                            |     |
|  │                                                         |     |
|  └─────────────────────────────────────────────────────────┘     |
+-------------------------------------------------------------------+
```

**API Endpoint:**
- `POST /api/packages`
- Requires Authorization Header and Role: `admin`

**Example Request:**
```json
{
  "name": "Premium",
  "price": 999,
  "requests_per_minute": null,
  "description": "Premium package for large businesses. Supports top search ranking"
}
```

**Required Information:**
- **Package Name** (required): e.g., Premium, Standard, Free
- **Price** (required): Price in Thai Baht
- **Requests per minute**: Number of questions allowed per minute (null = unlimited)
- **Description**: Package details

#### 4.2.2 Editing Packages

**Procedure:**

1. Find the package you want to edit
2. Click the **"Edit"** button
3. Edit information
4. Click **"Save"**

**API Endpoint:**
- `PUT /api/packages/{packageId}`

⚠️ **Important Warnings:**
- ⚠️ **Price changes will affect users using this package**
- ⚠️ **Notify users in advance before editing**
- ⚠️ **Rate limit changes take effect immediately**
- ⚠️ **Free package should not have price changed to other values**

#### 4.2.3 Deleting Packages

**Procedure:**

1. Find the package you want to delete
2. Click the **"Delete"** button
3. System will check if there are users using this package
4. If there are users, system will not allow deletion:

```
+-------------------------------------------------------------------+
|              ⚠️ Cannot Delete Package                              |
|                                                                   |
|  Package "Premium" has active users:                              |
|  • sponsor@example.com (3 businesses)                            |
|  • sponsor2@example.com (2 businesses)                           |
|                                                                   |
|  Please move all users to another package before deleting         |
|                                                                   |
|                    [OK]                                           |
+-------------------------------------------------------------------+
```

**API Endpoint:**
- `DELETE /api/packages/{packageId}`

⚠️ **Warnings:**
- ⚠️ **Cannot delete packages with active users**
- ⚠️ **Must move all users to another package first**
- ⚠️ **Package deletion cannot be recovered**
- ⚠️ **Free package should not be deleted**

---

## 5. Business Management

Admin has the same business management permissions as Staff (see details in "Staff Manual").

**Available Features:**
- ✅ Add businesses
- ✅ Edit businesses
- ✅ Delete businesses
- ✅ Manage business images
- ✅ Create embeddings

⚠️ **Note:** For detailed information, refer to the "Staff Manual" as it contains all the details.

---

## 6. Payment Slips

Admin has the same payment slip management permissions as Staff (see details in "Staff Manual").

**Available Features:**
- ✅ View all slips
- ✅ Approve slips
- ✅ Reject slips
- ✅ Filter by status

---

## 7. Business Image Management

Admin has the same image management permissions as Staff (see details in "Staff Manual").

---

## 8. System Configuration

### 8.1 Accessing System Configuration

Admin can access system configuration through Admin Panel.

**Important Settings:**

1. **Rate Limiting**
   - Number of questions allowed per minute per user
   - Rate limit settings by Role

2. **Package Configuration**
   - Settings for various packages
   - Price and privileges for each package

3. **System Settings**
   - Embedding Service settings
   - LLM API settings
   - Database settings

⚠️ **Note:** Most system configuration is done through Backend Configuration and Docker Compose, not through the Web Interface.

### 8.2 Checking Logs

**Via Terminal:**

```bash
# View API logs
docker logs vit-llm-chat-api-1 -f

# View Database logs
docker logs vit-llm-chat-database-1 -f

# View all logs
docker-compose -f docker-compose.prod.yml logs -f
```

---

## 9. Creating Embeddings

Admin has the same embedding creation permissions as Staff (see details in "Staff Manual").

**Additional for Admin:**

Can create embeddings for all businesses at once via script:

```bash
# Inside API Container
docker exec vit-llm-chat-api-1 node /path/to/batch-embedding.js
```

---

## 10. Data Backup and Recovery

### 10.1 Data Backup

**Backup Database:**

```bash
# Backup entire database
docker exec vit-llm-chat-database-1 pg_dump -U postgres llmchat_db > backup_$(date +%Y%m%d).sql

# Backup only important tables
docker exec vit-llm-chat-database-1 pg_dump -U postgres -t businesses -t users llmchat_db > partial_backup.sql
```

**Backup Files:**

```bash
# Backup image files
tar -czf images_backup_$(date +%Y%m%d).tar.gz /path/to/uploads/

# Backup config files
tar -czf config_backup_$(date +%Y%m%d).tar.gz /home/vit/docker-compose*.yml
```

### 10.2 Data Recovery

**Recover Database:**

```bash
# Recover from backup file
docker exec -i vit-llm-chat-database-1 psql -U postgres llmchat_db < backup_20250101.sql
```

⚠️ **Warnings:**
- ⚠️ **Recovery will overwrite existing data**
- ⚠️ **Stop services before recovery**
- ⚠️ **Test recovery in test system first**

---

## 11. Transaction Management

Admin has the same transaction management permissions as Staff (see details in "Staff Manual").

**API Paths:**
- `GET /api/transactions` - Retrieve all transactions
- `PUT /api/transactions/{id}/status` - Update transaction status

---

## 12. Related Commands and Paths

### 12.1 API Endpoints for Admin

**Users - Admin Only:**
```
GET    /api/users                         - Retrieve all users
GET    /api/users/:userId                 - Retrieve user information
POST   /api/users                         - Create new user
PUT    /api/users/:userId                 - Edit user
PUT    /api/users/:userId/status          - Suspend/activate
DELETE /api/users/:userId                 - Delete user
```

**Packages - Admin Only:**
```
POST   /api/packages                      - Create new package
PUT    /api/packages/:id                  - Edit package
DELETE /api/packages/:id                  - Delete package
```

**Others:**
- Same as Staff (see "Staff Manual")

### 12.2 Important File Paths

**Configuration:**
- `/home/vit/docker-compose.prod.yml` - Docker Compose for Production
- `/home/vit/docker-compose.dev.yml` - Docker Compose for Development

**Backend:**
- `/home/vit/llm-chat-backend/src/routes/user.routes.js` - Routes for users
- `/home/vit/llm-chat-backend/src/routes/package.routes.js` - Routes for packages
- `/home/vit/llm-chat-backend/src/controllers/user.controller.js` - Controller for users

**Database:**
- Database Name: `llmchat_db`
- Tables: `users`, `packages`, `businesses`, `business_packages`, `payment_slips`, `transactions`

### 12.3 Important Docker Commands

```bash
# View logs
docker logs vit-llm-chat-api-1 -f

# Restart service
docker restart vit-llm-chat-api-1

# Access container
docker exec -it vit-llm-chat-api-1 bash
docker exec -it vit-llm-chat-database-1 psql -U postgres -d llmchat_db

# Backup database
docker exec vit-llm-chat-database-1 pg_dump -U postgres llmchat_db > backup.sql

# Restore database
docker exec -i vit-llm-chat-database-1 psql -U postgres llmchat_db < backup.sql
```

### 12.4 Important SQL Commands

```sql
-- View all users
SELECT id, email, role, is_suspended, created_at FROM users ORDER BY created_at DESC;

-- View all packages
SELECT id, name, price, requests_per_minute FROM packages;

-- View users using packages
SELECT u.email, p.name as package_name, COUNT(bp.business_id) as business_count
FROM users u
LEFT JOIN business_packages bp ON u.id = bp.user_id
LEFT JOIN packages p ON bp.package_id = p.id
GROUP BY u.email, p.name;

-- View all Admins
SELECT id, email, first_name, last_name, created_at FROM users WHERE role = 'admin';

-- Suspend user
UPDATE users SET is_suspended = true WHERE email = 'user@example.com';

-- Activate user
UPDATE users SET is_suspended = false WHERE email = 'user@example.com';
```

---

## 13. Warnings and Precautions for Administrators

### 13.1 Warnings About User Management

⚠️⚠️⚠️ **Prohibited Actions:**
- ⚠️⚠️⚠️ **Do not delete primary Admin** (admin@example.com)
- ⚠️⚠️ **Do not suspend primary Admin**
- ⚠️⚠️ **Do not change primary Admin role to other**
- ⚠️ **Do not create too many Admins** (should have 1-2 primary Admins)

⚠️ **Caution Required:**
- Verify before deleting or suspending users
- Should not change user roles unnecessarily
- Record reasons every time role changes are made

### 13.2 Warnings About Package Management

⚠️⚠️⚠️ **Prohibited Actions:**
- ⚠️⚠️⚠️ **Do not delete Free package**
- ⚠️⚠️ **Do not edit packages with many users without notification**
- ⚠️ **Do not change Free package price to other values**

⚠️ **Caution Required:**
- Notify users in advance before editing packages
- Check number of users before deleting packages
- Should not change package prices too frequently

### 13.3 Warnings About Security

⚠️⚠️⚠️ **Security:**

1. **Login Information:**
   - ⚠️ **Never share Admin login information with anyone**
   - ⚠️ **Use strong passwords** (minimum 12 characters, include uppercase-lowercase, numbers, symbols)
   - ⚠️ **Change passwords periodically** (every 3-6 months)

2. **System Access:**
   - ⚠️ **Always log out after completion**
   - ⚠️ **Do not login on insecure computers**
   - ⚠️ **Regularly check login logs**

3. **Data Backup:**
   - ⚠️ **Backup data regularly** (recommended daily)
   - ⚠️ **Store backups in multiple locations**
   - ⚠️ **Test data recovery periodically**

### 13.4 Warnings About System Changes

⚠️ **Should Do:**
- ✅ Test changes in test system first
- ✅ Notify users before important changes
- ✅ Record all changes
- ✅ Backup data before major changes

⚠️ **Should Not Do:**
- ❌ Change system without planning
- ❌ Make multiple changes simultaneously
- ❌ Change production without testing

### 13.5 Administrator Checklist

**Before starting work each day:**
- [ ] Login to system
- [ ] Check logs for errors
- [ ] Check service status (Docker)
- [ ] Check payment slips awaiting review

**Weekly:**
- [ ] Backup database
- [ ] Check number of users and businesses
- [ ] Check usage logs
- [ ] Review changes made

**Monthly:**
- [ ] Test data recovery
- [ ] Security check
- [ ] Change password (if necessary)
- [ ] Review user permissions

---

## 14. Getting Help

If you encounter problems or need assistance:
1. Check logs: `docker logs vit-llm-chat-api-1 -f`
2. Check database: `docker exec -it vit-llm-chat-database-1 psql -U postgres -d llmchat_db`
3. Review recent changes
4. Contact development team (if available)

---

## Summary

Administrators have full access to manage the entire system, including:

✅ **User Management** - Create/edit/delete/suspend all users  
✅ **Package Management** - Create/edit/delete packages  
✅ **Business Management** - All features same as Staff  
✅ **Payment Slip Management** - All features same as Staff  
✅ **System Configuration** - Monitor and adjust system  

**URL:**
- Admin Panel: `https://frontend.sri-ketguide.com/#/admin`

**Admin Account:**
- Email: `admin@example.com`
- Password: `1234`
- Role: `admin` (Full privileges)

⚠️ **Important Warnings:**
- Admin has full access, use with caution
- Do not delete or modify primary Admin
- Backup data regularly
- Use strong passwords

---

**Last Updated:** October 2025  
**Version:** 1.0


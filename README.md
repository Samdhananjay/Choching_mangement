# EduManage Multi-Tenant Coaching Institute Management System (SaaS)

An enterprise-grade, responsive, secure, multi-tenant SaaS Coaching Institute Management System built with **Core PHP 8.2+ (Modular MVC Architecture)**, **MySQL 8+ / MariaDB**, **Bootstrap 5**, **FontAwesome 6**, and **Chart.js**.

---

## 1. System Architecture & Multi-Tenancy

### Shared Database with Discriminator-Based Tenant Isolation
The system employs strict tenant isolation at the architectural level:
- Every coaching institute is assigned a unique `institute_id`.
- All institute-scoped database tables include an `institute_id` column with foreign key constraints and dedicated database indices.
- **Tenant Isolation Middleware & Base Model**: `BaseModel` never trusts `institute_id` submitted via frontend forms or URL parameters. It derives `institute_id` strictly from the authenticated session (`$_SESSION['user']['institute_id']`).
- **Cross-Tenant Attack Prevention**: All CRUD queries (`find`, `where`, `all`, `create`, `update`, `delete`, `count`) automatically enforce `WHERE institute_id = :tenant_id`. Foreign primary keys manipulated via URLs (e.g. `?id=99`) safely return `NULL` or `404` if belonging to another tenant.
- **Super Administrator Access**: Super Admin (`institute_id = NULL`) has platform-wide authority and bypasses tenant isolation to manage institutes, subscription plans, platform revenue, and audit logs.

### Account Lock & Unlock System
- The Super Admin can transition any institute between `active`, `locked`, `suspended`, and `deactivated` states with a customized lock reason.
- When an institute is locked or suspended:
  - Institute Admins, Teachers, and Students are immediately blocked from logging in.
  - Active sessions are intercepted by `TenantMiddleware` and redirected to a dedicated `/account-locked` screen displaying the restriction notice and reason.
  - Super Admin maintains administrative access to investigate, unlock, or renew.

### Strict Teacher & Student Isolation
- **Teachers**: Can **only** see batches and students explicitly assigned to them via `teacher_batch_assignments`.
- **Students**: Can **only** view their personal admissions, attendance, fees, receipts, report cards, and assignments.

---

## 2. User Roles & Seed Login Credentials

All seed accounts have the default password: **`Password@123`**.

| User Role | Username | Email | Institute | Role Description |
| :--- | :--- | :--- | :--- | :--- |
| **Super Admin** | `superadmin` | `admin@coachingcloud.com` | *Platform Global* | Controls SaaS platform, plans, institutes, revenue, logs |
| **Institute Admin** | `apex_admin` | `admin@apexacademy.com` | Apex Academy of Science | Full institute management (staff, students, fees, exams) |
| **Institute Admin** | `zenith_admin` | `admin@zenithcommerce.com` | Zenith Commerce Institute | Separate tenant (verifies complete data isolation) |
| **Teacher** | `apex_teacher` | `robert.physics@apexacademy.com` | Apex Academy of Science | Assigned batches, roll call, assignments, study materials |
| **Student** | `apex_student` | `aarav.patel@gmail.com` | Apex Academy of Science | Profile, attendance, receipts, homework, timetable |
| **Locked Admin** | `global_admin` | `admin@globaliit.com` | Global IIT Prep (*Locked*) | Test account for account locked screen and restriction |

---

## 3. Technology Stack

- **Backend**: Core PHP 8.2+ (Modular MVC, Strict OOP, Singleton Database PDO, Custom Router, Custom Autoloader).
- **Frontend**: HTML5, CSS3, JavaScript (ES6), Bootstrap 5.3.3, FontAwesome 6.5.1, Chart.js 4.4.1.
- **Database**: MySQL 8+ / MariaDB 10.4+ (utf8mb4, InnoDB, Foreign Keys, Cascades).
- **Security**: `password_hash()` BCRYPT, Prepared Statements via PDO, CSRF Token validation on all POST requests, HTML output sanitization (`e()`), Login Rate Limiting (max 5 failed attempts per 15 minutes).

---

## 4. Project Directory Structure

```
multilevel_coching_mangement/
├── app/
│   ├── Controllers/
│   │   ├── AuthController.php
│   │   ├── SuperAdminController.php
│   │   ├── InstituteAdminController.php
│   │   ├── TeacherController.php
│   │   └── StudentController.php
│   ├── Models/
│   │   ├── BaseModel.php
│   │   ├── User.php
│   │   ├── Institute.php
│   │   ├── SubscriptionPlan.php
│   │   ├── Subscription.php
│   │   ├── Teacher.php
│   │   ├── Student.php
│   │   ├── Course.php
│   │   ├── Subject.php
│   │   ├── Batch.php
│   │   ├── Attendance.php
│   │   ├── Fee.php
│   │   ├── Examination.php
│   │   ├── Assignment.php
│   │   ├── StudyMaterial.php
│   │   ├── Notice.php
│   │   ├── Timetable.php
│   │   └── ActivityLog.php
│   ├── Middleware/
│   │   ├── AuthMiddleware.php
│   │   ├── RoleMiddleware.php
│   │   ├── TenantMiddleware.php
│   │   └── CsrfMiddleware.php
│   ├── Services/
│   │   ├── AuthService.php
│   │   ├── SubscriptionService.php
│   │   └── FileUploadService.php
│   ├── Helpers/
│   │   └── functions.php
│   └── Autoloader.php
├── config/
│   ├── config.php
│   └── database.php
├── database/
│   ├── Database.php
│   └── schema.sql
├── public/
│   ├── assets/
│   │   ├── css/style.css
│   │   └── js/main.js
│   ├── uploads/
│   │   ├── logos/
│   │   ├── materials/
│   │   ├── assignments/
│   │   └── submissions/
│   ├── .htaccess
│   └── index.php
├── resources/
│   └── views/
│       ├── layouts/
│       ├── auth/
│       ├── superadmin/
│       ├── admin/
│       ├── teacher/
│       ├── student/
│       └── errors/
├── routes/
│   ├── Router.php
│   └── web.php
├── tests/
│   └── test_system.php
├── .htaccess
└── index.php
```

---

## 5. Setup & Installation Instructions

### Prerequisites
- XAMPP with Apache and MySQL (PHP 8.2+)
- Running Apache on port 80 / 443
- MySQL running on port 3306

### Database Installation
1. Start Apache and MySQL in XAMPP.
2. The database `coaching_saas_db` has been created and imported with all 33 tables and seed records:
   ```powershell
   & "D:\xammp\mysql\bin\mysql.exe" -u root -e "CREATE DATABASE IF NOT EXISTS coaching_saas_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
   & "D:\xammp\mysql\bin\mysql.exe" -u root coaching_saas_db -e "source d:/xammp/htdocs/multilevel_coching_mangement/database/schema.sql"
   ```

### Accessing the System
Navigate in your browser to:
- **`http://localhost/multilevel_coching_mangement/`** or
- **`http://localhost/multilevel_coching_mangement/public/`**

---

## 6. Running the Automated Test Suite

Run the automated test suite using PHP CLI:
```powershell
& "D:\xammp\php\php.exe" tests\test_system.php
```

All 24 architectural, multi-tenant isolation, role hierarchy, quota check, and account lock tests will execute and confirm zero errors.

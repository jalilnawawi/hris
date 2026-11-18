# Table Structure of the Database HRIS

## Users Table (users)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| username    | VARCHAR(50) | UNIQUE, NOT NULL |
| password    | VARCHAR(255) | NULL |
| is_active   | BOOLEAN   | DEFAULT TRUE |
| role_id      | INT       | FOREIGN KEY REFERENCES roles(id) |
| last_login_at | TIMESTAMPTZ | NULL |
| employee_id  | UUID      | FOREIGN KEY REFERENCES employees(id) |
| telegram_chat_id | BIGINT    | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
indexes:
- idx_users_employee_id: (employee_id)
---
## User Tokens Table (user_tokens)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| user_id     | UUID      | NOT NULL, FOREIGN KEY REFERENCES users(id) |
| type        | VARCHAR(20) | NOT NULL CHECK (type IN ('refresh', 'activation', 'reset')) |
| token_hash  | VARCHAR(255) | UNIQUE, NOT NULL |
| expires_at  | TIMESTAMPTZ | NOT NULL |
| used_at     | TIMESTAMPTZ | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
indexes:
- idx_user_tokens_user_id: (user_id)
---
## Roles Table (roles)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(50) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Permissions Table (permissions)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| code        | VARCHAR(100) | UNIQUE, NOT NULL (mis. 'employee.read') |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
---
## Role Permissions Table (role_permissions)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| role_id     | INT       | NOT NULL, FOREIGN KEY REFERENCES roles(id) ON DELETE CASCADE |
| permission_id | INT     | NOT NULL, FOREIGN KEY REFERENCES permissions(id) ON DELETE CASCADE |
primary key: (role_id, permission_id)
---
## Employees Table (employees)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| employee_number | VARCHAR(20) | UNIQUE, NOT NULL |
| full_name   | VARCHAR(100) | NOT NULL |
| gender      | VARCHAR(10) | NULL |
| email       | VARCHAR(100) | UNIQUE, NOT NULL |
| phone_number | VARCHAR(15) | NULL |
| address     | VARCHAR(255) | NULL |
| position    | VARCHAR(50) | NULL |
| department_id | INT | FOREIGN KEY REFERENCES departments(id) |
| shift_id    | INT       | NULL, FOREIGN KEY REFERENCES master_shift(id) |
| reporting_manager_id | UUID | FOREIGN KEY REFERENCES employees(id) |
| date_of_birth | DATE      | NULL |
| date_of_joining | DATE      | NULL |
| marital_status | VARCHAR(20) | NULL |
| photo_url  | VARCHAR(255) | NULL |
| parent_name | VARCHAR(100) | NULL |
| parent_phone_number | VARCHAR(15) | NULL |
| major | VARCHAR(100) | NULL |
| employment_status | VARCHAR(20) | NOT NULL DEFAULT 'active' CHECK (employment_status IN ('active', 'resigned', 'terminated')) |
| resign_date | DATE      | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
indexes:
- idx_employees_department_id: (department_id)
- idx_employees_reporting_manager_id: (reporting_manager_id)
---
## Departments Table (departments)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(100) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Master Shift Table (master_shift)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(50) | UNIQUE, NOT NULL |
| start_time  | TIME      | NOT NULL |
| end_time    | TIME      | NOT NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
---
## Attendance Table (attendance)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| shift_id    | INT       | FOREIGN KEY REFERENCES master_shift(id) |
| date        | DATE      | NOT NULL |
| check_in    | TIMESTAMPTZ | NOT NULL |
| check_out   | TIMESTAMPTZ | NULL |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'present' CHECK (status IN ('present', 'late', 'absent', 'leave')) |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
indexes:
- idx_attendance_date: (date)
- idx_attendance_shift_id: (shift_id)
unique:
- uq_attendance_employee_date: (employee_id, date)
---
## Master Leave Table (master_leave)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(50) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Leave Balance Table (leave_balance)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL      | PRIMARY KEY |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| leave_id    | INT       | FOREIGN KEY REFERENCES master_leave(id) |
| year        | INT       | NOT NULL |
| balance     | INT       | NOT NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
indexes:
- idx_leave_balance_leave_id: (leave_id)
unique:
- uq_leave_balance_employee_leave_year: (employee_id, leave_id, year)
---
## Leave Requests Table (leave_requests)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| leave_id | INT      | FOREIGN KEY REFERENCES master_leave(id) |
| start_date  | DATE      | NOT NULL |
| end_date    | DATE      | NOT NULL |
| total_days  | INT       | NOT NULL |
| reason      | TEXT      | NOT NULL |
| approver_id | UUID      | NULL, FOREIGN KEY REFERENCES employees(id) |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'rejected', 'cancelled')) |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
indexes:
- idx_leave_requests_employee_id: (employee_id)
- idx_leave_requests_leave_id: (leave_id)
- idx_leave_requests_status: (status)
---
## Holidays Table (holidays)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| date        | DATE      | UNIQUE, NOT NULL |
| name        | VARCHAR(100) | NOT NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Master KPI Table (master_kpi)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| department_id | INT       | FOREIGN KEY REFERENCES departments(id) |
| name        | VARCHAR(100) | NOT NULL |
| description | VARCHAR(255) | NULL |
| target      | INT       | NOT NULL |
| is_achieve_below_target | BOOLEAN   | NOT NULL DEFAULT FALSE |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
unique:
- uq_master_kpi_department_name: (department_id, name)
---
## KPI Evaluations Table (kpi_evaluations)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| kpi_id      | INT       | FOREIGN KEY REFERENCES master_kpi(id) |
| score       | INT       | NOT NULL |
| period      | DATE      | NOT NULL (tanggal 1 bulan periode, mis. 2026-11-01) |
| evaluation_date | DATE      | NOT NULL |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'rejected')) |
| approver_id  | UUID      | FOREIGN KEY REFERENCES employees(id) |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
indexes:
- idx_kpi_evaluations_kpi_id: (kpi_id)
unique:
- uq_kpi_evaluations_employee_kpi_period: (employee_id, kpi_id, period)
---
## Payroll Components Table (payroll_components)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(100) | UNIQUE, NOT NULL |
| type        | VARCHAR(20) | NOT NULL CHECK (type IN ('allowance', 'deduction')) |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Payroll Table (payroll)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| salary_date | DATE      | NOT NULL |
| basic_salary | NUMERIC(15, 2) | NOT NULL |
| allowances  | NUMERIC(15, 2) | NOT NULL |
| deductions  | NUMERIC(15, 2) | NOT NULL |
| take_home_pay | NUMERIC(15, 2) | NOT NULL |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'draft' CHECK (status IN ('draft', 'approved', 'paid')) |
| approved_by | UUID      | NULL, FOREIGN KEY REFERENCES users(id) |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
indexes:
- idx_payroll_salary_date: (salary_date)
unique:
- uq_payroll_employee_salary_date: (employee_id, salary_date)
---
## Payroll Items Table (payroll_items)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| payroll_id  | UUID      | NOT NULL, FOREIGN KEY REFERENCES payroll(id) ON DELETE CASCADE |
| component_id | INT      | NOT NULL, FOREIGN KEY REFERENCES payroll_components(id) |
| amount      | NUMERIC(15, 2) | NOT NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
unique:
- uq_payroll_items_payroll_component: (payroll_id, component_id)
---
## Notifications Table (notifications)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| user_id     | UUID      | FOREIGN KEY REFERENCES users(id) |
| notification_type_id | INT       | FOREIGN KEY REFERENCES notification_types(id) |
| message     | TEXT      | NOT NULL |
| is_read     | BOOLEAN   | DEFAULT FALSE |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
indexes:
- idx_notifications_user_id_is_read: (user_id, is_read)
---
## Notification Types Table (notification_types)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(50) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
---
## Audit Logs Table (audit_logs)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| user_id     | UUID      | FOREIGN KEY REFERENCES users(id) |
| action      | VARCHAR(255) | NOT NULL |
| table_name  | VARCHAR(100) | NOT NULL |
| record_id   | VARCHAR(50) | NULL |
| old_data    | JSONB     | NULL |
| new_data    | JSONB     | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
---
## Worker Jobs Table (worker_jobs)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| job_name    | VARCHAR(100) | NOT NULL |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'running', 'done', 'failed')) |
| payload     | JSONB     | NULL |
| result      | JSONB     | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
indexes:
- idx_worker_jobs_status_created_at: (status, created_at)
---
## Worker Job Logs Table (worker_job_logs)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY DEFAULT gen_random_uuid() |
| job_id      | UUID      | FOREIGN KEY REFERENCES worker_jobs(id) |
| log_message | TEXT      | NOT NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
---
## System Settings Table (system_settings)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| key         | VARCHAR(100) | UNIQUE, NOT NULL |
| value       | TEXT      | NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMPTZ | DEFAULT CURRENT_TIMESTAMP |

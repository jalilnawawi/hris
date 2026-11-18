# Table Structure of the Database HRIS

## Users Table (users)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID    | PRIMARY KEY |
| username    | VARCHAR(50) | UNIQUE, NOT NULL |
| password    | VARCHAR(255) | NOT NULL |
| is_active   | BOOLEAN   | DEFAULT TRUE |
| role_id      | INT       | FOREIGN KEY REFERENCES roles(id) |
| last_login_at | TIMESTAMP | NULL |
| employee_id  | INT       | FOREIGN KEY REFERENCES employees(id) |
| telegram_chat_id | BIGINT    | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
indexes:
- idx_users_username: (username)
- idx_users_employee_id: (employee_id)
---
## Roles Table (roles)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(50) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Employees Table (employees)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id | UUID | PRIMARY KEY |
| employee_number | VARCHAR(20) | UNIQUE, NOT NULL |
| full_name   | VARCHAR(100) | NOT NULL |
| gender      | VARCHAR(10) | NULL |
| email       | VARCHAR(100) | UNIQUE, NOT NULL |
| phone_number | VARCHAR(15) | NULL |
| address     | VARCHAR(255) | NULL |
| position    | VARCHAR(50) | NULL |
| department_id | INT | FOREIGN KEY REFERENCES departments(id) |
| reporting_manager_id | UUID | FOREIGN KEY REFERENCES employees(id) |
| date_of_birth | DATE      | NULL |
| date_of_joining | DATE      | NULL |
| marital_status | VARCHAR(20) | NULL |
| photo_url  | VARCHAR(255) | NULL |
| parent_name | VARCHAR(100) | NULL |
| parent_phone_number | VARCHAR(15) | NULL |
| major | VARCHAR(100) | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
indexes:
- idx_employees_employee_number: (employee_number)
- idx_employees_email: (email)
- idx_employees_department_id: (department_id)
- idx_employees_reporting_manager_id: (reporting_manager_id)
---
## Departments Table (departments)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(100) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
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
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
---
## Attendance Table (attendance)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| shift_id    | INT       | FOREIGN KEY REFERENCES master_shift(id) |
| Date        | DATE      | NOT NULL |
| check_in    | TIMESTAMP | NOT NULL |
| check_out   | TIMESTAMP | NULL |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'present' |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
indexes:
- idx_attendance_employee_id: (employee_id)
- idx_attendance_date: (Date)
- idx_attendance_shift_id: (shift_id)
- idx_attendance_status: (status)
---
## Master Leave Table (master_leave)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(50) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Leave Balance Table (leave_balance)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL      | PRIMARY KEY |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| leave_id    | INT       | FOREIGN KEY REFERENCES master_leave(id) |
| balance     | INT       | NOT NULL DEFAULT 1 |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
indexes:
- idx_leave_balance_employee_id: (employee_id)
- idx_leave_balance_leave_id: (leave_id)
---
## Leave Requests Table (leave_requests)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| leave_id | INT      | FOREIGN KEY REFERENCES master_leave(id) |
| start_date  | DATE      | NOT NULL |
| end_date    | DATE      | NOT NULL |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'pending' |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
indexes:
- idx_leave_requests_employee_id: (employee_id)
- idx_leave_requests_leave_id: (leave_id)
- idx_leave_requests_status: (status)
---
## Master KPI Table (master_kpi)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| department_id | INT       | FOREIGN KEY REFERENCES departments(id) |
| name        | VARCHAR(100) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| target      | INT       | NOT NULL |
| is_achieve_below_target | BOOLEAN   | NOT NULL DEFAULT FALSE |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
indexes:
- idx_master_kpi_department_id: (department_id)
---
## KPI Evaluations Table (kpi_evaluations)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| kpi_id      | INT       | FOREIGN KEY REFERENCES master_kpi(id) |
| score       | INT       | NOT NULL |
| evaluation_date | DATE      | NOT NULL |
| approver_id  | UUID      | FOREIGN KEY REFERENCES employees(id) |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
indexes:
- idx_kpi_evaluations_employee_id: (employee_id)
- idx_kpi_evaluations_kpi_id: (kpi_id)
---
## Payroll Components Table (payroll_components)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(100) | UNIQUE, NOT NULL |
| type        | VARCHAR(20) | NOT NULL CHECK (type IN ('allowance', 'deduction')) |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Payroll Table (payroll)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| employee_id | UUID      | FOREIGN KEY REFERENCES employees(id) |
| salary_date | DATE      | NOT NULL |
| basic_salary | DECIMAL(10, 2) | NOT NULL |
| allowances  | DECIMAL(10, 2) | NOT NULL |
| deductions  | DECIMAL(10, 2) | NOT NULL |
| take_home_pay | DECIMAL(10, 2) | NOT NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
indexes:
- idx_payroll_employee_id: (employee_id)
- idx_payroll_salary_date: (salary_date)
---
## Notifications Table (notifications)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| user_id     | UUID      | FOREIGN KEY REFERENCES users(id) |
| notification_type_id | INT       | FOREIGN KEY REFERENCES notification_types(id) |
| message     | TEXT      | NOT NULL |
| is_read     | BOOLEAN   | DEFAULT FALSE |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
---
## Notification Types Table (notification_types)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| name        | VARCHAR(50) | UNIQUE, NOT NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
---
## Audit Logs Table (audit_logs)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| user_id     | UUID      | FOREIGN KEY REFERENCES users(id) |
| action      | VARCHAR(255) | NOT NULL |
| table_name  | VARCHAR(100) | NOT NULL |
| record_id   | UUID      | NULL |
| old_data    | JSONB     | NULL |
| new_data    | JSONB     | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
| created_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
| updated_by  | UUID      | FOREIGN KEY REFERENCES users(id) |
---
## Worker Jobs Table (worker_jobs)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| job_name    | VARCHAR(100) | NOT NULL |
| status      | VARCHAR(20) | NOT NULL DEFAULT 'pending' |
| payload     | JSONB     | NULL |
| result      | JSONB     | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
---
## Worker Job Logs Table (worker_job_logs)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| job_id      | UUID      | FOREIGN KEY REFERENCES worker_jobs(id) |
| log_message | TEXT      | NOT NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
---
## System Settings Table (system_settings)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | SERIAL    | PRIMARY KEY |
| key         | VARCHAR(100) | UNIQUE, NOT NULL |
| value       | TEXT      | NULL |
| description | VARCHAR(255) | NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |
---
## System Logs Table (system_logs)
| Column Name | Data Type | Constraints |
|-------------|-----------|-------------|
| id          | UUID      | PRIMARY KEY |
| log_level   | VARCHAR(20) | NOT NULL |
| message     | TEXT      | NOT NULL |
| created_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at  | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |


# 📝 Mini Leave Management System (FastAPI + SQLite)

A compact MVP Leave Management System for a 50-employee startup. Covers employee onboarding, leave requests, approvals, and balance tracking. Includes a simple **role-based access control** (HR vs Employee) via query parameter.

## 🚀 Quick Start

```bash
# 1) Create a virtual environment (optional)
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate

# 2) Install dependencies
pip install fastapi uvicorn sqlalchemy

# 3) Run
uvicorn main:app --reload

# API docs at
# http://127.0.0.1:8000/docs
```

SQLite DB file `leave.db` is created automatically.

---

## ✅ Features
- Add Employee (HR only)
- Apply for Leave
- Approve / Reject Leave (HR only)
- Fetch Leave Balance
- List Employee Leaves
- Overlap checks, date validation, and balance enforcement

### Default Balances
- Every new employee starts with **20 days** `leave_balance` (configurable in `models.py`).

### Roles (Simple RBAC for MVP)
- `hr` → can add employees, approve/reject leaves.
- `employee` → can apply for leave, view balance.
- **Pass role as query parameter:** `?role=hr` or `?role=employee` on restricted endpoints.

> ⚠️ This is for assignment purposes only. For production, use proper authentication (JWT) and a users table.

---

## 🧪 API Endpoints & Examples

### 1) Add Employee (HR Only)
`POST /employees?role=hr`
```json
{
  "name": "Alice",
  "email": "alice@example.com",
  "department": "Engineering",
  "joining_date": "2025-01-01",
  "role": "employee"
}
```
**Response**
```json
{ "message": "Employee added successfully", "id": 1 }
```

### 2) Apply for Leave
`POST /leaves`
```json
{
  "employee_id": 1,
  "start_date": "2025-02-10",
  "end_date": "2025-02-12",
  "reason": "Family trip"
}
```
**Response**
```json
{ "message": "Leave applied successfully", "leave_id": 1, "status": "pending" }
```

### 3) Approve / Reject Leave (HR Only)
`PUT /leaves/1?role=hr`
```json
{ "status": "approved" }
```
**Response**
```json
{ "message": "Leave approved" }
```

### 4) Get Leave Balance
`GET /employees/1/balance`
**Response**
```json
{ "employee_id": 1, "employee": "Alice", "leave_balance": 17 }
```

### 5) List Leaves for Employee
`GET /employees/1/leaves`

---

## 🧠 Edge Cases Handled
- ❌ Applying before joining date
- ❌ Invalid range (end_date < start_date)
- ❌ Overlapping requests (blocks overlap with any non-rejected leave)
- ❌ Insufficient balance at request time and approval time
- ❌ Employee not found
- ❌ Duplicate emails
- ❌ Re-approving or re-rejecting already finalised leaves

> Tip: We compute days as **inclusive**: `(end - start) + 1`.

---

## 🏗️ High Level System Design

### Architecture Diagram (Mermaid)
```mermaid
flowchart TD
    subgraph Client
        U1[HR]
        U2[Employee]
        UI[Swagger UI / Simple Frontend]
    end

    UI --> API[FastAPI Service]
    API --> RBAC[Simple Role Guard (?role=...)]
    API --> DB[(SQLite DB: employees, leaves)]

    U1 -->|Add/Approve/Reject| UI
    U2 -->|Apply/View Balance| UI
```

### How APIs & DB interact
- FastAPI endpoints use a `SessionLocal` (SQLAlchemy) per request.
- On **apply**: validate dates, balance, and overlaps, then insert a `leaves` row (status `pending`).
- On **approve**: transition `pending → approved`, and deduct days from `employees.leave_balance` in the same transaction.

### Scaling: 50 → 500 employees
- Swap **SQLite → Postgres/MySQL** (same SQLAlchemy models).
- Run multiple API workers behind a load balancer (Uvicorn/Gunicorn).
- Add DB indexes on `employees.email`, `leaves.employee_id, start_date, end_date`.
- For heavy reads: caching on balance endpoints or denormalized monthly leave summaries.
- Introduce proper auth (JWT) and a `users` table linked to employees.

---

## 🧩 Assumptions
- 20 annual leaves per employee (carry-forward not implemented).
- Only full-day leaves supported; half-days and holidays are out of scope.
- HR decisions are final; only **pending** leaves are mutable.
- Overlap prevention considers **pending + approved** leaves.

---

## 🔒 Improvements (Future)
- JWT auth + refresh tokens, password storage, and API keys
- Roles: Admin/HR/Manager/Employee, department-level approvals
- Leave types (sick/casual/LOP), accrual, carry-forward
- Public holidays calendar & half-days
- Notifications (email/Slack) and audit logs
- Frontend (React) with forms and dashboards
- CI/CD and containerization

---



## 📦 Repo Layout
```
leave-management/
│── main.py          # FastAPI entry point (routes + logic)
│── models.py        # SQLAlchemy + Pydantic models
│── database.py      # DB init
│── README.md        # This file
│── requirements.txt
```

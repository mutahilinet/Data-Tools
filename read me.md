# 🎟 library management Database – Admin & User Roles in Supabase

<div align="center">
  <img width="314" height="285" alt="Supabase Logo" src="https://github.com/user-attachments/assets/20661293-a214-4004-9042-657102fb0710" />
  <br/>
  <h3><b>Data Fundamentals Project</b></h3>
</div>

---

## 📗 Table of Contents

* [📖 About the Project](#about-project)  
* [🛠 Built With](#built-with)  
* [🚀 Live Demo](#live-demo)  
* [💻 Getting Started](#getting-started)  
* [💾 Sample SQL Queries & Policies](#sample-sql-queries)   
* [🛡 Security Notes](#security-notes)  
* [👥 Authors](#authors)  
* [🔭 Future Features](#future-features)  
* [🤝 Contributing](#contributing)  
* [⭐️ Show your support](#support)  
* [🙏 Acknowledgements](#acknowledgements)  
* [❓ FAQ](#faq)  
* [📝 License](#license)  

---

# 📖 About the Project <a name="about-project"></a>

This project demonstrates an **library management system** implemented using Supabase (PostgreSQL).
It integrates Row Level Security (RLS), Admin & User roles, and custom SQL policies to control data access and enforce least privilege principles.

The System includes tables: authors,publishers,students,books,borrow_records 
- ✅ UUID-based authentication via Supabase Auth
- ✅ Role-based access policies (Admin vs Regular User)
- ✅ Row Level Security (RLS) on all tables
- ✅ A secure function for admin-only operations

---

## 🛠 Built With <a name="built-with"></a>

- **Supabase** – PostgreSQL + Auth + Policy Management  
- **PostgreSQL** – Structured database engine and tables  
- **RLS Policies** - Fine-grained access control
- **SQL Functions** – Role-based admin actions

---

## 🚀 Live Demo <a name="live-demo"></a>

- [Supabase Dashboard](https://supabase.com/dashboard/project/pwsbzyjjqwxtqzzpaghy)  

---

## 💻 Getting Started <a name="getting-started"></a>

### Prerequisites
- Supabase account
- Basic SQL and PostgreSQL knowledge
-Git installed
- GitHub account for project submission  

### Setup

```bash
git clone https://github.com/Evans-dotcom/Data-Tools/tree/Data_Fundamentals-Branch
cd Data-Tools
```

```bash
Usage

Open Supabase SQL editor

Run schema.sql to create tables & sample data

Apply UUID + RLS setup:
Enable Row Level Security

ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE tickets ENABLE ROW LEVEL SECURITY;
ALTER TABLE payments ENABLE ROW LEVEL SECURITY;
ALTER TABLE events ENABLE ROW LEVEL SECURITY;


Apply user vs admin policies.
```

---

## 💾 Sample SQL Queries & Policies <a name="sample-sql-queries"></a>

### 1️⃣ User Policies
```sql
-- CREATE POLICY "Students can view their own borrow records"
ON borrow_records
FOR SELECT
USING (
  auth.uid() = (
    SELECT p.id
    FROM profiles p
    JOIN students s ON s.student_id = borrow_records.student_id
    WHERE p.id = auth.uid()
  )
);

CREATE POLICY "Students can insert their own borrow records"
ON borrow_records
FOR INSERT
WITH CHECK (
  auth.uid() = (
    SELECT p.id
    FROM profiles p
    JOIN students s ON s.student_id = borrow_records.student_id
    WHERE p.id = auth.uid()
  )
);

CREATE POLICY "Librarians have full access to borrow records"
ON borrow_records
FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid()
    AND role = 'librarian'
  )
)
WITH CHECK (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid()
    AND role = 'librarian'
  )
);




```sql
-- 

```

```sql
-- 
```
---

### 2️⃣ Admin Policies
```sql
-- -- Librarians can manage all borrow records
CREATE POLICY "Librarians have full access to borrow records"
ON borrow_records
FOR ALL
USING (
  EXISTS (
    SELECT 1
    FROM profiles
    WHERE id = auth.uid()
    AND role = 'librarian'
  )
)
WITH CHECK (
  EXISTS (
    SELECT 1
    FROM profiles
    WHERE id = auth.uid()
    AND role = 'librarian'
  )
);

```

```sql
-- CREATE POLICY "Librarians can manage all books"
ON books
FOR ALL
USING (
  EXISTS (
    SELECT 1
    FROM profiles
    WHERE profiles.id = auth.uid()
    AND profiles.role = 'librarian'
  )
)
WITH CHECK (
  EXISTS (
    SELECT 1
    FROM profiles
    WHERE profiles.id = auth.uid()
    AND profiles.role = 'librarian'
  )
);


CREATE POLICY "Librarians can manage all borrow records"
ON borrow_records
FOR ALL
USING (
  EXISTS (
    SELECT 1
    FROM profiles
    WHERE profiles.id = auth.uid()
    AND profiles.role = 'librarian'
  )
)
WITH CHECK (
  EXISTS (
    SELECT 1
    FROM profiles
    WHERE profiles.id = auth.uid()
    AND profiles.role = 'librarian'
  )
);

```
---
 ### 3️⃣ Example CRUD Queries with their output.
 ```sql
-- -- Show borrow records for the currently logged-in student
SELECT 
  br.record_id,
  b.title AS book_title,
  br.borrow_date,
  br.due_date,
  br.return_date
FROM borrow_records br
JOIN books b ON br.book_id = b.book_id
WHERE br.student_id = (
  SELECT s.student_id
  FROM students s
  JOIN profiles p ON p.id = auth.uid()
  WHERE p.role = 'student'
);


-

```

## User Roles and Output


**member loan history**
```sql
SELECT
    m.first_name || ' ' || m.last_name AS member_name,
    b.title AS book_title,
    b.author,
    l.loan_date,
    l.due_date,
    l.return_date
FROM
    Loans l
JOIN
    Members m ON l.member_id = m.member_id
JOIN
    Books b ON l.book_id = b.book_id
WHERE
    m.email = 'alice.j@library.org'; -- Use the member's unique email (like the auth UID);
```
  **Ouput upon Inserting & Viewing their borrowed books**

<img width="1397" height="597" alt="image" src="https://github.com/user-attachments/assets/07d298a4-b0a8-49d7-8a05-912fd8a51a83" />


 

## Admin Roles and Output

**This query selects the full name, role, and creation date for everyone who is not a regular student, giving the SuperAdmin a clean list of staff members**
```sql
--SELECT
    p.full_name,
    p.role,
    p.created_at AS profile_creation_date
FROM
    profiles p
WHERE
    p.role != 'student' -- Filter out standard student users
ORDER BY
    p.role, 
    p.created_at DESC;
```
<img width="1412" height="629" alt="image" src="https://github.com/user-attachments/assets/4596df6b-7d4a-4022-bb65-8e783091c74f" />



## 🛡 Security Notes <a name="security-notes"></a>

See full explanation of RLS, policies, and admin functions in 👉 [security_notes.md](https://github.com/DENNIS-MURITHI/Data-Fundamentals/blob/data_test_branch/security_notes.md)


---

## 👥 Authors <a name="authors"></a>

- **Evans Kibet**  
  GitHub: [@mutahilinet](https://github.com/mutahilinet) 
  

---

## 🔭 Future Features <a name="future-features"></a>

- Front-End Integration: Develop a web app interface for managing the library database.

Borrowing Analytics: Add reports for tracking most popular books and top-borrowing students.

Audit Logging: Implement a separate table to record all staff actions (add, edit, delete) on key records.

Digital Member IDs: Use student IDs or QR codes for quick, valid check-outs at the desk.

Automated Notifications: Send email/SMS reminders for overdue books and due dates.

Fine Management: Include system features for calculating and managing late fines, including the ability to process fine reversals/cancellations.

Role-Based Access Control (RBAC): Fully implement security policies to restrict actions based on user roles (SuperAdmin, Librarian, Student).
Add dashboard for revenue and sales insights
---

## 🤝 Contributing <a name="contributing"></a>

Open issues or pull requests are welcome.

---

## ⭐️ Show your support <a name="support"></a>

Give a ⭐️ if you like this project!

---

## 🙏 Acknowledgements <a name="acknowledgements"></a>

- Supabase docs for SQL & RLS policies  
- PostgreSQL official docs  

---

## ❓ FAQ <a name="faq"></a>

**Q: How do I test RLS policies?**  
A: Sign in as User vs Admin and try CRUD operations. Policies will restrict or allow access accordingly.  

**Q: Can I extend this to a front-end?**  
A: Yes, connect Supabase Auth with **Angular**, **Java**, or any front-end framework.  

---

## 📝 License <a name="license"></a>

This project is licensed under the MIT License - see the LICENSE file for details.

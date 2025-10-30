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
-- Users can view only their own tickets
CREATE POLICY "Users can view their own tickets"
ON tickets
FOR SELECT
USING (
  auth.uid() = (
    SELECT auth_user_id FROM customers WHERE customers.customer_id = tickets.customer_id
  )
);

```sql
-- Users can insert their own tickets
CREATE POLICY "Users can insert their own tickets"
ON tickets
FOR INSERT
WITH CHECK (
  auth.uid() = (
    SELECT auth_user_id FROM customers WHERE customers.customer_id = tickets.customer_id
  )
);
```

```sql
-- Users can view event details
CREATE POLICY "Users can view events"
ON events
FOR SELECT
USING (true);
```
---

### 2️⃣ Admin Policies
```sql
-- Admins can manage all tickets
CREATE POLICY "Admins have full access to tickets"
ON tickets
FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM customers WHERE auth_user_id = auth.uid() AND city = 'Admin'
  )
);

```sql
-- Admins can manage all payments
CREATE POLICY "Admins have full access to payments"
ON payments
FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM customers WHERE auth_user_id = auth.uid() AND city = 'Admin'
  )
);
```

```sql
-- Admins can view, edit, or delete all customers
CREATE POLICY "Admins can manage all customers"
ON customers
FOR ALL
USING (
  EXISTS (
    SELECT 1
    FROM users
    WHERE users.user_uuid = auth.uid()
    AND users.role = 'admin'
  )
);
```
```sql
CREATE POLICY "Admins can manage all event schedules"
ON event_schedules
FOR ALL
USING (
  EXISTS (
    SELECT 1
    FROM users
    WHERE users.user_uuid = auth.uid()
    AND users.role = 'admin'
  )
);
```
---
 ### 3️⃣ Example CRUD Queries with their output.
 ```sql
-- Show all events available for booking
SELECT * FROM events WHERE event_date >= CURRENT_DATE;

-- Show tickets purchased by a logged-in user
SELECT t.ticket_id, e.event_name, e.event_date
FROM tickets t
JOIN events e ON t.event_id = e.event_id
WHERE t.customer_id = (
  SELECT customer_id FROM customers WHERE auth_user_id = auth.uid()
);

-- Admin deletes a ticket (admin-only function)
SELECT delete_ticket_admin(3);

```
1️⃣ Fetch all tickets with customer and event details

<img width="1798" height="918" alt="image" src="https://github.com/user-attachments/assets/62fc6015-a39c-45d2-a7ca-8fbb866f1569" />

2️⃣ Fetch all payments with event info

<img width="1769" height="896" alt="image" src="https://github.com/user-attachments/assets/661341af-32fa-4212-8926-033d721c4b20" />

## User Roles and Output

**User purchases new tickets**
```sql
-- Insert new ticket (User only)
INSERT INTO tickets (event_id, customer_id, quantity, purchase_date)
VALUES (1, 1, 2, '2025-10-30');
```
**View tickets purchased by a specific user**
```sql
SELECT 
  c.full_name AS customer,
  e.event_name,
  e.location,
  t.quantity,
  t.purchase_date
FROM tickets t
JOIN customers c ON t.customer_id = c.customer_id
JOIN events e ON t.event_id = e.event_id
WHERE c.auth_user_id = 'eb08886f-6f46-4cae-a39e-7ad7619f7046';  -- User’s auth UID
```
  **Ouput upon Inserting & Viewing their tickets**

<img width="1846" height="826" alt="image" src="https://github.com/user-attachments/assets/97a8eb75-31d1-412b-be32-d4f325aae37a" />

**Viewing their tickets**
<img width="1794" height="860" alt="image" src="https://github.com/user-attachments/assets/4ae59f41-3be0-42d2-95a9-4c72dc77d2a3" />



  **User can also delete their purchased ticket**

```sql
DELETE FROM tickets
WHERE customer_id = 1 AND event_id = 1;

```
**User cancels (deletes) their ticket with id=1**

<img width="1900" height="838" alt="image" src="https://github.com/user-attachments/assets/c16dbed7-1f6b-4340-b135-f9592b64ae41" />



## Admin Roles and Output

**Admin add a new event and view before updating**

```sql
-- Admin adds an event
INSERT INTO events (event_name, event_date, location, price)
VALUES ('Developers Meetup', '2025-12-10', 'Mombasa', 1200.00);

```
<img width="1897" height="916" alt="image" src="https://github.com/user-attachments/assets/e5e85e81-17b8-4579-8dc8-785e63ff83b7" />


**Admin Update event details**
```sql
-- Admin updates event info
UPDATE events
SET price = 1500.00, location = 'Nairobi'
WHERE event_id = 6;

```
<img width="1808" height="880" alt="image" src="https://github.com/user-attachments/assets/664056d8-dd7c-4cb8-86bc-4a03fcd8575c" />

**Admin Delete an event**
```sql
-- Admin deletes an event
DELETE FROM events WHERE event_id = 6;

```
<img width="1877" height="896" alt="image" src="https://github.com/user-attachments/assets/6ccf6fef-a08d-47c9-b5b3-ec3e8bc69d18" />

---

## 🛡 Security Notes <a name="security-notes"></a>

See full explanation of RLS, policies, and admin functions in 👉 [security_notes.md](https://github.com/DENNIS-MURITHI/Data-Fundamentals/blob/data_test_branch/security_notes.md)


---

## 👥 Authors <a name="authors"></a>

- **Evans Kibet**  
  GitHub: [@EvansKibet](https://github.com/Evans-dotcom)  
  LinkedIn: [Evans Kibet](https://www.linkedin.com/in/evans-langat-680b05342/)  

---

## 🔭 Future Features <a name="future-features"></a>

- Integrate with front-end event booking portal
- Add analytics for most attended events and top-paying customers
- Implement audit logging for admin actions (create, update, delete events)
- Enable ticket QR code generation for entry validation
-Add email/SMS notifications for successful payments
-Include refund and cancellation management
-Implement role-based access (Admin, Customer)

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

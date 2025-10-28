
# Security Notes — library management system ~~ Data Fundamentals Project

This document explains the **security setup**, **Row Level Security (RLS)**, **roles**, and **policies** for the library management Database implemented on Supabase (Postgres). It describes how authentication (Supabase Auth) ties to data, how Admin and User roles are enforced, admin-only functions, and how to test everything step-by-step.

------------------------------------------------------------------------
## 🔑 Identity & Role Mapping

We use Supabase Auth for user authentication and a small role store to determine admin privileges.
auth.users (Supabase-managed) holds authenticated user accounts (UUID id).
profiles (recommended) stores user role information and maps to auth.users(id).
customers has an auth_user_id column (UUID) to map application records to authenticated users.
# Example schema snippets (role & linkage):
``` sql
CREATE TABLE IF NOT EXISTS profiles (
  id uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name text,
  role text NOT NULL DEFAULT 'user', -- 'user' | 'admin'
  created_at timestamptz NOT NULL DEFAULT now()
);

<img width="1824" height="721" alt="image" src="https://github.com/user-attachments/assets/e66ff657-017c-452c-9843-51e9df156697" />


```
# 🔒 Enable Row Level Security (RLS)

Enable RLS on every table you want protected. RLS denies access by default until policies are created.
``` sql
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE events ENABLE ROW LEVEL SECURITY;
ALTER TABLE tickets ENABLE ROW LEVEL SECURITY;
ALTER TABLE payments ENABLE ROW LEVEL SECURITY;

```

***The random uuid(id) generated are as shown below with string of characters***

```sql
SELECT * FROM authors;

`
<img width="1794" height="897" alt="image" src="https://github.com/user-attachments/assets/a9f9cc9f-a090-4d39-9317-ec9f5c739415" />


------------------------------------------------------------------------

## 👤 User Policies

Normal users have restricted access.
- events (public read, admin manage)

``` sql
-- View only their own favorites
CREATE POLICY "Public read events"
ON events FOR SELECT USING (true);

CREATE POLICY "Admins manage events"
ON events FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'));
```
- customers (users manage their own customer row; admins full access)
``` sql
CREATE POLICY "Admins full access customers"
ON customers FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'));

CREATE POLICY "Users can select own customer"
ON customers FOR SELECT
USING (auth_user_id = auth.uid());

CREATE POLICY "Users can update own customer"
ON customers FOR UPDATE
USING (auth_user_id = auth.uid())
WITH CHECK (auth_user_id = auth.uid());

CREATE POLICY "Users can insert own customer"
ON customers FOR INSERT
WITH CHECK (auth_user_id = auth.uid());
```
- tickets (users can manage tickets they own; admins full access)

``` sql
  CREATE POLICY "Admins full access tickets"
ON tickets FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'));

CREATE POLICY "Users can select own tickets"
ON tickets FOR SELECT
USING (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can insert own tickets"
ON tickets FOR INSERT
WITH CHECK (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can update own tickets"
ON tickets FOR UPDATE
USING (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
)
WITH CHECK (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can delete own tickets"
ON tickets FOR DELETE
USING (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);
```
- payments (users can manage payments for tickets they own; admins full access)
``` sql
CREATE POLICY "Admins full access payments"
ON payments FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'));

CREATE POLICY "Users can select payments for their tickets"
ON payments FOR SELECT
USING (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can insert payments for their tickets"
ON payments FOR INSERT
WITH CHECK (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can update payments for their tickets"
ON payments FOR UPDATE
USING (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
)
WITH CHECK (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can delete payments for their tickets"
ON payments FOR DELETE
USING (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

```

------------------------------------------------------------------------

## 👨‍💼 Admin Policies

Admins have **full access**.
Use **SECURITY DEFINER** functions with internal role checks as defense-in-depth.

- Delete event (admin only)
``` sql
CREATE OR REPLACE FUNCTION delete_event_by_admin(eid INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  IF NOT EXISTS (SELECT 1 FROM profiles p WHERE p.id = auth.uid() AND p.role = 'admin') THEN
    RAISE EXCEPTION 'Only admins may call delete_event_by_admin';
  END IF;
  DELETE FROM events WHERE event_id = eid;
END;
$$;
```
- Delete ticket (admin only):
  
``` sql
CREATE OR REPLACE FUNCTION delete_ticket_by_admin(tid INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  IF NOT EXISTS (SELECT 1 FROM profiles p WHERE p.id = auth.uid() AND p.role = 'admin') THEN
    RAISE EXCEPTION 'Only admins may call delete_ticket_by_admin';
  END IF;
  DELETE FROM tickets WHERE ticket_id = tid;
END;
$$;

```

------------------------------------------------------------------------

## ⚡️ Testing Roles & Policies

### ✅ User Tests
1.  **SELECT own tickets** (works)\
2.  **INSERT a new ticket purchase** (works)\
3.  **UPDATE or DELETE tickets/payments** (blocked)\
   
- **Create test accounts** in Supabase Auth:
 - Admin user (set profiles.role = 'admin')
 - Regular user A and user B
- **Create/Update profiles** or use signup trigger to auto-create a profile for each auth.users row.. **Populate sample data** (events, customers with auth_user_id, tickets, payments). Ensure customers.auth_user_id points to users.
- Test with **supabase** :
 -Sign in as regular user → request tickets and payments. Confirm user only sees their own.
 - Try to perform admin-only actions (update/delete events) as regular user → **should fail**.
- Sign in as **admin** → confirm full access, call admin RPCs (delete_event_by_admin).
- Use **SQL Editor** for debugging only (SQL Editor runs as service_role and bypasses RLS — do not use this for policy tests).

### ✅ Admin Tests

1.  **UPDATE event details** (works)\
2.  **DELETE event**(works)\
3.  **SELECT all tickets** (works)

------------------------------------------------------------------------

<img width="1903" height="832" alt="image" src="https://github.com/user-attachments/assets/58d58dc6-db66-4607-8d37-d2769784097b" />


## 🛠 Admin-only Function

Example: Admin deletes an event safely.

``` sql
CREATE OR REPLACE FUNCTION delete_event_safe(event_id INT)
RETURNS VOID
LANGUAGE SQL
SECURITY DEFINER
AS $$
  DELETE FROM events WHERE event_id = $1;
$$;

```

------------------------------------------------------------------------

<img width="1901" height="796" alt="image" src="https://github.com/user-attachments/assets/85a3c200-c0d4-408a-a43d-456e601bf6d4" />


## 📎 Reference

-   Linked to [README.md](https://github.com/Evans-dotcom/Data-Tools/blob/Data_Fundamentals-Branch/ReadMe.md)
-   Supabase Policies: <https://supabase.com/docs/guides/database/postgres/row-level-security>

  

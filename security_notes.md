
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
  full_name TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('librarian', 'student')) DEFAULT 'student',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);


<img width="1865" height="739" alt="image" src="https://github.com/user-attachments/assets/812afe2c-c714-409a-ba49-c962e9e87ca1" />


```
# 🔒 Enable Row Level Security (RLS)

Enable RLS on every table you want protected. RLS denies access by default until policies are created.
``` sql
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE students ENABLE ROW LEVEL SECURITY;
ALTER TABLE books ENABLE ROW LEVEL SECURITY;
ALTER TABLE borrow_records ENABLE ROW LEVEL SECURITY;
ALTER TABLE authors ENABLE ROW LEVEL SECURITY;
ALTER TABLE publishers ENABLE ROW LEVEL SECURITY;
-- 
```

***The random uuid(id) generated are as shown below with string of characters***

```sql
SELECT * FROM profiles;

`
<img width="1901" height="907" alt="image" src="https://github.com/user-attachments/assets/5a933baa-5f9d-45d3-9b27-5a344b28745e" />


------------------------------------------------------------------------

## 👤 User Policies

Normal users have restricted access.
- events (public read, admin manage)

``` sql
-- -- View only their own favorites
CREATE POLICY "Public read books"
ON books FOR SELECT USING (true);

CREATE POLICY "Librarians manage books"
ON books FOR ALL -- Applies to INSERT, UPDATE, DELETE
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'cataloger', 'superadmin')))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'cataloger', 'superadmin')));

CREATE POLICY "Librarians manage borrow_records"
ON borrow_records FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'superadmin')))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'superadmin')));


- customers (users manage their own customer row; admins full access)
``` sql
CREATE POLICY "Librarians full access students"
ON students FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'superadmin')))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'superadmin')));

CREATE POLICY "Students can select own student record"
ON students FOR SELECT
USING (auth_user_id = auth.uid());

CREATE POLICY "Students can update own loan records"
ON borrow_records FOR UPDATE
USING (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id
      and s.auth_user_id = auth.uid()
  )
)
WITH CHECK (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id
      and s.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Students can insert own student record"
ON students FOR INSERT
WITH CHECK (auth_user_id = auth.uid());
```
--(users can manage book records they own; admins full access)

``` sql
  CREATE POLICY "Librarians full access borrow_records"
ON borrow_records FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'superadmin')))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role IN ('librarian', 'superadmin')));

CREATE POLICY "Students can select own loan records"
ON borrow_records FOR SELECT
USING (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id -- Link loan to student
      and s.auth_user_id = auth.uid()             -- Check ownership via auth ID
  )

);

CREATE POLICY "Students can insert own loan records"
ON borrow_records FOR INSERT
WITH CHECK (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id  -- Links the new loan to a student ID
      and s.auth_user_id = auth.uid()               -- Verifies the student ID is owned by the current user
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


);

CREATE POLICY "Students can delete own loan records"
ON borrow_records FOR DELETE
USING (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id  -- Links the loan to a student ID
      and s.auth_user_id = auth.uid()               -- Verifies the student ID is owned by the current user
  )
);
);
```
- payments (users can manage payments for tickets they own; admins full access)
``` sql
CREATE POLICY "Admins full access payments"
ON payments FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'));

CREATE POLICY "Students can select own loan records"
ON borrow_records FOR SELECT
USING (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id -- Link loan to student ID
      and s.auth_user_id = auth.uid()             -- Verify ownership via auth ID
  )

);

CREATE POLICY "Students can insert own loan records"
ON borrow_records FOR INSERT
WITH CHECK (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id  -- Links the loan to a student ID
      and s.auth_user_id = auth.uid()               -- Verifies the student ID is owned by the current user
  )

  )
);

CREATE POLICY "Students can update own loan records"
ON borrow_records FOR UPDATE
USING (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id  -- Links the loan to the student's ID
      and s.auth_user_id = auth.uid()               -- Verifies the student ID is owned by the current user
  )
)

);
)
WITH CHECK (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id -- Link loan to student ID
      and s.auth_user_id = auth.uid()              -- Verify ownership via auth ID
  )
)
);

CREATE POLICY "Students can delete own loan records"
ON borrow_records FOR DELETE
USING (
  exists (
    select 1 from students s
    where s.student_id = borrow_records.student_id  -- Links the loan record to the student ID
      and s.auth_user_id = auth.uid()               -- Verifies the student ID is owned by the current user
  )

);

```

------------------------------------------------------------------------

## 👨‍💼 Admin Policies

Admins have **full access**.
Use **SECURITY DEFINER** functions with internal role checks as defense-in-depth.

- This function allows authorized library staff to delete a specific student record from the system.
``` sql
CREATE OR REPLACE FUNCTION delete_student_by_staff(sid INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER -- Executes with high privileges, bypassing RLS
AS $$
BEGIN
    -- CRITICAL ROLE CHECK: Ensure the caller has an authorized staff role
  IF NOT EXISTS (SELECT 1 FROM profiles p WHERE p.id = auth.uid() AND p.role IN ('librarian', 'superadmin')) THEN
    RAISE EXCEPTION 'Only authorized library staff may delete student records.';
  END IF;

    -- The actual administrative action: Delete the specific student record
  DELETE FROM students WHERE student_id = sid;

    -- Optional: If student deletion is blocked due to foreign key constraints,
    -- a more complex function would handle deleting or anonymizing associated
    -- borrow_records first.
    IF NOT FOUND THEN
        RAISE NOTICE 'Student ID % was not found.', sid;
    END IF;
END;
$$;
```
- library staff to delete a specific loan record from the transaction history.:
  
``` sql
CREATE OR REPLACE FUNCTION delete_loan_record_by_staff(rid INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER -- Executes with high privileges, bypassing RLS
AS $$
BEGIN
    -- CRITICAL ROLE CHECK: Ensure the caller has an authorized staff role
  IF NOT EXISTS (SELECT 1 FROM profiles p WHERE p.id = auth.uid() AND p.role IN ('librarian', 'superadmin')) THEN
    RAISE EXCEPTION 'Only authorized library staff may delete loan records.';
  END IF;

    -- The actual administrative action: Delete the specific loan record
  DELETE FROM borrow_records WHERE record_id = rid;

    -- Optional: Add a check to confirm deletion or raise an error if ID was not found
    IF NOT FOUND THEN
        RAISE NOTICE 'Loan record ID % was not found.', rid;
    END IF;
END;
$$;

```

------------------------------------------------------------------------

Test 1: Update Catalog (UPDATE)

Action: As a Librarian, successfully UPDATE a book's detail (e.g., change copies_available in the books table).

Expected Result: Action WORKS without an RLS error.

Test 2: Delete Inventory (DELETE via RPC)

Action: As a Librarian, successfully call the secure function (delete_book_safe or delete_student_by_staff).

Expected Result: Function WORKS, and the record is permanently deleted.

Test 3: View All Transactions (SELECT)

Action: As a Librarian, run a SELECT query on the borrow_records table.

Expected Result: Query WORKS, and all loan records (including every student's history) are returned.

Test 4: Student-Only Action (Failure Test)

Action: As a Librarian, try to perform an action restricted to the student role (e.g., try to register a second profile row with the Students can insert own student record policy).

Expected Result: Action is BLOCKED or fails gracefully, proving the role restrictions work in both directions.
-- Test 1: UPDATE book details 
UPDATE books
SET copies_available = 6
WHERE title = 'Beloved';

------------------------------------------------------------------------

<img width="1830" height="846" alt="image" src="https://github.com/user-attachments/assets/ca9bb68f-fd01-43ba-9130-b88a6889162b" />


## 🛠 Admin-only Function


``` sql
CREATE OR REPLACE FUNCTION delete_book_safe(book_id INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER -- Runs as the superuser, which is crucial for RLS bypass
AS $$
BEGIN
    -- CRITICAL CHECK: Ensure the caller has an authorized staff role
  IF NOT EXISTS (SELECT 1 FROM profiles p WHERE p.id = auth.uid() AND p.role IN ('librarian', 'superadmin')) THEN
    RAISE EXCEPTION 'Only authorized staff may call delete_book_safe.';
  END IF;

    -- The actual administrative action
  DELETE FROM books WHERE book_id = $1;
END;
$$;

```

------------------------------------------------------------------------

<img width="1875" height="602" alt="image" src="https://github.com/user-attachments/assets/ec1395ea-9eb0-43a3-89a4-19e7ed475c22" />


## 📎 Reference

-   Linked to https://github.com/mutahilinet/Data-Tools/edit/main/security_notes.md
-   Supabase Policies: <https://supabase.com/docs/guides/database/postgres/row-level-security>

  

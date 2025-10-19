# Data-Tools
Project
# My library management SQL Project

<a name="readme-top"></a>

<!-- TABLE OF CONTENTS -->

# 📗 Table of Contents

- [My SQL Project](#about-project)
- [📗 Table of Contents](#-table-of-contents)
- [📖 My SQL Project](#about-project)
  - [🛠 Built With ](#-built-with-)
    - [Tech Stack ](#tech-stack-)
    - [Key Features ](#key-features-)
  - [💻 Getting Started ](#-getting-started-)
    - [Prerequisites](#prerequisites)
    - [Setup](#setup)
    - [Usage](#usage)
  - [👥 Authors ](#-authors-)
  - [🔭 Future Features ](#-future-features-)
  - [🤝 Contributing ](#-contributing-)

<!-- PROJECT DESCRIPTION -->

# 📖 My SQL Project <a name="about-project"></a>

**My SQL Project** is a simple Database that uses SQL, Postgres via Supabase and R to create, query and secure a **Bookstore** database.

## 🛠 Built With <a name="built-with"></a>

### Tech Stack <a name="tech-stack"></a>
- SQL
- Postgres DB

<!-- Features -->

### Key Features <a name="key-features"></a>

- [ ] **Tables**
- [ ] **Schema**
- [ ] **Access control**

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->

## 💻 Getting Started <a name="getting-started"></a>

To rebuild this DB, follow these steps.

### Prerequisites

To run this project, you need:
- [A Supabase account](https://supabase.com/)
- [Knowledge on SQL](https://www.w3schools.com/sql/)
- A schema for creating your tables in the DB

<!-- ### Setup -->
### Setup

Copy the contents of this Readme.md to your Project's file

OR

Clone this repository to your desired folder:

```sh
  git clone https://github.com/joyapisi/readme-template-data
  cd budget-app
```

<!-- ### DB Creation -->

### DB Schema

- The DB is made up of 3 tables. Eaach table has 5 entries.
- To create the table, you will need a schema as shown below:

```sql
-- Drop old tables if they exist
DROP TABLE IF EXISTS borrow_records
DROP TABLE IF EXISTS books 
DROP TABLE IF EXISTS students
DROP TABLE IF EXISTS authors
DROP TABLE IF EXISTS publishers


--CREATE TABLE authors (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  country TEXT
);

--CREATE TABLE publishers (
    publisher_id SERIAL PRIMARY KEY, -- Primary Key, auto-incrementing ID
    publisher_name VARCHAR(255) NOT NULL UNIQUE, -- Unique name for the publisher
    city VARCHAR(100)
);

--CREATE TABLE students (
    student_id SERIAL PRIMARY KEY, -- Primary Key
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL, -- Email must be unique for each student
    enrollment_date DATE NOT NULL
);

-- CREATE TABLE books (
    book_id SERIAL PRIMARY KEY, -- Primary Key
    title VARCHAR(255) NOT NULL,
    isbn VARCHAR(13) UNIQUE NOT NULL, -- International Standard Book Number, must be unique
    publication_year SMALLINT, -- Changed YEAR to SMALLINT for broader compatibility
    copies_available INT NOT NULL DEFAULT 1, -- Number of copies currently available for loan

    -- Foreign Key references the authors table
    author_id INT,
    FOREIGN KEY (author_id) REFERENCES authors(author_id) ON DELETE RESTRICT,

    -- Foreign Key references the publishers table
    publisher_id INT,
    FOREIGN KEY (publisher_id) REFERENCES publishers(publisher_id) ON DELETE RESTRICT
);

--CREATE TABLE borrow_records (
    record_id SERIAL PRIMARY KEY, -- Primary Key
    
    -- Foreign Key references the books table
    book_id INT NOT NULL,
    FOREIGN KEY (book_id) REFERENCES books(book_id) ON DELETE RESTRICT,
    
    -- Foreign Key references the students table
    student_id INT NOT NULL,
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE RESTRICT,
    
    borrow_date DATE NOT NULL,
    due_date DATE NOT NULL,
    return_date DATE, -- NULL if the book has not yet been returned
    
    -- Constraint to ensure a book is not recorded as borrowed after it has been returned
    CHECK (return_date IS NULL OR return_date >= borrow_date)
);

-- 1. Insert Sample AUTHORS (5 rows)
INSERT INTO authors (first_name, last_name, nationality) VALUES
    ('Chimamanda', 'Adichie', 'Nigerian'),
    ('Ngũgĩ', 'wa Thiong''o', 'Kenyan'),
    ('Gabriel', 'Garcia Marquez', 'Colombian'),
    ('Toni', 'Morrison', 'American'),
    ('Chinua', 'Achebe', 'Nigerian');


-- 2. Insert Sample PUBLISHERS (3 rows)
INSERT INTO publishers (publisher_name, city) VALUES
    ('Anchor Books', 'New York'),
    ('Vintage', 'London'),
    ('Penguin Random House', 'Berlin');


-- 3. Insert Sample STUDENTS (4 rows)
INSERT INTO students (first_name, last_name, email, enrollment_date) VALUES
    ('Alex', 'Johnson', 'alex.j@uni.edu', '2022-09-01'),
    ('Sarah', 'Lee', 'sarah.lee@uni.edu', '2023-01-15'),
    ('Ben', 'Carter', 'ben.c@uni.edu', '2021-08-20'),
    ('Mia', 'Rodriguez', 'mia.r@uni.edu', '2024-02-28');


-- 4. Insert Sample BOOKS (5 rows)
-- We use the automatically generated IDs from the AUTHORS and PUBLISHERS tables (1, 2, 3, etc.)
INSERT INTO books (title, isbn, publication_year, copies_available, author_id, publisher_id) VALUES
    -- Author 1 (Adichie), Publisher 1 (Anchor)
    ('Half of a Yellow Sun', '9780307456789', 2006, 3, 1, 1),
    -- Author 2 (Ngũgĩ), Publisher 2 (Vintage)
    ('Weep Not, Child', '9780143105423', 1964, 2, 2, 2),
    -- Author 3 (Marquez), Publisher 1 (Anchor)
    ('One Hundred Years of Solitude', '9780060919672', 1967, 5, 3, 1),
    -- Author 4 (Morrison), Publisher 3 (Penguin)
    ('Beloved', '9781400033423', 1987, 1, 4, 3),
    -- Author 5 (Achebe), Publisher 2 (Vintage)
    ('Things Fall Apart', '9780385532589', 1958, 4, 5, 2);


-- 5. Insert Sample BORROW_RECORDS (3 rows)
-- This shows one returned book and two currently borrowed books.
INSERT INTO borrow_records (book_id, student_id, borrow_date, due_date, return_date) VALUES
    -- Alex (1) borrowed Half of a Yellow Sun (1) and returned it early
    (1, 1, '2024-05-01', '2024-05-15', '2024-05-14'),
    -- Sarah (2) borrowed One Hundred Years of Solitude (3) - currently out
    (3, 2, '2024-05-10', '2024-05-24', NULL),
    -- Ben (3) borrowed Beloved (4) - currently out
    (4, 3, '2024-05-05', '2024-05-19', NULL);

- The Tables should look like this in Supabase:
authors
<img width="1893" height="476" alt="image" src="https://github.com/user-attachments/assets/9a89f3ae-77d1-4ed2-a5c5-140db1e7e27b" />

books:
<img width="1881" height="445" alt="image" src="https://github.com/user-attachments/assets/d741319f-a0ff-416c-b50f-34c315c9af24" />

customers:
<img width="1881" height="505" alt="image" src="https://github.com/user-attachments/assets/354752e6-fa32-4aa8-a28f-bf99f98039f2" />

orders:
<img width="1902" height="517" alt="image" src="https://github.com/user-attachments/assets/fe99a68a-8950-4d87-82c1-25dcd3217a65" />

- The ERD screenshot from Supabase looks like this: 
<img width="1064" height="577" alt="image" src="https://github.com/user-attachments/assets/4b8a39b1-ff20-4bd3-be6f-f662b35ae49f" />

- To test the table, I used two queries: 

```sql
SELECT * FROM orders
WHERE name = "Nadine Gordimer"
````

```sql
SELECT * FROM books
WHERE in_stock = "TRUE"
````

- Here are the results of the queries:
<img width="1460" height="791" alt="image" src="https://github.com/user-attachments/assets/37cf0a4e-ca92-4d8d-8888-2cca0165d32b" />

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- AUTHORS -->

## 👥 Authors <a name="authors"></a>

👤 **Joy Phoebe**

- GitHub: [@joyapisi](https://github.com/joyapisi)
- Twitter: [@joyphoebe300](https://twitter.com/joyphoebe300)
- LinkedIn: [@joyapisi](https://linkedin.com/in/joyapisi)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- FUTURE FEATURES -->

## 🔭 Future Features <a name="future-features"></a>

- [ ] **Add security**
- [ ] **Link DB to R for visualisation purposes and further analyses**

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTRIBUTING -->

## 🤝 Contributing <a name="contributing"></a>

Contributions, issues, and feature requests are welcome!

Feel free to check the [issues page](../../issues/).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- SUPPORT -->

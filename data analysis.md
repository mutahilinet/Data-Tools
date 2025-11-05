# 📊 Library Management Data Analysis

<div align="center">
  <img width="200" height="200" alt="Library Analytics Logo" src="https://github.com/user-attachments/assets/20661293-a214-4004-9042-657102fb0710" />
  <br/>
  <h2><b>Library Management Analytics Project</b></h2>
</div>

## 📗 Table of Contents

- 📖 [About the Project](#about-project)
- 🛠 [Built With](#built-with)
- ✨ [Key Features](#key-features)
- 🚀 [Live Demo](#live-demo)
- 💻 [Getting Started](#getting-started)
  - Prerequisites
  - Setup
  - Usage
  - Connecting from Posit to Supabase
- 💾 [Schema SQL](#schema-sql)
- 📊 [R Data Analysis](#r-data-analysis)
- 📖 [Data Dictionary](#data-dictionary)
- 📈 [Power BI Dashboard Preview](#powerbi-dashboard)
- 👥 [Authors](#authors)
- 🔭 [Future Features](#future-features)
- 🤝 [Contributing](#contributing)
- ⭐ [Show your support](#support)
- 🙏 [Acknowledgements](#acknowledgements)
- ❓ [FAQ](#faq)
- 📝 [License](#license)

---

## 📖 About the Project <a name="about-project"></a>

This project explores and analyzes library operations and borrowing trends using a PostgreSQL database hosted on Supabase.  
It demonstrates how to track book borrowing, student activity, popular authors, and borrowing durations using SQL queries and R-based analytics (Posit Studio).

---

## 🛠 Built With <a name="built-with"></a>

### Tech Stack

<details>
<summary>Database & Hosting</summary>
<ul>
<li><a href="https://supabase.com">Supabase (PostgreSQL)</a> – backend database for books, students, and borrow records</li>
</ul>
</details>

<details>
<summary>SQL Queries</summary>
<ul>
<li>Schema creation, data population, and query optimization</li>
</ul>
</details>

<details>
<summary>R Data Analysis</summary>
<ul>
<li><a href="https://posit.co/">Posit / RStudio</a> for connecting to Supabase and running visual analytics</li>
<li>Libraries: DBI, dplyr, ggplot2 for database connection and EDA</li>
</ul>
</details>

---

## ✨ Key Features <a name="key-features"></a>

- Track borrow and return patterns across users  
- Identify top borrowed books and most active students  
- Analyze overdue returns and borrowing frequency  
- Visualize trends using R (ggplot2) and Power BI  

---

## 🚀 Live Demo <a name="live-demo"></a>

Backend-only project. Run SQL & R analysis directly.  
Supabase Dashboard

---

## 💻 Getting Started <a name="getting-started"></a>

### Prerequisites

- Supabase account  
- R / Posit IDE  
- R packages: DBI, RPostgres, dplyr, ggplot2  
- Git installed  

### Setup

```bash
git clone https://github.com/DENNIS-MURITHI/library-management-data-analysis.git
cd library-management-data-analysis
```

### Usage

1. Open Supabase → SQL Editor  
2. Run the `schema.sql` script below to create tables  
3. Connect to Supabase from R using `connect_db.R`  
4. Run the analysis code in `analysis.R`  

### Connecting from Posit to Supabase <a name="posit-supabase-connection"></a>

```r
install.packages(c("DBI", "RPostgres", "dplyr", "ggplot2"))
library(DBI)

connect_db <- function() {
  dbConnect(
    RPostgres::Postgres(),
    dbname = "your_dbname",
    host = "your-project.supabase.co",
    port = 5432,
    user = "your_username",
    password = "your_password",
    sslmode = "require"
  )
}

con <- connect_db()
dbListTables(con)
```

---
## output for the database connection in posit
<img width="1872" height="771" alt="image" src="https://github.com/user-attachments/assets/ae5c6000-bf8c-44ff-87c9-0877b9a5985e" />

## 💾 Schema SQL <a name="schema-sql"></a>

<details>
<summary>Click to expand schema.sql for Library Management Data Analysis</summary>

```sql
CREATE TABLE authors (
    author_id SERIAL PRIMARY KEY, -- Primary Key, auto-incrementing ID using SERIAL
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    nationality VARCHAR(50),
    -- Constraint to ensure the combination of first and last name is unique
    UNIQUE (first_name, last_name)
);

CREATE TABLE publishers (
    publisher_id SERIAL PRIMARY KEY, -- Primary Key, auto-incrementing ID
    publisher_name VARCHAR(255) NOT NULL UNIQUE, -- Unique name for the publisher
    city VARCHAR(100)
);

CREATE TABLE students (
    student_id SERIAL PRIMARY KEY, -- Primary Key
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL, -- Email must be unique for each student
    enrollment_date DATE NOT NULL
);

CREATE TABLE books (
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

CREATE TABLE borrow_records (
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

CREATE TABLE IF NOT EXISTS profiles (
  id uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name text,
  role text NOT NULL CHECK (role IN ('librarian', 'student')) DEFAULT 'student',
  created_at timestamptz NOT NULL DEFAULT now()
);

-- Insert sample authors

INSERT INTO authors (first_name, last_name, nationality)

VALUES

('George', 'Orwell', 'British'),

('Jane', 'Austen', 'British'),

('Mark', 'Twain', 'American'),

('Chinua', 'Achebe', 'Nigerian');
 
-- Insert publishers

INSERT INTO publishers (publisher_name, city)

VALUES

('Penguin Books', 'London'),

('Oxford Press', 'Oxford'),

('Random House', 'New York');
 
-- Insert students

INSERT INTO students (first_name, last_name, email, enrollment_date)

VALUES

('Alice', 'Njeri', 'alice@uni.edu', '2024-01-10'),

('Brian', 'Otieno', 'brian@uni.edu', '2024-02-15'),

('Clara', 'Wambui', 'clara@uni.edu', '2024-03-01'),

('Dennis', 'Murithi', 'dennis@uni.edu', '2024-04-05');
 
-- Insert books

INSERT INTO books (title, isbn, publication_year, copies_available, author_id, publisher_id)

VALUES

('1984', '9780451524935', 1949, 4, 1, 1),

('Pride and Prejudice', '9781503290563', 1813, 2, 2, 2),

('Adventures of Huckleberry Finn', '9780142437179', 1884, 3, 3, 3),

('Things Fall Apart', '9780385474542', 1958, 5, 4, 1);
 
-- Insert borrow records

INSERT INTO borrow_records (book_id, student_id, borrow_date, due_date, return_date)

VALUES

(1, 1, '2025-01-01', '2025-01-14', '2025-01-10'),

(2, 2, '2025-02-10', '2025-02-24', NULL),

(3, 3, '2025-02-12', '2025-02-26', '2025-02-20'),

(4, 4, '2025-03-05', '2025-03-19', NULL);

 
```
</details>

---

## 📊 R Data Analysis <a name="r-data-analysis"></a>

<details>
<summary>Click to expand full R analysis</summary>

```r
source("connect_db.R")
library(DBI)
library(dplyr)
library(ggplot2)

con <- connect_db()

# 1️⃣ Most Borrowed Books
most_borrowed <- dbGetQuery(con, "
  SELECT b.title, COUNT(br.book_id) AS borrow_count
  FROM borrow_records br
  JOIN books b ON br.book_id = b.book_id
  GROUP BY b.title
  ORDER BY borrow_count DESC;
")

ggplot(most_borrowed, aes(x=reorder(title, borrow_count), y=borrow_count, fill=title)) +
  geom_col(show.legend=FALSE) +
  coord_flip() +
  labs(title='📚 Most Borrowed Books', x='Book Title', y='Times Borrowed') +
  theme_minimal()

# 2️⃣ Active Students
active_students <- dbGetQuery(con, "
  SELECT s.full_name, COUNT(br.student_id) AS total_borrowed
  FROM borrow_records br
  JOIN students s ON br.student_id = s.student_id
  GROUP BY s.full_name
  ORDER BY total_borrowed DESC;
")

ggplot(active_students, aes(x=reorder(full_name, total_borrowed), y=total_borrowed, fill=full_name)) +
  geom_col(show.legend=FALSE) +
  coord_flip() +
  labs(title='👩‍🎓 Most Active Borrowers', x='Student', y='Books Borrowed') +
  theme_minimal()

```
</details>

### Example Plots

📘 **return status of borrowed books**  
<img width="1864" height="892" alt="image" src="https://github.com/user-attachments/assets/0e716d5b-cd6c-44ab-9674-102ca6b1b902" />

🎓 **author distribution by nationality**  
<img width="1726" height="872" alt="image" src="https://github.com/user-attachments/assets/46f45d03-2215-408a-853d-22e3c75644c0" />

---

---

## 📖 Data Dictionary <a name="data-dictionary"></a>

📄 **View Full Data Dictionary:**  
Library Data Dictionary

---

## 👥 Authors <a name="authors"></a>

👤 **mutahilinet**  
- GitHub: [@mutahilinet](https://github.com/mutahilinet)  
- LinkedIn: [LinkedIn](#)

---

## 🔭 Future Features <a name="future-features"></a>

- Add overdue fine analytics  
- Monthly borrowing trends dashboard (Power BI)  
- Predictive modeling for book demand  
- Integration with front-end dashboard  

---

## 🤝 Contributing <a name="contributing"></a>

Pull requests and suggestions are welcome!  
Fork the repo and open an issue.

---

## ⭐ Show your support <a name="support"></a>

If you like this project, give it a ⭐ on GitHub!

---

## 🙏 Acknowledgements <a name="acknowledgements"></a>

- Supabase for database hosting  
- Posit (RStudio) for visualization tools  
- Power BI for dashboard design inspiration  

---

## ❓ FAQ <a name="faq"></a>

**Q1:** How do I connect R to Supabase?  
👉 Use RPostgres + DBI libraries and your Supabase credentials.

**Q2:** What if my tables don’t appear in R?  
👉 Run `dbListTables(con)` to confirm connection before running queries.

---

## 📝 License <a name="license"></a>

This project is licensed under the **MIT License** – see the LICENSE file for details.

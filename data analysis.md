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
CREATE TABLE students (
  student_id SERIAL PRIMARY KEY,
  full_name VARCHAR(100),
  department VARCHAR(50),
  email VARCHAR(100) UNIQUE
);

CREATE TABLE books (
  book_id SERIAL PRIMARY KEY,
  title VARCHAR(150),
  author VARCHAR(100),
  genre VARCHAR(50),
  year_published INT
);

CREATE TABLE borrow_records (
  record_id SERIAL PRIMARY KEY,
  student_id INT REFERENCES students(student_id),
  book_id INT REFERENCES books(book_id),
  borrow_date DATE,
  return_date DATE
);

-- Sample Data
INSERT INTO students (full_name, department, email)
VALUES
('Alice Njeri', 'Computer Science', 'alice@library.org'),
('Brian Otieno', 'Mathematics', 'brian@library.org'),
('Clara Wambui', 'Economics', 'clara@library.org'),
('Dennis Murithi', 'Data Science', 'dennis@library.org');

INSERT INTO books (title, author, genre, year_published)
VALUES
('Data Analytics for All', 'Hadley Wickham', 'Data Science', 2021),
('Database Systems', 'Elmasri & Navathe', 'Computer Science', 2016),
('Linear Algebra Simplified', 'Gilbert Strang', 'Mathematics', 2019),
('Econometrics 101', 'Jeff Wooldridge', 'Economics', 2017);

INSERT INTO borrow_records (student_id, book_id, borrow_date, return_date)
VALUES
(1, 1, '2025-01-02', '2025-01-12'),
(2, 3, '2025-01-15', '2025-01-22'),
(3, 4, '2025-02-10', NULL),
(4, 2, '2025-02-14', '2025-02-24');
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

# 3️⃣ Borrowing Duration Analysis
borrow_duration <- dbGetQuery(con, "
  SELECT title, (return_date - borrow_date) AS duration
  FROM borrow_records br
  JOIN books b ON br.book_id = b.book_id
  WHERE return_date IS NOT NULL;
")

ggplot(borrow_duration, aes(x=title, y=duration, fill=title)) +
  geom_col(show.legend=FALSE) +
  coord_flip() +
  labs(title='⏱ Borrow Duration per Book', x='Book', y='Days Borrowed') +
  theme_minimal()
```
</details>

### Example Plots

📘 **Most Borrowed Books**  
<img width="1366" height="630" src="https://github.com/user-attachments/assets/a00864a0-99b8-4b2a-8e77-4cdfe6e73caa" />

🎓 **Most Active Borrowers**  
<img width="1366" height="630" src="https://github.com/user-attachments/assets/12d08288-53ce-4880-8c05-ff0382909a74" />

⏱ **Borrow Duration by Book**  
<img width="1366" height="630" src="https://github.com/user-attachments/assets/4b673d5b-c9c8-4814-8412-63ed767f8f62" />

---

## 📈 Power BI Dashboard Preview <a name="powerbi-dashboard"></a>

Power BI dashboard visualization placeholder.

---

## 📖 Data Dictionary <a name="data-dictionary"></a>

📄 **View Full Data Dictionary:**  
Library Data Dictionary

---

## 👥 Authors <a name="authors"></a>

👤 **Dennis Murithi**  
- GitHub: [@DENNIS-MURITHI](https://github.com/DENNIS-MURITHI)  
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

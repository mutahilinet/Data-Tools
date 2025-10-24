# Data Dictionary for library management system Database 

 This data dictionary describes the structure and purpose of the database tables used in the library management system. It defines each table, its columns, data types, and relationships to help developers and analysts clearly understand the data model. 

---
 
## Table: authors
Stores information about authors who wrote the books.  
 
<img width="1241" height="491" alt="image" src="https://github.com/user-attachments/assets/0c880ec4-f60c-472b-8281-22702450b021" />

---
 
## Table: publishers
Contains details about publishers of the books in the database.
 <img width="1124" height="367" alt="image" src="https://github.com/user-attachments/assets/d9487b1f-8236-4b05-896e-66af0b5fb43b" />

 
---
 
## Table: students
Records students who borrowed books. 
<img width="1066" height="450" alt="image" src="https://github.com/user-attachments/assets/a705fd9c-098f-4432-944d-04e0f1a1501b" />



---
 
## Table: books
Stores information about all the books in the library.
 <img width="1216" height="741" alt="image" src="https://github.com/user-attachments/assets/a6a86f78-1ab1-4d29-8918-957c221433aa" />


---



## Table :borrow records
Stores record of books borrowed by students.
<img width="1135" height="706" alt="image" src="https://github.com/user-attachments/assets/732b8a2f-98e0-47b4-bc2c-d3238389f07f" />


---

 **Notes:**  
Primary keys uniquely identify each record, while foreign keys (FK) connect related tables.
In this library system:

books.author_id → links to authors

books.publisher_id → links to publishers

borrow_records.book_id → links to books

borrow_records.student_id → links to students

This schema supports queries such as:

List all books written by a specific author

Show books borrowed by a student

Find overdue borrow records

Display books published by a certain publisher

Check available copies or borrowing history for each student

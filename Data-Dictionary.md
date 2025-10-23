# Data Dictionary for Event Ticketing Database 

 This data dictionary describes the structure and purpose of the database tables used in the Event Ticketing project. It defines each table, its columns, data types, and relationships to help developers and analysts clearly understand the data model. 

---
 
## Table: customers
Stores information about users who purchase tickets for events.  
 
<img width="909" height="350" alt="image" src="https://github.com/user-attachments/assets/7bedea0d-758d-493a-bbe7-e560b92506ab" />

---
 
## Table: events
Contains details about upcoming or past events available for ticket purchase. 
 <img width="870" height="380" alt="image" src="https://github.com/user-attachments/assets/38f3cae2-7372-408e-9f7e-82a217fb4799" />
 
---
 
## Table: tickets
Records tickets purchased by customers for specific events. 
<img width="863" height="429" alt="image" src="https://github.com/user-attachments/assets/89439568-939a-4f28-b90c-177033b7b62e" />

---
 
## Table: payments
Stores payment details for tickets purchased by customers  
 <img width="914" height="341" alt="image" src="https://github.com/user-attachments/assets/227813d3-36dd-41e9-9281-821e77855423" />

---
 **Notes:**  
- Primary keys uniquely identify rows in each table.
Foreign keys (FK) connect relationships across tables (tickets → customers/events, payments → tickets).
This schema supports queries like:
- List all tickets purchased by a specific customer
- Find all upcoming events after a specific date
- Show total payments received for a given event"
- Display customer details and their purchased event names
and many more queries based on your criteria. 

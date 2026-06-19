# Library Management System using SQL Project 

## Project Overview

**Project Title**: Library Management System  


This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/najirh/Library-System-Management---P2/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD](https://github.com/najirh/Library-System-Management---P2/blob/main/library_erd.png)

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.


```sql
create database library_db;

use  library_db;

DROP TABLE IF EXISTS books;
create table books (
isbn varchar(10) primary key,
book_title varchar(75),
category varchar(16),
rental_price decimal(10,2),
status_ varchar(10),
author varchar(30),
publisher varchar(30)
);

drop table if exists members;
create table members (
member_id varchar(10) primary key,
member_name varchar(20),
member_address varchar(25),
reg_date date 
);

drop table if exists branch;
create table branch (
branch_id varchar(10) primary key,
manager_id varchar(10),
branch_address varchar(15),
contact_no varchar(15)
);

drop table if exists issued_status;
create table issued_status (
issued_id varchar(10) primary key,
issued_member_id varchar(10),
issued_book_name varchar(70),
issued_date date,
issued_book_isbn varchar(20),
issued_emp_id varchar(10)
);

drop table if exists return_status;
create table return_status (
return_id varchar(10) primary key,
issued_id varchar(10),
return_book_name varchar(80),
return_date date,
return_book_isbn varchar(50)
);

drop table if exists employees;
create table employees (
emp_id varchar(10) primary key,
emp_name varchar (25),
position varchar(15),
salary int,
branch_id varchar(10)
);
```
---- imported csv files data using import wizard 

-- creating relationships between tables because i didnt add foreign key while creating tables. so here i m gonna use alter table to create relationsship btw tables(data model)--

```sql
ALTER TABLE employees
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

alter table issued_status
add constraint fk_member_id
foreign key (issued_member_id)
references members(member_id);

alter table issued_status
add constraint fk_emp_id
foreign key (issued_emp_id)
references employees(emp_id);

alter table issued_status
add constraint fk_book_isbn
foreign key (issued_book_isbn)
references books(isbn);

alter table return_status
add constraint fk_return_book_isbn
foreign key (return_book_isbn)
references books(isbn);
```


### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

-- Task 1. Create a New Book Record  in books 
-- error data too long for column isbn at row 1

alter table books
modify column isbn varchar(50);

```sql
insert into books ( isbn , book_title, category, rental_price, status_, author, publisher)
values ("978-1-60129-456-2", 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```
**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT ISSUED_EMP_ID, COUNT(*)
FROM ISSUED_STATUS
GROUP BY 1
HAVING COUNT(*) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
create table total_book_issued_count 
SELECT b.isbn , b.book_title, count(i.issued_book_name) as issued_count
from issued_status i
join books b
on i.issued_book_isbn = b.isbn
group by  1,2;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT SUM(b.RENTAL_PRICE) AS Total_rental_revenue, b.category, count(*)
 from issued_status i
 join books b
 on i.issued_book_isbn = b.isbn
 group by 2;

```

9. **List Members Who Registered in the Last 180 Days**:
```sql
select * from members
where reg_date >= curdate() - interval 180 day ;
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT e1.emp_id, e1.emp_name, e1.position, e1.salary, b.*, e2.emp_name as manager
FROM employees as e1
JOIN 
branch as b
ON e1.branch_id = b.branch_id    
JOIN
employees as e2
ON e2.emp_id = b.manager_id
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
create table expensive_books
select book_title, rental_price
from books 
where rental_price > 7;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT * FROM issued_status i
LEFT JOIN
return_status as r
ON r.issued_id = i.issued_id
WHERE r.return_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
select i.issued_member_id, m.member_name, b.book_title, i.issued_date, current_date() - i.issued_date as overdue_days
from issued_status i
join members m
on m.member_id = i.issued_member_id 
join books b
on  b.isbn = i.issued_book_isbn 
left join return_status r 
on r.issued_id = i.issued_id
where r.return_date is null and (CURRENT_DATE - i.issued_date) > 30;
```



**Task 14: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
create table branch_report
select b.branch_id, b.manager_id, count(i.issued_id), count(r.return_id), sum(bk.rental_price)
from issued_status i
join employees e 
on e.emp_id = i.issued_emp_id
join branch b
on e.branch_id = b.branch_id
left join return_status r
on i.issued_id = r.issued_id
join books bk
on i.issued_book_isbn = bk.isbn
group by 1,2;

SELECT * FROM branch_reports;
```

**Task 15: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

create table active_members
select * from members
where member_id in
(select distinct issued_member_id from issued_status
where issued_date > curdate() - interval 2 month);

SELECT * FROM active_members;

```


**Task 16: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
select e.emp_name, b.branch_id, count(i.issued_id) as books_processed
from issued_status i
join employees e
on e.emp_id = i.issued_emp_id
join branch b
on e.branch_id = b.branch_id
group by 1,2
order by 3 desc
limit 3;
```
-- other approach

```sql
with emp_stats as (
 select e.emp_name, b.branch_id, count(i.issued_id) as books_processed, rank() over (order by count(i.issued_id) desc) as rnk
from issued_status i
join employees e
on e.emp_id = i.issued_emp_id
join branch b
on e.branch_id = b.branch_id
group by 1,2
)
select e.emp_name, b.branch_id, count(i.issued_id) as books_processed
from emp_stats
where rnk <=3;
```


## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.





#3/11/2024 

---
# Manipulating Databases
## EXPLAIN:

```sql
/* ONLY TAKES COMMAND AS PARAMETER */
EXPLAIN (QUERY Statement) ; 

EXPLAIN SELECT * FROM gfgtable;
```

## CREATE:

```sql
CREATE Database db_name

CREATE SCHEMA schema_name

CREATE table_name (
	column_name [data_type] [constraints]
)

Example: 

CREATE table tutorial.employees (
id numeric primary key,
first_name varchar not null,
last_name varchar not null,
email varchar,
hire_date date default current_date,
department varchar default 'Unassigned'
);
```
 
## ALTER:

```sql
ALTER [Object] [Object_name] [Command];

Example: 
-- add New column 
ALTER table tutorial.employees add column age INT;
-- drop column
ALTER table tutorial.employees drop column age;
-- set default value
ALTER table tutorial.employees alter column department set default 'reassigned';
```
## INSERT: 

```sql
INSERT into [table_name] (column1,column2,...) values (value1,value2,...);

-- insert data into columns
INSERT into tutorial.employees (id,first_name,last_name,email) 
values (1,'Jhon','Doe','johndoe@example.com');
```
## UPDATE:

```sql
UPDATE table_name
	set column1 = value1,column2 = value2,.....
WHERE condition;

--Change column values after creation
UPDATE tutorial.employees
	set first_name ='jane',
	last_name = 'Doe',
	email = 'janedoe@example.com'
WHERE id = 2;
```
## DELETE:

```sql
DELETE from table_name WHERE condition;

-- delete a row
DELETE from tutorial.employees where id = 3;
--delete multiple rows 
DELETE from tutorial.employees where id in (1,4);
```
## TRUNCATE:

```sql
TRUNCATE table table_name;

TRUNCATE table tutorial.employees;
```
## MERGE:

```sql
-- Not supported in all Databases support it
MERGE into table1 as t1
using table2 as t2
on t1.id = t2.id
when matched then
	update set
		t1.title = t2.title
when not matched then
	insert(id,title)
	values(t2.id,t2.title);
-- Merge table1 and table2
MERGE INTO tutorial.employees AS e
USING tutorial.employees_two AS e2
ON e.id = e2.id
WHEN MATCHED THEN
    UPDATE SET 
        first_name = e2.first_name,
        last_name = e2.last_name,
        email = e2.email,
        hire_date = e2.hire_date,
        department = e2.department
WHEN NOT MATCHED THEN
    INSERT (id, first_name, last_name, email, hire_date, department)
    VALUES (e2.id, e2.first_name, e2.last_name, e2.email, e2.hire_date, e2.department);
```
## DROP:

```sql
DROP table table_name

-- Remove table
DROP table tutorial.employees
```

# Manipulating Date Time in SQL

## DateTime Intro:
```sql
-- Timestamp format (T is not necessary)
select timestamp '2023-04-01T14:30:00';

-- diffrenet date formats that are the same
select date '2023-04-01';
select date '04/01/2023';

select date 'April 1,2023';
select date '1 April 2023';

-- timestamp with time zone
select timestamp with time zone '2023-04-01 14:30:00+02' --UTC +2

-- current values
select current_date;
select current_time;
select current_timestamp;

-- Truncating dates & timestamps
select DATE_TRUNC('month', DATE '2023-04-15');
select DATE_TRUNC('year', DATE '2023-04-15');
select DATE_TRUNC('hour', TIMESTAMP '2023-04-15 14:30:00');

-- calculating age (difference between to dates)
-- Returns an interval
SELECT AGE(DATE '2023-04-01', DATE '2022-03-01');
SELECT AGE(TIMESTAMP '2023-04-01 14:30:00', TIMESTAMP '2022-03-01 12:00:00');
SELECT AGE(DATE '2022-03-01', CURRENT_DATE);
```

## Different Datetime Types:

```sql
-- Type casting (:: is used for casting )
select '2023-01-01'::date;
select '15:30:00'::time;
select '2023-01-01'::timestamp;

-- Returns an interval
select '15:30:00'::time - '12:30:00'::time;
```
## Timezones:

```sql
-- Selecting timezone
SELECT current_setting('timezone');

-- Creating table with timezone column 
CREATE table demo(
	tz_demo timestamptz);

-- Convert timezone
SELECT timezone('America/Denver', tz_demo)

-- Other Time Functions
SELECT 
	current_time,
	current_date,
	current_timestamp,
	now(), 
	timezone('UTC', now()) as now_timezone_convert,
	localtime,
	localtimestamp,
	TIMEOFDAY()
```
## Intervals:

```sql
-- Creating an interval 
SELECT INTERVAL '1 day 2 hours 30 minutes';

-- Add intervals
	SELECT NOW() + INTERVAL '1 day';
	
-- Subtract intervals
SELECT AGE(NOW(), '2022-03-18');

-- Extracting Date
SELECT EXTRACT('year' FROM NOW()),
	   date_part('year', NOW());
```
# Complex Data Types
## ENUM:

```sql
-- Create an Enum
CREATE TYPE weekday AS ENUM ('Monday', 'Tuesday', 'Wednesday', 
			'Thursday', 'Friday', 'Saturday', 'Sunday');

-- using ENUM as a datatype
create table enum_demo (
id serial primary key,
day_of_week weekday not null);

-- When using order by data is order by the order data was entered in the enum for example here the Monday will be the first when ordering Ascendingly

-- Adding a check constraint 
alter table enum_demo 
add column wage float check (wage >= 0),
add column another_day varchar check (another_day in ('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'));

-- ENUM is should be used when you want certain values and the order matter other than that just use CHECK constraint 
```

## ARRAY:

```sql
-- Creating an array Type
CREATE TABLE array_table (
  id SERIAL PRIMARY KEY,
  myarray INTEGER[] );

-- Inserting values into the array
insert into array_table (myarray)
values (array[1, 2, 3, 4]);

-- Query an array that has certain value METHOD 1
SELECT *
FROM array_table
WHERE 2 = ANY(myarray);

-- Query an array that has certain value METHOD 2
select *
from array_table
where array[9, 27, 43, 64]::integer[] = myarray;

-- Unesting (Unpivoting) array
select id, unnest(myarray)
from array_table ;
```
## RANGE:

```sql
-- Creating Range columns
CREATE TABLE job_board (
id SERIAL PRIMARY KEY,
job TEXT,
salary NUMERIC,
salary_range numrange,
salary_intrange int4range);

-- Inserting Values in Table
INSERT INTO job_board (job, salary, salary_range,salary_intrange)
VALUES 
('Engineer I', 120000, NUMRANGE(95000, 130000),INT4RANGE(95000, 130000)),
('Engineer II', 150000, NUMRANGE(135000, 170000),INT4RANGE(135000, 170000)),
('Engineer III', 210000, NUMRANGE(185000,250000),INT4RANGE(185000,250000));

-- Selecting rows including the range for INT4RANGE
SELECT * FROM job_board
where salary_intrange @> 95000;

-- Selecting rows including the range for NUMRANGE
-- It must be casted to numeric cuz numrange include decimals
SELECT * FROM job_board
where salary_range @> 95000::numeric;
```
## NESTED DATA:

```sql
-- Create JSON column 
CREATE table customers (
id SERIAL PRIMARY KEY,
name TEXT,
address JSONB);

-- Inserting values into JSONB
INSERT INTO customers (name,address)
VALUES ('jhon doe', '{"street": "123 Main st","city": "New York", "state": "NY", "zip": "10001"}');

-- Selecting values from JSONB
SELECT
	address->>'street' as street,
	address->>'city' as city,
	address->>'state' as state,
	address->>'zip' as zip
FROM customers
WHERE name = 'jhon doe';

-- Update a value in JSONB
UPDATE customers
SET address = jsonb_set(address,'{city}', '"Los Anglos"')
WHERE name = 'jhon doe';
```
# Advanced Query Techniques
## OVER:

```sql
-- Creating a running total (cumulative sum) column
SELECT order_id, customer_id,
SUM(order_total) OVER (ORDER BY order_date) AS cumulative_total
FROM orders;

-- Getting all values equal to max order total
with temp_table as (
SELECT order_id, customer_id,order_date, order_total,
MAX(order_total) OVER (PARTITION BY customer_id) as max_order_per_cust
FROM orders o )

select * from temp_table
where order_total = max_order_per_cust;

-- Another way to get all max values using row_number()
with temp_table as (
SELECT order_id, customer_id,order_date, order_total,
row_number() OVER (PARTITION BY customer_id order by order_total desc) as max_order_per_cust_alt
FROM orders o )

select * from temp_table
where max_order_per_cust = 1;
```
## CROSS JOIN:

```sql
select
	student_id,
	advisor,
	building_name
from
	class_unnormalized
cross join buildings b;
```
## LATERAL JOIN :

```sql
-- Using lateral join
select
	u.username,
	o.order_id,
	o.order_date
from
	users u
left join lateral (
	select
		order_id,
		order_date
	from
		orders
	where
		user_id = u.user_id
	order by
		order_date desc
	
	limit 1
) o on true;
```
## CROSS JOIN LATERAL :

```sql
-- using cross join lateral 
-- used for unpivoting data
-- some dbms has unique unpivoting operators using them will be better
select c.student_id, t.*
from class_unnormalized c
cross join lateral (
	values
	(c.class1, 'class1'),
	(c.class2, 'class2'),
	(c.class3, 'class3')
) as t (subject, class_num)
order by student_id;
```
## COALESCE:

```sql
-- Coalesce gets the first Non-Null Value
select id, name,
coalesce(salary, department, bonus) as tru_salary
from employees;
```
## CASE:

```sql
-- Case statment syntax 
-- Order of cases matters
select
	name,
	salary,
	case
		when salary < 60000 then 'entry level'
		when salary < 100000 then 'mid'
		when salary < 200000 then 'big baller'
		when salary < 200000 or bonus > 0 then 'really big baller'
		else 'uncaught exception'
	end as salary_classification
from
	employees ;
```
## CONCAT:

```sql
-- Concat Syntax
select
	f_name,
	l_name,
	concat(f_name, ' ', l_name) as full_name
from
	employees;
```
## RECURSIVE CTE:

```sql
-- using recursive cte to create date column only using a start and end values
with recursive date_table as (
select
	'2023-01-01': :date da_date union all select da_date + 1
from date_ table
where da date < '2023-02-01':: date
)

select *
from date_table;

-- using recursive cte to create a hierarchy in table
WITH recursive managers AS
(
	SELECT '' AS hierarchy_lvl,
	employee_id,
	manager_id,
	title AS employee_title
	FROM employees
	WHERE title = 'The Boss'
	UNION ALL
	SELECT hierarchy_lvl || '-',
	e. employee_id,
	e.manager_id,
	e.title
	FROM employees e
	JOIN managers m
	ON e.manager_id = m.employee_id )

SELECT 
	hierarchy_lvl || employee_title AS title,
	employee_id,
	manager_id
FROM managers

```
# Database Normalization
## DATA NORMALIZATION:

```sql
-- 1st Normal Form is all about unpivoting the data ( Romve multivalue attributes)
-- 2nd Normal Form is all about having no partial dependency
-- 3rd Normal Form is all about having no transitive partial dependency
```

# Performance & Control
## STORED PROCEDURES & UDFs:

```sql
-- creating a procedure
-- It is better to do all the table maniplation in a stored procedure rather that a UDF
CREATE OR REPLACE PROCEDURE insert_employee (
-- Input parameters
p_first_name VARCHAR, p_last_name VARCHAR, p_department_id INTEGER, p_salary NUMERIC
)
-- Specify the language 
-- plpgsql: procedural language postgreSQL
LANGUAGE plpgsql
AS $$
BEGIN
INSERT INTO employees (first_name, last_name, department_id, salary)
VALUES (p_first_name, p_last_name, p_department_id, p_salary);
END;
$$;

select * from employees;
-- Calling the procedure
call insert_employee('David', 'Martinez', 3, 155000);


-- Creating a UDF: user defined function
CREATE OR REPLACE FUNCTION average_salary (p_department_id INTEGER)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
DECLARE
v_avg_salary NUMERIC;
BEGIN
	SELECT AVG(salary) INTO V_avg_salary
	FROM employees
	WHERE department_id = _department_id;
	RETURN v_avg_salary;
END;
$$;

-- Using the UDF
SELECT average_salary(1);
```
## TEMP TABLES:

```sql
-- Temp tables are delete when the user session is done 
-- Creating a temp table
CREATE temporary TABLE employees (
	id SERIAL PRIMARY KEY,
	f_name VARCHAR (255),
	l_name VARCHAR(255),
	salary varchar,
	department VARCHAR (255),
	bonus varchar
);
```
## MATERIALIZED VIEW:

```sql
-- Creating a Materialized View
CREATE MATERIALIZED VIEW customer_orders AS
SELECT
	customers.customer_id,
	customers.first_name,
	customers.last_name,
	orders.order_id,
	orders.order_date,
	orders.total_amount
FROM
	customers
JOIN orders ON customers.customer_id = orders.customer_id;

-- Refreshing the Materialized View
-- Used after making any change in the original tables to include it in the Materialized View
Refresh materialized view customer_orders;
```
## TRANSACTIONS:

```sql 
-- Controlling the transactions manually
BEGIN TRANSACTION;

-- Insert a new customer
INSERT INTO customers (name, email) VALUES ('John Doe', 'john.doe@example.com');

-- Create a savepoint
SAVEPOINT sp1;

-- Insert a new order
INSERT INTO orders (customer_id, product_id, quantity) VALUES (1, 101, 5);

-- If there's an issue, rollback to sp1
ROLLBACK TO sp1;

-- Continue with the transaction
UPDATE customers SET email = 'johndoe@example.com' WHERE name = 'John Doe';

-- you can only rollback before commiting 
COMMIT;

-- End Transaction automatically commit  
BEGIN TRANSACTION;
```

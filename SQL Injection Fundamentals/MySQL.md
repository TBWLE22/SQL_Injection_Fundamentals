## Intro to MySQL
### Structured Query Language (SQL)
SQL syntax can differ from one RDBMS to another. However, they are all required to follow the [ISO standard](https://en.wikipedia.org/wiki/ISO/IEC_9075) for Structured Query Language.
- SQL can be used to perform the following actions:
	- Retrieve data
	- Update data
	- Delete data
	- Create new tables and databases
	- Add / remove users
	- Assign permissions to these users
### Command Line
- **`mysql` Utility:**
    - Used to **authenticate and interact** with MySQL or MariaDB databases.
- **Authentication Flags:**
    - `-u` flag: Specifies the **username**.
        
    - `-p` flag: Prompts for the **password**.
- **Security Tip:**
    - Leave the `-p` flag **empty** (do not enter the password on the command line).
        
    - This prevents the password from being stored in **cleartext** in files like `bash_history`.
````shell-session
Whog@htb[/htb]$ mysql -u root -p
````

- It is also possible to use the password directly in the command, though this should be avoided, as it could lead to the password being kept in logs and terminal history:
````shell-session
Whog@htb[/htb]$ mysql -u root -p<password>
````

> [!Tip]  There shouldn't be any spaces between '-p' and the password.
- **Superuser Access:**
    - The above example is for logging in as **`root`** , which provides full privileges to execute all commands.
        
    - Other users may have **limited privileges** based on their role.
- **Viewing Privileges:**
    - Use the **`SHOW GRANTS`** command to view the privileges of the current user (discussed later).
- **Host and Port Options:**
    - If no host is specified, MySQL connects to **`localhost`** by default.
        
    - Use the **`-h`** flag to specify a **remote host**.
        
    - Use the **`-P`** flag to specify a **custom port**.

>[!Note] The default MySQL/MariaDB port is (3306), but it can be configured to another port. It is specified using an uppercase `P`, unlike the lowercase `p` used for passwords.

### Creating a database
- **Starting SQL Interaction:**
    
    - After logging in with the `mysql` utility, you can begin executing **SQL queries**.
- **Creating a Database:**
    
    - Use the **`CREATE DATABASE`** statement to create a **new database** within MySQL.

```mysql
mysql> CREATE DATABASE users;

Query OK, 1 row affected (0.02 sec)
```

- **Query Termination:**
    
    - MySQL **requires a semicolon (`;`)** at the end of each command.
        
- **Viewing Databases:**
    
    - Use the **`SHOW DATABASES;`** command to list all available databases.
        
- **Switching Databases:**
    
    - Use the **`USE users;`** command to switch to the `users` database.
```mysql
mysql> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| users              |
+--------------------+

mysql> USE users;

Database changed
```

>[!Note] SQL statements aren't case sensitive, which means 'USE users;' and 'use users;' refer to the same command. However, the database name is case sensitive, so we cannot do 'USE USERS;' instead of 'USE users;'. So, it is a good practice to specify statements in uppercase to avoid confusion.

### Tables
- **Tables in DBMS:**
    
    - Data is stored in **tables** made up of **rows** (horizontal) and **columns** (vertical).
        
    - The intersection of a row and column is called a **cell**.
- **Table Structure:**
    
    - Each table has a **fixed set of columns**.
        
    - Each column has a defined **data type**.
- **Data Types:**
    
    - Define the type of values a column can hold (e.g., **numbers**, **strings**, **dates**, **binary data**).
        
    - MySQL supports **various data types**, including some **DBMS-specific** types.
```mysql
mysql> CREATE TABLE logins (
    ->     id INT,
    ->     username VARCHAR(100),
    ->     password VARCHAR(100),
    ->     date_of_joining DATETIME
    ->     );
Query OK, 0 rows affected (0.03 sec)
```
-  A table named **`logins`** is created with **four columns**.
        
- **Column Details:**
    
    - **`id`**: An **integer** used to uniquely identify each row.
        
    - **`username`** and **`password`**: **Strings** (VARCHAR) with a maximum length of **100 characters**.
        
        - Inputs exceeding this length will cause an **error**.
            
    - **`date_of_joining`**: A **DATETIME** column that stores the **date and time** an entry was added.
- **List All Tables in a Database:**
    
    - Use the **`SHOW TABLES;`** statement to display all tables in the current database.
- **View Table Structure:**
    
    - Use the **`DESCRIBE table_name;`** command to see:
        
        - Column names
            
        - Data types
            
        - Nullability
            
        - Keys
            
        - Default values (if any)
```mysql
mysql> DESCRIBE logins;

+-----------------+--------------+
| Field           | Type         |
+-----------------+--------------+
| id              | int          |
| username        | varchar(100) |
| password        | varchar(100) |
| date_of_joining | date         |
+-----------------+--------------+
4 rows in set (0.00 sec)
```

### Table Properties
- The `CREATE TABLE` query allows setting **properties** for the table and its columns.
    
- One common property is **`AUTO_INCREMENT`**, used with integer columns like `id`.
    
- `AUTO_INCREMENT` automatically **increments the value by 1** whenever a new row is added.
    
- This is commonly used to **generate unique IDs** for each record in the table.

```sql
  id INT NOT NULL AUTO_INCREMENT,
```

- **`NOT NULL`**: Ensures that a column **cannot have NULL values**, meaning the field is **required** during insertion.
    
- **`UNIQUE`**: Ensures that **all values in the column are different**, preventing **duplicate entries**.

```sql
username VARCHAR(100) UNIQUE NOT NULL,
```

- The **`DEFAULT`** keyword sets a **default value** for a column if no value is provided during insertion.
    
- For example, using `DEFAULT NOW()` in a `DATETIME` column sets the **current date and time** automatically when a new record is added.

```sql
date_of_joining DATETIME DEFAULT NOW(),
```

- **`PRIMARY KEY`** uniquely identifies each record in a table.
    
- It **ensures uniqueness** and **non-nullability** for the column(s) assigned as the primary key.
    
- Typically, the **`id`** column is set as the **primary key** in many tables.
    
- This key is used to **refer to records** in relational databases and link tables together.
```sql
PRIMARY KEY (id)
```

- The final CREATE TABLE query will be as follows:
```sql
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
    );

```

---
## SQL Statements
### INSERT Statement
- The **INSERT** statement is used to add new records to a given table. The statement following the below syntax:
	
```sql
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
```
	
- The syntax above requires the user to fill in values for all the columns present in the table.
	
```sql
mysql> INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');
Query OK, 1 row affected (0.00 sec) 
--- The query above shows how to add a new login to the logins table, with appropriate values for each column. 
```
	
- We can skip filling columns with default values, such as `id` and `date_of_joining`.
	
- This can be done by specifying the column names to insert values into a table selectively:
	
```sql
INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);
```
	
>[!Note] Skipping columns with the 'NOT NULL' constraint will result in an error, as it is a required value.
	
```sql
mysql> INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');
Query OK, 1 row affected (0.00 sec)
```
	
- We inserted a username-password pair in the example above while skipping the `id` and `date_of_joining` columns.
	
>[!Callout] The examples insert cleartext passwords into the table, for demonstration only. This is a bad practice, as passwords should always be hashed/encrypted before storage.
	
- We can also insert multiple records at once by separating them with a comma:
	
```sql
mysql> INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0
-- The query above inserted two new records at once.
```
### SELECT Statement
- We use **SELECT** statement to retrieve data  from tables. The general syntax to view the entire table is as follows:
	
```sql
SELECT * FROM table_name;
```
	
- The asterisk symbol (\*) acts as a wildcard and selects all the columns. The **FROM** keyword is used to denote the table to select from. 
- It is possible to view data present in specific columns as well:
	
```sql
SELECT column1, column2 FROM table_name;
```
	
- The query above will select data present in column1 and column2 only.
	

```sql
mysql> SELECT * FROM logins; -- This query looks at all records present in the logins table. 

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec) -- We can see the four records which were entered before. 


mysql> SELECT username,password FROM logins; -- This query selects just the username and password columns while skipping the other two. 

+---------------+------------+
| username      | password   |
+---------------+------------+
| admin         | p@ssw0rd   |
| administrator | adm1n_p@ss |
| john          | john123!   |
| tom           | tom123!    |
+---------------+------------+
4 rows in set (0.00 sec)
```
### DROP Statement
- We can use **DROP** to remove tables and databases from the server.
	
```sql
mysql> DROP TABLE logins;

Query OK, 0 rows affected (0.01 sec)

mysql> SHOW TABLES;

Empty set (0.00 sec)
```
	
- As we can see, the table was removed entirely.
	
>[!Note] The 'DROP' statement will permanently and completely delete the table with no confirmation, so it should be used with caution.
### ALTER Statement
- We can use **ALTER** to change the name of any table and any of its fields or to delete or add a new column to an existing table
	
```sql
mysql> ALTER TABLE logins ADD newColumn INT; -- Adds a new column newColumn to the logins table using ADD
Query OK, 0 rows affected (0.01 sec)
```
	
- To rename a column, we can use **RENAME COLUMN**:
```sql
mysql> ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;
Query OK, 0 rows affected (0.01 sec)
```
	
- We can also change a column's datatype with **MODIFY**:
```sql
mysql> ALTER TABLE logins MODIFY newerColumn DATE;
Query OK, 0 rows affected (0.01 sec)
```
	
- We can drop a column using **DROP**:
```sql
mysql> ALTER TABLE logins DROP newerColumn;
Query OK, 0 rows affected (0.01 sec)
```
	
- We can use any of the above statements with any existing table, as long as we have enough privileges to do so.
### UPDATE Statement
- **UPDATE** statement can be used to update specific records within a table, based on certain conditions. Its general syntax is:
```sql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
```
	
- We specify the table name, each column and its new value, and the condition for updating records. 
```sql
mysql> UPDATE logins SET password = 'change_password' WHERE id > 1;

Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0


mysql> SELECT * FROM logins;

+----+---------------+-----------------+---------------------+
| id | username      | password        | date_of_joining     |
+----+---------------+-----------------+---------------------+
|  1 | admin         | p@ssw0rd        | 2020-07-02 00:00:00 |
|  2 | administrator | change_password | 2020-07-02 11:30:50 |
|  3 | john          | change_password | 2020-07-02 11:47:16 |
|  4 | tom           | change_password | 2020-07-02 11:47:16 |
+----+---------------+-----------------+---------------------+
4 rows in set (0.00 sec)
--- The query above updated all passwords in all records where the id was more significant than 1.
```
	
>[!note] We have to specify the 'WHERE' clause with UPDATE, in order to specify which records get updated. The 'WHERE' clause will be discussed next.

---
## Query Results
### Sorting Results
- We can sort the results of any query using **ORDER BY** and specifying the column to sort by:
```sql
mysql> SELECT * FROM logins ORDER BY password;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec)
```
	
- By default, the sort is done in ascending order, but we can also sort the results by **ASC** or **DESC**.
```sql
mysql> SELECT * FROM logins ORDER BY password DESC;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec)
```
	
- It is also possible to sort by multiple columns, to have a secondary sort for duplicate values in one column:
```sql
mysql> SELECT * FROM logins ORDER BY password DESC, id ASC;

+----+---------------+-----------------+---------------------+
| id | username      | password        | date_of_joining     |
+----+---------------+-----------------+---------------------+
|  1 | admin         | p@ssw0rd        | 2020-07-02 00:00:00 |
|  2 | administrator | change_password | 2020-07-02 11:30:50 |
|  3 | john          | change_password | 2020-07-02 11:47:16 |
|  4 | tom           | change_password | 2020-07-02 11:50:20 |
+----+---------------+-----------------+---------------------+
4 rows in set (0.00 sec)
```
### LIMIT results
- In case our query returns a large number of records, we can LIMIT the results to what we want only.
```sql
mysql> SELECT * FROM logins LIMIT 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```
	
- If we wanted to LIMIT results with an offset, we could specify the offset before the LIMIT count:
```sql
mysql> SELECT * FROM logins LIMIT 1, 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```
	
>[!Note] The offset marks the order of the first record to be included, starting from 0. For the above, it starts and includes the 2nd record, and returns two values.
### WHERE Clause
- To filter or search for specific data, we can use conditions with the **SELECT** statement using the **WHERE** clause, to fine-tune the results:
```sql
SELECT * FROM table_name WHERE <condition>; --Returns all records which satisfy the given condition
```
	
```sql
mysql> SELECT * FROM logins WHERE id > 1; -- Select all records where the value of id is greater than 1.

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
3 rows in set (0.00 sec)
--  As we can see, the first row with its id as 1 was skipped from the output.
```
	
```sql
mysql> SELECT * FROM logins where username = 'admin'; -- Selects the record where the username is admin

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  1 | admin    | p@ssw0rd | 2020-07-02 00:00:00 |
+----+----------+----------+---------------------+
1 row in set (0.00 sec)
```
- Similarly, We can use the **UPDATE** statement to update certain records that meet a specific condition.
	
>[!Note] String and date data types should be surrounded by single quote (') or double quotes ("), while numbers can be used directly.
### LIKE Clause
- SQL clause **LIKE**, enables selecting records by matching a certain pattern.
	
```sql
mysql> SELECT * FROM logins WHERE username LIKE 'admin%'; --  Retrieves all records with usernames starting with admin.

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | administrator | adm1n_p@ss | 2020-07-02 15:19:02 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```
	
- The **%** symbol acts as a wildcard and matches all characters after **admin**. It is used to match zero or more characters. 
	
- Similarly, the _ symbol is used to match exactly one character. 
	
```sql
mysql> SELECT * FROM logins WHERE username like '___'; -- Matches all usernames with exactly three characters in them.

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  3 | tom      | tom123!  | 2020-07-02 15:18:56 |
+----+----------+----------+---------------------+
1 row in set (0.01 sec)
```
---
## SQL Operators
- SQL also supports **Logical Operators** to use multiple conditions at once. The most common logical operators are **AND, OR** and **NOT.**
### AND Operator
- The AND operator takes in two conditions and returns true or false based on their evaluation.
```sql
condition1 AND condition2
```
	
- The result of the **AND** operation is true if and only if both **condition1** and **condition2** evaluate to true.
```sql
mysql> SELECT 1 = 1 AND 'test' = 'test';
-- The first query returned true as both expressions were evaluated as true
+---------------------------+
| 1 = 1 AND 'test' = 'test' |
+---------------------------+
|                         1 | 
+---------------------------+
1 row in set (0.00 sec) 

mysql> SELECT 1 = 1 AND 'test' = 'abc';
-- The second query returned false as the second condition 'test' = 'abc' is false.
+--------------------------+
| 1 = 1 AND 'test' = 'abc' |
+--------------------------+
|                        0 |
+--------------------------+
1 row in set (0.00 sec)
```
	
- In MySQL terms:
	- Any **non-zero** value is considered **true**, and it usually returns the value **1** to signify **true**. 
	- **0** is considered **false**. 
### OR Operator
- The **OR** operator takes in two expressions as well, and returns **true** when at least one of them evaluates to **true**.
```sql
mysql> SELECT 1 = 1 OR 'test' = 'abc';
-- The first query evaluated to true as the condition 1 = 1 is true. 
+-------------------------+
| 1 = 1 OR 'test' = 'abc' |
+-------------------------+
|                       1 |
+-------------------------+
1 row in set (0.00 sec)

mysql> SELECT 1 = 2 OR 'test' = 'abc';
-- The second query has two false conditions, resulting in false output.
+-------------------------+
| 1 = 2 OR 'test' = 'abc' |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.00 sec)
```
### NOT Operator
- The **NOT** operator simply toggles a boolean value 'i.e. **true** is converted to **false** and vice versa.
```sql
mysql> SELECT NOT 1 = 1;
-- The first query resulted in false because it is the inverse of the evaluation of 1 = 1, which is true, so its inverse is false.
+-----------+
| NOT 1 = 1 |
+-----------+
|         0 |
+-----------+
1 row in set (0.00 sec)

mysql> SELECT NOT 1 = 2;
-- The second query returned true, as the inverse of 1 = 2 'which is false' is true.
+-----------+
| NOT 1 = 2 |
+-----------+
|         1 |
+-----------+
1 row in set (0.00 sec)
```
### Symbol Operators
- The **AND, OR** and **NOT** operators can also be represented as **&&, ||** and **!**, respectively. 
```sql
mysql> SELECT 1 = 1 && 'test' = 'abc';

+-------------------------+
| 1 = 1 && 'test' = 'abc' |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SELECT 1 = 1 || 'test' = 'abc';

+-------------------------+
| 1 = 1 || 'test' = 'abc' |
+-------------------------+
|                       1 |
+-------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SELECT 1 != 1;

+--------+
| 1 != 1 |
+--------+
|      0 |
+--------+
1 row in set (0.00 sec)
```
### Operators in queries
- Let us look at how these operators can be used in queries. 
```sql
mysql> SELECT * FROM logins WHERE username != 'john';
-- The query lists all records where the username is NOT john
+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
3 rows in set (0.00 sec)
```
	
```sql
mysql> SELECT * FROM logins WHERE username != 'john' AND id > 1;
-- The query selects users who have their id greater than 1 AND username NOT equal to john.
+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```

### Multiple Operator Precedence
- SQL supports various other operations such as addition, division as well as bitwise operations. 
	
- Thus, a query could have multiple expressions with multiple operations at once. 
	
- The order of these operations is decided through operator precedence.
	
- Here is a list of common operations and their precedence, as seen in the MariaDB Documentation:
	- Division (**/**), Multiplication (**\***), and Modulus (**%**)
    - Addition (**+**) and subtraction (**-**)
    - Comparison (**=, >, <, <=, >=, !=, LIKE**)
    - NOT (**!**)
    - AND (**&&**)
    - OR (**||**)
- Operations at the top are evaluated before the ones at the bottom of the list.
	
```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
```
	
- The query has four operations: **!=, AND, >**, and **-**. From the operator precedence, we know that subtraction comes first, so it will first evaluate **3 - 2** to **1**.
	
```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 1;
```
	
- We have two comparison operations, **>** and **!=**.
	
- Both of these are of the same precedence and will be evaluated together.
	
- So, it will return all records where username is not **tom**, and all records where the **id** is greater than 1, and then apply **AND** to return all records with both of these conditions.
	
```sql
mysql> select * from logins where username != 'tom' AND id > 3 - 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-03 12:03:53 |
|  3 | john          | john123!   | 2020-07-03 12:03:57 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```
---
---

## Subverting Query Logic
### Authentication Bypass
- Consider the following administrator login page.
	- 
![[Pasted image 20250522112406.png]]
	-
- We can log in with the administrator credentials `admin / p@ssw0rd`.
	
![[Pasted image 20250522112440.png]]
	
- The page also displays the SQL query being executed to understand better how we will subvert the query logic.
	
- Our goal is to log in as the admin user without using the existing password.
	
```sql
SELECT * FROM logins WHERE username='admin' AND password = 'p@ssw0rd';
```
	
- The login page accepts **username and password** inputs.
    
- It uses an **SQL query with the `AND` operator** to find matching records.
    
- If the database **returns a matching record**, the login condition evaluates to **true**.
    
- On success, the **admin record is returned**, and login is validated.
    
- If **incorrect credentials** are entered, the **query returns no record**, and the login fails.
	
![[Pasted image 20250522112833.png]]
	
-  The login failed due to the wrong password leading to a `false` result from the `AND` operation.
### SQLi Discovery
- Before attempting authentication bypass, we must **check if the login form is vulnerable to SQL injection**.
    
- This is done by **injecting test payloads** into the username field.
    
- The goal is to observe if the **page behavior changes** or any **SQL errors appear**.
    
- Common test payloads include:
    
    - `' OR '1'='1`
        
    - `admin' --`
        
    - `admin' #`
        
    - `' OR 1=1 --`
        
- If the page responds differently or logs in without a valid password, it suggests **SQL injection vulnerability**.
	
| Payload | URL Encoded |
| ------- | ----------- |
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |
	
>[!Note] In some cases, we may have to use the URL encoded version of the payload. An example of this is when we put our payload directly in the URL 'i.e. HTTP GET request'.
	
- Let's start by injecting a single quote.
	
![[Pasted image 20250522113504.png]]
	
- We see that a SQL error was thrown instead of the `Login Failed` message.
	
```sql
-- The resulting query is as follows.
SELECT * FROM logins WHERE username=''' AND password = 'something';
```
	
- Entering a **single quote (`'`)** can cause a **syntax error** due to an **odd number of quotes** in the SQL query.
    
- To fix this and make the injected query valid, there are two common approaches:
    
    - **Comment out** the rest of the original SQL query using comment syntax like `--`, `#`, or `/* */`.
        
    - **Balance the quotes** by using an **even number of quotes** in the input so that the final SQL statement remains syntactically correct.
        
- These techniques help attackers **inject SQL code successfully** .
### OR Injection
- To **bypass authentication**, we need the SQL query to **always return true**.
    
- This can be achieved by abusing the **`OR` operator** in SQL injection.
    
- According to **MySQL operator precedence**, `AND` is evaluated **before** `OR`.
    
- Therefore, if **any condition connected with `OR` is true**, the **entire query** evaluates to true.
    
- A common always-true condition used in SQL injection is: `'1'='1'`.
    
- To avoid syntax errors from unmatched quotes:
    
    - Inject `'1'='1` (with a missing ending quote).
        
    - The closing quote from the original SQL query balances the input.
        
- This causes the SQL query to return **true**, **bypassing login checks**.
	
- So, if we inject the below condition and have an `OR` operator between it and the original condition, it should always return `true`.
	
```sql
admin' or '1'='1
```
	
```sql
-- The final query is as follows.
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```
	
- This means the following:
	- If username is `admin`  
	    `OR`
	- If `1=1` return `true` 'which always returns `true`'  
	    `AND`
	- If password is `something`
![[Pasted image 20250522114309.png]]
	
- The **`AND` operator** is evaluated **first** due to SQL operator precedence.
    
- If the `AND` condition fails, it returns **false**.
    
- Then, the **`OR` operator** is evaluated.
    
- If **either side** of the `OR` is **true**, the entire condition becomes **true**.
    
- The injected condition `'1'='1'` is **always true**.
    
- As a result, the overall SQL query evaluates to **true**, even with invalid credentials.
    
- This allows the attacker to **bypass authentication** and **gain access**.
### Authentication Bypass with OR operator
- Let us try this as the username and see the response.
	
![[Pasted image 20250522114529.png]]
	 -
- We were able to log in successfully as admin. However, what if we did not know a valid username?
	
![[Pasted image 20250522114617.png]]
	
- The login failed because `notAdmin` does not exist in the table and resulted in a false query overall.
	
![[Pasted image 20250522114705.png]]
	
- To successfully log in once again, we will need an overall `true` query.
	
- This can be achieved by injecting an `OR` condition into the password field, so it will always return `true`. 
	
- Let us try `something' or '1'='1` as the password.
	
![[Pasted image 20250522114859.png]]
	
- Adding an `OR` condition like `' OR '1'='1` results in an **always true** SQL query.
    
- This makes the **entire `WHERE` clause** evaluate to true, returning **all records**.
    
- The **first record** (often the admin) is typically used for login.
    
- There's **no need for valid credentials**; the injection alone is enough.
	
![[Pasted image 20250522115119.png]]
	
- This works since the query evaluate to `true` irrespective of the username or password.

---
## Using Comments
### Comments
- SQL, like other languages, supports the use of comments.
    
- Comments help in documenting SQL queries or ignoring parts of them during execution.
    
- **Types of comments in MySQL:**
    
    - `--` (double dash) for line comments.
        
    - `#` (hash) for line comments.
        
    - `/* ... */` for inline or block comments (less common in SQL injection).
        
- The `--` style is commonly used in SQL injections to comment out the remaining part of a query. For example:
	
```mysql
mysql> SELECT username FROM logins; -- Selects usernames from the logins table 

+---------------+
| username      |
+---------------+
| admin         |
| administrator |
| john          |
| tom           |
+---------------+
4 rows in set (0.00 sec)
```
	
> [!Note]  In SQL, using two dashes only is not enough to start a comment. So, there has to be an empty space after them, so the comment starts with (-- ), with a space at the end. This is sometimes URL encoded as (--+), as spaces in URLs are encoded as (+). To make it clear, we will add another (-) at the end (-- -), to show the use of a space character.
	
- The `#` symbol can be used as well.
	
```mysql
mysql> SELECT * FROM logins WHERE username = 'admin'; # You can place anything here AND password = 'something'

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  1 | admin    | p@ssw0rd | 2020-07-02 00:00:00 |
+----+----------+----------+---------------------+
1 row in set (0.00 sec)
```
	
>[!Tip] If we are entering our payload in the URL within a browser, a (#) symbol is usually considered as a tag, and will not be passed as part of the URL. In order to use (#) as a comment within a browser, we can use '%23', which is an URL encoded (#) symbol.
### Auth Bypass with comments
- Looking back to the previous example, let's inject **admin'--** as username.
	
```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```
	
- As we can see from the syntax highlighting, the username is now `admin`, and the remainder of the query is now ignored as a comment.
	
- This way, we can ensure that the query does not have any syntax issues.
	
- Let us try using these on the login page, and log in with the username `admin'--` and anything as the password.
	
![[Pasted image 20250522120128.png]]
	
- Thus, we were able to bypass the authentication, as the new modified query checks for the username, with no other conditions.
### Another Example
- **SQL supports parentheses** to control the order of condition evaluation.
    
- **Expressions inside parentheses** are evaluated **before** other parts of the SQL query.
    
- This is useful when the application needs to **enforce precedence** among logical conditions.
    
- Helps ensure that **complex logical operations** are executed correctly. Consider the following:
	
![Admin panel showing an SQL query execution: SELECT * FROM logins WHERE (username='admin' AND id > 1) AND password='437b930db84b8079c2dd804a71936b5f'; with a message: Login failed!](https://academy.hackthebox.com/storage/modules/33/paranthesis_fail.png)
	
- The query includes a condition to ensure the **user's id is greater than 1**, effectively **blocking admin (usually id=1) login**.
    
- The **password is hashed before being used** in the query.
    
- Hashing the password **prevents SQL injection via the password field**, since the input is transformed before the query executes.
	
- Let us try logging in with valid credentials `admin / p@ssw0rd` to see the response.
	
![Admin panel showing an SQL query execution: SELECT * FROM logins WHERE (username='admin' AND id > 1) AND password='0f359740bd1cda994f8b55330c86d845'; with a message: Login failed!](https://academy.hackthebox.com/storage/modules/33/paranthesis_valid_fail.png)
	
- The login failed even though we supplied valid credentials because the admin’s ID equals 1. So let us try logging in with the credentials of another user, such as `tom`.
	
![Admin panel showing an SQL query execution: SELECT * FROM logins WHERE (username='tom' AND id > 1) AND password='f86a3c565937e6315864d1a43c48e7'; with a message: Login successful as user: tom"](https://academy.hackthebox.com/storage/modules/33/tom_login.png)
	
- Logging in as the user with an id not equal to 1 was successful.
	
- So, how can we log in as the admin? We know from the previous section on comments that we can use them to comment out the rest of the query.
	
- So, let us try using `admin'--` as the username.
	
![Admin panel showing an SQL query execution: SELECT * FROM logins WHERE (username='admin'--' AND id > 1) AND password='437b930db84b8079c2dd804a71936b5f'; with an error message: You have an error in your SQL syntax; check the manual for the right syntax near '437b930db84b8079c2dd804a71936b5f' at line 1](https://academy.hackthebox.com/storage/modules/33/paranthesis_error.png)
	
- The login failed due to a syntax error, as a closed one did not balance the open parenthesis.
	
- To execute the query successfully, we will have to add a closing parenthesis.
	
- Let us try using the username `admin')--` to close and comment out the rest.
	
![Admin panel showing an SQL query execution: SELECT * FROM logins WHERE (username='admin'--' AND id > 1) AND password='437b930db84b8079c2dd804a71936b5f'; with a message: Login successful as user: admin"](https://academy.hackthebox.com/storage/modules/33/paranthesis_success.png)
	
- The query was successful, and we logged in as admin. The final query as a result of our input is.
	
```sql
SELECT * FROM logins where (username='admin')
```
---
## Union Clause
 - Another type of SQL injection is injecting entire SQL queries executed along with the original query.
### Union
- **SQL UNION Clause**:
    
    - Used to **combine results** from multiple `SELECT` statements.
        
    - Helps in **retrieving data from different tables or even databases** in one query.
        
- **UNION Injection**:
    
    - A type of **SQL Injection** where the attacker uses the `UNION` keyword to inject a second `SELECT` query.
        
    - Enables **exfiltration of data from other tables** not originally intended to be accessed by the application.
- Let us try using the **UNION** operator in a sample database and try viewing the content of the ports table.
	
```mysql
mysql> SELECT * FROM ports;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| ZZ-21    | Shenzhen  |
+----------+-----------+
3 rows in set (0.00 sec)
```
	
```mysql
mysql> SELECT * FROM ships; -- Viewing the output of the ships tables

+----------+-----------+
| Ship     | city      |
+----------+-----------+
| Morrison | New York  |
+----------+-----------+
1 rows in set (0.00 sec)
```
	-
```mysql 
mysql> SELECT * FROM ports UNION SELECT * FROM ships; -- Using UNION to combine both results.

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| Morrison | New York  |
| ZZ-21    | Shenzhen  |
+----------+-----------+
4 rows in set (0.00 sec) 
-- As shown above, some of the rows belong to the `ports` table while others belong to the `ships` table.
```
	
- `UNION` combined the output of both `SELECT` statements into one, so entries from the `ports` table and the `ships` table were combined into a single output with four rows.
	
>[!Note]  The data types of the selected columns on all positions should be the same.
### Even Columns
- **UNION Clause Requirement**:
    
    - The `UNION` operator can **only combine results** from `SELECT` statements that have the **same number of columns**.
        
- **Mismatch Error**:
    
    - If two `SELECT` queries joined by `UNION` **do not return the same number of columns**, an **SQL error** occurs (usually indicating a column count mismatch).
        
- **Implication in UNION Injection**:
    
    - During a UNION-based SQL injection attack, attackers must **guess or determine the correct number of columns** in the original query to successfully append their own query.
    
```mysql
mysql> SELECT city FROM ports UNION SELECT * FROM ships;
ERROR 1222 (21000): The used SELECT statements have a different number of columns
```
	
- The above query results in an error, as the first `SELECT` returns one column and the second `SELECT` returns two.
	
- Once we have two queries that return the same number of columns, we can use the `UNION` operator to extract data from other tables and databases.
	
```sql
SELECT * FROM products WHERE product_id = 'user_input'
```
	
- We can inject a `UNION` query into the input, such that rows from another table are returned.
	
```sql
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '
```
	
- The above query would return `username` and `password` entries from the `passwords` table, assuming the `products` table has two columns.
### Un-even Columns
- When using `UNION`, the number of columns in both queries must **match**.
    
- If the original query returns **n columns**, our injected `SELECT` must also return **n columns**.
    
- To achieve this, we can fill the remaining columns with **dummy values** like:
    
    - Strings: `'junk'`
        
    - Numbers: `1`, `2`, etc.
        
- For example, if the original query has **3 columns**, and we only want to extract one (e.g., a password), we can do:
```sql
' UNION SELECT password, 'junk', 'junk' FROM users--
```
    
- This technique ensures that:
    
    - The syntax is correct
        
    - The injection returns useful data alongside placeholder junk values.
    
>[!Note] When filling other columns with junk data, we must ensure that the data type matches the columns data type, otherwise the query will return an error. For the sake of simplicity, we will use numbers as our junk data, which will also become handy for tracking our payloads positions, as we will discuss later.
---
	
>[!Tip ] For advanced SQL injection, we may want to simply use 'NULL' to fill other columns, as 'NULL' fits all data types.
	
- The `products` table has two columns in the above example, so we have to `UNION` with two columns.
	
- If we only wanted to get one column 'e.g. `username`', we have to do `username, 2`, such that we have the same number of columns.
	
```sql
SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords
```
	
- If we had more columns in the table of the original query, we would have to add more numbers to create the remaining required columns. 
	
- For example, if the original query used `SELECT` on a table with four columns, our `UNION` injection would be.
	
```sql
UNION SELECT username, 2, 3, 4 from passwords-- '
```
	
```mysql
mysql> SELECT * from products where product_id UNION SELECT username, 2, 3, 4 from passwords-- '

+-----------+-----------+-----------+-----------+
| product_1 | product_2 | product_3 | product_4 |
+-----------+-----------+-----------+-----------+
|   admin   |    2      |    3      |    4      |
+-----------+-----------+-----------+-----------+
-- Our wanted output of the query is found at the first column of the second row, while the numbers filled the remaining columns.
```
 ---
## Union Injection 
- Consider the following:
	-
![[Pasted image 20250522125910.png]]
	
- We see a potential SQL injection in the search parameters. We apply the SQLi Discovery steps by injecting a single quote (`'`), and we do get an error:
	
![[Pasted image 20250522125946.png]]
	
- Since we caused an error, this may mean that the page is vulnerable to SQL injection. 
	
- This scenario is ideal for exploitation through Union-based injection, as we can see our queries' results.
### Detect number of columns
- It is essential to figure out the number of columns selected by the server, before proceeding with exploiting Union-based queries.
	
 - There are two methods of detecting the number of columns
	- Using `ORDER BY`
	- Using `UNION`
### Using ORDER BY
- The **`ORDER BY` clause** helps determine the number of columns in an SQL query.
    
- Inject `ORDER BY 1--` to check if sorting by the first column works.
    
- Increase the column number incrementally (e.g., `ORDER BY 2--`, `ORDER BY 3--`, etc.).
    
- When the query causes an **error or blank page**, the specified column does **not exist**.
    
- The **last successful column number** before the failure indicates the **total number of columns**.
    
- Example: If `ORDER BY 4--` fails but `ORDER BY 3--` works, the query has **3 columns**.
	
```sql
-- an extra dash (-) at the end, represents that there is a space after (--)
' order by 1-- - 
```
	
![[Pasted image 20250522131754.png]]
	
```sql
-- Sorting by the second column.
' order by 2-- -
```
	
![[Pasted image 20250522131924.png]]
	
- We do the same for column `3` and `4` and get the results back. However, when we try to `ORDER BY` column 5, we get the following error which means that this table has exactly 4 columns.
	
![[Pasted image 20250522132018.png]]
### Using UNION
- Another way to detect the number of columns is by using **`UNION` injection**.
    
- Inject **`UNION SELECT`** statements with varying numbers of columns.
    
- This method produces **errors until the correct number** of columns is used.
    
- Start testing with a **3-column UNION query**, such as:  
    `UNION SELECT 1,2,3--`
    
- If the query succeeds and returns results, the number of columns is **correct**.
    
- Keep adjusting the number of columns in the UNION query until you get a **successful result**.
	
```sql
' UNION select 1,2,3-- -
' -- We get an error saying that the number of columns don't match.
```
	
![[Pasted image 20250522132337.png]]
	
```sql
-- Trying 4 columns to see the response.
cn' UNION select 1,2,3,4-- -
```
	
![[Pasted image 20250522132423.png]]
	-
- We successfully get the results, meaning once again that the table has 4 columns.
### Location of Injection
- A SQL query may return multiple columns, but the **web app might display only some**.
    
- If injected data is in a column that **is not shown** on the page, its output won’t be visible.
    
- Therefore, it’s important to identify which columns are **actually printed** on the web page.
    
- This helps determine **where to place the payload** for visible output.
    
- In the example above, even if the query returns 1, 2, 3, 4 — only **2, 3, and 4** are shown, revealing usable columns for injection.
	
![[Pasted image 20250522132759.png]]
	
- Not all columns from a SQL query are displayed to the user (e.g., **ID fields** are often hidden).
    
- In a previous example, **columns 2, 3, and 4** were printed, indicating suitable locations for SQL injection output.
    
- **Avoid injecting into non-visible columns** (like column 1) since their output won’t be seen.
    
- Using **numbers as junk data** (e.g., 1, 2, 3...) helps identify which columns are displayed.
    
- To test if **actual database data** can be retrieved, a query like `@@version` can be injected into a visible column (e.g., column 2).
	
![[Pasted image 20250522132945.png]]
	
- As we can see, we can get the database version displayed.
---
---

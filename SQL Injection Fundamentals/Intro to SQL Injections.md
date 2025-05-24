### Use of SQL in Web Applications
- Web applications use MySQL databases to store and retrieve data.
    
- The DBMS must be installed, configured, and running on the backend server.
    
- Once set up, web applications can interact with the database.
    
- In PHP web applications, MySQL can be accessed using SQL syntax within PHP code.
    
- PHP provides built-in functions (e.g., `mysqli_connect()`) to connect and interact with MySQL databases.
	
```php
```php
// Connecting to the database, and start using the MySQL database through MySQL syntax
$conn = new mysqli("localhost", "root", "password", "users");
$query = "select * from logins";
$result = $conn->query($query);
```
	
- The query's output will be stored in `$result`, and we can print it to the page or use it in any other way.
	
```php
// Print all returned results of the SQL query in new lines.
while($row = $result->fetch_assoc() ){
	echo $row["name"]."<br>";
}
```
	
- Web applications commonly use user input to retrieve data from databases.
    
- For example, a user search query is sent to the web application.
    
- The application uses this input to perform a search in the database.
    
- This enables dynamic and personalized data retrieval based on user interaction.
	
```php
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```
	
- ***If we use user-input within an SQL query, and if not securely coded, it may cause a variety of issues, like SQL Injection vulnerabilities.***
### What is an Injection?
- In the above example, user input is accepted and directly passed to the SQL query without sanitization.
	
>[!Note] Sanitization refers to the removal of any special characters in user-input, in order to break any injection attempts.
	
-  **Injection** happens when user input is treated as code by the application.
    
- This can alter the program's execution flow.
    
- Attackers may use special characters (e.g., `'`) to escape input boundaries.
    
- They can inject code like **JavaScript** or **SQL** (e.g., SQL Injection).
    
- **Lack of input sanitization** increases the risk of successful injection attacks.
### SQL Injection
- **SQL Injection** happens when user input is directly inserted into an SQL query string without sanitization.
    
- It allows attackers to manipulate the SQL query logic.
    
- The risk increases when input is **not filtered or sanitized** properly.
    
- Previous example demonstrated SQL queries using **raw user input**, making them vulnerable.
	
```php
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```
	
- In typical cases, the `searchInput` would be inputted to complete the query, returning the expected outcome.
	
- Any input we type goes into the following SQL query.
	
```sql
select * from logins where username like '%$searchInput'
```
	
- If a user inputs `admin`, the query becomes `'%admin'`—treated as a regular search term.
    
- Inputs like `SHOW DATABASES;` are also interpreted as search terms: `'%SHOW DATABASES;'`.
    
- However, **lack of input sanitization** allows attackers to manipulate the query.
    
- For example, inputting `1'; DROP TABLE users;` ends the string early and **injects real SQL code**.
    
- This results in the final query being altered to execute destructive commands like `DROP TABLE users;`.
	
```php
'%1'; DROP TABLE users;'
```
	
>[!Note] Notice how we added a single quote (') after "1", in order to escape the bounds of the user-input in ('%$searchInput').
	
-  The final executed query will be as follows.
	
```sql
select * from logins where username like '%1'; DROP TABLE users;'
-- Once the query is run, the` users `table will get deleted.
```
	
- As we can see from the syntax highlighting, we can escape the original query's bounds and have our newly injected query execute as well.
	
>[!Note] In the above example, for the sake of simplicity, we added another SQL query after a semi-colon (;). Though this is actually not possible with MySQL, it is possible with MSSQL and PostgreSQL.
### Syntax Errors
- The previous example of SQL injection would return an error:
	
```php
Error: near line 1: near "'": syntax error
```
	
- This is because of the last trailing character, where we have a single extra quote (`'`) that is not closed, which causes a SQL syntax error when executed.
	
```sql
select * from logins where username like '%1'; DROP TABLE users;'
```
	
- When user input is in the middle of an SQL query, injected code must ensure the **entire SQL statement remains valid**.
    
- **Syntax errors** after injection can cause the query to fail.
    
- **Attackers usually don't know** the original SQL query structure ( due to no source code access).
    
- To **bypass this**, attackers use:
    
    - **SQL comments** to ignore the rest of the original query.
        
    - **Multiple single quotes (`'`)** to close open strings properly and avoid syntax errors.
        
- These techniques help attackers **craft valid injections** even without full query knowledge.
### Types of SQL Injections
- SQL Injections are categorized based on how and where we retrieve their output.
	
![[Pasted image 20250522110258.png]]
	-
- **In-band SQL Injection**: Output is directly visible on the front end.
    
    - **Union Based**: Uses `UNION` to combine results and show output in a visible column.
        
    - **Error Based**: Forces SQL or PHP errors that leak data in the error message.
        
- **Blind SQL Injection**: Output is not visible; relies on observing application behavior.
    
    - **Boolean Based**: Uses conditions (e.g., `IF`) to check if output changes based on true/false.
        
    - **Time Based**: Uses delays (e.g., `SLEEP()`) to measure response time and infer data.
        
- **Out-of-band SQL Injection**: No direct or indirect output; data is exfiltrated to external sources, 'i.e., DNS record,' and then attempt to retrieve it from there.

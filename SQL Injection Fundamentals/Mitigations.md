## Mitigating SQL Injection
### Input Sanitization
- Here's the snippet of the code from the authentication bypass section we discussed earlier.
	
```php
<SNIP>
  $username = $_POST['username'];
  $password = $_POST['password'];

  $query = "SELECT * FROM logins WHERE username='". $username. "' AND password = '" . $password . "';" ;
  echo "Executing query: " . $query . "<br /><br />";

  if (!mysqli_query($conn ,$query))
  {
          die('Error: ' . mysqli_error($conn));
  }

  $result = mysqli_query($conn, $query);
  $row = mysqli_fetch_array($result);
<SNIP>
```
	
- The script **takes `username` and `password` directly from the POST request** and embeds them into an SQL query.
    
- This approach is **vulnerable to SQL injection**, allowing attackers to manipulate the query.
    
- To prevent this, **user input should be sanitized** before being used in SQL statements.
    
- One method is using **`mysqli_real_escape_string()`**, a function in PHP's MySQLi extension.
    
- This function **escapes special characters** like `'` and `"`, neutralizing potentially malicious input.
	
```php
<SNIP>
$username = mysqli_real_escape_string($conn, $_POST['username']);
$password = mysqli_real_escape_string($conn, $_POST['password']);

$query = "SELECT * FROM logins WHERE username='". $username. "' AND password = '" . $password . "';" ;
echo "Executing query: " . $query . "<br /><br />";
<SNIP>
```
	
- The snippet above shows how the function can be used.
	-
![[Pasted image 20250523132051.png]]
	
- As expected, the injection no longer works due to escaping the single quotes. A similar example is the [pg_escape_string()](https://www.php.net/manual/en/function.pg-escape-string.php) which used to escape PostgreSQL queries.
### Input Validation
- User input can also be validated based on the data used to query to ensure that it matches the expected input.
	
-  For example, when taking an email as input, we can validate that the input is in the form of `...@email.com`, and so on.
	-
- Consider the following code snippet from the ports page, which we used `UNION` injections on.
	-
```php
<?php
if (isset($_GET["port_code"])) {
	$q = "Select * from ports where port_code ilike '%" . $_GET["port_code"] . "%'";
	$result = pg_query($conn,$q);
    
	if (!$result)
	{
   		die("</table></div><p style='font-size: 15px;'>" . pg_last_error($conn). "</p>");
	}
<SNIP>
?>
```
	
- We see the GET parameter `port_code` being used in the query directly. 
	
- It's already known that a port code consists only of letters or spaces.
	
- We can restrict the user input to only these characters, which will prevent the injection of queries. 
	
- A regular expression can be used for validating the input.
	
```php
<SNIP>
$pattern = "/^[A-Za-z\s]+$/";
$code = $_GET["port_code"];

if(!preg_match($pattern, $code)) {
  die("</table></div><p style='font-size: 15px;'>Invalid input! Please try again.</p>");
}

$q = "Select * from ports where port_code ilike '%" . $code . "%'";
<SNIP>
```
	
- The code is modified to use the [preg_match()](https://www.php.net/manual/en/function.preg-match.php) function, which checks if the input matches the given pattern or not. 
	
- The pattern used is `[A-Za-z\s]+`, which will only match strings containing letters and spaces. Any other character will result in the termination of the script.
	
![[Pasted image 20250523133702.png]]
	
```sql
-- Use this query to test.
'; SELECT 1,2,3,4-- -
```
	
![[Pasted image 20250523133754.png]]
	
- As seen in the image above, input with injected queries was rejected by the server.
### User Privileges
- **DBMS allows creation of users with specific permissions** to enforce access control.
    
- **Users should be given only the minimum privileges** necessary to perform their tasks.
    
- **Superusers or admin-level accounts should never be used** by web applications.
    
- These high-privilege accounts can access sensitive functions, which may lead to **server compromise if exploited**.
    
- Always implement **principle of least privilege** when assigning database permissions.
	
```mysql
MariaDB [(none)]> CREATE USER 'reader'@'localhost';

Query OK, 0 rows affected (0.002 sec)


MariaDB [(none)]> GRANT SELECT ON ilfreight.ports TO 'reader'@'localhost' IDENTIFIED BY 'p@ssw0Rd!!';

Query OK, 0 rows affected (0.000 sec)
```
	
- The commands above add a new MariaDB user named `reader` who is granted only `SELECT` privileges on the `ports` table. 
	
- We can verify the permissions for this user by logging in.
	
```mysql
Whog@htb[/htb]$ mysql -u reader -p

MariaDB [(none)]> use ilfreight;
MariaDB [ilfreight]> SHOW TABLES;

+---------------------+
| Tables_in_ilfreight |
+---------------------+
| ports               |
+---------------------+
1 row in set (0.000 sec)


MariaDB [ilfreight]> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;

+--------------------+
| SCHEMA_NAME        |
+--------------------+
| information_schema |
| ilfreight          |
+--------------------+
2 rows in set (0.000 sec)


MariaDB [ilfreight]> SELECT * FROM ilfreight.credentials;
ERROR 1142 (42000): SELECT command denied to user 'reader'@'localhost' for table 'credentials'
```
	
- The snippet above confirms that the `reader` user cannot query other tables in the `ilfreight` database. 
	
- The user only has access to the `ports` table that is needed by the application.
### Web Application Firewall
- **Web Application Firewalls (WAFs)** help detect and block **malicious HTTP requests**.
    
- They are effective in **preventing SQL Injection**, even if the application has flawed logic.
    
- WAFs can be:
    
    - **Open-source** (e.g., _ModSecurity_)
        
    - **Premium/commercial** (e.g., _Cloudflare_)
        
- Most WAFs come with **default rulesets** targeting common web attacks.
    
- For example, requests containing terms like **`INFORMATION_SCHEMA`** may be **automatically blocked**, as they are often used in SQL injection attacks.
### Parameterized Queries
-  **Parameterized queries** are a safe way to handle user input in SQL queries.
    
- They use **placeholders** instead of directly embedding user input into the query.
    
- This approach ensures that the input is **properly escaped**, preventing SQL injection.
    
- In **PHP**, placeholders are filled using **specific functions** provided by database drivers (e.g., `mysqli`, `PDO`).
    
- Example: Instead of writing raw SQL with variables, we define the query with `?` or named parameters and bind the actual input later.
	
- Consider the following code:
	
```php
<SNIP>
  $username = $_POST['username'];
  $password = $_POST['password'];

  $query = "SELECT * FROM logins WHERE username=? AND password = ?" ;
  $stmt = mysqli_prepare($conn, $query);
  mysqli_stmt_bind_param($stmt, 'ss', $username, $password);
  mysqli_stmt_execute($stmt);
  $result = mysqli_stmt_get_result($stmt);

  $row = mysqli_fetch_array($result);
  mysqli_stmt_close($stmt);
<SNIP>
```
	
- The query is modified to contain two placeholders, marked with `?` where the username and password will be placed. 
	
- We then bind the username and password to the query using the [mysqli_stmt_bind_param()](https://www.php.net/manual/en/mysqli-stmt.bind-param.php) function. 
	
- This will safely escape any quotes and place the values in the query.
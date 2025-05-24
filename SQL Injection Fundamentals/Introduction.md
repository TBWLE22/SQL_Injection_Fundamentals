#### **1. Role of Databases in Web Applications**

- Modern web apps use databases to store/retrieve data such as content and user info.
    
- Web apps interact with databases in real-time using HTTP(S) requests.
    
- Queries are issued to the database based on user input to generate responses.


#### **2. Three-Tier Architecture**

- **Tier I**: User interacts with client application.
    
- **Tier II**: Client connects to an application server.
    
- **Tier III**: Application server connects to a DBMS (Database Management System).

#### **3. SQL Injection (SQLi) Overview**

- SQLi occurs when attackers inject malicious SQL into queries via user input.
    
- It manipulates the query sent to the database to perform unintended operations.
    
- Commonly targets **relational databases** (e.g., MySQL).
    
- Similar attacks on **non-relational databases** are called **NoSQL injection**.


#### **4. How SQL Injection Works**

- Attacker injects SQL code using characters like `'` or `"` to escape input boundaries.
    
- Alters query logic to:
    
    - Inject new SQL statements.
        
    - Use stacked queries or UNION-based attacks.
        
- Output is interpreted or captured from the application front-end.


#### **5. Use Cases and Impact**

- **Data Theft**: Retrieve sensitive info (logins, passwords, credit card data).
    
- **Login Bypass**: Access systems without valid credentials.
    
- **Privilege Escalation**: Gain unauthorized access to admin features.
    
- **Server Exploitation**: Read/write files, place backdoors, take control of server/website.

#### **6. Causes of SQL Injection**

- Poorly written application code.
    
- Insecure back-end server or database privileges.

#### **7. Prevention Techniques**

- **Secure Coding**: Validate and sanitize user inputs.
    
- **Principle of Least Privilege**: Restrict back-end user permissions.
    
- Use **prepared statements** and **parameterized queries**.
# Intro to Databases
- Web applications use back-end databases to store content and information.
    
- Stored data can include:
    
    - Core web assets (e.g., images, files)
        
    - Content (e.g., posts, updates)
        
    - User data (e.g., usernames, passwords)
        
- Various types of databases exist, each suited to different use cases.
    
- Earlier applications used file-based databases, which became inefficient as data grew.
    
- This inefficiency led to the use of Database Management Systems (DBMS) for better performance and management.
### Database Management Systems
- A **Database Management System (DBMS)** is used to create, define, host, and manage databases.
- Types of DBMS include:
    
    - File-based systems
        
    - Relational DBMS (RDBMS)
        
    - NoSQL
        
    - Graph-based DBMS
        
    - Key/Value stores
- DBMS can be accessed via:
    
    - Command-line tools
        
    - Graphical user interfaces (GUIs)
        
    - APIs (Application Programming Interfaces)
- DBMS is widely used in sectors like:
    
    - Banking
        
    - Finance
        
    - Education
- Key features of a DBMS include:

| **Feature**                 | **Description**                                                                                                                                                                            |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Concurrency`               | A real-world application might have multiple users interacting with it simultaneously. A DBMS makes sure that these concurrent interactions succeed without corrupting or losing any data. |
| `Consistency`               | With so many concurrent interactions, the DBMS needs to ensure that the data remains consistent and valid throughout the database.                                                         |
| `Security`                  | DBMS provides fine-grained security controls through user authentication and permissions. This will prevent unauthorized viewing or editing of sensitive data.                             |
| `Reliability`               | It is easy to backup databases and rolls them back to a previous state in case of data loss or a breach.                                                                                   |
| `Structured Query Language` | SQL simplifies user interaction with the database with an intuitive syntax supporting various operations.                                                                                  |

---
### Architecture
The diagram below details a two-tiered architecture.
![[Pasted image 20250520155209.png]]
- **Tier I (Client-side):**
    
    - Includes websites or GUI applications.
        
    - Handles high-level interactions (e.g., user login, comments).
        
    - Sends data to Tier II via API calls or other requests.
- **Tier II (Middleware):**
    
    - Interprets client-side events.
        
    - Prepares data for the DBMS using specific libraries and drivers.
        
    - Acts as the bridge between the client and the database.
- **Database Interaction:**
    
    - Middleware sends queries to the DBMS.
        
    - DBMS performs operations such as:
        
        - Insertion
            
        - Retrieval
            
        - Deletion
            
        - Updating
    - DBMS returns data or error codes after processing.
- **Hosting Considerations:**
    
    - Application server and DBMS can be hosted on the same machine.
        
    - For large-scale applications, databases are typically hosted separately for better **performance** and **scalability**.
---
---
## Types of Databases
Databases, in general, are categorized into `Relational Databases` and `Non-Relational Databases`. Only Relational Databases utilize SQL, while Non-Relational databases utilize a variety of methods for communications.
### Relational Databases
-  Most common type of database.
    - Uses a **schema** to define the structure of stored data.
- **Example Scenario:**
    - A company selling products stores information about:
        - Customers
            
        - Products sold
            
        - Payment and order details
    - Each data category is stored in a **separate table**:
        - Table 1: Customer information
            
        - Table 2: Product details and cost
            
        - Table 3: Order information and payment data
- **Tables and Keys:**
    - Tables (or entities) are linked using **keys** (e.g., customer ID, product ID).
        
    - Keys enable fast access and summarization of data.
        
    - Relationships between tables help maintain data consistency.
        
    - Changes in one table affect related tables predictably and systematically.
- **RDBMS (Relational Database Management System):**
    - Links tables using keys to manage integrated data.
        
    - Popular for being easy to learn, use, and understand.
        
    - Originally used by large companies, now widely adopted.
- **Examples of RDBMS:**
    - Microsoft Access
        
    - MySQL
        
    - SQL Server
        
    - Oracle
        
    - PostgreSQL
- **Sample Table Design:**
    - `users` table: `id`, `username`, `first_name`, `last_name`, etc.
        
    - `posts` table: `id`, `user_id`, `date`, `content`, etc.
        
    - `user_id` in `posts` references `id` in `users` — showing table relationships.
![[Pasted image 20250520161330.png]]
- **Linking Tables:**
    - The `id` from the `users` table links to `user_id` in the `posts` table.
        
    - This allows retrieval of user details for each post without duplicating data.
- **Multiple Keys:**
    - A table can have **more than one key** to link with different tables.
        
    - Example:
        - The `id` in the `posts` table can link to a `comments` table.
            
        - Each comment belongs to a specific post via this relationship.
- **Benefit:**
    - Efficient data organization and retrieval.
        
    - Avoids redundancy and supports scalable, maintainable design.

> [!Reminder] The relationship between tables within a database is called a Schema.
- **Efficiency of Relational Databases:**
    - Enables **quick and easy data retrieval** across multiple related tables.
        
    - Example: Retrieve all information about a specific user from different tables using a **single query**.
- **Advantages:**
    - Fast and reliable performance.
        
    - Ideal for managing **large datasets** with a **clear structure**.
        
    - Promotes efficient and systematic data management.
- **Popular Example:**
    - **MySQL** is a widely used relational database.
### Non-relational Databases
-  Do **not** use tables, rows, columns, relationships, or schemas.
    - Store data using **various models** based on data type.
        
    - Offer **high scalability** and **flexibility** due to lack of fixed structure.
- **Best Use Case:**
    - Ideal for storing **unstructured or semi-structured** data.
        
    - Suitable when the dataset does not follow a strict schema.
- **Four Common NoSQL Storage Models:**
    1. **Key-Value**
        
    2. **Document-Based**
        
    3. **Wide-Column**
        
    4. **Graph**
- **Key-Value Model Example:**
    - Stores data in **key-value pairs** (e.g., JSON or XML).
        
    - Each key maps to a value containing all associated data.
![[Pasted image 20250520162259.png]]
The above example can be represented using JSON as:
```json
```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```
It looks similar to a dictionary item in languages like `Python` or `PHP` (i.e. `{'key':'value'}`), where the `key` is usually a string, and the `value` can be a string, dictionary, or any class object.

The most common example of a NoSQL database is `MongoDB`.

>[!Reminder] Non-relational Databases have a different method for injection, known as NoSQL injections. SQL injections are completely different than NoSQL injections.

---
---

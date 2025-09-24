# Elevate_Labs_Task2
DML, Null handling

Interview Questions: 1. Difference between NULL and 0?
  Conceptual meaning:
    >NULL means the value is unknown, missing, or not applicable.
    >0 is a definite numeric value representing zero.
  Storage & operations:
    >NULL is a special marker, not a number or string, and typically uses an internal flag rather than “space.”
    >Arithmetic on NULL (5 + NULL) returns NULL because any calculation with an unknown value is still unknown.
    >Arithmetic on 0 behaves normally (5 + 0 = 5).
  Comparisons:
    >You cannot use = NULL or <> NULL; you must use IS NULL or IS NOT NULL.
    >NULL = NULL is not TRUE—it is UNKNOWN.
  Practical example: In an Orders table, Discount = NULL might mean “discount not yet determined,” while Discount = 0 means “confirmed: no   discount.”
  Indexing: Most database engines can index NULLs, but they treat them as a distinct value.

2. What is a Default Constraint?
  Definition: Provides an automatic value for a column when an INSERT omits that column.
  Purpose: Ensures consistent data entry and simplifies application code because you don’t have to supply every column.
  Types of defaults:
    Static (literal): DEFAULT 'Pending'.
    Dynamic (function): DEFAULT CURRENT_TIMESTAMP or DEFAULT UUID().
  Example:
    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        Status  VARCHAR(20) DEFAULT 'Pending',
        CreatedAt DATETIME DEFAULT CURRENT_TIMESTAMP
    );
  If you insert only the OrderID, the database sets Status to Pending and CreatedAt to the current time.
  Maintenance tip: Defaults can be altered later with ALTER TABLE … ALTER COLUMN … SET DEFAULT.

3.How does IS NULL work?
  Role: Tests whether a column value is actually NULL.
  Three-valued logic: SQL’s TRUE/FALSE/UNKNOWN logic means column = NULL can never be TRUE; hence IS NULL is mandatory.
  Use cases:
    >Find incomplete records: SELECT * FROM Employees WHERE Bonus IS NULL;.
    >Conditional output: SELECT COALESCE(Bonus, 0) to treat NULL as 0.
  Indexes: Many engines can optimize IS NULL checks using indexes.
  Difference from empty string: An empty string '' is a real value, not NULL.

4. How do you update multiple rows?
  Single statement: Use UPDATE with a WHERE clause that matches many rows:
    UPDATE Employees
    SET Salary = Salary * 1.1
    WHERE Department = 'Sales';
  Multi-table updates: Join tables to update based on related data:
    UPDATE Employees e
    JOIN Departments d ON e.DeptID = d.DeptID
    SET e.Bonus = e.Bonus + 500
    WHERE d.Region = 'West';
  Performance: One bulk update is faster and transactional compared to looping row-by-row in application code.
  Safety: Always preview with SELECT … WHERE … to ensure the WHERE clause is correct before running the update.

5️. .Can we insert partial values?
  Why allowed: Not all columns need a value every time. If a column allows NULL or has a DEFAULT, it can be omitted.
  Syntax:
    INSERT INTO Employees (Name, HireDate)
    VALUES ('Maria', CURRENT_DATE);
  Rules:
    Omitted columns must either allow NULL or have a DEFAULT.  
    If a column is NOT NULL and lacks a default, the insert fails.
  Benefits: Simplifies code when many columns have auto-generated or optional values.
  Tip: Always name columns explicitly to avoid breakage if table structure changes.

6️. What happens if a NOT NULL field is left empty?
  Immediate result: Database throws an error (e.g., ERROR: Column 'Email' cannot be null).
  Insert behavior:
    >Omitting the column is fine if it has a DEFAULT; otherwise it fails.
    >Providing NULL explicitly fails.
  Reason: Ensures critical data (like primary keys, login IDs, foreign keys) is never missing.
  Design advice: Use NOT NULL on columns that must always be known, such as usernames or timestamps.

7.How do you rollback a deletion?
  Transactional rollback:
    START TRANSACTION;
    DELETE FROM Employees WHERE Department = 'Temp';
    -- If you change your mind
    ROLLBACK;
    
    No rows are permanently removed until you COMMIT.
  Requirements: The table’s storage engine must support transactions (e.g., InnoDB in MySQL, PostgreSQL, SQL Server).  
  After commit: Rollback is impossible; you would need backups or point-in-time recovery tools.
  Good practice: Always wrap critical DELETEs in a transaction and double-check with a SELECT.

8.Can we insert values into specific columns only?
  Purpose: Allows inserting values only where needed, letting defaults or NULLs fill the rest. 
  Example:
    INSERT INTO Products (Name, Price)
    VALUES ('Widget', 19.99);
  Advantages:
    >Flexibility when schema evolves (adding columns doesn’t break inserts).
    >Reduces the need to supply meaningless placeholders for optional columns.

9.How to insert values using SELECT?
  Definition: Copying or transforming data from existing tables or queries directly into another table.
  Example:
    INSERT INTO ArchivedOrders (OrderID, CustomerID, Total)
    SELECT OrderID, CustomerID, Total
    FROM Orders
    WHERE OrderDate < '2024-01-01';
  Uses:
    >Archiving old records.
    >Creating summary or reporting tables.
    >Migrating data during a schema redesign.
  Performance: Usually faster than exporting and re-importing data because it’s all server-side.

10.What is ON DELETE CASCADE?
  Definition: A foreign key clause that automatically deletes child rows when the parent row is removed.
  Purpose: Maintains referential integrity without manual cleanup of dependent data.
  Example:
    CREATE TABLE OrderItems (
        ItemID   INT PRIMARY KEY,
        OrderID  INT,
        FOREIGN KEY (OrderID)
            REFERENCES Orders(OrderID)
            ON DELETE CASCADE
    );
  Behavior: Deleting a row from Orders automatically removes all related OrderItems.
  Caution:
    >Powerful but dangerous—accidental parent deletion can cascade widely.
    >Use only when dependent data is truly meaningless without the parent.

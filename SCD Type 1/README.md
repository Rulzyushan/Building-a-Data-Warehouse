Create Database and Schema

```sql
-- Drop and recreate the 'DataWarehouse' database
IF EXISTS (SELECT 1 FROM sys.databases WHERE name = 'SCD_Database')
BEGIN
    ALTER DATABASE SCD_Database SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE SCD_Database;
END;
GO


CREATE DATABASE SCD_Database;
GO

USE SCD_Database;
GO

-- Create Schemas
CREATE SCHEMA SCD_bronze;
GO
```

Create and Populate the dim_customers Table

```sql
-- Drop the table if it already exists to avoid duplication errors
IF OBJECT_ID('dim_customers', 'U') IS NOT NULL
    DROP TABLE dim_customers;

-- Create dim_customers table with a unique constraint on customer_id
CREATE TABLE dim_customers (
    row_id INT IDENTITY(1,1) PRIMARY KEY,    -- Surrogate key that auto-increments
    customer_id INT UNIQUE,       -- Add UNIQUE constraint on customer_id
    first_name VARCHAR(50),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100)
);

INSERT INTO dim_customers (customer_id, first_name, last_name, phone, email)
VALUES 
    (1001, 'John', 'Doe', '555-1234', 'john.doe@example.com'),
    (1002, 'Jane', 'Smith', '555-5678', 'jane.smith@example.com'),
    (1003, 'James', 'Brown', '555-8765', 'james.brown@example.com');
```

![Screenshot1.png](SCD1/Screenshot1.png?raw=true)

Create and populate the staging_customers Table

```sql
-- Drop the table if it already exists to avoid duplication errors
IF OBJECT_ID('staging_customers', 'U') IS NOT NULL
    DROP TABLE staging_customers;

-- Create staging_customers table
CREATE TABLE staging_customers (
    row_id INT IDENTITY(1,1) PRIMARY KEY,    -- Surrogate key that auto-increments
    customer_id INT UNIQUE,       -- Add UNIQUE constraint on customer_id
    first_name VARCHAR(50),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100)
);

INSERT INTO staging_customers (customer_id, first_name, last_name, phone, email)
VALUES 
    (1001, 'John', 'Doe', '555-4321', 'john.doe@newemail.com'),   -- Updated email and phone for existing customer
    (1002, 'Jane', 'Smith', '555-9999', 'jane.smith@newemail.com'), -- Updated phone and email for existing customer
    (1004, 'Emily', 'Davis', '555-1111', 'emily.davis@example.com'); -- New customer record
```
![Screenshot2.png](SCD1/Screenshot2.png?raw=true)

Implementation of SCD Type 1

```sql
MERGE dim_customers AS target
USING staging_customers AS source
ON target.customer_id = source.customer_id
WHEN MATCHED THEN
    UPDATE SET
        first_name = source.first_name,
        last_name = source.last_name,
        phone = source.phone,
        email = source.email
WHEN NOT MATCHED BY TARGET THEN
    INSERT (customer_id, first_name, last_name, phone, email)
    VALUES (source.customer_id, source.first_name, source.last_name, source.phone, source.email);
```
![Screenshot3.png](SCD1/Screenshot3.png?raw=true)

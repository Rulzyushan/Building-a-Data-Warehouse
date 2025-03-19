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

Create and Populate the dim_customers_scd3 Table

```sql
-- Drop the table if it already exists to avoid duplication errors
IF OBJECT_ID('dim_customers_scd3', 'U') IS NOT NULL
    DROP TABLE dim_customers_scd3;

-- Create dim_customers_scd3 table
CREATE TABLE dim_customers_scd3 (
    row_id INT IDENTITY(1,1) PRIMARY KEY,  -- Surrogate key that auto-increments (IDENTITY in SQL Server)
    customer_id INT UNIQUE,                -- Add UNIQUE constraint on customer_id
    first_name VARCHAR(50),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    previous_phone VARCHAR(20),            -- Column to store previous phone number
    email VARCHAR(100),
    previous_email VARCHAR(100)            -- Column to store previous email
);

-- Insert initial data into dim_customers_scd3
INSERT INTO dim_customers_scd3 (customer_id, first_name, last_name, phone, previous_phone, email, previous_email)
VALUES 
    (1001, 'John', 'Doe', '555-1234', NULL, 'john.doe@example.com', NULL),
    (1002, 'Jane', 'Smith', '555-5678', NULL, 'jane.smith@example.com', NULL),
    (1003, 'James', 'Brown', '555-8765', NULL, 'james.brown@example.com', NULL);
```

![Screenshot1.png](SCD3/Screenshot1.png?raw=true)

Create and populate the staging_customers_scd3 Table

```sql
-- Drop the table if it already exists to avoid duplication errors
IF OBJECT_ID('staging_customers_scd3', 'U') IS NOT NULL
    DROP TABLE staging_customers_scd3;

-- Create staging_customers_scd3 table
CREATE TABLE staging_customers_scd3 (
    row_id INT IDENTITY(1,1) PRIMARY KEY,  -- Surrogate key that auto-increments
    customer_id INT UNIQUE,                -- Add UNIQUE constraint on customer_id
    first_name NVARCHAR(50),               -- Use NVARCHAR for Unicode support
    last_name NVARCHAR(100),
    phone NVARCHAR(20),
    email NVARCHAR(100)
);

-- Insert new and updated records into staging_customers_scd3
INSERT INTO staging_customers_scd3 (customer_id, first_name, last_name, phone, email)
VALUES 
    (1001, 'John', 'Doe', '555-4321', 'john.doe@newemail.com'),    -- Updated phone and email for existing customer
    (1002, 'Jane', 'Smith', '555-9999', 'jane.smith@newemail.com'),-- Updated phone and email for existing customer
    (1004, 'Emily', 'Davis', '555-1111', 'emily.davis@example.com'); -- New customer record
```
![Screenshot2.png](SCD3/Screenshot2.png?raw=true)

Implementation of SCD Type 3

```sql
MERGE dim_customers_scd3 AS target
USING staging_customers_scd3 AS source
ON target.customer_id = source.customer_id  -- Match on customer_id
WHEN MATCHED THEN
    UPDATE SET
        first_name = source.first_name,
        last_name = source.last_name,
        previous_phone = target.phone,   -- Move current phone to previous_phone
        phone = source.phone,            -- Update with new phone
        previous_email = target.email,  -- Move current email to previous_email
        email = source.email             -- Update with new email
WHEN NOT MATCHED BY TARGET THEN
    INSERT (customer_id, first_name, last_name, phone, previous_phone, email, previous_email)
    VALUES (source.customer_id, source.first_name, source.last_name, source.phone, NULL, source.email, NULL);
```
![Screenshot3.png](SCD3/Screenshot3.png?raw=true)

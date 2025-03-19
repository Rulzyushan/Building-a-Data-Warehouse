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

Create and Populate the dim_customers_scd2 Table

```sql
-- Drop the table if it already exists to avoid duplication errors
IF OBJECT_ID('dim_customers_scd2', 'U') IS NOT NULL
    DROP TABLE dim_customers_scd2;

-- Create dim_customers table with a unique constraint on customer_id
CREATE TABLE dim_customers_scd2 (
    row_id INT IDENTITY(1,1) PRIMARY KEY,    -- Surrogate key that auto-increments
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100),
    start_date DATE DEFAULT GETDATE(),
    end_date DATE DEFAULT NULL,
    is_current CHAR(1) DEFAULT 'Y'
);

-- Insert initial data into the table
INSERT INTO dim_customers_scd2 (customer_id, first_name, last_name, phone, email)
VALUES 
    (1001, 'John', 'Doe', '555-1234', 'john.doe@example.com'),
    (1002, 'Jane', 'Smith', '555-5678', 'jane.smith@example.com'),
    (1003, 'James', 'Brown', '555-8765', 'james.brown@example.com');
```

![Screenshot1.png](SCD2/Screenshot1.png?raw=true)

Create and populate the staging_customers_scd2 Table

```sql
-- Drop the table if it already exists to avoid duplication errors
IF OBJECT_ID('staging_customers_scd2', 'U') IS NOT NULL
    DROP TABLE staging_customers_scd2;

-- Create staging_customers table
CREATE TABLE staging_customers_scd2 (
    row_id INT IDENTITY(1,1) PRIMARY KEY,    -- Surrogate key that auto-increments
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100)
);

-- Insert data into the staging_customers_scd2 table
INSERT INTO staging_customers_scd2 (customer_id, first_name, last_name, phone, email)
VALUES 
    (1001, 'John', 'Doe', '555-4321', 'john.doe@newemail.com'),   -- Updated email and phone for existing customer
    (1002, 'Jane', 'Smith', '555-9999', 'jane.smith@newemail.com'), -- Updated phone and email for existing customer
    (1004, 'Emily', 'Davis', '555-1111', 'emily.davis@example.com'); -- New customer record
```
![Screenshot2.png](SCD2/Screenshot2.png?raw=true)

Implementation of SCD Type 2

BEGIN TRANSACTION;
-- Step 1: Close out old records
UPDATE dim_customers_scd2
SET 
    end_date = CAST(GETDATE() AS DATE),
    is_current = 'N'
WHERE customer_id IN (
    SELECT staging.customer_id
    FROM staging_customers_scd2 staging
    JOIN dim_customers_scd2 dim
      ON staging.customer_id = dim.customer_id
      AND dim.is_current = 'Y'
    WHERE 
        staging.first_name <> dim.first_name OR
        staging.last_name <> dim.last_name OR
        staging.phone <> dim.phone OR
        staging.email <> dim.email
);

-- Step 2: Insert new records
INSERT INTO dim_customers_scd2 (
  customer_id, 
  first_name, 
  last_name, 
  phone, 
  email,
  start_date,
  end_date,
  is_current
)
SELECT 
  staging.customer_id,
  staging.first_name,
  staging.last_name,
  staging.phone,
  staging.email,
  CAST(GETDATE() AS DATE),
  NULL,
  'Y'
FROM staging_customers_scd2 staging
LEFT JOIN dim_customers_scd2 dim
  ON staging.customer_id = dim.customer_id
  AND dim.is_current = 'Y'
WHERE dim.customer_id IS NULL
   OR (
     staging.first_name <> dim.first_name OR
     staging.last_name <> dim.last_name OR
     staging.phone <> dim.phone OR
     staging.email <> dim.email
   );

COMMIT TRANSACTION;
```
![Screenshot3.png](SCD2/Screenshot3.png?raw=true)

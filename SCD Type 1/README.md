Create and populate the dim_customers Table

```sql
-- Drop the table if it already exists to avoid duplication errors
DROP TABLE IF EXISTS dim_customers;

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

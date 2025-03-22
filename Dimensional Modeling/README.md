
### **Relational Database**

![Screenshot1.png](IMG/Screenshot1.png?raw=true)

The optimal schema design for an operational information system implemented in a relational database is one that adheres to the third normal form (3NF) or follows the entity-relationship model [3, 4]. Such a design ensures efficient and consistent handling of atomic transactions, enabling high-performance insertion, deletion, and updating of data [3, 4]. Operational systems, often referred to as online transaction processing (OLTP) systems, rely on this schema design to maintain predictability and reliability in managing real-time transactional data.

### **Steps of the Dimensional Design Process**

- Identifying the business process.
- Identifying the dimensions.
- Identifying the facts.
****

#### **Business Processes**
In Relational Database (OLTP, We will work with the above tables and columns. We can identify the following processes:

- Ordering a bike by a customer.
- Shipping a bike from a store to a customer.
- Snapsh the stock of bikes in each store for each day.
  
The reason for considering ordering and shipping a bike as two different processes is that the order date and the shipment date are not necessarily identical, meaning the two processes do not occur simultaneously. Thus, each should be evaluated in a separate fact table.

**Some example questions we want to answer:**

- What are the total sales by product, customer, and time period?
- What is the inventory level of each product in each store?
- What are the shipment times for orders?
- Which stores have the highest sales?
- What are the best-selling products?
  
**Identify Facts**

**Identify Dimensions**

### **Query for Dimension Tables**
```sql
-- Dimension Tables
CREATE TABLE dim_customer (
    customer_id INT PRIMARY KEY,
    first_name NVARCHAR(50),
    last_name NVARCHAR(50),
    phone NVARCHAR(15),
    email NVARCHAR(100),
    street NVARCHAR(100),
    zip_code NVARCHAR(10),
    state NVARCHAR(50)
);

CREATE TABLE dim_staff (
    staff_id INT PRIMARY KEY,
    first_name NVARCHAR(50),
    last_name NVARCHAR(50),
    phone NVARCHAR(15),
    email NVARCHAR(100),
    active BIT,
    manager_id INT,
    manager_first_name NVARCHAR(50),
    manager_last_name NVARCHAR(50)
);

CREATE TABLE dim_product (
    product_id INT PRIMARY KEY,
    product_name NVARCHAR(100),
    list_price DECIMAL(10, 2),
    model_year INT,
    brand_name NVARCHAR(50),
    category_name NVARCHAR(50)
);

CREATE TABLE dim_store (
    store_id INT PRIMARY KEY,
    store_name NVARCHAR(100),
    phone NVARCHAR(15),
    email NVARCHAR(100),
    street NVARCHAR(100),
    zip_code NVARCHAR(10),
    city NVARCHAR(50)
);

CREATE TABLE dim_date (
    date_id INT PRIMARY KEY,
    date DATE,
    day_name NVARCHAR(10),
    day_of_month INT,
    week_of_month INT,
    week_of_year INT,
    month INT,
    month_name NVARCHAR(10),
    quarter INT,
    year INT,
    is_weekend BIT
);

CREATE TABLE diff_hierarchy (
    staff_id INT,
    subordinate_id INT,
    hierarchy_depth INT,
    PRIMARY KEY (staff_id, subordinate_id),
    FOREIGN KEY (staff_id) REFERENCES dim_staff(staff_id),
    FOREIGN KEY (subordinate_id) REFERENCES dim_staff(staff_id)
);
```

### **Query for Fact Tables**
```sql
-- Fact Tables
CREATE TABLE fact_bike_order (
    order_date_id INT,
    requirement_date_id INT,
    customer_id INT,
    staff_id INT,
    store_id INT,
    product_id INT,
    order_id INT PRIMARY KEY,
    quantity INT,
    list_price DECIMAL(10, 2),
    discount DECIMAL(5, 2),
    order_amount DECIMAL(10, 2),
    discounted_order_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES dim_customer(customer_id),
    FOREIGN KEY (staff_id) REFERENCES dim_staff(staff_id),
    FOREIGN KEY (store_id) REFERENCES dim_store(store_id),
    FOREIGN KEY (product_id) REFERENCES dim_product(product_id),
    FOREIGN KEY (order_date_id) REFERENCES dim_date(date_id),
    FOREIGN KEY (requirement_date_id) REFERENCES dim_date(date_id)
);

CREATE TABLE fact_bike_shipment (
    shipment_date_id INT,
    customer_id INT,
    staff_id INT,
    store_id INT,
    product_id INT,
    order_id INT,
    quantity INT,
    list_price DECIMAL(10, 2),
    discount DECIMAL(5, 2),
    shipment_amount DECIMAL(10, 2),
    discounted_shipment_amount DECIMAL(10, 2),
    PRIMARY KEY (order_id, shipment_date_id),
    FOREIGN KEY (customer_id) REFERENCES dim_customer(customer_id),
    FOREIGN KEY (staff_id) REFERENCES dim_staff(staff_id),
    FOREIGN KEY (store_id) REFERENCES dim_store(store_id),
    FOREIGN KEY (product_id) REFERENCES dim_product(product_id),
    FOREIGN KEY (shipment_date_id) REFERENCES dim_date(date_id)
);

CREATE TABLE fact_store_stock (
    date_id INT,
    store_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (date_id, store_id, product_id),
    FOREIGN KEY (store_id) REFERENCES dim_store(store_id),
    FOREIGN KEY (product_id) REFERENCES dim_product(product_id),
    FOREIGN KEY (date_id) REFERENCES dim_date(date_id)
);
```

### **Data Insertion for Dimension Tables**
```sql
-- Drop schema if exists and create schema
IF SCHEMA_ID('dwh') IS NOT NULL
    DROP SCHEMA dwh;
GO
CREATE SCHEMA dwh;
GO

-- ==================
-- Product Dimension
-- ==================
SELECT 
    p.product_id,
    p.product_name,
    p.list_price,
    p.model_year,
    b.brand_name,
    c.category_name
INTO dwh.dim_product
FROM bike_stores.products p
JOIN bike_stores.brands b
    ON p.brand_id = b.brand_id
JOIN bike_stores.categories c
    ON p.category_id = c.category_id;

ALTER TABLE dwh.dim_product
ADD CONSTRAINT dim_product_pk PRIMARY KEY (product_id);

-- ===================
-- Customer Dimension
-- ===================
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    COALESCE(c.phone, 'Unknown') AS phone,
    COALESCE(c.email, 'Unknown') AS email,
    COALESCE(c.street, 'Unknown') AS street,
    COALESCE(c.zip_code, 'Unknown') AS zip_code,
    COALESCE(c.state, 'Unknown') AS state
INTO dwh.dim_customer
FROM bike_stores.customers c;

ALTER TABLE dwh.dim_customer
ADD CONSTRAINT dim_customer_pk PRIMARY KEY (customer_id);

-- ================
-- Store Dimension
-- ================
SELECT 
    s.store_id,
    s.store_name,
    COALESCE(s.phone, 'Unknown') AS phone,
    COALESCE(s.email, 'Unknown') AS email,
    s.street,
    s.zip_code,
    s.city
INTO dwh.dim_store
FROM bike_stores.stores s;

ALTER TABLE dwh.dim_store
ADD CONSTRAINT dim_store_pk PRIMARY KEY (store_id);

-- ================
-- Staff Dimension
-- ================
SELECT 
    s.staff_id,
    s.first_name,
    s.last_name,
    COALESCE(s.phone, 'Unknown') AS phone,
    COALESCE(s.email, 'Unknown') AS email,
    s.active,
    s.manager_id,
    s2.first_name AS manager_first_name,
    s2.last_name AS manager_last_name
INTO dwh.dim_staff
FROM bike_stores.staffs s
LEFT JOIN bike_stores.staffs s2
    ON s.manager_id = s2.staff_id;

ALTER TABLE dwh.dim_staff
ADD CONSTRAINT dim_staff_pk PRIMARY KEY (staff_id);

-- ==========================
-- Staff Hierarchy Bridge
-- ==========================
WITH sh AS (
    SELECT
        staff_id,
        staff_id AS subordinate_id,
        0 AS hierarchy_depth
    FROM bike_stores.staffs
    UNION ALL
    SELECT 
        sh.staff_id,
        s.staff_id AS subordinate_id,
        sh.hierarchy_depth + 1 AS hierarchy_depth
    FROM sh
    JOIN bike_stores.staffs s
        ON sh.subordinate_id = s.manager_id
)
SELECT 
    staff_id,
    subordinate_id,
    hierarchy_depth
INTO dwh.staff_hierarchy
FROM sh;

ALTER TABLE dwh.staff_hierarchy
ADD CONSTRAINT staff_hierarchy_d_staff_fk 
    FOREIGN KEY (staff_id) REFERENCES dwh.dim_staff(staff_id);

ALTER TABLE dwh.staff_hierarchy
ADD CONSTRAINT staff_hierarchy_d_subordinate_fk 
    FOREIGN KEY (subordinate_id) REFERENCES dwh.dim_staff(staff_id);

-- ===============
-- Date Dimension
-- ===============
CREATE TABLE dwh.dim_date (
    date_id INT NOT NULL,
    date DATE NOT NULL,
    day_name VARCHAR(20) NOT NULL,
    day_of_month INT NOT NULL,
    week_of_month INT NOT NULL,
    week_of_year INT NOT NULL,
    month INT NOT NULL,
    month_name VARCHAR(20) NOT NULL,
    quarter INT NOT NULL,
    year INT NOT NULL,
    is_weekend BIT NOT NULL
);

-- Insert data into dim_date
;WITH date_seq AS (
    SELECT 
        DATEADD(DAY, seq.num, '2016-01-01') AS d
    FROM (
        SELECT TOP 2000 ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) - 1 AS num
        FROM sys.objects
    ) seq
)
INSERT INTO dwh.dim_date
SELECT 
    CONVERT(INT, CONVERT(CHAR(8), d, 112)) AS date_id,
    d AS date,
    DATENAME(WEEKDAY, d) AS day_name,
    DAY(d) AS day_of_month,
    DATEPART(WEEK, d) - DATEPART(WEEK, DATEFROMPARTS(YEAR(d), MONTH(d), 1)) + 1 AS week_of_month,
    DATEPART(WEEK, d) AS week_of_year,
    MONTH(d) AS month,
    DATENAME(MONTH, d) AS month_name,
    DATEPART(QUARTER, d) AS quarter,
    YEAR(d) AS year,
    CASE 
        WHEN DATEPART(WEEKDAY, d) IN (7, 1) THEN 1 -- Sunday = 1, Saturday = 7
        ELSE 0
    END AS is_weekend
FROM date_seq
ORDER BY date_id;

ALTER TABLE dwh.dim_date
ADD CONSTRAINT dim_date_pk PRIMARY KEY (date_id);

ALTER TABLE dwh.dim_date
ADD CONSTRAINT dim_date_date_u UNIQUE(date);
```

### **Data Insertion for Fact Tables**
```sql
-- ===========
-- Order Fact
-- ===========
SELECT
    CONVERT(INT, CONVERT(CHAR(8), o.order_date, 112)) AS order_date_id,
    CONVERT(INT, CONVERT(CHAR(8), o.required_date, 112)) AS requirement_date_id,
    o.customer_id,
    o.staff_id,
    o.store_id,
    oi.product_id,
    o.order_id,
    oi.quantity,
    oi.list_price,
    oi.discount,
    oi.list_price * oi.quantity AS order_amount,
    (oi.list_price - oi.discount) * oi.quantity AS discounted_order_amount
INTO dwh.fact_bike_order
FROM bike_stores.orders o
JOIN bike_stores.order_items oi
    ON o.order_id = oi.order_id;

ALTER TABLE dwh.fact_bike_order 
ADD CONSTRAINT f_bike_order_date_fk      
    FOREIGN KEY (order_date_id) REFERENCES dwh.dim_date(date_id);

ALTER TABLE dwh.fact_bike_order 
ADD CONSTRAINT f_bike_order_requirement_d_date_fk      
    FOREIGN KEY (requirement_date_id) REFERENCES dwh.dim_date(date_id);

ALTER TABLE dwh.fact_bike_order 
ADD CONSTRAINT f_bike_order_d_customer_fk      
    FOREIGN KEY (customer_id) REFERENCES dwh.dim_customer(customer_id);

ALTER TABLE dwh.fact_bike_order 
ADD CONSTRAINT f_bike_order_d_product_fk      
    FOREIGN KEY (product_id) REFERENCES dwh.dim_product(product_id);

ALTER TABLE dwh.fact_bike_order 
ADD CONSTRAINT f_bike_order_d_staff_fk      
    FOREIGN KEY (staff_id) REFERENCES dwh.dim_staff(staff_id);

ALTER TABLE dwh.fact_bike_order 
ADD CONSTRAINT f_bike_order_d_store_fk      
    FOREIGN KEY (store_id) REFERENCES dwh.dim_store(store_id);

-- ===============
-- Shipment Fact
-- ===============
SELECT
    CONVERT(INT, CONVERT(CHAR(8), o.shipped_date, 112)) AS shipment_date_id,
    o.customer_id,
    o.staff_id,
    o.store_id,
    oi.product_id,
    o.order_id,
    oi.quantity,
    oi.list_price,
    oi.discount,
    oi.list_price * oi.quantity AS shipment_amount,
    (oi.list_price - oi.discount) * oi.quantity AS discounted_shipment_amount
INTO dwh.fact_bike_shipment
FROM bike_stores.orders o
JOIN bike_stores.order_items oi
    ON o.order_id = oi.order_id
WHERE o.shipped_date IS NOT NULL;

ALTER TABLE dwh.fact_bike_shipment 
ADD CONSTRAINT f_bike_shipment_d_date_fk      
    FOREIGN KEY (shipment_date_id) REFERENCES dwh.dim_date(date_id);

ALTER TABLE dwh.fact_bike_shipment 
ADD CONSTRAINT f_bike_shipment_d_customer_fk      
    FOREIGN KEY (customer_id) REFERENCES dwh.dim_customer(customer_id);

ALTER TABLE dwh.fact_bike_shipment 
ADD CONSTRAINT f_bike_shipment_d_product_fk      
    FOREIGN KEY (product_id) REFERENCES dwh.dim_product(product_id);

ALTER TABLE dwh.fact_bike_shipment 
ADD CONSTRAINT f_bike_shipment_d_staff_fk      
    FOREIGN KEY (staff_id) REFERENCES dwh.dim_staff(staff_id);

ALTER TABLE dwh.fact_bike_shipment 
ADD CONSTRAINT f_bike_shipment_d_store_fk      
    FOREIGN KEY (store_id) REFERENCES dwh.dim_store(store_id);

-- ============================
-- Store Stock Fact
-- ============================
SELECT 
    CONVERT(INT, CONVERT(CHAR(8), '2021-06-23', 112)) AS date_id,
    store_id,
    product_id,
    quantity
INTO dwh.fact_store_stock
FROM bike_stores.stocks;

ALTER TABLE dwh.fact_store_stock
ADD CONSTRAINT f_store_stock_d_store_fk
    FOREIGN KEY (store_id) REFERENCES dwh.dim_store(store_id);

ALTER TABLE dwh.fact_store_stock
ADD CONSTRAINT f_store_stock_d_product_fk
    FOREIGN KEY (product_id) REFERENCES dwh.dim_product(product_id);

ALTER TABLE dwh.fact_store_stock
ADD CONSTRAINT f_store_stock_d_date_fk
    FOREIGN KEY (date_id) REFERENCES dwh.dim_date(date_id);
```

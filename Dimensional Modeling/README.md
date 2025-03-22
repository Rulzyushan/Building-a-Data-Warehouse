
### **Relational Database**

![Screenshot1.png](IMG/Screenshot1.png?raw=true)

The optimal schema design for an operational information system implemented in a relational database is one that adheres to the third normal form (3NF) or follows the entity-relationship model [3, 4]. Such a design ensures efficient and consistent handling of atomic transactions, enabling high-performance insertion, deletion, and updating of data [3, 4]. Operational systems, often referred to as online transaction processing (OLTP) systems, rely on this schema design to maintain predictability and reliability in managing real-time transactional data.

### **Steps of the Dimensional Design Process**

- Identifying the business process.
- Declaring the gain.
- Identifying the dimensions.
- Identifying the facts.
****
#### **Business Processes**
In database, we can identify the following processes:

- Ordering a bike by a customer.
- Shipping a bike from a store to a customer.
- Snapsh the stock of bikes in each store for each day.
  
The reason for considering ordering and shipping a bike as two different processes is that the order date and the shipment date are not necessarily identical, meaning the two processes do not occur simultaneously. Thus, each should be evaluated in a separate fact table.

#### **Grain**

1. **Grain Definition:**
The grain defines what a single row in a fact table represents. It specifies the level of detail or granularity of the data.
For example, in a sales process, the grain could be "one row per sale per day per customer per product."

2. **Importance of Declaring Grain Early:**
The grain must be declared before identifying dimensions or facts because all dimensions and facts must align with the chosen grain.
If the grain is not clearly defined, the design may not support the necessary analyses.

3. **Atomic Grain:**
Atomic grain refers to the lowest level of detail captured by a business process.
Kimball and Ross recommend starting with atomic-grained data because it provides the most flexibility for answering a wide range of user queries, including those that are unpredictable.

4. **Example of Grain in Ordering and Shipping Processes:**
In the example provided, the lowest level of data for analyzing ordering and shipping processes is:

 - Per date
 - Per customer
 - Per product
 - Per store
 - Per staff

   If the grain were defined only by date, customer, and product, it would not allow analysis by staff or store attributes (e.g., store 
   location). This would limit the ability to answer certain business questions.

5. **Multiple Grains for the Same Process:**
If two different grains are needed for the same process, separate fact tables should be created for each grain.
For example, one fact table might have a grain of "per date, per customer, per product," while another might have a grain of "per date, per store, per staff."

6. **Practical Implications:**
The passage gives an example of analyzing delayed shipments by customer zip code. If the customer dimension were missing from either the fact_bike_order or fact_bike_shipment fact tables, it would be impossible to analyze delays based on customer addresses.
Similarly, the design should allow for easy extension of queries to analyze delays by other dimensions, such as store location or shipment month.

7. Flexibility in Querying:
By defining the grain at the atomic level and including all relevant dimensions (e.g., customer, store, staff), the data warehouse becomes more flexible and capable of answering a wide variety of business questions.
****

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

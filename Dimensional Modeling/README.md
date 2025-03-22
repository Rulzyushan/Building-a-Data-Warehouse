
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

# Building-a-Data-Warehouse
### **What is a data warehouse**
A data warehouse is a system, which consolidates and stores enterprise information from diverse sources in a form suitable for analytical querying and reporting to support business intelligence and data analytics initiatives.
### **Why data warehouse**
- Fact-based decisions taken at the speed of business as end-users can effortlessly access and work with a company’s historical information as well as current information collected from disparate heterogeneous systems.
- Decision-making based on high-quality information, because prior to entering a data warehouse, data undergoes comprehensive cleansing and transformation processes. In addition to this, many data management activities become automated, which helps eliminate error-prone manual data aggregation.   
- When a data warehouse is integrated with self-service BI solutions, such as Power BI or Tableau, data culture is adopted naturally across a company. 
- Due to the unified approach to data governance, which besides other things implies solid definition and management of data security policies, the risk of data breaches and leaks is minimized.
### **Data warehouse architecture**
When you create the architecture of your future data warehouse, you have to take into account multiple factors, such as how many data sources will connect to the data warehouse, the amount of information in each of them together with its nature and complexity, your analytics objectives, existing technology environment, and soon.
- **Data Sources:** The different data sources include databases, CRM systems, flat files, APIs, etc., which are the origins of the data. Each source can have varied formats and volumes of data.‍
- **ETL (Extract, Transform, Load):** ETL process helps you to extract data from multiple sources, transform it into a format suitable by the data warehouse, and load it into the central repository.‍
- **Data Warehouse Database:** A data warehouse database is the central repository where the transformed data is stored. It is usually a relational database management system (RDBMS) optimized for complex queries and analytics.‍
- **Metadata Repository:** The metadata repository stores information, such as data definitions, data lineage, and data relationships, about the data stored in the warehouse. Metadata is essential for understanding the data’s context and makes your data easier to find, access, and use.‍
- **Query and Reporting Tools:** These tools allow you to extract actionable insights from the warehouse data. Query tools enable ad-hoc queries, and reporting tools help create structured reports and visual dashboards.
### **Approaches to building a data warehouse**
The two fundamental design methods, which are used to build a data warehouse, are Inmon’s (Top-down) and Kimball’s (Bottom-up) approaches. 

**Inmon’s approach**

Within Inmon’s approach, firstly, a centralized repository for enterprise information is designed according to a normalized data model, where atomic data is stored in tables that are grouped together by subject areas with the help of joins. After the enterprise data warehouse is built, the data stored there is used to structure data marts.

Inmon’s approach is more preferable in cases when you need to:
- Get a single source of truth while ensuring data consistency, accuracy and reliability
- Quickly develop data marts with no effort duplication for extracting data from original sources, cleansing, etc.
However, one of the major constraints of this method is that the setup and implementation is more time and resource-consuming compared to Kimball’s approach.

**Kimball’s approach**

Kimball’s approach suggests that dimensional data marts should be created first, then if required, a company may proceed with creating a logical enterprise data warehouse.
The advocates of this approach point out that since dimensional data marts require minimal normalization, such data warehouse projects take less time and resources.  On the other hand, you may find duplicate data in tables and have to repeat ETL activities, as each data mart is created independently.

### **Dimensional modeling**

Dimensional Modeling (DM) is a data structure technique optimized for data storage in a Data warehouse. The purpose of dimensional modeling is to optimize the database for faster retrieval of data. The concept of Dimensional Modelling was developed by Ralph Kimball and consists of fact and dimension tables.

A dimensional model in data warehouse is designed to read, summarize, analyze numeric information like values, balances, counts, weights, etc. in a data warehouse. In contrast, relation models are optimized for addition, updating and deletion of data in a real-time Online Transaction System. in the relational mode, normalization and ER models reduce redundancy in data. On the contrary, dimensional model in data warehouse arranges data in such a way that it is easier to retrieve information and generate reports.

### **Normalization vs. Denormalization**

- Normalization is the technique of dividing the data into multiple tables to reduce data redundancy and inconsistency and to achieve data integrity. On the other hand, Denormalization is the technique of combining the data into a single table to make data retrieval faster.
- Normalization is used in OLTP system, which emphasizes on making the insert, delete and update anomalies faster. As against, Denormalization is used in OLAP system, which emphasizes on making the search and analysis faster.
- Data integrity is maintained in normalization process while in denormalization data integrity harder to retain.
- Redundant data is eliminated when normalization is performed whereas denormalization increases the redundant data
- Normalization increases the number of tables and joins. In contrast, denormalization reduces the number of tables and join.
- Disk space is wasted in denormalization because same data is stored in different places. On the contrary, disk space is optimized in a normalized table.
- Normalization and denormalization are useful according to the situation. Normalization is used when the faster insertion, deletion and update anomalies, and data consistency are necessarily required. On the other hand, Denormalization is used when the faster search is more important and to optimize the read performance. It also lessens the overheads created by over-normalized data or complicated table joins.

### **Star schema design**

Star schema is a dimensional modeling design technique adopted by relational data warehouses.
- **Dimension tables** describe the entities relevant to your organization and analytics requirements. Broadly, they represent the things that you model. Things could be products, people, places, or any other concept, including date and time.
- **Fact tables** store measurements associated with observations or events. They can store sales orders, stock balances, exchange rates, temperature readings, and more. Fact tables contain dimension keys together with granular values that can be aggregated.

### **Dimension table**

In a dimensional model, a dimension table describes an entity relevant to your business and analytics requirements. Broadly, dimension tables represent the things that you model. Things could be products, people, places, or any other concept, including date and time.

****

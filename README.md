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

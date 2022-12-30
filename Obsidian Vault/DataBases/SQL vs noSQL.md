# [Difference between SQL and NoSQL](https://www.geeksforgeeks.org/difference-between-sql-and-nosql/)
When it comes to choosing a database the biggest decisions is picking a **relational (SQL)** or **non-relational (NoSQL)** data structure. While both the databases are viable options still there are certain key differences between the two that users must keep in mind when making a decision. 

## The Main Differences

1.  **Type –**   
    [SQL](https://www.geeksforgeeks.org/sql-tutorial/) databases are primarily called as Relational Databases [(RDBMS)](https://www.geeksforgeeks.org/rdbms-full-form/); whereas [NoSQL database](https://www.geeksforgeeks.org/introduction-to-nosql/) are primarily called as non-relational or distributed database. 
2.  **Language –**   
    SQL databases defines and manipulates data based [structured query language (SQL)](https://www.geeksforgeeks.org/structured-query-language/). Seeing from a side this language is extremely powerful. SQL is one of the most versatile and widely-used options available which makes it a safe choice especially for great complex queries. But from other side it can be restrictive. SQL requires you to use predefined [schemas](https://www.geeksforgeeks.org/create-schema-in-sql-server/) to determine the structure of your data before you work with it. Also all of your data must follow the same structure. This can require significant up-front preparation which means that a change in the structure would be both difficult and disruptive to your whole system.   
    A NoSQL database has dynamic schema for unstructured data. Data is stored in many ways which means it can be document-oriented, column-oriented, graph-based or organized as a KeyValue store. This flexibility means that documents can be created without having defined structure first. Also each document can have its own unique structure. The syntax varies from database to database, and you can add fields as you go. 
3.  **Scalability –**   
    In almost all situations SQL databases are vertically scalable. This means that you can increase the load on a single server by increasing things like [RAM](https://www.geeksforgeeks.org/random-access-memory-ram/), [CPU](https://www.geeksforgeeks.org/central-processing-unit-cpu/) or [SSD](https://www.geeksforgeeks.org/introduction-to-solid-state-drive-ssd/). But on the other hand NoSQL databases are horizontally scalable. This means that you handle more traffic by sharding, or adding more servers in your NoSQL database. It is similar to adding more floors to the same building versus adding more buildings to the neighborhood. Thus NoSQL can ultimately become larger and more powerful, making these databases the preferred choice for large or ever-changing data sets.
4.  **Structure –**   
    SQL databases are table-based on the other hand NoSQL databases are either key-value pairs, document-based, graph databases or wide-column stores. This makes relational SQL databases a better option for applications that require multi-row transactions such as an accounting system or for legacy systems that were built for a relational structure. 
5.  **Property followed –**   
    SQL databases follow [ACID properties](https://www.geeksforgeeks.org/acid-properties-in-dbms/) (Atomicity, Consistency, Isolation and Durability) whereas the NoSQL database follows the Brewers [CAP theorem](https://www.geeksforgeeks.org/the-cap-theorem-in-dbms/) (Consistency, Availability and Partition tolerance). 
6.  **Support –**   
    Great support is available for all SQL database from their vendors. Also a lot of independent consultations are there who can help you with SQL database for a very large scale deployments but for some NoSQL database you still have to rely on community support and only limited outside experts are available for setting up and deploying your large scale NoSQL deployments.

# Key highlights on SQL vs NoSQL

## SQL

- RELATIONAL DATABASE MANAGEMENT SYSTEM (RDBMS)
- These databases have fixed or static or predefined schema
- These databases are not suited for [[Hierarchical Database]] storage
- These databases are best suited for complex queries
- Vertically Scalable
- Follows #ACID property

**Examples:** [MySQL](https://www.geeksforgeeks.org/mysql-common-mysql-queries/), [PostgreSQL](https://www.geeksforgeeks.org/what-is-postgresql-introduction/), Oracle, MS-SQL Server, etc

## NoSQL

- Non-relational or distributed database system.
- They have dynamic schema
- These databases are best suited for hierarchical data storage.
- These databases are not so good for complex queries
- Horizontally scalable
- Follows CAP(consistency, availability, partition tolerance)

**Examples:** [MongoDB](https://www.geeksforgeeks.org/mongodb-tutorial/), [GraphQL](https://www.geeksforgeeks.org/graphql-query/), [HBase](https://www.geeksforgeeks.org/architecture-of-hbase/), [Neo4j](https://www.geeksforgeeks.org/neo4j-introduction/), [Cassandra,](https://www.geeksforgeeks.org/apache-cassandra-nosql-database/), Redis, etc
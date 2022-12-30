A hierarchical database is a data model in which data is stored in the form of records and organized into a tree-like structure, or parent-child structure, in which one parent node can have many child nodes connected through links.

![Simple Hierarchical Database diagram](https://assets-global.website-files.com/620d42e86cb8ec4d0839e59d/6230e9d760057557e278e91c_61cb63ea50556dfc275b7d47_Hierarchical-Database-Model-example.png)

# FAQs

## What is a Hierarchical Database?

As the name suggests, the hierarchical database model is most appropriate for use cases in which the main focus of information gathering is based on a concrete hierarchy, such as several individual employees reporting to a single department at a company.

The schema for hierarchical databases is defined by its tree-like organization, in which there is typically a root “parent” directory of data stored as records that links to various other subdirectory branches, and each subdirectory branch, or child record, may link to various other subdirectory branches.

The hierarchical database structure dictates that, while a parent record can have several child records, each child record can only have one parent record. Data within records is stored in the form of fields, and each field can only contain one value. Retrieving hierarchical data from a hierarchical database architecture requires traversing the entire tree, starting at the root node

## Hierarchical Database vs Relational Database

A [relational database](https://www.heavy.ai/technical-glossary/relational-database) is a digital database based on the relational model, which organizes data in sets of tables with columns and rows. The rows, also known as records, represent a collection of related values, and each row is identified by a unique key. Each column holds attributes of a specific data type, and each record typically has a value for each attribute.

The difference between relational and hierarchical databases lies in the data structures. While the hierarchical database architecture is tree-like, data in a relational database is stored in tables with a unique identifier for each record. A relational database structure facilitates easy identification and access of data in relation to other data points in the database. The tables are separate from physical storage structures, which enables database administrators to alter physical data storage without reorganizing the database tables themselves.

Characteristics of hierarchical database models include their simplicity, but also their lack of flexibility. Hierarchical structures, unlike, relational databases, do not describe many-to-one relationships or many-to-many relationships due to the fact that child records can only have a single parent.  

## Hierarchical Database Model Advantages and Disadvantages

The key advantage of a hierarchical database is its ease of use. The one-to-many organization of data makes traversing the database simple and fast, which is ideal for use cases such as website drop-down menus or computer folders in systems like Microsoft Windows OS. Due to the separation of the tables from physical storage structures, information can easily be added or deleted without affecting the entirety of the database. And most major programming languages offer functionality for reading tree structure databases.

The major disadvantage of hierarchical databases is their inflexible nature. The one-to-many structure is not ideal for complex structures as it cannot describe relationships in which each child node has multiple parents nodes. Also the tree-like organization of data requires top-to-bottom sequential searching, which is time consuming, and requires repetitive storage of data in multiple different entities, which can be redundant.
# Data and Tables

## Databases

There are many kinds of databases, but what they all have in common is persistence and data structures for storing or searching data. It is also possible for multiple users or programs to access or modify data at the same time.

How do we structure application data?

In memory (Ephemeral, Temporary)              | Durable storage (Persistent, Durable)
--------------------------------------------- | ----------------------------------------------------------
simple variables: numbers, strings            | flat files on disk: text, XML, JSON
data structures: lists, dictionaries, objects | databases: key-value store, navigational DB, relational DB

### Relational Databases

Relational databases offer very flexible tools for querying with aggregation and join operations to summarize data. We can also set up constraints which are rules to ensure that changes within the data is consistent.

## Tables

We store all our data in the form of tables. Below is an anatomy of a table: ![](/images/table.png)

## Data Types and Meaning

Each data entry has a **data type** similar to how we assign variables in code. A column of countries will store the name of a country as a string or text and the population will be a numeric or integer data type.

We also have to think about the **meaning** of each data type as it is represented in the real world. For example, you wouldn't have an entry of someone's full name in the column only set for surnames.

## Aggregations

An aggregation is a database operation that summarizes multiple rows into a single row. Most of the aggregations only work with numbers.

Here are some of the most common aggregations in SQL:

Question             | Aggregation
-------------------- | -----------
How many rows?       | COUNT
What's the average?  | AVG
What's the greatest? | MAX
What's the least?    | MIN
What's the sum?      | SUM

## Queries and Results

### How queries happen

![](/images/how-queries-happen.png)

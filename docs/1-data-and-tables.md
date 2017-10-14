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

When our code fetches data out of a database, it does so by sending a query (SQL). In response, the database will send us a result containing a new table with the data we asked for.

Depending on the environment, our code could be talking to a database over the network or it might be calling a library that keeps a database local. ![](/images/how-queries-happen.png)

```sql
// Examples of SQL

=# select 2+2;
 ?column?
----------
        4
(1 row)

=# select 2+2 as sum;
 sum
-----
   4
(1 row)
```

## Related Tables

A database will usually have several tables in it. A value with the same meaning can occur in different tables and have different column names. We can derive new tables by linking up existing tables using joins.

## Uniqueness and Keys

Whenever we want to unambiguously relate a row of one table to a row of another, we have to have a unique value. It's common that most database systems do that for you.

A column that uniquely identifies the rows in a table can be called a **primary key**.

## Joining Tables

Joining those tables gives us a new table, which contains information taken from the two original tables. The join statement has a join condition that tells the database how to match up the rows from one table with the rows from the other.

animals:

name (string) | species (string) | birthdate (date)
------------- | ---------------- | ----------------
Max           | gorilla          | 2001-04-13
Sue           | gorilla          | 1998-06-12
Max           | moose            | 2012-02-20
Alison        | llama            | 1997-11-24
George        | gorilla          | 2011-01-09
Spot          | iguana           | 2010-07-23
Ratu          | orangutan        | 1989-09-15
Eli           | llama            | 2002-02-22

diet:

species (string) | food (string)
---------------- | -------------
llama            | plants
brown bear       | fish
brown bear       | meat
brown bear       | plants
orangutan        | plants
orangutan        | insects

This SQL query will join the two tables to find out what foods each animal can eat:

```sql
SELECT animals.name,
       animals.species,
       diet.food
  FROM animals
  JOIN diet ON animals.species = diet.species
  WHERE diet.food = 'fish';
```

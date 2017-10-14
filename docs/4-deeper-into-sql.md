# Deeper Into SQL

## Normalized Design

Normalization is the central idea in database design. In a normalized database, the relationships among the tables match the relationships that are really there among the data.

- [A Simple Guide to 5 Normal Forms in Relational Database Theory](http://www.bkent.net/Doc/simple5.htm)
- [Wikipedia: Database Normalization](https://en.wikipedia.org/wiki/Database_normalization)

### Simple Rules of Normalization

1. **Every row has the same number of columns.** In practice, the database system won't let us literally have different numbers of columns in different rows. But if we have columns that are sometimes empty (null) and sometimes not, or if we stuff multiple values into a single field, we're bending this rule.

2. **There is a unique _key_, and everything in a row says something about the key.** The key may be one column or more than one. It may even be the whole row, as in the diet table. But we don't have duplicate rows in a table.

3. **Facts that don't belong to a key belong in a different table.** The example here was the items table, which had items, their locations, and the location's street addresses in it. The address isn't a fact about the item; it's a fact about the location. Moving it to a separate table saves space and reduces ambiguity, and we can always reconstitute the original table using a join.

4. **Tables shouldn't imply relationships that don't exists.** The example here was the job_skills table, where a single row listed one of a person's technology skills (like 'Linux') and one of their language skills (like 'French'). This made it look like their Linux knowledge was specific to French, or vice versa ... when that isn't the case in the real world. Normalizing this involved splitting the tech skills and job skills into separate tables.

Note: You will sometimes hear about denormalization as an approach to making database queries faster by avoiding joins.

## Create Table and Types

Adding a new table to your database is done with the create table command.

```sql
create table tablename (
  column1 type [constraints],
  column2 type [constraints],
          .
          .
          .
  [row constraints]
);
```

Note that different database systems support many different data types. ![](/images/db-types.png)

## Creating and Dropping

- `create database name [options]`
- `drop database name [options]`
- `drop table name [options]`

Note: The [options] are described in the PostgreSQL docs.

After creating a database in `psql`, connect to it with: `\c name`

### Serial: a PostgreSQL-specific way to create an autoincrementing column.

When creating a table with data type serial and insert data into rows, the database does a bunch of work for us to make sure the serial column gets populated correctly. It creates a sequence which is an internal data structure and sets it as the default value for the column.

## Declaring Primary Key

Primary keys are meant to uniquely identify the thing that a row is about.

```sql
create table students (
  id serial primary key,
  name text,
  birthdate date
);

create table postal_places (
  postal_code text,
  country text,
  name text,
  primary key(postal_code, country)
);
```

If a duplicate primary key is attempting to be added to the database, it will signal some type of error.

## Declaring relationships

`references` provides referential integrity - columns that are supposed to refer to each other are guaranteed to do so.

```sql
create table sales (
  sku text references products,
  sale_date date,
  count integer
);
```

## Foreign Keys

A column with a references constraint is referred to as a **foreign key**. A foreign key is a column or set of columns in a table, that uniquely identifies rows in another table.

```sql
create table students (
  id serial primary key,
  name text
);

create table courses (
  id text primary key,
  name text
);

create table grades (
  student integer references students (id),
  course text references courses (id),
  grade text
);
```

## Self Joins

A self join is a join in which a table is joined with itself. ![](/images/self-join.png)

```python
# This query is intended to find pairs of roommates.
# Simple self join style

QUERY = '''
SELECT a.id, b.id, a.building, a.room
  FROM residences AS a, residences AS b
  WHERE a.building = b.building AND a.room = b.room
  ORDER BY a.building, a.room;
'''
```

Results:

id     | id     | building | room
------ | ------ | -------- | ----
413001 | 881256 | Crosby   | 10
496747 | 741532 | Crosby   | 19
612413 | 931027 | Crosby   | 31
170267 | 958827 | Dolliver | 1
104131 | 707536 | Dolliver | 14
477801 | 505241 | Dolliver | 8
118199 | 824292 | Kendrick | 1A
105540 | 231742 | Kendrick | 3B

## Left Join (aka Left Outer Join) and Right Join

The LEFT JOIN keyword returns all records from the left table (table1), and the matched records from the right table (table2). The result is NULL from the right side, if there is no match.

```sql
-- Here are two tables describing bugs found in some programs.
create table programs (
   name text,
   filename text
);
create table bugs (
   filename text,
   description text,
   id serial primary key
);

-- The query below is intended to count the number of bugs in each program.
SELECT programs.name, count(bugs.id) AS num
  FROM programs
  LEFT JOIN bugs ON programs.filename = bugs.filename
  GROUP BY programs.name
  ORDER BY num;
```

Results:

name               | num
------------------ | ---
Your Database Code | 0
Sweet Spreadsheet  | 2
Fancy Website      | 3

The RIGHT JOIN keyword returns all records from the right table (table2), and the matched records from the left table (table1). The result is NULL from the left side, when there is no match.

## SQL JOIN Cheatsheet

![](https://www.codeproject.com/KB/database/Visual_SQL_Joins/Visual_SQL_JOINS_orig.jpg)

## Subqueries

- [Info about Subqueries](https://sqlbolt.com/topic/subqueries)
- [Subquery Expressions](https://www.postgresql.org/docs/10/static/functions-subquery.html)

A subquery can be referenced anywhere a normal table can be referenced. Inside a FROM clause, you can JOIN subqueries with other tables, inside a WHERE or HAVING constraint, you can test expressions against the results of the subquery, and even in expressions in the SELECT clause, which allow you to return data directly from the subquery. They are generally executed in the same logical order as the part of the query that they appear in, as described in the last lesson.

Because subqueries can be nested, each subquery must be fully enclosed in parentheses in order to establish proper hierarchy. Subqueries can otherwise reference any tables in the database, and make use of the constructs of a normal query (though some implementations don't allow subqueries to use LIMIT or OFFSET).

### Scalar Subqueries

A scalar subquery is an ordinary SELECT query in parentheses that returns exactly one row with one column.

```sql
SELECT name, (SELECT max(pop) FROM cities WHERE cities.state = states.name)
    FROM states;
```

## Views

A **view** is a SELECT query stored in the database in a way that lets you use it like a table.

```sql
-- Example

CREATE VIEW course_size AS
  SELECT course_id, count(*) AS num
  FROM enrollment
  GROUP BY course_id;
```

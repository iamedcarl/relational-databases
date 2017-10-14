# Elements of SQL

## Data types in SQL

Here is a sample of data types that SQL supports. The exact list may differ from one database to another. Here is the list for [PostgreSQL].

Text and string types

- text - a string of any length, like Python str or unicode types.
- char(n) - a string of exactly n characters.
- varchar(n) - a string of up to n characters.

Numeric types

- integer - an integer value, like Python int.
- real - a floating-point value, like Python float. Accurate up to six decimal places.
- double precision - a higher-precision floating-point value. Accurate up to 15 decimal places.
- decimal - an exact decimal value.

Date and time types

- date - a calendar date; including year, month, and day.
- time - a time of day.
- timestamp - a date and time together.

_NOTE: Always put 'single quotes' around text strings and date/time values._

## Comparison Operators

The comparison operators in SQL are almost the same as the ones in Python: < for less than, > for greater than, != for not equal, <= for less than or equal, and so forth. One difference is that SQL uses = instead of == to represent equality. You can apply all the basic comparison operators to strings, numbers, dates, and other values.

## Select clauses

### where

The **where** clause expresses _restrictions_ -- filtering a table for rows that follow a particular rule. where supports equalities, inequalities, and boolean operators (among other things):

- `where species = 'gorilla'` - return only rows that have 'gorilla' as the value of the species column.
- `where name >= 'George'` - return only rows where the name column is alphabetically after 'George'.
- `where species != 'gorilla' and name != 'George'` - return only rows where species isn't 'gorilla' and name isn't 'George'.

### limit / offset

The **limit** clause sets a limit on how many rows to return in the result table. The optional **offset** clause says how far to skip ahead into the results. So **limit 10 offset 100** will return 10 results starting with the 101st.

### order by

The **order by** clause tells the database how to sort the results -- usually according to one or more columns. So **order by species, name** says to sort results first by the species column, then by name within each species. Ordering happens before limit/offset, so you can use them together to extract pages of alphabetized results. (Think of the pages of a dictionary.)

The optional **desc** modifier tells the database to order results in descending order -- for instance from large numbers to small ones, or from Z to A.

### group by

The **group by** clause is only used with aggregations, such as **max** or **sum**. Without a **group by** clause, a select statement with an aggregation will aggregate over the whole selected table(s), returning only one row. With a **group by** clause, it will return one row for each distinct value of the column or expression in the **group by** clause.

#### Query Example:

```python
# Write a query that returns all the species in the zoo, and how many animals of
# each species there are, sorted with the most populous species at the top.
#
# The result should have two columns:  species and number.
#
# The animals table has columns (name, species, birthdate) for each individual.

QUERY = "select species, count(*) as number from animals group by species order by number desc;"
```

Results:

species    | number
---------- | ------
gorilla    | 9
llama      | 9
orangutan  | 6
alpaca     | 5
ferret     | 5
jackal     | 5
sea lion   | 5
yak        | 5
zebra      | 5
iguana     | 4
moose      | 4
brown bear | 3
camel      | 3
dingo      | 3
hyena      | 3
platypus   | 3
raccoon    | 3
warthog    | 3
narwhal    | 2
unicorn    | 2
echidna    | 1
mongoose   | 1

## Insert: Adding rows

**insert into** _table_ ( column1, column2, ... ) **values** ( val1, val2, ... );

If the values are in the same order as the table's columns (starting with the first column), you don't have to specify the columns in the insert statement:

**insert into** _table_ **values** ( val1, val2, ... );

For instance, if a table has three columns (a, b, c) and you want to insert into a and b, you can leave off the column names from the insert statement. But if you want to insert into b and c, or a and c, you have to specify the columns.

A single insert statement can only insert into a single table. (Contrast this with the select statement, which can pull data from several tables using a join.)

## Joins

![](/images/join.png)

### More Join Practice

```python
#
# List all the taxonomic orders, using their common names, sorted by the number of
# animals of that order that the zoo has.
#
# The animals table has (name, species, birthdate) for each individual.
# The taxonomy table has (name, species, genus, family, t_order) for each species.
# The ordernames table has (t_order, name) for each order.
#
# Be careful:  Each of these tables has a column "name", but they don't have the
# same meaning!  animals.name is an animal's individual name.  taxonomy.name is
# a species' common name (like 'brown bear').  And ordernames.name is the common
# name of an order (like 'Carnivores').

EXPLICIT_QUERY = '''
SELECT ordernames.name, count(*) as num
  FROM ordernames
  JOIN taxonomy ON ordernames.t_order = taxonomy.t_order
  JOIN animals ON taxonomy.name = animals.species
  GROUP BY ordernames.name
  ORDER BY num DESC;
'''

SIMPLE_QUERY = '''
SELECT ordernames.name, count(*) as num
  FROM animals, taxonomy, ordernames
  WHERE animals.species = taxonomy.name
   AND taxonomy.t_order = ordernames.t_order
  GROUP BY ordernames.name
  ORDER BY num desc
'''
```

## Having: After Aggregating

The **having** clause works like the **where** clause, but it applies after **group by** aggregations take place. The syntax is like this:

```sql
select columns
  from tables
  group by column
  having condition;
```

It doesn't usually make sense to use **having** without **group by**.

A few more examples:

```sql
# explicit join style
SELECT food, count(animals.name) as num
  FROM diet
  JOIN animals ON diet.species = animals.species
  GROUP BY food
  HAVING num = 1
```

And here is another:

```sql
# simple join style
SELECT food, count(animals.name) as num
  FROM diet, animals
  WHERE diet.species = animals.species
  GROUP BY food
  HAVING num = 1
```

## The one thing SQL is terrible at...

Listing your tables and columns in a standard way or in code. You can only look it up at the database console:

- PostgreSQL: `\dt` and `\d` _tablename_
- MySQL: `show tables` and `describe` _tablename_
- SQLite: `.tables` and `.schema` _tablename_ -

[postgresql]: https://www.postgresql.org/docs/10/static/datatype.html

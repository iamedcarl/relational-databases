# Python DB-API

## What is a DB-API?

Functions called to connect to a database, to send queries, and to get results.

Each database system has it's own library.

![](/images/db-api.png)

## Writing code with DB-API

1. `import sqlite3` which is the library for the database we are using. For PostgreSQL, we would use `import psyco-pg`.
2. Connect to the database: `conn = sqlite3.connect("Cookies")`. The string "Cookies" is the name of the database to connect to. If using a database over a network, you may have to specify the hostname, username, password, and other information. The `connect(...)` returns a connect object which is good until you `close()` the connection.
3. Next, make a cursor object `cursor = conn.cursor()`. The cursor that runs queries and fetches results.

  ```python
  cursor.execute("select ...")
  results_all = cursor.fetchall()
  results_one = cursor.fetchone()
  ```

4. Finally, `conn.close()` the connection.

Below is an example of a python program that uses a DB-API library to query a database:

```python
import sqlite3

# Fetch some student records from the database.
db = sqlite3.connect("students")
c = db.cursor()
query = "select name, id from students order by name;"
c.execute(query)
rows = c.fetchall()

# First, what data structure did we get?
print "Row data:"
print rows

# And let's loop over it too:
print
print "Student names:"
for row in rows:
  print "  ", row[0]

db.close()
```

Results:

```
Row data:
[(u'Diane Paiwonski', 773217), (u'Harry Evans-Verres', 172342), (u'Hoban Washburne', 186753), (u'Jade Harley', 441304), (u'Jonathan Frisby', 917151), (u'Melpomene Murray', 102030), (u'Robert Oliver Howard', 124816), (u'Taylor Hebert', 654321), (u'Trevor Bruttenholm', 162636)]

Student names:
   Diane Paiwonski
   Harry Evans-Verres
   Hoban Washburne
   Jade Harley
   Jonathan Frisby
   Melpomene Murray
   Robert Oliver Howard
   Taylor Hebert
   Trevor Bruttenholm
```

## Insert in DB-API

```python
import psycopg2

# Connecting to database 'testdb'
pg = psycopg2.connect("testdb")

# Create cursor object
c = pg.cursor()

# Insert data into the database
c.execute("insert into names values ('Kira Rose');")

# Commit to save changes. Always after 'execute'!
pg.commit()

# Close connection
pg.close()
```

### Atomicity

A transaction happens as a whole or not at all.

One thing nice about transactions and rollbacks is that if your code crashes in the middle of a database transaction, you can be sure it won't have written half-finished changes.

## PostgreSQL

The `psql` command-line tool is really powerful. There's a complete reference to it in the [PostgreSQL documentation].

- `\dt` - list all the tables in the database.
- `\dt+` - list tables plus additional information (notably, how big each table is on disk).
- `\H` - switch between printing tables in plain text vs. HTML.

Here's a fun one to run in `psql` while your forum web app is running:

```sql
select *
  from table \watch
```

(Note that `\watch` replaces the semicolon.) This will display the contents of the posts table and refresh it every two seconds, so you can see changes to the table as you use the app.

# Bobby Tables (SQL Injection)

![](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)

[A Guide Preventing SQL Injection](http://bobby-tables.com/)

**UNSAFE**: The problem with the below code is that user input gets inserted into the database in an unsafe way. Some text inserted are considered SQL code instead of text. For example if a user were to submit a post `); delete from posts; --`, it will delete all posts!

```python
db = psycopg2.connect(database=DBNAME)
c = db.cursor()
c.execute("insert into posts values ('%s')" % content)
...
```

**SAFE**: When we execute a query, we can put a `%s` in the query text and after a tuple parameter to execute the call.

```python
db = psycopg2.connect(database=DBNAME)
c = db.cursor()
c.execute("insert into posts values (%s)", (content,))
...
```

## Spammy Tables (Script Injection Attack)

Inserting the below to a database will treat it as text, but the browser will interpret as working JavaScript code. This will spam the forum every 2500 ms. This is a security problem called a script injection attack.

```html
<script>
setTimeout(function() {
    var tt = document.getElementById('content');
    tt.value = "<h2 style='color: #FF6699; font-family: Comic Sans MS'>Spam, spam, spam, spam,<br>Wonderful spam, glorious spam!</h2>";
    tt.form.submit();
}, 2500);
</script>
```

To solve this security issue, you can use the Python Bleach library that does HTML sanitization. [Bleach docs](http://bleach.readthedocs.io/en/latest/)

The best approach to handle this is to store the user's input, then clean the bad stuff out before displaying the posts back to users.

From one perspective, it doesn't matter what users put in the database as long as it doesn't break the database (like Bobby Tables) and it can't get out to harm other users.

_Output sanitization_ -- cleaning up users' posts before displaying them -- makes sure of the latter.

The downside of not sanitizing inputs is that we have to treat the contents of the database as possibly unsafe.

## Updating in DB-API

The syntax of the update statement:

```sql
update table
  set column = value
  where restriction ;
```

The **like** operator supports a simple form of text pattern-matching. Whatever is on the left side of the operator (usually the name of a text column) will be matched against the pattern on the right. The pattern is an SQL text string (so it's in 'single quotes') and can use the `%` sign to match any sub-string, including the empty string.

Example - finding 'salmon' in column 'fish':

```sql
... fish like 'salmon'
... fish like 'salmon%'
... fish like 'sal%'
... fish like '%n'
... fish like 's%n'
... fish like '%al%'
... fish like '%'
... fish like '%%%'
```

## Deleting from DB-API

The syntax for the delete command:

```sql
delete from table
  where restriction ;
```

[postgresql documentation]: https://www.postgresql.org/docs/10/static/app-psql.html

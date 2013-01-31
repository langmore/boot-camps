SQL and Relational Databases
============================

Setup instructions
------------------

1. Install Firefox
2. Install SQLite Manager for Firefox (Tools -> Add-ons -> Search -> SQLite Manager -> Install)
3. Restart Firefox
3. Download the [sample database](https://dl.dropbox.com/u/22164876/babynames.sqlite)
4. Open SQLite Manager: (Tools -> SQLite Manager)

Brief introduction
------------------

* Relational databases store data in structured tabular format
  * Structure here means columns and rows
* Each column has the same type
* Data from tables is retrieved using special queries
* SQL (Structured Query Language) is used to compose such queries

Goals
-----
* Understand common relational database concepts
* Learn basic SQL syntax
* Be able to get data using SQL using different criteria
* Be able to do groupby operations using SQL
* Be able to join tables together
* If there is time, basic insertion/deletion/updating

Setup our test database
-----------------------

1. Start a New Database **Database -> New Database**
2. Start the import **Database -> Import**
3. Select the file to import
4. Give the table a name (or use the default)
5. If the first row has column headings, check the appropriate box
6. Make sure the delimiter and quotation options are correct
7. Press **OK**
8. When asked if you want to modify the table, click **OK**
9. Set the data types for each field

***Exercise: Import the names and names_by_ethnicity tables***

Getting data
------------
An SQL query is generally composed of SQL keywords, names for
tables and columns, constants or variables. In the examples below,
<variable> is intended to be replaced by actual table, column, or
other values. While SQL is case insensitive, we will be using
ALL CAPS for keywords to make it easier to understand.

### Basic querying

To get data from a database, use the SQL **SELECT** keyword.

The simplest query is to get a single data table:

  SELECT * FROM <table>

The wildcard * indicates that all columns are selected.

***Concept check: write a query that returns all the data in the table named "names"

Often a data table contains more columns than we care about.
To get a single column, change the wildcard to a colum name:

  SELECT <column> FROM <table>

For example, ***SELECT year_of_birth FROM names*** will get just the birth years.
But that's not very useful, most of the time we want multiple columns from the table.
If you have multiple columns, just put them in a comma separated list:

  SELECT <col1>, <col2>, <col3>, ..., <colN> FROM <table>

***Concept check: select the year_of_birth, gender, and name from the "names" table.

### Sneak peek

Data tables can grow to be fairly large. It can be slow to select the entire table if
all you wanted to do was to see a few rows from the table. For this purpose there is
the LIMIT keyword:

  SELECT <columns> FROM <table> LIMIT <N>

This returns just the first N rows of data

### Summary stats

OK, so the "names" table is fairly large, but just how big is it? We can use the
**COUNT** keyword to find out!

  SELECT COUNT(*) FROM <table>

OK, that's a lot of data. What's the time span of the data collection?

  SELECT MIN(year_of_birth) FROM names

  SELECT MAX(year_of_birth) FROM names

### Calculated values

In addition to preset statistics, SQL allows you to compute many types of generic
derived data. Suppose we wanted the ratio of hispanic to black baby name counts,
we could write:

  SELECT year_of_birth, name, hispanic_count / black_count FROM names_by_ethnicity

When we run the query, the computation is evaluated for each row and appended to that row,
in a new column. Expressions can use any field, any arithmetic operators (+ - * /),
and built-in functions like ABS or ROUND.

To make the data more readable, we can use the AS keyword:

  SELECT year_of_birth, name, (hispanic_count / black_count) AS ratio
  FROM names_by_ethnicity

### Unique values

Woah, that's a lot of data! But there are lots of repeated names. I wonder just
how many distinct names there are. For that we have the DISTINCT keyword:

    SELECT DISTINCT <column> FROM <table>

If we select more than one column, then the distinct tuples of values are returned

    SELECT DISTINCT <col1>, <col2> FROM <table>

For example, "SELECT DISTINCT year_of_birth, name FROM names" would return both
(1920, MARY) and (1921, MARY), where as "SELECT DISTINCT name FROM names" would
return MARY only once.

***Concept check: write a query that returns the first 10% of the unique birth-year/name pairs
from the names table (hint: nesting is allowed)

Filtering
---------
We can already do quite a bit using the few keywords that we have learned so far.
However, the power of SQL goes much further than that. Using the WHERE keyword
along with a few other keywords and operators, we can contruct queries
that implements complex and precise filtering criteria.

Let's say we only wanted to look at data from the most recent year:

  SELECT * FROM names WHERE year_of_birth=2008

For string data, we need to wrap the value in quotes:

  SELECT * FROM names WHERE name="MARY"

The part of the query after the WHERE keyword is commonly called
the **WHERE clause**.

More than one criteria can be chained together to form a more complex
WHERE clause using operators like AND and OR. Comparison operators
like >, <, >=, <=, and != are also valid in WHERE clauses.

***Exercise: suppose I'm interested in WWII. How would I find out the number
of unique female names between 1939 and 1945?

### Order of operations

Parentheses are useful to create a tree of filtering criteria in the where clause.
Suppose I wanted to get asian male baby names that has more than 100 counts
and also black female baby names that has more than 125 counts:

  SELECT * FROM names_by_ethnicity
  WHERE (gender="MALE" AND asian_count>100) OR (gender="FEMALE" AND black_count>125)

***Question: what would be returned if the query wasn't specified using parentheses?

### Some short cuts

We saw before that we can use "year_of_birth >= 1939 AND year_of_birth <= 1945" to
specify a range of filtering values. To save us some typing, SQL has a BETWEEN
keyword to achieve the same effect:

  SELECT <columns> FROM <table> WHERE <col> BETWEEN <minimum> AND <maximum>

We can use OR to select multiple values, like **name = "MARY" OR name = "MICHELL"**
but it gets cumbersome if we have several values. SQL has a IN keyword (and NOT IN)
to save us time:

  SELECT <columns> FROM <table> WHERE <col> IN (<val1>, <val2>, ..., <valN>)

***Concept check: write a query to get all babies named Angela, Pamela, Monica,
Erica, or Rita between 1995 and 2000.

### Building complicated queries

Just a brief note on query strategy. Always make start simple and build it
up while testing to make sure the partial results are as you expected.
This will save you substantial debugging time.

Sorting
-------

Another very commonly used feature of SQL is sorting results via ORDER BY.
The names_by_ethnicity table is order by year, gender, and total count.
We could reorder the results so it's sorted by total count, year, then gender:

    SELECT * FROM names_by_ethnicity ORDER BY all_count, year_of_birth, gender

By default, the sorting is ascending. We could alternately use DESC to get descending order.

    SELECT name FROM names_by_ethnicity ORDER BY all_count DESC, year_of_birth ASC, gender DESC

Notice that the ordering columns don't appear in the SELECT. That's because the filtering and ordering
are performed before the column selection.

***Exercise: return the top 1000 rows of names_by_ethnicity sorted by the total rank, names
in reverse alphabetical order, then by gender.

***Exercise: Let’s try to combine what we’ve learned so far. Write a query that
returns the 100 most common hispanic male names from the year 2005.


Aggregation
-----------
Aggregation allows us to combine results group records based on value
and calculate combined values in groups.

We've already seen COUNT, MIN, and MAX from before. There are several
other aggregations like SUM and AVG.

So let's say we want to look at the results year by year and get
the total count of all baby names by year

    SELECT year_of_birth, COUNT(name) FROM names

Oops, what just happened?

The database counted all the names, but that's only one single number.
The query didn't specify how to match to the year_of_birth, so
the database engine just returned something willy-nilly.

In order to correct the query, we need to let the database know that
the year_of_birth should be used to form the groups within which the
counting operation will happen. This is done using the GROUP BY
keyword. Any aggregations will happen within group and any non-aggregated
columns in the query MUST also appear in the group by.

    SELECT year_of_birth, COUNT(name)
    FROM names
    GROUP BY year_of_birth

***Concept check: get the number of occurences of the most common name
by year and gender

***Exercise: for each gender and year of birth, get the
difference between the most common and least common real names
(i.e., exclude baby names "MALE" and "FEMALE") as a percentage
of the total number of babies by gender/year.

Joins
-----
To combine data from two tables we use the SQL JOIN command,
which comes after the FROM command.

    SELECT * FROM names JOIN names_by_ethnicity

Because we didn't specify HOW the tables should be joined
together, the database simply returned the cartesian
product of the rows. Just look at how many rows were in
our result!

In order to join tables in a useful manner, we need the ON
keyword to match related columns between the tables.

  SELECT * FROM names JOIN names_by_ethnicity
  ON names.year_of_birth = names_by_ethnicity.year_of_birth
  AND names.name = names_by_ethnicity.name
  AND names.gender = names_by_ethnicity.gender

Just like WHERE, ON can use AND/OR and the comparison operators
to join together complex join behavior.

Notice that when selecting using the wildcard, all columns from
all joined tables are returned. For equivalent columns, we can
simply use <table>.<column> to select it:

  SELECT names.year_of_birth, names.name, names.gender,
  names.count, names.rank, names_by_ethnicity.all_count, names_by_ethnicity.all_rank
  FROM names JOIN names_by_ethnicity
  ON names.year_of_birth = names_by_ethnicity.year_of_birth
  AND names.name = names_by_ethnicity.name
  AND names.gender = names_by_ethnicity.gender

***Exercise: check for discrepancies in the data and return rows where
the total counts differ between names and names_by_ethnicity

***Exercise: return the sum of the absolute difference of the total counts
between the two tables by year and gender for the top 20 ranked names
(ranking according to the names table)


Adding data to existing tables
------------------------------

INSERT INTO <table> VALUES (<comma separated list>)

Deleting data
-------------

DELETE FROM <table> WHERE <criteria>

Updating data
-------------

UPDATE <table> SET <col1>=<val1>, <col2>=<val2>
WHERE <criteria>
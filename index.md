---
---

Databases using SQL in R
==========================

Instructor: Mary Shelley

SQL 
------------------
SQL (Structured Query Language) is a high-level language for interacting with relational databases. 
Commands use intuitive English words but can be strung together and nested in powerful ways.

Connecting to the data
-----------------------
Recall the portal mammals database is available on pgstudio.research.sesync.org

Database Host: localhost
Database Name: portal
Username: student
Password: synthesis


Basic queries
-------------
Let’s write a SQL query that selects only the year column from the surveys
table.

    SELECT year FROM surveys;

	
A note on style: we have capitalized the words SELECT and FROM because they are SQL keywords.
Unlike R, SQL is case insensitive, but it helps for readability – good style. 

If we want information about other fields, we can just add a new column to the list of fields,
right after SELECT:
    
	SELECT year, month, day FROM surveys;
	
Or we can select all of the columns in a table using the wildcard *
    
	SELECT * FROM surveys;

### Limit

We can use the LIMIT statement to select only the first few rows. This is particularly helpful when getting
a feel for very large tables.

	SELECT year, species_id FROM surveys LIMIT 10;
		
### Unique values

If we want only the unique values so that we can quickly see what species have
been sampled we use ``DISTINCT``

    SELECT DISTINCT species_id FROM surveys;

If we select more than one column, then the distinct pairs of values are
returned

    SELECT DISTINCT year, species FROM surveys;
	
	
### Calculated values

We can also do calculations with the values in a query.
For example, if we wanted to look at the mass of each individual
on different dates, but we needed it in kg instead of g we would use

    SELECT year, month, day, weight/1000.0 FROM surveys;

When we run the query, the expression ``weight / 1000.0`` is evaluated for each row
and appended to that row, in a new column.  Expressions can use any fields, any
arithmetic operators (+ - * /) and a variety of built-in functions. For
example, we could round the values to make them easier to read.

    SELECT plot_id, species_id, sex, weight, ROUND(weight / 1000.0, 2) FROM surveys;

The underlying data in the wgt column of the table does not change. The query, which exists separately from the data,
simply displays the calculation we requested in the query result window pane. You can assign the new column a name by typing "AS weight_kg" after the expression

***EXERCISE: Write a query that returns
             the year, month, day, species ID, and weight in mg***

Filtering
---------

Databases can also filter data – selecting only the data meeting certain
criteria.  For example, let’s say we only want data for the species Dipodomys
merriami, which has a species code of DM.  We need to add a WHERE clause to our
query:

    SELECT * FROM surveys WHERE species_id='DM';

We can do the same thing with numbers.
Here, we only want the data since 2000:

    SELECT * FROM surveys WHERE year >= 2000;
	
We can use more sophisticated conditions by combining tests with AND and OR.
For example, suppose we want the data on Dipodomys merriami starting in the year
2000:

    SELECT * FROM surveys WHERE (year >= 2000) AND (species_id = 'DM');

Note that the parentheses aren’t needed, but again, they help with readability.
They also ensure that the computer combines AND and OR in the way that we
intend.

If we wanted to get data for any of the Dipodomys species,
which have species codes DM, DO, and DS we could combine the tests using OR:

    SELECT * FROM surveys WHERE (species_id = "DM") OR (species_id = "DO") OR (species_id = "DS");	

  
Building more complex queries
-----------------------------

Now, lets combine the above queries to get data for the 3 Dipodomys species from
the year 2000 on.  This time, let’s use IN as one way to make the query easier
to understand.  It is equivalent to saying ``WHERE (species = "DM") OR (species
= "DO") OR (species = "DS")``, but reads more neatly:

    SELECT * FROM surveys WHERE (year >= 2000) AND (species IN ("DM", "DO", "DS"));

    SELECT *
    FROM surveys
    WHERE (year >= 2000) AND (species IN ("DM", "DO", "DS"));

We started with something simple, then added more clauses one by one, testing
their effects as we went along.  For complex queries, this is a good strategy,
to make sure you are getting what you want.  Sometimes it might help to take a
subset of the data that you can easily see in a temporary database to practice
your queries on before working on a larger or more complicated database.

***EXERCISE: Write a query that returns
   The day, month, year, species ID, and weight (in kg) for
   individuals caught on Plot 1 that weigh more than 75 g***

Sorting
-------

We can also sort the results of our queries by using ORDER BY.
For simplicity, let’s go back to the species table and alphabetize it by taxa.

	"SELECT * FROM species ORDER BY taxa ASC;

The keyword ASC tells us to order it in Ascending order.
We could alternately use DESC to get descending order.

    SELECT * FROM species ORDER BY taxa DESC;

ASC is the default.

We can also sort on several fields at once.
To truly be alphabetical, we might want to order by genus then species.

    SELECT * FROM species ORDER BY genus ASC, species ASC;

***Exercise: Write a query that returns
             year, species, and weight in kg from the surveys table, sorted with
             the largest weights at the top***


Order of execution
------------------

Another note for ordering. We don’t actually have to display a column to sort by
it.  For example, let’s say we want to order by the species ID, but we only want
to see genus and species.

    SELECT genus, species FROM species ORDER BY species_id ASC;

We can do this because sorting occurs earlier in the computational pipeline than
field selection.

The computer is basically doing this:

1. Filtering rows according to WHERE
2. Sorting results according to ORDER BY
3. Displaying requested columns or expressions.


Order of clauses
----------------

The order of the clauses when we write a query is dictated by SQL: SELECT, FROM, WHERE, ORDER BY
and we often write each of them on their own line for readability.


***Exercise: Let's try to combine what we've learned so far in a single query.
Using the surveys table write a query to display the three date
fields, species ID, and weight in kilograms (rounded to two decimal places), for
rodents captured in 1999, ordered alphabetically by the species ID.***


Aggregation
-----------

Aggregation allows us to group records based on field values and
calculate combined values in groups (or for a table as a whole).

Let’s go to the surveys table and find out how many individuals there are.
Using the wildcard simply counts the number of records (rows)

    SELECT COUNT(*) FROM surveys;

We can also find out how much all of those individuals weigh.

    SELECT COUNT(*), SUM(weight) FROM surveys;

***Do you think you could output this value in kilograms, rounded to 3 decimal
   places?***

    SELECT ROUND(SUM(weight)/1000.0, 3) FROM surveys;

There are many other aggregate functions included in SQL including
MAX, MIN, and AVG.

***From the surveys table, can we use one query to output the total weight,
   average weight, and the min and max weights? How about the range of weight?***

Now, let's see how many individuals were counted in each species. We do this
using a GROUP BY clause

    SELECT species_ID, COUNT(*)
    FROM surveys
    GROUP BY species_ID;
					 
					 
GROUP BY tells SQL what field or fields we want to use to aggregate the data.
If we want to group by multiple fields, we give GROUP BY a comma separated list.

***EXERCISE: Write queries that return:***
***1. How many individuals were counted in each year***
***2. Average weight of each species in each year**
Hint: To exclude missing data from the average, we can use the SQL test for missing IS NULL (or in this case, IS NOT NULL)


We can order the results of our aggregation by a specific column, including the
aggregated column.  Let’s count the number of individuals of each species
captured, ordered by the count

    SELECT species_id, COUNT(*)
    FROM surveys
    GROUP BY species_id
    ORDER BY COUNT(*);

Joins
-----

To combine data from two tables we use the SQL JOIN command, which comes after
the FROM command.

We also need to tell the computer which columns provide the link between the two
tables using the word ON.  We want to join data with the same
species codes.

    SELECT *
    FROM surveys
    JOIN species ON surveys.species_id = species.species_id;")

ON is like WHERE, it filters things out according to a test condition.  We use
the table.colname format to tell the manager what column in which table we are
referring to.

We often won't want all of the fields from both tables, so anywhere we would
have used a field name in a non-join query, we can use *table.colname*

For example, what if we wanted information on when individuals of each
species were captured, but instead of their species ID we wanted their
actual species names.
	SELECT surveys.year, surveys.month, surveys.day, species.genus, species.species
	FROM surveys
	JOIN species ON surveys.species_id = species.species_id;

***Exercise: Write a query that returns the genus, the species, and the weight
   of every individual captured at the site***

Joins can be combined with sorting, filtering, and aggregation.  So, if we
wanted average mass of the individuals on each type of plot, we could use
	SELECT plots.plot_type, AVG(surveys.weight)
	FROM surveys
	JOIN plots
	ON surveys.plot = plots.plot_id
	GROUP BY plots.plot_type;
	

Using SQL from R
----------------
R, like most major programming languages, has a library for interacting with SQL databases of various kinds. 

To connect to a postgresql database, open RStudio and install/load the RPostgreSQL package.

	install.packages("RPostgreSQL")
	library(RPostgreSQL)

We need to "open a connection" to the database so that R can communicate with it. This will serve as a pipeline to send data back and forth.
It identifies which databsae we are using, where it lives, and verifies our credentials (user/pwd).

    drv <- dbDriver("PostgreSQL")
	dbHost <- "pgstudio.research.sesync.org"
	dbUser <- "student"
	dbName <- "portal"
	
	con <- dbConnect(drv, user=dbUser, host=dbHost, dbname = dbName, password=.rs.askForPassword("Enter password:"))
	
Now we can use this connection to read the output of any SQL query directly into a dataframe in R
    surv <- dbGetQuery(con, "SELECT * FROM surveys WHERE year > 2000;")

We can also write data into the database from R. dbWriteTable can be used to create a new table.
	d <- data.frame(x=letters, y=1:26)
	dbWriteTable(con, "mks", d )
	
We can check the results in a number of ways, including looking in pgstudio or querying through ROUND
	dbGetQuery(con,"select * from mks;")	
	
We can also add rows to an existing table using dbWriteTable
	d2<-data.frame(x="AA",y=27)
	dbWriteTable(con, "mks", d2, append=T)

Once we're done with a session, it's good practice to close the connection because there is a limit to the total number of connections the server can support at any one time	
	dbDisconnect(con)	
	
-----------------------
Recall the portal mammals database is available on pgstudio.research.sesync.org

Database Host: localhost
Database Name: portal
Username: student
Password: synthesis

	
	   
Additional Resources and Information
------------------------------------
* A few types of queries in SQL, in addition to SELECT, will cover most of what you might want to do.
  UPDATE change column values; CREATE generates  a new, blank table; DELETE removes rows from a table. 	   
  All of these can employ the concepts of calculation, filtering, aggregation, and joining in their execution.
 
* Database design tips: https://www.periscope.io/blog/better-sql-schema.html

* Documentation for the RPostgreSQL package: https://cran.r-project.org/web/packages/RPostgreSQL/RPostgreSQL.pdf
 
	   
Adapted by Mary Shelley for SESYNC ci-spring 2016 from Data Carpentry SQL lesson, authored by Ethan White
https://github.com/datacarpentry/archive-datacarpentry/blob/master/lessons/sql/sql.md

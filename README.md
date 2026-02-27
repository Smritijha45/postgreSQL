# Notes on PostgreSQL

## Database Administration

### Database Creation

The minimum SQL command to create a database is:

```sql
CREATE DATABASE mydb;
```

This creates a copy of the *template1* database. Any role with `CREATEDB` privilege can create new databases. Always use *template1*. Do not edit *template0*. *template0* is your backup in case you've screwed *template1* up.

### Using Schemas

Schemas organize your database into logical groups. If you have dozens of tables in your database, consider sorting them into schemas. Objects must have unique names within a schema but need not be unique across the database. For example, if you are a car dealer, you can place all tables of cars you own and their maintenance records into a cars schema. Place all your staff into an employees schema and place all customer-related information into a customers schema.

Another common way to organize schemas is by roles. This can be particularly handy with applications that serve multiple clients whose data must be kept separate (software multitenancy). Think Cloud-PBX, Logging-SaaS, etc.

```sql
CREATE SCHEMA customer1;
CREATE SCHEMA customer2;
```

Put the same tables into each schema and insert records into the schema that corresponds with the client. Finally create different login roles for each schema with the **same name as the schema**. Products are now completely isolated in their respective schemas. When customers log in to your database to do stuff, they will be able to access only information pertaining to their own product.

### Backup

PostgreSQL ships with three utilities for backup: *pg_dump*, *pg_dumpall*, and *pg_basebackup*.

Use *pg_dump* to back up specific databases. To back up all databases in plain text along with server globals, use *pg_dumpall*, which needs to run under a superuser
account so that it back up all databases. Use *pg_basebackup* to do system-level disk backup of all databases.

Two popular open source ones you might want to consider are [pgBackRest](https://pgbackrest.org/) and [Barman](https://www.pgbarman.org/). These offer additional features like backup scheduling, multiserver support, and restore shortcuts.

To create a compressed, single database backup:

```bash
pg_dump -h localhost -p 5432 -U someuser -F c -b -v -f mydb.backup mydb
```

To create a plain-text single database backup, including a -C option, which stands for `CREATE DATABASE`:

```bash
pg_dump -h localhost -p 5432 -U someuser -C -F p -b -v -f mydb.backup mydb
```

Use the *pg_dumpall* utility to back up all databases on a server into a single plain-text file. This comprehensive backup automatically includes server globals such as table‐space definitions and roles.

It’s a good idea to back up globals on a daily basis. Although you can use *pg_dumpall* to back up databases as well, we prefer backing up databases individually using *pg_dump* or using *pg_basebackup* to do a PostgreSQL service-level backup. Restoring from a huge plain-text backup can take a very long time. Using *pg_basebackup* in conjunction with streaming replication is the fastest way to recover from major server failure.

Backup all globals and tablespace definitions only:

```bash
pg_dumpall -h localhost -U postgres --port=5432 -f myglobals.sql --globals-only
```

### Restore

There are two ways to restore data in PostgreSQL from backups created with *pg_dump* or *pg_dumpall*:

* Use *psql* to restore plain-text backups generated with *pg_dumpall* or *pg_dump*.
* Use *pg_restore* to restore compressed, TAR, and directory backups created with *pg_dump*.

To restore a plain-text backups and stop if any errors is found:

```bash
psql -U postgres --set ON_ERROR_STOP=on -f myglobals.sql
```

To restore to a specific database:

```bash
psql -U postgres -d mydb -f select_objects.sql
```

If you backed up using *pg_dump* and chose a format such as TAR, custom, or directory, you have to use the *pg_restore* utility to restore.

Some of *pg_restore*’s features are:

* Perform parallel restores using the -j (equivalent to --jobs= ) option to indicate the number of threads to use.
* Generate a table of contents file from your backup file to check what has been backed up.
* Selectively restore, even from within a backup of a full database.
* *pg_restore* is (mostly) backward-compatible. You can back up a database on an older version of PostgreSQL and restore to a newer version.

To perform a restore using *pg_restore*, first create the database anew using SQL:

```sql
CREATE DATABASE mydb;
```

Then restore:

```bash
pg_restore --dbname=mydb --jobs=4 --verbose mydb.backup
```

If the name of the database is the same as the one you backed up, you can create and restore the database in one step:

```bash
pg_restore --dbname=postgres --create --jobs=4 --verbose mydb.backup
```

> A restore will not re-create objects already present in a database. If you have data in the database, and you want to replace it with what’s in the backup, you need to add the `--clean` switch to the *pg_restore* command.

## Basic Table Creation

```sql
CREATE TABLE logs (
  log_id serial PRIMARY KEY,
  user_name varchar(50),
  description text,
  log_ts timestamp with time zone NOT NULL DEFAULT current_timestamp
);
CREATE INDEX idx_logs_log_ts ON logs USING btree (log_ts);
```

1. `serial` is the data type used to represent an incrementing autonumber. Adding a serial column automatically adds an accompanying sequence object to the database schema. A serial data type is always an integer with the default value set to the next value of the sequence object. Each table usually has just one serial column, which often serves as the primary key. For very large tables, you should opt for the related `bigserial`.

2. `varchar` is shorthand for "character varying", a variable-length string similar to what you will find in other databases. You don’t need to specify a maximum length; if you don’t, `varchar` will be almost identical to the text data type.

3. `text` is a string of indeterminate length. It’s never followed by a length restriction.

4. `timestamp` with time zone (shorthand `timestamptz`) is a date and time data type, always stored in UTC. It displays date and time in the server’s own time zone unless you tell it to otherwise.

## Identity

New in version 10 is the `IDENTITY` qualifier for a column. `IDENTITY` is a more standard-compliant way of generating an autonumber for a table column.

```sql
CREATE TABLE logs (
  log_id int GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  user_name varchar(50),
  description text,
  log_ts timestamp with time zone NOT NULL DEFAULT current_timestamp
);
CREATE INDEX idx_logs_log_ts ON logs USING btree (log_ts);
```

Under what cases would you prefer to use `IDENTITY` over serial? The main benefit of the `IDENTITY` construct is that an identity is always tied to a specific table, so incrementing and resetting the value is managed with the table. A serial, on the other hand, creates a sequence object that may or may not be reused by other tables and needs to be dropped manually when it’s no longer needed. If you wanted to reset the number of a serial, you’d need to modify the related `SEQUENCE` object, which means knowing what the name of it is.

## Indexes

> Index names must be unique within a given schema.

### Operator Classes

> Why is the planner not taking advantage of my index?

A particular index will work only against a given set of opclasses. To see this complete list, execute the query to get a comprehensive view.

```sql
SELECT
  am.amname AS index_method,
  opc.opcname AS opclass_name,
  opc.opcintype::regtype AS indexed_type,
  opc.opcdefault AS is_default
FROM pg_am am INNER JOIN pg_opclass opc ON opc.opcmethod = am.oid
WHERE am.amname = 'btree'
ORDER BY index_method, indexed_type, opclass_name;
```

index_method | opclass_name | indexed_type | is_default
------------ | ------------ | ------------ | ----------
btree | bool_ops | boolean | true
btree | text_ops | text | true
btree | text_pattern_ops | text | false
btree | varchar_ops | text | false
btree | varchar_pattern_ops | text | false

B-Tree against `text_ops` (aka `varchar_ops` ) doesn’t include the `~~` operator (the `LIKE` operator), so none of your `LIKE` searches can use an index in the `text_ops` opclass. If you plan on doing many wildcard searches on `varchar` or `text` columns, you’d be better off explicitly choosing the `text_pattern_ops` / `varchar_pattern_ops` opclass for your index. To specify the opclass, just append the opclass after the column name, as in:

```sql
CREATE INDEX idx1 ON census.lu_tracts USING btree (tract_name text_pattern_ops);
```

Finally, remember that each index you create works against only a single opclass. If you would like an index on a column to cover multiple opclasses, you must create separate indexes. To add the default index text_ops to a table, run:

```sql
CREATE INDEX idx2 ON census.lu_tracts USING btree (tract_name);
```

Now you have two indexes against the same column. (There’s no limit to the number of indexes you can build against a single column.) The planner will choose `idx2` for basic equality queries and `idx1` for comparisons using `LIKE`.

See the article [Why is My Index Not Used?](http://www.postgresonline.com/journal/archives/78-Why-is-my-index-not-being-used.html) for more info on troubleshooting index issues.

### Functional Indexes

PostgreSQL lets you add indexes to functions of columns. Functional indexes prove their usefulness in mixed-case textual data. PostgreSQL is a case-sensitive database. To perform a case-insensitive search you could create a functional index:

```sql
CREATE INDEX idx ON featnames_short
USING btree (upper(fullname) varchar_pattern_ops);
```

This next example uses the same function to uppercase the fullname column before comparing. Since we created the index with the same `upper(fullname)` expression, the planner will be able to use the index for this query:

```sql
SELECT fullname FROM featnames_short WHERE upper(fullname) LIKE 'S%';
```

### Partial Indexes

Partial indexes (sometimes called filtered indexes) are indexes that cover only rows fitting a predefined `WHERE` condition.

```sql
CREATE TABLE subscribers (
  id serial PRIMARY KEY,
  name varchar(50) NOT NULL,
  is_active boolean);
```

We add a partial index to guarantee uniqueness only for current subscribers:

```sql
CREATE UNIQUE INDEX uq ON subscribers USING btree(lower(name)) WHERE is_active;
```

> Functions used in the index’s `WHERE` condition must be immutable. This means you can’t use time functions like `CURRENT_DATE` or data from other tables (or other rows of the indexed table) to determine whether a record should be indexed. For example, this index is both PARTIAL and functional because what it indexes is `upper(name)` (not `name`). For an easy way to not have to worry about this, check out *Partial Indexes* in chapter 6 of the [book](http://shop.oreilly.com/product/0636920052715.do).

## Views

### Single Table Views

```sql
CREATE OR REPLACE VIEW finance.vw_activities_2018 AS
SELECT activity_type_id, val, yr, activity_id FROM finance.activities WHERE yr = 2018;
```

As of version 9.3, you can alter the data in this view by using `INSERT`, `UPDATE`, or `DELETE` commands. Updates and deletes will abide by any `WHERE` condition you have as part of your view. Check out *Single Table Views* in chapter 7 of the [book](http://shop.oreilly.com/product/0636920052715.do) for more info on how to do that.

### Handy Constructions

See *Handy Constructions* in chapter 7 of the [book](http://shop.oreilly.com/product/0636920052715.do) for more info on how each shortcut works.

* `DISTINCT ON` behaves like `DISTINCT`, but with two enhancements: you can specify which columns to consider as distinct and to sort the remaining columns.
* `LIMIT` and `OFFSET`
* Shorthand Casting allows casting of the text `2011-1-11` to a date by using `'2011-1-11'::date` instead of `CAST('2011-1-11' AS date)`
* Multirow Insert
* `ILIKE` for Case-Insensitive Search
* `ANY` Array Search
* Set-Returning Functions in `SELECT`
* Restricting `DELETE`, `UPDATE`, and `SELECT` from Inherited Tables
* `DELETE USING`
* Returning affected records to the User
* `UPSERT`s: `INSERT ON CONFLICT UPDATE`
* Composite Types in Queries
* Dollar Quoting
* `DO`
* `FILTER` Clause for Aggregates
* Percentiles and Mode

## Writing Functions

Basic function structure:

```sql
CREATE OR REPLACE FUNCTION func_name(arg1 arg1_datatype DEFAULT arg1_default)
RETURNS some type | set of some type | TABLE (..) AS
$$
BODY of function
$$
LANGUAGE language_of_function
```

Arguments can have default values, which allow the caller of the function to omit them. Optional arguments must be positioned after nonoptional arguments in the function definition.

Argument names are optional but are useful because they let you refer to an argument by name inside the function body. For example, think of a function that is defined to take three input arguments (two being optional):

```sql
log_event(msg text, severity numeric DEFAULT 3, facility text DEFAULT 'router')
```

You can refer to the arguments by name (`msg`, `severity`, etc.) inside the body of the function. If they are not named, you need to refer to the arguments inside the function by their order in the argument list: $1, $2, and $3.

If you name the arguments, you also have the option of using named notation when calling the function:

```sql
log_event(msg => 'Some message', severity => 2)
```

### Basic SQL Function

This example shows a primitive SQL function that inserts a row into a table and returns a scalar value:

```sql
CREATE OR REPLACE FUNCTION write_to_log(param_user_name varchar, param_description text)
RETURNS integer AS
$$
INSERT INTO logs(user_name, description) VALUES($1, $2)
RETURNING log_id;
$$
LANGUAGE 'sql' VOLATILE;
```

To call the function, execute something like:

```sql
SELECT write_to_log('mike', 'Logged out at 09:55 AM.') AS new_id;
```

## Query Performance Tuning


### EXPLAIN

```sql
EXPLAIN (ANALYZE)
SELECT artist_id, artist_name
FROM catalog.artists
WHERE artist_name = 'Daft Punk';
```

Using `EXPLAIN` alone gives us estimated plan costs. Using `EXPLAIN` in conjunction with `ANALYZE` gives us both estimated and actual costs to execute the plan.

> Get a graphical break down of the output by pasting the result into [PostgreSQL's explain analyze made readable](https://explain.depesz.com/).

An even better visualizer is [Postgres EXPLAIN Visualizer (Pev)](http://tatiyants.com/pev). For best results, use `EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS, FORMAT JSON)`. Psql users can export the plan to a file using:

```bash
psql -qAt -f explain.sql > analyze.json
```

## Replication and External Data


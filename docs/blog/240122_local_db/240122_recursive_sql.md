# Recursive SQL

Using the fresh local PostgreSQL database from the [previous chapter](./240122_local_db.md), we'll explore recursive SQL queries.

## Example data

Let's create a table with some example data. This example is inspired by the PostgreSQL Tutorial on [PostgreSQL Recursive Query](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-recursive-query/). Using employees and reporting lines are just an intuitive example, we can imagine that this could be any hierarchical data.

Create two tables, `employees` and `reporting_lines` in your database. Already, we have slightly modified the tutorial to use a foreign key constraint on the `reporting_lines` table. The reason for this will become clear later.

```sql
DROP TABLE IF EXISTS employees;
DROP TABLE IF EXISTS reporting_lines;
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    full_name VARCHAR NOT NULL
);

CREATE TABLE reporting_lines (
    employee_id INT NOT NULL,
    manager_id INT NOT NULL,
    PRIMARY KEY (employee_id, manager_id),
    FOREIGN KEY (employee_id) REFERENCES employees (employee_id),
    FOREIGN KEY (manager_id) REFERENCES employees (employee_id)
);
```

```sql
INSERT INTO employees (employee_id, full_name)
VALUES
    (1, 'Michael North'),
    (2, 'Megan Berry'),
    (3, 'Sarah Berry'),
    (4, 'Zoe Black'),
    (5, 'Tim James'),
    (6, 'Bella Tucker'),
    (7, 'Ryan Metcalfe'),
    (8, 'Max Mills'),
    (9, 'Benjamin Glover'),
    (10, 'Carolyn Henderson'),
    (11, 'Nicola Kelly'),
    (12, 'Alexandra Climo'),
    (13, 'Dominic King'),
    (14, 'Leonard Gray'),
    (15, 'Eric Rampling'),
    (16, 'Piers Paige'),
    (17, 'Ryan Henderson'),
    (18, 'Frank Tucker'),
    (19, 'Nathan Ferguson'),
    (20, 'Kevin Rampling');

INSERT INTO reporting_lines (employee_id, manager_id)
VALUES
    (2, 1),
    (3, 1),
    (4, 1),
    (5, 1),
    (6, 2),
    (7, 2),
    (8, 2),
    (9, 2),
    (10, 3),
    (11, 3),
    (12, 3),
    (13, 4),
    (14, 4),
    (15, 4),
    (16, 7),
    (17, 7),
    (18, 8),
    (19, 8),
    (20, 8);
```

We then create a view to make it easier to query the data. This view is equivalent to the tutorials `employees` table from the tutorial!

```sql
CREATE VIEW employees_with_managers AS
SELECT
    e.employee_id,
    e.full_name,
    r.manager_id
FROM
    employees e
LEFT JOIN reporting_lines r ON e.employee_id = r.employee_id;
```

```sql
SELECT
    *
FROM
    employees_with_managers;
```

```sql
 employee_id |     full_name     | manager_id
-------------+-------------------+------------
           2 | Megan Berry       |          1
           3 | Sarah Berry       |          1
           4 | Zoe Black         |          1
           5 | Tim James         |          1
           6 | Bella Tucker      |          2
           7 | Ryan Metcalfe     |          2
           8 | Max Mills         |          2
           9 | Benjamin Glover   |          2
          10 | Carolyn Henderson |          3
          11 | Nicola Kelly      |          3
          12 | Alexandra Climo   |          3
          13 | Dominic King      |          4
          14 | Leonard Gray      |          4
          15 | Eric Rampling     |          4
          16 | Piers Paige       |          7
          17 | Ryan Henderson    |          7
          18 | Frank Tucker      |          8
          19 | Nathan Ferguson   |          8
          20 | Kevin Rampling    |          8
           1 | Michael North     |
(20 rows)
```

## Recursive queries

Recursive queries are useful when we have hierarchical data. For example, we can use recursive queries to find all the employees that report to a given manager.

Let's say we want to find all the employees that report to `Megan Berry` (employee with `employee_id` 2). We can do this with a recursive query.

When writing a recursive query, we need to define two parts:

- **Anchor member** - the first part of the query, the part that doesn't reference the recursive part
- **Recursive member** - the second part of the query, the part that references the recursive part

The anchor member is the query that returns the first row(s). Since we are interested in `Megan Berry`'s subordinates, the anchor member becomes:

```sql
SELECT
    employee_id,
    full_name
FROM
    employees_with_managers
WHERE
    employee_id = 2;
```

```
 employee_id |  full_name
-------------+-------------
           2 | Megan Berry
(1 row)
```

The recursive member is the query that references the recursive part. In our case, we want to find all the employees that report to `Megan Berry`. We can do this with the following query:

```sql
SELECT
    e.employee_id,
    e.full_name
FROM
    employees_with_managers e
INNER JOIN
    subordinates s ON e.employee_id = s.manager_id;
```

```
 employee_id |     full_name
-------------+-------------------
           6 | Bella Tucker
           7 | Ryan Metcalfe
           8 | Max Mills
           9 | Benjamin Glover
(4 rows)
```

We can combine these two queries into a single recursive query using the `UNION` operator:

```sql
WITH RECURSIVE subordinates AS (
    SELECT
        employee_id,
        manager_id,
        full_name
    FROM
        employees_with_managers
    WHERE
        employee_id = 2

    UNION

    SELECT
        e.employee_id,
        e.manager_id,
        e.full_name
    FROM
        employees_with_managers e
    INNER JOIN subordinates s ON s.employee_id = e.manager_id
)
SELECT * FROM subordinates;
```

```
 employee_id | manager_id |    full_name
-------------+------------+-----------------
           2 |          1 | Megan Berry
           6 |          2 | Bella Tucker
           7 |          2 | Ryan Metcalfe
           8 |          2 | Max Mills
           9 |          2 | Benjamin Glover
          16 |          7 | Piers Paige
          17 |          7 | Ryan Henderson
          18 |          8 | Frank Tucker
          19 |          8 | Nathan Ferguson
          20 |          8 | Kevin Rampling
(10 rows)
```

This query returns all the employees that report to `Megan Berry`, including subordinates of subordinates, and so on.

## `UNION` vs `UNION ALL` in recursive queries

The `UNION` operator removes duplicate rows, whereas `UNION ALL` does not. In our example, we don't have any duplicate rows, so we can use either `UNION` or `UNION ALL`.

As stated in the beginning of this post, we have modified the original tutorial a bit. We did this to now be able to illustrate the difference between `UNION` and `UNION ALL` in recursive queries. In the original example, a employee could not report to more than one manager. With this set up we can, so let's imagine that we suddenly have a circular reporting line. For example, `Megan Berry` now also reports to `Kevin Rampling`:

```sql
INSERT INTO reporting_lines (employee_id, manager_id)
VALUES
    (2, 20);
```

If we run our recursive query from above (using `UNION`), we get the following result:

```
 employee_id | manager_id |    full_name
-------------+------------+-----------------
           2 |          1 | Megan Berry
           2 |         20 | Megan Berry
           6 |          2 | Bella Tucker
           7 |          2 | Ryan Metcalfe
           8 |          2 | Max Mills
           9 |          2 | Benjamin Glover
          16 |          7 | Piers Paige
          17 |          7 | Ryan Henderson
          18 |          8 | Frank Tucker
          19 |          8 | Nathan Ferguson
          20 |          8 | Kevin Rampling
```

However, if we use `UNION ALL`, we notice that the docker container we are using crashes (after a while)! Let's try to understand why. The query, although using "recursive" as keyword, is run iteratively. The first iteration returns the anchor member:

```
 employee_id | manager_id |    full_name
-------------+------------+-----------------
           2 |          1 | Megan Berry
           2 |         20 | Megan Berry
```

The _recursive_ member's first iteration returns the direct subordinates(s) of `Megan Berry`:

```
 employee_id | manager_id |    full_name
-------------+------------+-----------------
           6 |          2 | Bella Tucker
           7 |          2 | Ryan Metcalfe
           8 |          2 | Max Mills
           9 |          2 | Benjamin Glover
```

The recursive member's second iteration returns the direct subordinates(s) of the above rows.

```
 employee_id | manager_id |    full_name
-------------+------------+-----------------
          16 |          7 | Piers Paige
          17 |          7 | Ryan Henderson
          18 |          8 | Frank Tucker
          19 |          8 | Nathan Ferguson
          20 |          8 | Kevin Rampling
```

The recursive member's third iteration returns the direct subordinates(s) of the above rows. Employees 16-19 do not have any subordinates, but employee 20 does: `Megan Berry`! This is where the circular reference occurs. When using `UNION`, the duplicate row is removed and the query terminates because there are no more new rows to process. However, when using `UNION ALL`, the duplicate row is not removed and the query continues to run forever.

What can we do to mitigate the risk of running a query which won't terminate? Can we use a `LIMIT` clause? Unfortunately not, since it is only applied tp the final result.

As a safety net, we can set the timeout for the session. In Postgres, this is done like so:

```sql
SET statement_timeout = 10000; -- 10 seconds
```

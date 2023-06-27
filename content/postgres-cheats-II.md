+++
title = "Postgres Notes II"
date = 2023-06-27
category = "Prog"
draft = false

[taxonomies]
tags = ["postgres"]
+++

Part II of the Postgres notes

<!-- more -->

## Joins

Joins let us fetch data that is shared among different tables.

Types of joins

1. Inner Join - only returns values that are present on both of the tables. If a column is empty in any of the columns being matched, it is omitted.
2. Left Join - will return everything for the table on the left even if, it's counterpart on the right is null.
3. Right Join - inverse of Left Join (left join^-1)
4. Full Join - combined the results of the left and the right joins.
5. Cross Join - can be used to generate all possible pairings or combinations between two tables.
6. Self Join - useful when you want to combine rows from the same table based on a condition. (Use chatgpt for a visual example)

## Unions

They are used to concatenate rows from different sources.

Our students schema

```sql
  student_id SERIAL PRIMARY KEY,
  first_name TEXT,
  age SMALLINT,
  birth_date DATE,
```

It is also possible to do pattern matching with sql. For example, using our students schema, what if we wanted to search for all student's who's name began with letter `J`

```sql
SELECT first_name, student_id
FROM students
WHERE first_name SIMILAR TO 'J%'
```

To match the ending, i.e get students whose names end with an 'h'

```sql
SELECT first_name, student_id
FROM students
WHERE first_name SIMILAR TO 'J%'
```

## REGEX

Here is a short primer on [regular expressions](https://youtu.be/85pG_pDkITY?t=5135).

suppose we wanted to get a students name whose first_name starts with 'Be' and last_name ends with

```sql
SELECT first_name, student_id
FROM students
WHERE first_name ~ '^Be'
AND last_name ~ 'mi$'
```

You can also use logical or, e.g to search whether a last_name ends with `rk` or `rc`

```sql
SELECT first_name, last_name
FROM students
WHERE last_name ~ 'rc|rk'
```

The `group by` clause is very useful when arranging identical data into groups using some functions. For example, the query below check the number of occurrences that students birthdays' occur.

```sql
SELECT (EXTRACT MONTH FROM birth_DATE) AS month, count(*)  AS occurrences
FROM students
GROUP BY month
ORDER BY occurrences;
```

## Views

A view is the result of a query and is stored in a virtual table.

```sql
CREATE VIEW students_overview AS SELECT students.first_name, students.last_name, subject_name
JOIN tests.student_id = students.id
```

## Functions

```sql
create or replace function fn_add_ints(int, int)
returns int as
$body$
select $1 + $2;
$body$

language sql
```

To use the created function

```sql
    select fn_add_ints(3,4);
```

To list available functions using `psql`

```sql
\df
```

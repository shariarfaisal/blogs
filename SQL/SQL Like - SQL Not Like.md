# SQL Like - SQL Not Like

```SQL```

SQL LIKE is used with WHERE clause to search for a pattern for a column. Wildcards are the one which is used for specifying the pattern. There are two wildcards that are used with the LIKE operator.


1. %: Percentage is used for representation of single, multiple or no occurrence.
2. _: The underscore is used for representation of a single character.

 To use SQL LIKE operator, we must be very sure of the usage of the wildcard position as that will define the search pattern.


# SQL Like Syntax


SQL Like operator can be used with any query with where clause. So we can use it with Select, Delete, Update etc.


```
SELECT column FROM table_name WHERE column LIKE pattern;

UPDATE table_name SET column=value WHERE column LIKE pattern;

DELETE FROM table_name WHERE column LIKE pattern;

```


In the sql like syntax mentioned above the “pattern” is the one that is defined by the usage of wildcards.


# SQL Like Example


Let’s try to understand the usage of SQL LIKE statement along with wildcards by some examples. Consider the following Customer table for the example.





CustomerId
CustomerName




1
Amit


2
John


3
Annie




1. 
Find customer name with name starting with ‘A’.
SELECT CustomerName FROM Customer WHERE CustomerName LIKE 'A%';

Output: Amit Annie

2. 
Find customer name with name ending with ‘e’.
SELECT CustomerName FROM Customer WHERE CustomerName LIKE '%e'

Output: Annie

3. 
Find customer name with name starting with ‘A’ and ending with ‘t’.
SELECT CustomerName FROM Customer WHERE CustomerName LIKE 'A%t'

Output: Amit

4. 
Find customer name with name containing ‘n’ at any position.
SELECT CustomerName FROM Customer WHERE CustomerName LIKE '%n%'

Output: Annie John

5. 
Find customer name with name containing ‘n’ at second position.
SELECT CustomerName FROM Customer WHERE CustomerName LIKE '_n%'

Output: Annie

6. 
Find customer name with name containing ‘i’ at third position and ending with ‘t’.
SELECT CustomerName FROM Customer WHERE CustomerName LIKE '__i%t'

Output: Amit


# SQL Not Like


Sometimes we want to get records that doesn’t match the like pattern. In that case we can use sql not like operator. SQL not like statement syntax will be like below.


```
SELECT column FROM table_name WHERE column NOT LIKE pattern;

UPDATE table_name SET column=value WHERE column NOT LIKE pattern;

DELETE FROM table_name WHERE column NOT LIKE pattern;

```


As an example, let’s say we want the list of customer names that don’t start with ‘A’. Below query will give us the required result set.


```
SELECT CustomerName FROM Customer WHERE CustomerName NOT LIKE 'A%';

```


Output: John


# SQL Multiple Like


We can have multiple like statements in SQL query. For example, if we want a list of customer names starting from ‘Jo’ and ‘Am’ then we will have to use multiple like statements like below.


```
SELECT CustomerName FROM Customer WHERE CustomerName LIKE 'Am%' OR CustomerName LIKE 'Jo%';

```


That’s all for SQL like operator and SQL not like operator examples.



# SQL Data Types

```SQL```

## Introduction


SQL data types define the type of value that can be stored in a table column. For example, if you want a column to store only integer values, you can define its data type as INT.


SQL data types can be broadly divided into the following categories.


1. Numeric data types such as: INT, TINYINT, BIGINT, FLOAT, REAL, etc.
2. Date and Time data types such as: DATE, TIME, DATETIME, etc.
3. Character and String data types such as: CHAR, VARCHAR, TEXT, etc.
4. Unicode character string data types such as: NCHAR, NVARCHAR, NTEXT, etc.
5. Binary data types such as: BINARY, VARBINARY, etc.
6. Miscellaneous data types - CLOB, BLOB, XML, CURSOR, TABLE, etc.

In this article, you will learn about different categories of SQL data types.


# Relational Database Vendor Differences



Note: Not all data types are supported by every relational database vendor.

For example, the Oracle database doesn’t support DATETIME, and MySQL doesn’t support CLOB. When designing database schemas and writing SQL queries, make sure to check if the data types are supported.



Note: Data types listed here don’t include all the data types. These are the most commonly used data types. Some relational database vendors have their own data types that might not be listed here.

For example, Microsoft SQL Server has MONEY and SMALLMONEY data types, but since they’re not supported by other popular database vendors, they’re not listed here.



Note: Every relational database vendor has its own maximum size limit for different data types.

Be sure to select the appropriate data type for your particular scenario.


# SQL Numeric Data Types





Data Type
From
To




BIT
1
0


TINYINT
0
255


SMALLINT
-32,768
32,767


INT
-2,147,483,648
2,147,483,647


BIGINT
-9,223,372,036,854,775,808
9,223,372,036,854,775,807


DECIMAL
-10^38 + 1
10^38 - 1


NUMERIC
-10^38 + 1
10^38 - 1


FLOAT
-1.79E+308
1.79E+308


REAL
-3.40E+38
3.40E+38




# SQL Date and Time Data Types





Data Type
Description




DATE
Stores date in the format YYYY-MM-DD


TIME
Stores time in the format HH:MI:SS


DATETIME
Stores date and time information in the format YYYY-MM-DD HH:MI:SS


TIMESTAMP
Stores number of seconds passed since the Unix epoch ('1970-01-01 00:00:00' UTC)


YEAR
Stores year in a 2-digit or 4-digit format. Range 1901 to 2155 in 4-digit format. Range 70 to 69, representing 1970 to 2069.




# SQL Character and String Data Types





Data Type
Description




CHAR
Fixed length with a maximum length of 8,000 characters


VARCHAR
Variable-length storage with a maximum length of 8,000 characters


VARCHAR(max)
Variable-length storage with provided max characters, not supported in MySQL


TEXT
Variable-length storage with a maximum size of 2GB data





Note: These data types are for character streams. They should not be used with Unicode data.

# SQL Unicode Character and String Data Types





Data Type
Description




NCHAR
Fixed length with a maximum length of 4,000 characters


NVARCHAR
Variable-length storage with a maximum length of 4,000 characters


NVARCHAR(max)
Variable-length storage with provided max characters


NTEXT
Variable-length storage with a maximum size of 1GB data





Note: These data types are not supported in MySQL databases.

# SQL Binary Data Types





Data Type
Description




BINARY
Fixed length with a maximum length of 8,000 bytes


VARBINARY
Variable-length storage with a maximum length of 8,000 bytes


VARBINARY(max)
Variable-length storage with provided max bytes


IMAGE
Variable-length storage with a maximum size of 2 GB binary data




# SQL Miscellaneous Data Types





Data Type
Description




CLOB
Character large objects that can hold up to 2 GB


BLOB
For large binary objects


XML
For storing XML data


JSON
For storing JSON data




# Conclusion


In this article, you learned about different categories of SQL data types.


Continue your learning with more SQL tutorials.


References


- Oracle Database Dataypes
- MySQL Data Types


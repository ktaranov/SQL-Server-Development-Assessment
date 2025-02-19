---
title: Data Type Conventions
permalink: findings/data-type-conventions
parent: Findings
nav_order: 3
layout: default
---

# Data Type Conventions
{: .no_toc }
Poor data type choices can have significant impact on a database design and performance. A best practice is to right size the data type by understanding the data.

---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

[Back to top](#top)

---

## Columns Named the Same Have Different Data Types
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/123)

There are two situations where this is going to hurt performance. When you have a mismatch of data types in a JOIN and a WHERE clause. SQL Server will need to convert one of them to match the others data type. This is called implicit conversion.

**What will it hurt**

- Indexes will not be used correctly
- Missing index requests will not be logged in the DMV
- Extra CPU cycles are going to be required for the conversion

[Back to top](#top)

---

## Using of Deprecated Data Type
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/30)

- Do not use the deprecated data types below.
  - ``text``
  - ``ntext``
  - ``image``
  - ``timestamp``

There is no good reason to use ``text`` or ``ntext``. They were a flawed attempt at BLOB storage and are there only for backward compatibility. Likewise, the WRITETEXT, UPDATETEXT and READTEXT statements are also deprecated. All this complexity has been replaced by the ``varchar(MAX)`` and ``nvarchar(MAX)`` data types, which work with all of SQL Server’s string functions.

[Back to top](#top)

---

## Email Address Column
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/44)

An email address column should be set to ``nvarchar(254)`` to leave 2 characters for <> if needed.

[Back to top](#top)

---

## URL Column
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/45)

A URL column should be set to ``nvarchar(2083)``.

[Back to top](#top)

---

## Overuse of (n)varchar(MAX)
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/46)

You might be overusing ``(n)varchar(MAX)`` on your table.

``(n)varchar(MAX)`` columns can be included in an index but not as a key. Queries will not be able to perform an index seek on this column. 

``(n)varchar(MAX)`` should only every be used if the size of the field is known to be over 8K for ``varchar`` and 4K for ``nvarchar``

Since SQL Server 2016 if the size of the cell is < 8K characters for ``varchar(MAX)`` it will be treated as Row data. If > 8K it will be treated as a Large Object (LOB) for storage purposes.

[Back to top](#top)

---

## Boolean Column Not Using bit
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/47)

Use the ``bit`` data type for boolean columns. These columns will have names like IsSpecialSaleFlag. 

[Back to top](#top)

---

## Using float or real
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/65)

The ``float`` (8 byte) and ``real`` (4 byte) data types are suitable only for specialist scientific use since they are approximate types with an enormous range (-1.79E+308 to -2.23E-308, 0 and 2.23E-308 to 1.79E+308, in the case of ``float``). Any other use needs to be regarded as suspect, and a ``float`` or ``real`` used as a key or found in an index needs to be investigated. The ``decimal`` type is an exact data type and has an impressive range from -10^38+1 through 10^38-1. Although it requires more storage than the ``float`` or ``real`` types, it is generally a better choice.

[Back to top](#top)

---

## Using sql_variant
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/67)

The ``sql_variant`` type is not your typical data type. It stores values from a number of different data types and is used internally by SQL Server. It is hard to imagine a valid use in a relational database. It cannot be returned to an application via ODBC except as binary data, and it isn’t supported in Microsoft Azure SQL Database.

[Back to top](#top)

---

## Using User-Defined Data Type
**Check Id:** 10

User-defined data types should be avoided whenever possible. They are an added processing overhead whose functionality could typically be accomplished more efficiently with simple data type variables, table variables, or temporary tables.


[Back to top](#top)

---

## Using datetime Instead of datetimeoffset
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/80)

``datetimeoffset`` defines a date that is combined with a time of a day that has time zone awareness and is based on a 24-hour clock. This allows you to use ``datetimeoffset AT TIME ZONE [timezonename]`` to convert the datetime to a local time zone. 

Use this query to see all the timezone names:

```sql
SELECT * FROM sys.time_zone_info
```


[Back to top](#top)

---

## Using datetime or datetime2 Instead of date
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/81)

Even with data storage being so cheap, a saving in a data type adds up and makes comparison and calculation easier. When appropriate, use the ``date`` or ``smalldatetime`` type. Narrow tables perform better and use less resources.


[Back to top](#top)

---

## Using datetime or datetime2 Instead of time
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/82)

Being frugal with memory is important for large tables, not only to save space but also to reduce I/O activity during access. When appropriate, use the ``time`` or ``smalldatetime`` type. Queries too are generally simpler on the appropriate data type.


[Back to top](#top)

---

## Using money Data Type
**Check Id:** 28

The ``money`` data type confuses the storage of data values with their display, though it clearly suggests, by its name, the sort of data held. Use ``decimal(19, 4)`` instead. It is proprietary to SQL Server. 

``money`` has limited precision (the underlying type is a ``bigint`` or in the case of ``smallmoney`` an ``int``) so you can unintentionally get a loss of precision due to roundoff errors. While simple addition or subtraction is fine, more complicated calculations that can be done for financial reports can show errors. 

Although the ``money`` data type generally takes less storage and takes less bandwidth when sent over networks, it is generally far better to use a data type such as the ``decimal(19, 4)`` type that is less likely to suffer from rounding errors or scale overflow.


[Back to top](#top)

---

## Using varchar Instead of nvarchar for Unicode Data
**Check Id:** [None yet, click here to view the issue](https://github.com/EmergentSoftware/SQL-Server-Development-Assessment/issues/84)

You can't require everyone to stop using national characters or accents any more. Names are likely to have accents in them if spelled properly, and international addresses and language strings will almost certainly have accents and national characters that can’t be represented by 8-bit ASCII!

**Column names to check:**
- FirstName
- MiddleName
- LastName
- FullName
- Suffix
- Title
- ContactName
- CompanyName
- OrganizationName
- BusinessName
- Line1
- Line2
- CityName
- TownName
- StateName
- ProvinceName

[Back to top](#top)

---
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
---
layout: post
title: "SQL Query : Get top X tables by space occupied"
excerpt_separator: "<!--more-->"
tags: 
    - SQL Server Queries
---

### Objective
Get information on top X number of tables by total space occupied within current database.

<!--more-->
<br>

### Query
{% highlight sql %}
SELECT TOP 10 -- Replace 10 with desired number
	SCHEMA_NAME([tab].[schema_id]) + '.' + [tab].[name] AS [table], 
    CAST(SUM([spc].[used_pages] * 8)/1024.00 AS NUMERIC(36, 2)) AS [used_mb],
    CAST(SUM([spc].[total_pages] * 8)/1024.00 AS NUMERIC(36, 2)) AS [allocated_mb]
from 
	[sys].[tables] AS [tab]
	INNER JOIN [sys].[indexes] AS [ind] ON [tab].[object_id] = [ind].[object_id]
	INNER JOIN [sys].[partitions] AS [part] ON [ind].[object_id] = [part].[object_id] AND [ind].[index_id] = [part].[index_id]
	INNER JOIN [sys].[allocation_units] AS [spc] ON [part].[partition_id] = [spc].[container_id]
GROUP BY 
	SCHEMA_NAME([tab].[schema_id]) + '.' + [tab].[name]
ORDER BY
	SUM([spc].[used_pages]) DESC	
{% endhighlight %}

<br>

### Output
![top-tables-output](/til/assets/sql-queries-output/top-tables.png)

<br>

## Reference Links
- [sys.tables](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-tables-transact-sql)
- [sys.indexes](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-indexes-transact-sql)
- [sys.partitions](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-partitions-transact-sql)
- [sys.allocation_units](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-allocation-units-transact-sql)
---
layout: post
title: "Check space occupied by database"
excerpt_separator: "<!--more-->"
tags: 
    - SQL Server Queries
---

**Objective** <br>
Get information on space occupied by the given database.

<!--more-->

<br>

**Query**
{% highlight sql %}
SELECT 
	RTRIM([name]) AS [Segment Name], 
	[groupid] AS [Group Id], 
	[filename] AS [File Name],
	CAST([size]/128.0 AS DECIMAL(10,2)) AS [Size in MB],
	CAST(FILEPROPERTY([name], 'SpaceUsed')/128.0 AS DECIMAL(10,2)) AS [Space Used],
	CAST([size]/128.0-(FILEPROPERTY([name], 'SpaceUsed')/128.0) AS DECIMAL(10,2)) AS [Available Space],
	CAST((CAST(FILEPROPERTY(name, 'SpaceUsed')/128.0 AS DECIMAL(10,2))/CAST(size/128.0 AS DECIMAL(10,2)))*100 AS DECIMAL(10,2)) AS [Percent Used]
FROM 
	[sysfiles]
ORDER BY 
	[groupid] DESC	
{% endhighlight %}

<br>

**Output** <br>
![db-space-output](/til/assets/sql-queries-output/db-space.png)
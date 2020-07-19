---
layout: post
title: "CA2227 : Collection properties should be read only"
excerpt_separator: "<!--more-->"
tags: 
    - C#
    - code quality
---

**Why?**

> MSDN says, *A writable collection property allows a user to replace the collection with a completely different collection. A read-only property stops the collection from being replaced, but still allows the individual members to be set. If replacing the collection is a goal, the preferred design pattern is to include a method to remove all the elements from the collection, and a method to repopulate the collection.*

<!--more-->

Consider following class, 

{% highlight csharp %}
public class Temp
{
    public List<string> Data {get; set;}
}

{% endhighlight %}

Here code analyzer will throw error because rule says Collections do not need setters. <br>
Now this allows me to do anything with Data object. I can assign any arbitrary `List<string>` to it or add/remove any entry from collection etc. <br>
This mutability of property can result in some accidental issues like following <br>

{% highlight csharp %}
var abc = new Temp();
abc.Data.Add("abc_Test1");

//after some code lines 

var lst1 = new List<string>();
lst1.Add("lst_Test1");
lst1.Add("lst_Test2");
abc.Data = lst1;

//after some code lines 

//What to expect here? Initially it was abc_Test1 but object was reassigned so result will be lst_Test1.
Console.WriteLine(abc.Data[0]); 
{% endhighlight %}

Now consider following case, 

{% highlight csharp %}
public class Temp
{
    private List<string> _data = new List<string>();
    public List<string> Data {get{ return _data; }}
}
{% endhighlight %}

or (as per new construct)


{% highlight csharp %}
public class Temp
{
    public List<string> Data { get; } = new List<string>();
}
{% endhighlight %}

Here I have removed setter from Data property & have added a private backing field. It is not possible to set property value directly. It is already initialaized with private backing field. 


{% highlight csharp %}
var abc = new Temp();
abc.Data = new List<string>();
{% endhighlight %}

^^ This will throw error now. But I can still add or remove entries from collection. So following is still valid.

{% highlight csharp %}
abc.Data.Add("Test1");
abc.Data.Remove("Test1");
{% endhighlight %}

This makes Data property immutable. **You can use methods provided by the object but direct object assignment is not possible.**

This is the recommended pattern. You can see same behavior in most of the .NET types. Simple example will be the Exception class in .NET. Try following code & you will be able to see it yourself. 

{% highlight csharp %}
var excp = new Exception();
excp.Data = new Dictionary<string,string>(); // Error : This Property or indexer 'Exception.Data' cannot be assigned to -- it is read-only.
{% endhighlight %}


## When to suppress this error?

As per MSDN, *You can suppress the warning if the property is part of a Data Transfer Object (DTO) class.* <br>

In our case you supress warning if,
- Entity class is being used as DTO object for handling data fetched from database.
- Entity class is being used as class used for deserialization of any JSON object. Newtonsoft.Json needs all properties as settable to be able to deserialize JSON properly.

Any other case, you should not suppress this warning. Instead update it to follow right pattern. 

#### Reference : 
- [https://docs.microsoft.com/en-us/visualstudio/code-quality/ca2227](https://docs.microsoft.com/en-us/visualstudio/code-quality/ca2227)

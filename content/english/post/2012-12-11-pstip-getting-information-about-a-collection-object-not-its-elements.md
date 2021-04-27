---
title: '#PSTip Getting information about a collection object, not its elements'
author: Shay Levy
type: post
date: 2012-12-11T19:25:40+00:00
url: /2012/12/11/pstip-getting-information-about-a-collection-object-not-its-elements/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
You have a collection of values in a variable. When you pipe the variable to the _Get-Member_ cmdlet you get the type and members of each item in that collection. PowerShell unrolls the collection and sends each item through the pipeline, one at a time, to the _Get-Member_ cmdlet.

```
PS> $array = "one",2
PS> $array | Get-Member
   TypeName: System.String
Name             MemberType            Definition
----             ----------            ----------
Clone            Method                System.Object Clone(), System.Object ...
CompareTo        Method                int CompareTo(System.Object value), i...
(...)

   TypeName: System.Int32
Name        MemberType   Definition
----        ----------   ----------
CompareTo   Method       int CompareTo(System.Object value), int CompareTo...
Equals      Method       bool Equals(System.Object obj), bool Equals(int o...
(...)
```

Most of the time that&#8217;s the desired output. However, there are cases where you&#8217;ll need to get the members of the collection itself. To do so, you can choose one of three approaches. Call the _GetType_ method on a variable:

```
PS> $array.GetType()
IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array
```

Pass the collection object to the _InputObject_ parameter of the _Get-Member_ cmdlet:

```
PS> Get-Member -InputObject $array
   TypeName: System.Object[]
Name           MemberType            Definition
----           ----------            ----------
Count          AliasProperty         Count = Length
Add            Method                int IList.Add(System.Object value)
Address        Method                System.Object&, mscorlib, Version=4.0.0...
Clear          Method                void IList.Clear()
Clone          Method                System.Object Clone(), System.Object IC...
CompareTo      Method                int IStructuralComparable.CompareTo(Sys...
(...)
```

Or use the unary comma operator to create an array with one member. That way when the array is unrolled the inner array will pass through the pipeline as one item.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; ,$array | Get-Member
</pre>
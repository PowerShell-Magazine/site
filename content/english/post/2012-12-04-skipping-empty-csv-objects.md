---
title: Skipping empty CSV objects
author: Shay Levy
type: post
date: 2012-12-04T18:00:14+00:00
url: /2012/12/04/skipping-empty-csv-objects/
categories:
  - How To
tags:
  - How To

---
Working with CSV files in PowerShell is a common practice. You import the file, loop on its records and you&#8217;re good to go. Sometimes however you may find yourself in a situation where you get a file that has blank lines in it, and those lines can break your script. Consider the following CSV content:

<pre class="brush: powershell; title: ; notranslate" title="">## sample.csv ##
column1,column2,column3
Value1,Value2,Value3
Value1,Value2,Value3
&lt;empty line&gt;
&lt;empty line&gt;
&lt;empty line&gt;
## file ends here
</pre>

On the surface, nothing looks suspicious when you import the file:

```
PS> Import-Csv sample.csv
column1  column2  column3
-------  -------  -------
Value1   Value2   Value3
Value1   Value2   Value3
PS>
```

But if you pipe it to Format-List you can clearly see what&#8217;s going on. You get empty objects for each empty line in the file.

```
PS> Import-Csv sample.csv | Format-List

column1 : Value1
column2 : Value2
column3 : Value3

column1 : Value1
column2 : Value2
column3 : Value3

column1 :
column2 :
column3 :

column1 :
column2 :
column3 :

column1 :
column2 :
column3 :
```

To filter out empty objects you need to test that all properties are not equal to an empty string and throw them away.

You might be attempted to do that with:

<pre class="brush: powershell; title: ; notranslate" title="">Import-Csv sample.csv |
Where-Object {$_.column1 -ne '' -and $_.column1 -ne '' -and $_.column1 -ne ''}
</pre>

But what if each record has 20 properties, or even more? This is where the PSObject property comes to rescue. In a nutshell, PSObject allows us to work with any object in the same way without really knowing its structure. PowerShell wraps the base object in a PSObject and provide us a simplified and consistent view of the object, its methods, properties, and so on. One of the properties of PSObject is Properties, and it gives us a list of properties of the base object.

On a related note, PSObject and other members are not visible when you pipe an object to the Get-Member cmdlet. To reveal those members add the -Force switch to Get-Member.

For our purpose, we can process the properties list and filter out those who have a Value of null.

```
Import-Csv sample.csv |
Where-Object { ($_.PSObject.Properties | ForEach-Object {$_.Value}) -ne $null} |
Format-List

column1 : Value1
column2 : Value2
column3 : Value3

column1 : Value1
column2 : Value2
column3 : Value3
```

In PowerShell 3.0 and the new [Member Enumeration feature][1] we can get the same result in less characters:

<pre class="brush: powershell; title: ; notranslate" title="">Import-Csv sample.csv |
Where-Object { $_.PSObject.Properties.Value -ne $null}
</pre>

I logged an [Import-Csv feature enhancement,][2] and you can add your vote if you&#8217;d like to have a built-in option of ignoring empty lines.

[1]: http://blogs.msdn.com/b/powershell/archive/2012/06/14/new-v3-language-features.aspx
[2]: https://connect.microsoft.com/PowerShell/feedback/details/767851/import-csv-and-blank-lines


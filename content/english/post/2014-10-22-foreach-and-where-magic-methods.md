---
title: ForEach and Where magic methods
author: Kirk Munro
type: post
date: 2014-10-22T16:00:38+00:00
url: /2014/10/22/foreach-and-where-magic-methods/
categories:
  - How To
  - Language
tags:
  - How To
  - Language

---
_ForEach_ and _Where_ are two frequently used concepts that have been available in PowerShell since version 1 came out in 2006.  _ForEach_ has been available as both a statement and a cmdlet (_ForEach-Object_), allowing you to iterate through a collection of objects and take some action once for each object in that collection.  _Where_ has been available as a cmdlet (_Where-Object_), allowing you to filter out items in a collection that don’t pass some condition, a condition which may be evaluated using either the properties on objects in the collection or the objects themselves that belong to the collection.  The ForEach ability to take action on items in a collection and the Where ability to filter a collection are features that are highly useful and are included in one form or another in the logic behind many PowerShell scripts, commands and modules, regardless of what version of PowerShell is being used.  In fact, they are so heavily used that they have been a focus area for improvement of performance, functionality and syntax in PowerShell versions 3.0 and 4.0.

With the release of Windows PowerShell 4.0, two new “magic” methods were introduced for collection types that provide a new syntax for accessing _ForEach_ and _Where_ capabilities in Windows PowerShell.  These methods are aptly named _ForEach_ and _Where_. I call these methods “magic” because they are quite magical in how they work in PowerShell.  They don’t show up in _Get-Member_ output, even if you apply -Force and request _-MemberType All_.  If you roll up your sleeves and dig in with reflection, you can find them; however, it requires a broad search because they are private extension methods implemented on a private class.  Yet even though they are not discoverable without peeking under the covers, they are there when you need them, they are faster than their older counterparts, and they include functionality that was not available in their older counterparts, hence the “magic” feeling they leave you with when you use them in PowerShell.  Unfortunately, these methods remain undocumented even today, almost a year since they were publicly released, so many people don’t realize the power that is available in these methods.  This article will try to correct that by explaining where they can be used and how they work so that you can leverage this magic when you use PowerShell.

### A note about PowerShell 3.0

Before I get into explaining how the _ForEach_ and _Where_ methods work, I need to mention something with respect to these two methods and PowerShell 3.0.  While it is true that the _ForEach_ and _Where_ methods were only made available in PowerShell 4.0 and later versions, PowerShell 3.0 is still very widely used in many environments, and unless you’re using PowerShell in an environment that has standardized on PowerShell 4.0 and later, you may find yourself wishing you could take advantage of the syntax provided by the new methods when using PowerShell 3.0.  I felt this was a limitation worth addressing, so as part of the [TypePx module that I recently published on GitHub][1] and in the [PowerShell Resource Gallery][2] (aka the _PowerShellGet_ public repository, currently in limited preview), I included _ForEach_ and _Where_ script methods that are functionally equivalent to the methods introduced in PowerShell 4.0 so that you can leverage the new syntax and functionality even if you’re using PowerShell 3.0.  There are a few shortcomings in this implementation, which I will highlight later in this article.

### The ForEach method

_ForEach_ is a method that allows you to rapidly iterate through a collection of objects and take some action on each object in that collection.  This method provides faster performance than its older counterparts (the _foreach_ statement and the _ForEach-Object_ cmdlet), and it also simplifies some of the most common actions that you may want to take on the objects in the collection.  Any objects that are output by this method are returned in a generic collection of type _System.Collections.ObjectModel.Collection\`1[psobject]_.

There are six supported ways that you can invoke this method, and each of these will be explained in greater detail below.  The supported arguments that can be used when invoking the ForEach method are as follows:

  * _ForEach(scriptblock expression)_
  * _ForEach(type convertToType)_
  * _ForEach(string propertyName)_
  * _ForEach(string propertyName, object[] newValue)_
  * _ForEach(string methodName)_
  * _ForEach(string methodName, object[] arguments)_
  * _ForEach(scriptblock expression, object[] arguments)_

Note that these are supported argument pairings, not different overloads available for the ForEach method.  Using any argument pairings other than these may result in errors that do not clearly identify what the actual problem is.

### ForEach(scriptblock expression) and ForEach(scriptblock expression, object[] arguments)

If you pass a script block expression into the _ForEach_ method, you are able to perform the same kind of tasks that you would in a script block that you would use with the _foreach_ statement or the _ForEach-Object_ cmdlet.  Also, like the _ForEach-Object_ cmdlet, the _$__ and _$PSItem_ variables both reference the current item that is being processed.  Any arguments that you provide beyond the initial script block argument will be used as arguments for the script block. This is just like how the -ArgumentList parameter works on the ForEach-Object cmdlet. Here is an example demonstrating how you might use this to execute a script block on each item in a collection:

<pre class="brush: powershell; title: ; notranslate" title=""># Get a set of services
$services = Get-Service c*
# Display the names and display names of all services in the collection
$services.foreach{"$($_.Name) ($($_.DisplayName))"}
# Select a property name to expand using a script block argument
$services.foreach({param([string]$PropertyName); $_.$PropertyName}, 'DisplayName')
</pre>

You may have noticed something odd about this syntax, because I didn’t wrap the script block itself in brackets.  You can wrap it in round brackets, however that is optional in PowerShell 4.0 or later because the PowerShell parser was enhanced to allow for the brackets to be omitted whenever you are invoking a method that accepts a single script block argument.  Also, like the _foreach_ statement and the _ForEach-Object_ cmdlet, the script block that you provide is invoked in the current scope.  That means that any variable assignments you make inside of that script block will persist after the ForEach method has finished executing.

### ForEach(type convertToType)

Unique to the _ForEach_ method, you can pass a type into the _ForEach_ method if you want to convert every item in a collection into another type.  For example, imagine you have a collection of objects and you want to convert those objects into their string equivalent.  Here is what that would look like with the _ForEach_ method:

<pre class="brush: powershell; title: ; notranslate" title=""># Get a collection of processes
$processes = Get-Process
# Convert the objects in that collection into their string equivalent
$processes.foreach([string])
</pre>

You could have performed the same task by typecasting the collection into an array of type string (e.g. _[string[]]$processes_), and typecasting the array is in fact significantly faster, however there’s a very good chance you wouldn’t even notice the difference in execution time unless you were working with a very, very large collection.  Despite the time difference, I will tend to prefer the _ForEach_ method syntax in certain situations if it allows me to maintain elegance in the implementation by avoiding extra round brackets in the scripts I write.

### ForEach(string propertyName)

In PowerShell 3.0 and later, a second parameter set was added to _ForEach-Object_ to allow you to more easily retrieve the value of a specific property by simply passing in a property name as the only parameter value for _ForEach-Object_.  This convenience has been offered in the _ForEach_ method as well.  Here is an example demonstrating how you can iterate through a collection and return a property of that collection:

<pre class="brush: powershell; title: ; notranslate" title=""># Get all services whose name starts with "w"
$services = Get-Service w*
# Return the names of those services
$services.foreach('Name')
</pre>

Of course since version 3.0 of PowerShell, you could simply invoke _$services.Nam_e to get the names of all services, and that will complete faster than the _ForEach_ method alternative (although you’ll only notice the performance difference in very large collections in the order of hundreds of thousands of objects); however, that only works for properties that are not on the collection itself, and it is a syntax that some scripters are not comfortable with due to the implicit nature of what the command does. The new ForEach method syntax provides you with a more explicit alternative that has the added benefit of being a little more self-documenting as well.

### ForEach(string propertyName, object[] newValue)

Not only can you retrieve a property on a collection of objects, you can set a property on a collection of objects as well. This is functionality that is not available in the other foreach’s, unless you explicitly create the script block to do so. To set the property, you simply provide the property name and the value you want to use when setting that property, like this:

<pre class="brush: powershell; title: ; notranslate" title=""># Note, this is not a realistic example
# This would be used more commonly on configuration data
$services = Get-Service c*
# Now change the display names of every service to some new value
$services.foreach('DisplayName','Hello')
</pre>

Just like assignments you would make using the equals operator, PowerShell will attempt to convert whatever you provide as the new value into the appropriate type for the property being assigned.

### ForEach(string methodName) and ForEach(string methodName, object[] arguments)

To invoke a method, you simply provide the method name as the first argument, and then the arguments for that method as the second, third, fourth, etc. arguments. If the method does not take any arguments, you can simply pass in the method name and it will be invoked without any arguments. Here’s an example showing how you could kill a bunch of processes running a specific program:

<pre class="brush: powershell; title: ; notranslate" title=""># Get all processes running Chrome
$processes = Get-Process -Name Chrome
# Now kill all of those processes
$processes.foreach('Kill')
</pre>

Here’s another example, this time using a method with arguments while showing how you could verify that commands are following best practices by using appropriate names and aliases for commonly used parameters:

<pre class="brush: powershell; title: ; notranslate" title=""># Get all commands that have a ComputerName parameter
$cmds = Get-Command -ParameterName ComputerName
# Now show a table making sure the parameter names and aliases are consistent
$cmds.foreach('ResolveParameter','ComputerName') | Format-Table Name,Aliases
</pre>

As you can see from those results, there are definitely some inconsistencies in the implementation of ComputerName parameters that should be corrected.

That covers all of the functionality that is currently available in the ForEach method. As you can see, there is not a lot of new functionality offered in this method, but the syntax improvements when you are performing a simple task on a collection of objects are nice, and the _ForEach_ method performance improvements when compared to the equivalent _foreach_ statement for _ForEach-Object_ pipeline are definitely a welcome improvement as well. With that explanation out of the way, let’s move on to the _Where_ method.

### The Where method

_Where_ is a method that allows you to filter a collection of objects.  This is very much like the _Where-Object_ cmdlet, but the _Where_ method is also like _Select-Object_ and _Group-Object_ as well, includes several additional features that the _Where-Object_ cmdlet does not natively support by itself. This method provides faster performance than _Where-Object_ in a simple, elegant command.  Like the _ForEach_ method, any objects that are output by this method are returned in a generic collection of type _System.Collections.ObjectModel.Collection\`1[psobject]_.

There is only one version of this method, which can be described as follows:

<pre class="brush: powershell; title: ; notranslate" title="">Where(scriptblock expression[, WhereOperatorSelectionMode mode[, int numberToReturn]])
</pre>

As indicated by the square brackets, the expression script block is required and the mode enumeration and the _numberToReturn_ integer argument are optional, so you can invoke this method using 1, 2, or 3 arguments.  If you want to use a particular argument, you must provide all arguments to the left of that argument (i.e. if you want to provide a value for _numberToReturn_, you must provide values for mode and expression as well).

### Where(scriptblock expression)

The most basic invocation of the Where method simply takes a script block expression as an argument.  The script block expression will be evaluated once for each object in the collection that is being processed, and if it returns true, the object will be returned by the _Where_ method.  This is the functional equivalent of calling the _Where-Object_ cmdlet and passing it a script block.  Like the _Where-Object_ cmdlet, the _$__ and _$PSItem_ variables can be used to refer to the current item that is being processed while inside of the script block.

Here is a very simple example, showing how you could get a list of running services.

<pre class="brush: powershell; title: ; notranslate" title=""># Get all services
$services = Get-Service
# Now filter out any services that are not running
$services.where{$_.Status -eq 'Running'}
</pre>

This doesn’t offer any new functionality, but it offers much faster performance than _Where-Object_ and the syntax is quite easy to follow, so you really should consider this for your scripts when you are performing client-side filtering of collections that you have stored in a variable.

### Where(scriptblock expression, WhereOperatorSelectionMode mode[ ,int numberToReturn])

When you start looking at the optional parameters for the _Where_ method, things start getting much more interesting.  Windows PowerShell version 4.0 included a new enumeration with a typename of _System.Management.Automation.WhereOperatorSelectionMode_.  Note the suffix of that typename: “_SelectionMode_”.  It is used to provide powerful selection capabilities in a Where syntax.  Here are the values included in this enumeration, along with their definitions:

| Default   | Filter the collection using the expression script block, to a maximum count if one was provided or defaulting to all objects in the collection if no maximum count was provided in numberToReturn. |
| --------- | ------------------------------------------------------------ |
| First     | Return the first N objects that pass the expression script block filter, defaulting to only 1 object if a specific count was not requested in numberToReturn. |
| Last      | Return the last N objects that pass the expression script block filter, defaulting to only 1 object if a specific count was not requested in numberToReturn. |
| SkipUntil | Skip objects in the collection until an object passes the expression script block filter, and then return the first N objects, defaulting to all remaining objects if no maximum count was provided in numberToReturn. |
| Until     | Return the first N objects in a collection until an object passes the expression script block filter, defaulting to all objects leading up to the first object that passed if no maximum count was provided in numberToReturn. |
| Split     | Split a collection into two, placing all objects that pass the expression script block filter into the first collection up to a maximum count if one was provided in numberToReturn, or all objects that pass if no maximum count was provided, and placing all other objects that are not put in the first collection into the second collection. |

Each of these offers some unique value when you are processing collections of data, so I’ll provide more details of each selection mode below.

#### Default

Not surprisingly, the default value of the mode argument is ‘Default’.  The default selection mode offers the same functionality that you get when you don’t provide a selection mode at all.  For example, we could have written the last line of our previous example like this:

<pre class="brush: powershell; title: ; notranslate" title=""># Now filter out any services that are not running
$services.where({$_.Status -eq 'Running'},'Default')
</pre>

In that example, the extra argument isn’t necessary though, because it does the exact same thing that it would do if you didn’t provide the argument.  You could also provide the maximum number of objects that you want to return while using Default selection mode using the _numberToReturn_ argument, like this:

<pre class="brush: powershell; title: ; notranslate" title=""># Get the first 10 services in our collection that are running
$services.where({$_.Status -eq 'Running'},'Default',10)
</pre>

It is important to note that exact functionality is also available when using the First selection mode (which we’ll talk about in a moment), so it really isn’t practical to use any of the optional parameters at all when you are using the Default selection mode.

#### First

As you might have guessed, the _First_ selection mode allows you to select the first object(s) in the collection that pass the script block expression filter. When you use _First_ without a value for the _numberToReturn_ argument, or when you use _First_ with a value of 0 for the _numberToReturn_ argument, only the first object that passes the filter will be returned.  You can optionally specify how many objects to return in the _numberToReturn_ argument, in which case that many objects will be returned (assuming there are that many objects that pass the filter).

Here are some examples using our services collection showing the First selection mode in action:

<pre class="brush: powershell; title: ; notranslate" title=""># Get the first service in our collection that is running
$services.where({$_.Status -eq 'Running'},'First')
# Get the first service in our collection that is running
$services.where({$_.Status -eq 'Running'},'First',1)
# Get the first 10 services in our collection that are running
$services.where({$_.Status -eq 'Running'},'First',10)
</pre>

Note that the second command in these examples returns the same results as the first command because it is simply explicitly passing in the default value of the numberToReturn argument when the First selection mode is used.

#### Last

The _Last_ selection mode functions much like the _First_ selection mode, allowing you to select the last object(s) in the collection that pass the script block expression filter.  When you use Last without a value for the _numberToReturn_ argument, or when you use Last with a value of 0 for the _numberToReturn_ argument, only the last object that passes the filter will be returned.  You can optionally specify how many objects to return in the _numberToReturn_ argument, in which case that many objects will be returned (assuming there are that many objects that pass the filter).

Here are some examples using our services collection showing the _Last_ selection mode in action:

```
# Get the last service in our collection that is running
$services.where({$_.Status -eq 'Running'},'Last')

# Get the last service in our collection that is running
$services.where({$_.Status -eq 'Running'},'Last',1)

# Get the last 10 services in our collection that are running
$services.where({$_.Status -eq 'Running'},'Last',10)
```

Also like the _First_ selection mode examples, the second command in these examples returns the same results as the first command because it is simply explicitly passing in the default value of the _numberToReturn_ argument when the _Last_ selection mode is used.

#### SkipUntil

The _SkipUntil_ selection mode allows you to skip all objects in a collection until you find one that passes the script block expression filter.  Once you find an object that passes the filter, _SkipUntil_ mode will either return all objects remaining in the collection if no value or a value of 0 was provided to the _numberToReturn_ argument, or it will return the first N remaining objects in the collection if a value greater than zero was provided to the _numberToReturn_ argument.  In both cases, the results will include the first object that passed the filter.

Here are some examples using a subset of our services collection to show the _SkipUntil_ selection mode in action:

<pre class="brush: powershell; title: ; notranslate" title=""># Get a collection of services whose name starts with "c"
$services = Get-Service c*
# Skip all services until we find one with a status of "Running"
$services.where({$_.Status -eq 'Running'},'SkipUntil')
# Skip all services until we find one with a status of "Running", then
# return the first 2
$services.where({$_.Status -eq 'Running'},'SkipUntil',2)
</pre>

#### Until

The Until selection mode provides the opposite functionality of the _SkipUntil_ selection mode.  It allows you to return objects in a collection until you find one that passes the script block expression filter.  Once you find an object that passes the filter, the Where method stops processing objects.  If you don’t provide a value for the _numberToReturn_ argument, or if you provide a value of 0, the _Until_ selection mode will return all objects in the collection leading up to the first one that passes the script block expression filter.  If you do provide a value for the _numberToReturn_ argument that is greater than 0, the Until selection mode will return at most that number of objects, meaning that it may not even find an object that passes the script block expression filter.

Here are some examples using a different subset of our services collection to show the Until selection mode in action:

<pre class="brush: powershell; title: ; notranslate" title=""># Get a collection of services whose name starts with "p"
$services = Get-Service p*
# Return all services until we find one with a status of "Stopped"
$services.where({$_.Status -eq 'Stopped'},'Until')
# Return the first 2 services unless we find one with a status of
# "Stopped" first
$services.where({$_.Status -eq 'Stopped'},'Until',2)
</pre>

#### Split

Split selection mode is unique.  Instead of returning a subset of the collection you start with in a new collection, it returns a new collection that internally contains two separate collections.  What those nested collections contain depends on how you use Split selection mode.  Split allows you to split a collection of objects into two.  By default, if you don’t provide a value for the _numberToReturn_ argument or if you provide a value of 0 for the _numberToReturn_ argument, Split will place all objects that pass the script block expression filter into the first nested collection, and all other objects (those that don’t pass the script block expression filter) into the second nested collection.  If you do provide a value greater than 0 for the _numberToReturn_ argument, split will limit the size of the first collection to that maximum amount, and all remaining objects in the collection, even those that match the script block expression filter, will be placed into the second collection.

Here are some examples showing how Split selection mode can be used to split up a collection of objects different ways:

<pre class="brush: powershell; title: ; notranslate" title=""># Get all services
$services = Get-Service
# Split the services into two groups: Running and not Running
$running,$notRunning = $services.Where({$_.Status -eq 'Running'},'Split')
# Show the Running services
$running
# Show the services that are not Running
$notRunning
# Split the services into the same two groups, but limit the Running group
# to a maximum of 10 items
$10running,$others = $services.Where({$_.Status -eq 'Running'},'Split',10)
# Show the first 10 Running services
$10running
# Show all other services
$others
</pre>

As you can see from this example, Split is quite a powerful selection mode, providing a mix of filtering, grouping, and selection in a single command call.

This collection of selection modes makes the _Where_ method fast and elegant yet at the same time more powerful than the _Where-Object, Group-Object_ and Select-Object combined in a single pipeline.  What’s not to love about that?

### Shortcomings in the ForEach and Where script methods in TypePx

As I mentioned earlier in this article, I have written type extensions for PowerShell 3.0 and later and packaged them up into a module called _[TypePx][1]._  _TypePx_ is a script module that was written entirely in PowerShell, and it runs on PowerShell 3.0 or later.  If you are using PowerShell 3.0 (and only if you are using PowerShell 3.0), _TypePx_ defines _ForEach_ and _Where_ script methods that mimic the behaviour of the _ForEach_ and Where methods in PowerShell 4.0 and later.  While the extended type system in PowerShell makes it possible to mimic this behaviour, there are a few shortcomings due to the implementation being in PowerShell that impacted how far I was able to go with these type extensions.  This section will describe some of the differences and limitations that exist in the _ForEach_ and _Where_ script methods in _TypePx_ that you may want to be aware of if you are using PowerShell 3.0.

### Script blocks invoked from a script method run in a child scope

Unlike the _ForEach_ and _Where_ methods implemented as part of PowerShell 4.0 or later which invoke the expression script block in the current scope where the method is called, the _ForEach_ and Where script methods implemented in PowerShell 3.0 invoke the expression script block in a child scope.  This is a limitation in PowerShell that has been there since the very beginning (a limitation, I might add, that I think is by far the biggest shortcoming of PowerShell as a language).

Due to this limitation, any variables that you assign inside of an expression script block will only be modified in the child scope.  This has implications if your expression script block is intended to update a variable in the scope in which you invoke ForEach or Where.  It is unlikely this would cause a problem while using Where because it is not very common to modify variables in a Where expression script block, but in _ForEach_ script blocks this may pose a problem so you need to keep this in mind if you use these extensions.

I should note that I would like to remove this limitation altogether, and I believe I will be able to do so, however at the time I wrote this article I have not yet implemented a fix for this.

### Most, but not all collections will have ForEach and Where methods

In PowerShell 4.0 and later, the _ForEach_ and _Where_ methods are magically made available for all types that implement _IEnumerable_ except for _String_, _XmlNode_, and types that implement _IDictionary_.  In PowerShell, the extended type system does not allow extensions to be created for interfaces, only for types.  This is a challenge if you want to create a broad extension like _ForEach_ and _Where_.  In the current implementation of these extensions in TypePx, the TypePx module finds all types in all assemblies loaded in the current app domain, and for all non-generic types that define _IEnumerable_ but not _IDictionary_ (excluding _String_ and _XmlNode_), plus for all generic types that define _IEnumerable_ but not _IDictionary_ for generic collections of _PSObject_, _Object_, _String_, _Int32_, or _Int64,_ the _ForEach_ and _Where_ script methods will be created.

This covers a large number of types that in my own testing has been sufficient, however you may run into types where you want to use these methods and they are not available.  If that is the case, let me know through GitHub and I’ll see what I can do.  This is also a limitation I would like to remove, but I need more time to research how to implement the equivalent functionality in a compiled assembly where I may be able to define it more like it is defined in PowerShell 4.0 and later.

### PowerShell scripts are not as fast as compiled code

This probably goes without saying, but when you write something in PowerShell, which is an interpreted language, it will not run as quickly as it would if you wrote the equivalent logic in a language like C#.  This is absolutely the case for the _ForEach_ and _Where_ script methods, which internally use _ForEach-Object_, Where-Object, and pipelining to mimic the behaviour of the native _ForEach_ and _Where_ methods.  In this case, the advantage of having these commands comes from the elegant syntax and functionality they provide, plus being able to use these in scripts for PowerShell 3.0 and 4.0.  The performance benefits in ForEach and Where are only in the PowerShell 4.0 native implementation.

### PowerShell 3.0 requires brackets around all method parameters

I mentioned in the examples above that I was able to invoke a method with a single literal script block parameter without wrapping that literal script block in additional brackets.  That capability exists only in PowerShell 4.0 or later due to improvements that were made to the parser in that release.  In PowerShell 3.0, the parser does not support this, so brackets are always required in order for the _ForEach_ and _Where_ script methods to work with a single literal script block parameter in that version.

### Conclusion

When I started trying to mimic the behaviour of the _ForEach_ and Where magic methods in PowerShell 3.0, I didn’t quite realize how much functionality they provided.  Digging into the technical details behind these methods so that I could create the extensions I wanted in _TypePx_ helped uncover all of the hidden features in these very powerful methods, and I am very happy to share those with you in this article.  I hope this information helps you leverage this wonderful new set of features in your PowerShell work, even if you’re still using PowerShell 3.0.  Happy scripting!

[1]: https://github.com/KirkMunro/TypePx
[2]: https://msconfiggallery.cloudapp.net/packages/TypePx/
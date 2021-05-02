---
title: Creating PowerShell custom objects
author: Jakub Jareš
type: post
date: 2013-02-04T17:00:21+00:00
url: /2013/02/04/creating-powershell-custom-objects/
categories:
  - How To
tags:
  - How To

---
Creating PowerShell custom objects is a common task that some find hard to do right. In this article I&#8217;ll show a sample code that has some of the common flaws. We will go through the code, describe what is wrong, and I’ll show one way to correct it. Then I will add a few more ways to create custom objects. The example code is as follows:

```
$groups = 'Group1', 'Group2'
$users = 'User1', 'User2'

$objectCollection=@()

$object = New-Object PSObject
Add-Member -InputObject $object -MemberType NoteProperty -Name Group -Value ""
Add-Member -InputObject $object -MemberType NoteProperty -Name User -Value ""

$groups | ForEach-Object {
	$groupCurrent = $_
	$users | ForEach-Object {
		$userCurrent = $_
   		$object.Group = $groupCurrent
		$object.User = $userCurrent
		$objectCollection += $object
	}
}

$objectCollection
```

The code above creates custom objects with two properties:  Group and User. The properties are filled with data from the $groups variable and the _$users_ variable. Combining two and two items we get four objects in total. This is the output we expect:

<pre class="brush: powershell; title: ; notranslate" title="">Group  User
-----  ----
Group1 User1
Group1 User2
Group2 User1
Group2 User2
</pre>

Implementing this we first create the data to work on and the object using the _New-Object_ cmdlet. This object will be used as a template so we also add empty properties using the _Add-Member_. Then we use two loops, one placed into the other. The first for each loop iterates over the values in the _$groups_ variable and the second iterates over the values in the $users variable; this enables us to easily combine the values. In both loops we save the item we are currently working with to appropriately named variable, _$groupCurrent_ or _$userCurrent_. In the inner loop we use these variables to set the properties of our template object to correct values, we also add the object to the _$objectCollection_ there.

After processing all items we output the _$objectCollection_ array and get this result:

<pre class="brush: powershell; title: ; notranslate" title="">Group  User
-----  ----
Group2 User2
Group2 User2
Group2 User2
Group2 User2
</pre>

The code does not work as expected. The property values are the same for all four objects.

The problem is that we create the object ‘before’ we enter the foreach loop. We assign the information to the object and add it to the collection, but we still use the _same_ object. When we added it to the collection we added just a ‘reference’ to the object, not the object itself. What we ended up with was just four ‘shortcuts’ to the same object. To fix that we need to create new object each time in the second ForEach loop, as in:

```
$groups = 'Group1', 'Group2'
$users = 'User1', 'User2'

$objectCollection=@()

$groups | ForEach-Object {
	$groupCurrent = $_

	$users | ForEach-Object {
        $userCurrent = $_

        $object = New-Object PSObject
        Add-Member -InputObject $object -MemberType NoteProperty -Name Group -Value ""
        Add-Member -InputObject $object -MemberType NoteProperty -Name User -Value ""

        $object.Group = $groupCurrent
        $object.User = $userCurrent

        $objectCollection += $object
	}
}

$objectCollection
```

The script is fixed and works as expected but there are still few things we can do easier. First, creating the array and saving the objects manually is not necessary, we can just assign the output to a variable and PowerShell will create the collection automatically. Second we can use the &#8216;Property&#8217; parameter of the _New-Object_ to create the properties easier. In the end the code may look as such:

```
$groups = 'Group1', 'Group2'
$users = 'User1', 'User2'

$objectCollection = $groups |
    ForEach-Object {
    	$groupCurrent = $_
        $users | ForEach-Object {
        	$userCurrent = $_
            $properties = @{
            	User=$userCurrent;
               	Group = $groupCurrent
             }
			New-Object -TypeName PSObject -Property $properties
		}
	}
$objectCollection
```

This way of creating custom objects is great when you create the object in just one place, and if the object has just few properties.

Sometimes you need to create an object that has properties set to default values and use it as template.  In situations like this you can use the _Copy()_ method of _PSObject_. Let’s rewrite our sample code to reflect this need, and add one property called ‘dummy’ that we set to default value:

```
$groups = 'Group1', 'Group2'
$users = 'User1', 'User2'

$properties = @{User=''; Group = ''; Dummy = 'Default'}
$objectTemplate = New-Object -TypeName PSObject -Property $properties

$objectCollection = $groups |
	ForEach-Object {
    	$groupCurrent = $_
        $users | ForEach-Object {
        	$userCurrent = $_
			$objectCurrent = $objectTemplate.PSObject.Copy()
			$objectCurrent.group = $groupCurrent
			$objectCurrent.user = $userCurrent
			$objectCurrent
		}
	}
$objectCollection | ft –AutoSize

Group  User  Dummy
-----  ----  -----
Group1 User1 Default
Group1 User2 Default
Group2 User1 Default
Group2 User2 Default
```

As in the first example, we created the object before entering the foreach loops, but this time we saved it in the $objectTemplate variable and more importantly we used a copy of the template object in the loops. This time it works because we don’t use just the reference to the object but we create a new copy of the object in the inner foreach loop.

This approach is great but after you use the _$objectTemplate_ for the last time you should use its _Dispose()_ method to free the memory allocated, and also you must not forget to initialize the object before the first use. Sure the amount of allocated memory is pretty insignificant, and you can set the object defaults as the first thing in the script, but why complicating it? Create a new helper function called ‘_New-ReportLine_’ that takes ‘User’, ‘Group’ and ‘Dummy’ parameters and outputs the new object. See the next example:

```
Function New-ReportLine ($Group='GroupDefault',$User='UserDefault',$Dummy='Default' )
{
	New-Object -TypeName psObject -Property @{Group = $group; User=$user; Dummy= $Dummy}
}

$groups = 'Group1', 'Group2'
$users = 'User1', 'User2'

$objectCollection = $groups |
	ForEach-Object {
    	$groupCurrent = $_
        $users | ForEach-Object {
        	$userCurrent = $_
			New-ReportLine -Group $groupCurrent -User $userCurrent
		}
	}

$objectCollection
Group  User  Dummy
-----  ----  -----
Group1 User1 Default
Group1 User2 Default
Group2 User1 Default
Group2 User2 Default
```

Here we basically took the same approach as in the third example (we are creating the object in the inner foreach loop), but this time we use the default parameter values and the scoping of the function. That approach enables us to get rid of the need to call _Dispose()_ and still we have the ability to set the properties to default values.

To show you another example where the pros of this approach really shows up see the next example:

```
function IsServerRemotable ($Name) {$name -eq 'localhost'}
function IsServerOnline ($Name) {$name -eq 'localhost'}

Function New-ReportLine ($Server,[switch]$isOnline,[switch]$isRemotable)
{
	New-Object -TypeName psObject -Property @{Server=$Server; isOnline=$isOnline; isRemotable=$isRemotable}
}

$servers = 'localhost', 'nonexistent'

$report = $servers | ForEach-Object {
	if (IsServerOnline -Name $_)
	{
		if (IsServerRemotable -Name $_)
		{
			#server is both online and remotable
			New-ReportLine -Server $_ -isOnline -isRemotable
         }
		else
		{
			#server is just online
			New-ReportLine -Server $_ -isOnline
		}
		}
		else
		{
			#server is not online
			New-ReportLine -Server $_
		}
	}

$report

Server      isOnline isRemotable
------      -------- -----------
localhost   True     True
nonexistent False    False
```

The function _New-ReportLine_ creates a new object when invoked. The Server parameter accepts the name of the server. The _isOnline_ and the _isRemotable_ parameters default to False as switch parameters always do when they are not present in the function call. In the script body we pipe the items from the $servers to the foreach loop and test if the server is online. If the server is online we test if the server is accessible by PowerShell remoting. If the server is accessible by PowerShell remoting appropriate object is returned.  In other cases we also call the ‘constructor’ function using appropriate set of parameters.

Both _IsServerOnline_ and _IsServerRemotable_ are just pseudo functions to make the examples work of the shelf.
---
title: '#PSTip Working with Excel constants'
author: Shay Levy
type: post
date: 2013-03-20T18:00:41+00:00
url: /2013/03/20/pstip-working-with-excel-constants/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When automating Microsoft Excel, almost every time you&#8217;ll need to include some constant values to define the behaviour of the code. Excel (and other Office applications) makes extensive use of constant values via enumeration objects. For example, to right align cell content you need to know in advance the value of the _xlRight_ constant. Most of the time you&#8217;ll lookup the value using an Internet search or use some VBA-Fu techniques and hard code it in your script.

```
$xlRight = -4152
$sheet.Range('A1').HorizontalAlignment = $xlRight
```

In this tip I want to suggest a convenient  way  to work with those constants, a way that utilizes tab completion and that also doesn&#8217;t require you to know the numeric value of the constant in question. Now it&#8217;s very easy to discover all constants that have &#8216;right&#8217; in their name.

![](/images/xlEnum.png)

The following code iterates over all Enumeration objects of Excel, converge all values into one PowerShell custom object. Now whenever you need to specify one of the constants, it&#8217;s very easy to discover the value.

```
# create Excel object
$xl = New-Object -ComObject Excel.Application

# create new PowerShell object
$xlEnum = New-Object -TypeName PSObject

# get all Excel exported types of type Enum
$xl.GetType().Assembly.GetExportedTypes() | Where-Object {$_.IsEnum} | ForEach-Object {
	# create properties from enum values
	$enum = $_
	$enum.GetEnumNames() | ForEach-Object {
    	$xlEnum | Add-Member -MemberType NoteProperty -Name $_ -Value $enum::($_)
	}
}
```

Now you can use constants without hard coding them first.

$sheet.Range('A1').HorizontalAlignment = $xlEnum.xlRight


If you still need to find the numeric value:

```
PS> $xlEnum.xlRight.value__
-4152
```


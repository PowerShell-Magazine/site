---
title: Preparing for a PowerShell interview!
author: Ravikanth C
type: post
date: 2013-07-15T16:00:29+00:00
url: /2013/07/15/preparing-for-a-powershell-interview/
views:
  - 40257
post_views_count:
  - 5408
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
Recently, I was interviewing candidates for a couple of PowerShell automation vacancies. Being a PowerShell enthusiast, I, naturally, focused on understanding how well these candidates understand PowerShell and how well they can solve some problems on the fly. The idea I had in mind when I started these interviews was to select a couple of them with at least beginner level knowledge of PowerShell.

This post is a compilation of notes I made during these interviews and some more thoughts on how the candidates or aspirants can prepare themselves better to face a PowerShell interview. This is certainly not a post about the questions you can expect in a PowerShell interview and the answers. If you are a PowerShell novice, you may find this post useful in getting started with PowerShell.

Here is what I had in my notes.

### **Understand what Windows PowerShell is!**

My first question was always **&#8220;Tell me, what is Windows PowerShell?&#8221;**

PowerShell is not just any other shell. PowerShell is an object-based distributed automation engine, scripting language, and command line shell. I don&#8217;t expect a beginner to understand every aspect of what I just mentioned but at the least, I expect a candidate to describe the object-based nature of PowerShell. Saying that &#8220;PowerShell is object-based because it uses .NET Framework under the covers&#8221; is not a very accurate statement. PowerShell is object-based because it deals with objects and not text. Let me show you an example.

Here is how I get the size of a folder (including its sub-folders and so on) in a DOS batch script:

```shell
@echo off
For /F "tokens=*" %%a IN ('"dir /s /-c /a | find "bytes" | find /v "free""') do Set xsummary=%%a
For /f "tokens=1,2 delims=)" %%a in ("%xsummary%") do set xfiles=%%a&set xsize=%%b
Set xsize=%xsize:bytes=%
Set xsize=%xsize: =%
Echo Size is: %xsize% Bytes
```


Do you see the pain? How many of you understand what exactly the batch script is doing?

Well, let us see how we do that in PowerShell.

```powershell
Get-ChildItem –Recurse | Measure-Object -Property Length -Sum
```


Simple? At least, it looks simple on the surface. This is made possible because PowerShell deals with objects&#8211;the self-describing entities. These objects have properties and one such property on a file object is _Length_ that describes the size of the file in bytes. So, when we add up the size of all files in a folder, we get the size of the folder. If you compare this to the DOS batch example, we are not dealing with any temporary variables or text parsing. We are simply piping the output of _Get-ChildItem_ cmdlet to _Measure-Object_ and summing the value of _Length_ property of each object that passes through the pipeline.

Now, that is a very simplified version of explaining object-based nature of PowerShell. You can give more such examples as you start using PowerShell.

Another observation from these interviews was how the candidates answered questions on properties and methods available on objects. The question was like &#8220;**get me the value of CPU affinity (without giving the actual property name) of a process**&#8220;. How do you do that?

I have seen quite a few doing the following:

```powershell
$process = Get-Process -Name notepad
```


So far so good. And, then, they start doing:

_$process.<Tab> <Tab> <Tab> &#8230; <Tab>_ until they reach the required property.

Well, this certainly gets you the property but what if the property is the 99th one in a set of 100 properties? Not scalable, right? This is where knowing how to use the shell helps you. There is a better way to do that in the shell.

If you know the exact property name:

```powershell
Get-Process -Name Notepad | Select-Object ProcessorAffinity
```


Or

```powershell
$process = Get-Process -Name Notepad
$process.ProcessorAffinity
```


If you don&#8217;t know the exact property name, don&#8217;t keep using Tab! Instead, discover PowerShell. That is the next thing I want to discuss.

### Learn to discover PowerShell

One of the most important things for a beginner is to understand how to use the following cmdlets:

  * Get-Command
  * Get-Member
  * Get-Help

#### Get-Command

_Get-Command_ gives a list of all commands available in a PowerShell session. One common observation I made during these interviews was the candidates try a non-existent or incorrect cmdlet name and then keep wondering why that was not working. If we know how to discover PowerShell, we can use the _Get-Command_ cmdlet to test if the command we are trying exists or not. For example:

```powershell
PS C:\> Get-Command -Name Get-ComputerName
Get-Command : The term 'Get-ComputerName' is not recognized as the name of a cmdlet, function, script file, or operable program. Check
the spelling of the name, or if a path was included, verify that the path is correct and try again.
At line:1 char:1
+ Get-Command -Name Get-ComputerName
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : ObjectNotFound: (Get-ComputerName:String) [Get-Command], CommandNotFoundException
+ FullyQualifiedErrorId : CommandNotFoundException,Microsoft.PowerShell.Commands.GetCommandCommand
```


See that? You can also use _Get-Command_ to get commands with a specific noun or verb or even use wildcards to filter.

```powershell
Get-Command -Noun Host
Get-Command -Verb Write
Get-Command N*
```


#### Get-Member

In an earlier paragraph, on using the shell, we saw an example on getting specific properties of an object. But, you can do so only if you know the exact property name. How do we discover what properties and methods are available on an object? _Get-Member_ helps here.

```powershell
Get-Process | Get-Member
```


The above command gives all properties, methods, and events of a particular object type. You can filter it further to see the type of member you are interested in.

```powershell
Get-Process | Get-Member -MemberType Method
Get-Process | Get-Member -MemberType Property
Get-Process | Get-Member -MemberType Event
```


#### Get-Help

Another common mistake across the board was the lack of knowledge in using the built-in help. PowerShell built-in cmdlets come with lot of help around how to use the cmdlet. Of course, with PowerShell 3.0 and above, you have to first update this help content using _Update-Help_ cmdlet. When you don&#8217;t know the exact syntax of a PowerShell cmdlet, you can always use _Get-Help_ cmdlet. This gives you information about cmdlet parameters, how to use each of those parameters, and provides some examples. When in doubt, you should first check the local help system.

```powershell
Get-Help Get-Command
Get-Help Get-Member -Detailed
Get-Help Get-Process –Examples
Get-Help Get-Service –Parameter InputObject
```


### Use the shell

I don&#8217;t expect a beginner to have the knowledge of writing scripts and modules. I look for people who know how to use PowerShell as a shell. For this, you must have really used the built-in cmdlets. This includes simple things like listing the services, processes, files, and folders. If you don&#8217;t know how to use _Get-ChildItem_ to perform a recursive search for files, I am sure you wouldn&#8217;t have even touched the surface of PowerShell. IMHO, you are not even a beginner.

When learning PowerShell, you should always start at the shell. More or less everything that runs at the PowerShell command prompt, runs as a script too. Start automating simple tasks at the shell using the built-in cmdlets and the PowerShell language. Eventually, you will learn to write scripts. This is one of the most important aspects of getting started with PowerShell.

I have also heard candidates saying that they know only product-specific PowerShell cmdlets and they can use only those cmdlets. Now, tell me how you would automate a task in PowerShell that configures a product using only product-specific cmdlets without ever using any of the built-in cmdlets or pipeline or PowerShell language. This is not possible. When someone says so, I know that they don&#8217;t understand the basics or never used the shell.

#### Know the pipeline

For working with PowerShell and to make efficient use of it, you need to understand what pipeline is and how you can combine multiple commands using a pipeline. I showed examples using the pipeline in the above paragraph but did not really explain what it is. Running simple commands is not a difficult task. You will realize the real benefit of PowerShell command line only if you can use the pipeline to combine multiple commands to achieve a bigger task. A complete discussion on pipeline will easily be a 50 &#8211; 75 pages of book. Let us keep it simple for the purpose of this article.

Think of PowerShell pipeline as an assembly line in a manufacturing unit. In a manufacturing unit, components move from one station to another and the final output gets built along the way. The end of the pipeline is where we see the completed product. Similar to this analogy, when we combine multiple PowerShell cmdlets using pipeline, the output of one command becomes an input to the next command in the pipeline. For example:

```powershell
Get-Process -Name s* | Where-Object { $_.HandleCount -lt 100 }
```


In the above command, one or more process objects from _Get-Process_ cmdlet is given as an input to the _Where-Object_ cmdlet. The _Where-Object_ cmdlet then filters the object array for the objects with _HandleCount_ property less than 100. You can certainly write something like this without a pipeline too. Let us see how that is done.

```powershell
$process = Get-Process -name s*
foreach ($proc in $process) {
   if ($proc.HandleCount -lt 100) {
       $proc
   }
}
```


As you see, there is more code. While that is not a big deal, you will see a difference in how the output is generated. Related to this, many candidates mentioned that the first command completes execution and all objects move from first command to second. This is not accurate. In case of a pipeline, the objects stream from the first command to the second command as soon they are available.

This previous example combines only two commands which may look very trivial. But, think of something like the below example:

```powershell
Get-ChildItem -Recurse -Force |
Where-Object {$_.Length -gt 10MB} |
Sort-Object -Property Length -Descending |
Select-Object Name, @{name='Size (MB)'; expression={$_.Length/1MB}}, Extension |
Group-Object -Property extension
```


If you are beginner, I don&#8217;t expect you to understand this or write something like this. But, this shows the usefulness of pipeline. The above command (or pipeline of commands) grabs all files that are bigger than 10MB, sorts them by file size in descending order, and groups them by the file extension. Can you try converting this code snippet to achieve the same task without using the pipeline?

### Don&#8217;t over-engineer

PowerShell, by its very nature, provides more than one way to do things. Of course, not all of these approaches are the same. There are efficient and inefficient ways. There are simple and complicated ways.

So, when I ask &#8220;**get me the name of the computer**&#8220;, I don&#8217;t expect you to write a WMI query.

```powershell
Get-WmiObject -Class Win32_ComputerSystem | Select-Object -Property Name
```


or

```powershell
Get-WmiObject -Class Win32_OperatingSystem | Select-Object -Property CSName
```


You bet, these commands get you the local computer name. But, understand that there is an easy and better way to do that.

```powershell
$env:ComputerName
```


Oh, I got the classic _hostname_ command as an answer as well. Nothing wrong. I use it many times. But, it is a PowerShell interview. Right?

### Write scripts

This has been another constant disappointment. You may know how to run scripts others have written. But, would that make you a scripter? No way. Reading scripts written by others will certainly help you understand the best and worst practices. But, it won&#8217;t help you learn if you are a beginner. You learn only when you start writing scripts on your own.

Also, don&#8217;t tell an interviewer that you wrote advanced functions unless you know how to describe common parameters, parameter types, cmdletbinding, and how Begin, Process, and End blocks work and why they are required. If you don&#8217;t understand these concepts, I don&#8217;t consider that you have ever written an advanced function in PowerShell. The advanced functions are different from the regular functions we write in PowerShell. When you add _CmdletBinding()_ attribute to a function definition, the basic behavior of a function changes. Why not see an example?

Here is a regular function that accepts two numbers as an input and returns the sum as output.

```powershell
function sum {
    param (
       $number1,
       $number2
    )
    $number1 + $number2
}

PS C:\> sum 10 30
40
```

Now, call this function with different number of parameters.

```powershell
PS C:\> sum 10 30 40
40
```


See that? Although we have only two parameters in the function definition, it accepts three arguments and simply ignores the third argument. Now, add the _CmdletBinding()_ attribute and see how the behavior changes.

```powershell
function sum {
   [CmdletBinding()]
   param (
      $number1,
      $number2
   )
   $number1 + $number2
}
```


Do the test again with same set of arguments as earlier!

```powershell
PS C:\> sum 10 30
40

PS C:\> sum 10 30 40
sum : A positional parameter cannot be found that accepts argument '40'.
At line:1 char:1
+ sum 10 30 40
+ CategoryInfo : InvalidArgument: (:) [sum], ParameterBindingException
+ FullyQualifiedErrorId : PositionalParameterNotFound,sum
```

Got it? The basic behavior of handling input parameters got changed after we added the _CmdletBinding()_ attribute. This is an advanced PowerShell function. But, this is just the beginning. I won&#8217;t go into the details of advanced functions here but as an interviewer, I&#8217;d expect you to know more if you ever tell me that you wrote advanced functions.

### Don&#8217;t depend on search engines

One thing I heard from many candidates is that they depend on Google when they want to write a script. They just use a search engine, take the ready-made solution for the problem and use it. This has been a common response before they give up after a few minutes into writing a script or completing a task. I use search engines when I am completely stuck and cannot move forward with what I am doing. Or when I know that someone already had written a script and I don’t want to re-invent the wheel. But, I won’t use this method when my goal is to learn PowerShell. A search engine is the easiest way to find a solution but, you won&#8217;t learn anything but how to use search engine. Especially, when you are a beginner, using the ready-made scripts won’t take you anywhere in your learning. And, of course, this is when I generally end the interview! 🙂

Good luck!
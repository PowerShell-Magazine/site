---
title: Calling native commands from PowerShell
author: Jakub Jareš
type: post
date: 2013-01-02T17:00:14+00:00
url: /2013/01/02/calling-native-commands-from-powershell/
categories:
  - How To
tags:
  - How To

---
Every now and then I find myself facing task I am not sure how to solve using without using native commands. Just writing the native command followed by its parameters in PowerShell host and hitting Enter is usually not enough to run the command successfully. There are several ways to make the command work, let me show you the one I found most convenient.

In my examples I am going to use &#8216;_icacls_&#8216;, native command that allows you to set file permissions easily. I don&#8217;t want to make any changes to your current files so first create a temporary file using technique shown [in this tip][1] and save its path into the _$path_ variable.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $path = [IO.Path]::GetTempFileName()
</pre>

Next run the command as you would in Windows Command line to see it raise an exception in PowerShell:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; icacls $path /grant Administrators:(D,WDAC)
At line:1 char:38
+ icacls $path /grant Administrators:(D,WDAC)
+                                      ~
Missing argument in parameter list.
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : MissingArgument
</pre>

To be able to run the command in PowerShell host successfully you need to put the problematic statement in single quotes like this:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; icacls $path /grant 'Administrators:(D,WDAC)'
processed file: C:\Users\mo\AppData\Local\Temp\tmpE4F2.tmp
Successfully processed 1 files; Failed processing 0 files
</pre>

Figuring out what must be put in single quotes can get tiresome, but fortunately there is way to automate it with function like this:


    function Invoke-NativeExpression
    {
        param (
    	[Parameter(Mandatory=$true,ValueFromPipeline=$true,Position=0)]
    	[string]$Expression
        )
        process
        {
            $executable,$arguments = $expression -split ' '
            $arguments = $arguments | foreach {"'$_'"}
            $arguments = $arguments -join ' '
            $command = $executable + ' ' + $arguments
    
        if ($command)
            {
                Write-Verbose "Invoking '$command'"
                Invoke-Expression -command $command
            }
        }
    }
The function converts the original expression to this format _executable &#8216;argument1&#8217; &#8216;argument2_ and passes it to the _Invoke-Expression_ cmdlet.

Using the shown function the native command can be called as such:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; "icacls $path /grant Administrators:(D,WDAC)" | Invoke-NativeExpression
processed file: C:\Users\mo\AppData\Local\Temp\tmpE4F2.tmp
Successfully processed 1 files; Failed processing 0 files
</pre>

You can also pass the command as named parameter or by position. Making calling the native commands easy.

In PowerShell 3.0, there is a new symbol called the stop-parsing symbol (_&#8211;%_) that makes using native commands a little easier. The _&#8211;%_ symbol prevents PowerShell from parsing anything until the end of the line enabling you to call the command like this:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Icacls $path --% /grant Administrators:(D,WDAC)
</pre>

Notice that you must place the symbol after all the variables that should be expanded, in this case _$path_, making this approach applicable [only in some cases][2]. If you are finished playing with the temporary file you can delete it by this command:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Remove-Item -Force -Path $path
</pre>

[1]: /2012/12/04/pstip-generate-a-zero-byte-temporary-file-on-disk
[2]: http://technet.microsoft.com/en-us/library/hh847892.aspx
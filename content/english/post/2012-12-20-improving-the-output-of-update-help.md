---
title: Improving the output of Update-Help
author: Shay Levy
type: post
date: 2012-12-20T17:00:48+00:00
url: /2012/12/20/improving-the-output-of-update-help/
categories:
  - How To
tags:
  - How To

---
If you&#8217;ve updated PowerShell to version 3.0 you probably noticed that help is no longer shipped in the box. PowerShell 3.0 includes a new feature, the [Updatable Help system][1], which allows you to ensure that help on the local computer stays up to date.

PowerShell is now a part of the operating system and operating systems gets to be updated only during service packs or specific patches. The Updatable Help system and its cmdlets (_*-Help_) make it easy to download and install help files, or updating exiting help files, as soon as newer help files become available.

To download and install help files for the first time, use the _Update-Help_ cmdlet. When executed, _Update-Help_ goes through a series of steps to get the latest help versions of the modules installed on your system:

  1. It determines which modules support updatable help by checking the _HelpInfoUri_ key in the module manifest file.  The _HelpInfoUri_ contains the Internet location where each module stores its updatable help files.
  2. Compares each module local help files with the newest help files that are available for each module.
  3. Downloads the new files (packaged in a cab file).
  4. Unwraps the help file package, verifies that the files are valid, and then installs the help files in the language-specific subdirectory of the module directory.

**Note**: There are a few things to take into account when using _Update-Help_. First, you must run PowerShell as an administrator as help files are written to the installation folder of PowerShell and that happens to be under the _System32_ folder. PowerShell allows you to update help files once every 24 hours. To override this behaviour you must specify the _-Force_ switch.

By default, the _Update-Help_ cmdlet doesn&#8217;t generate any output. When executed, it displays a progress bar that prints information about the current module update phase.

![](/images/updateHelp.png)

If you want to see what&#8217;s going on under the hood, include the _-Verbose_ switch:

![](/images/updateHelp1.png)

One of the things that really annoys me is the output of the _-Verbose_ switch, the way it is written makes it very hard to read and determine which files and modules has been updated. In this post I want to introduce you to a new feature in PowerShell 3.0 that can help you change the way the verbose information is displayed in the console.

In the previous version of PowerShell it was very hard to capture the output of the _Verbose_ stream, or any other PowerShell-related stream. Luckily, this has changed in PowerShell 3.0 and now it is a very easy thing to do. We can now redirect and merge any of the pipeline output streams (see list below) to text files. You can read more about this in [about_Redirection][2] help topic.

By default, the _Update-Help_ command doesn’t write anything to the pipeline so we can safely merge the verbose stream to the standard output stream and parse it without having to worry about information from both sources gets mixed together.

Each redirection operator uses a character to represent each output type:

  * All output
  * 1 &#8211; Success output
  * 2 &#8211; Errors
  * 3 &#8211; Warning messages
  * 4 &#8211; Verbose output
  * 5 &#8211; Debug messages

The _Verbose_ stream constant is 4, and _Success_ output is 1, so we use the redirection operator (e.g &#8216;>&#8217;) to funnel the output:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $uh = Update-Help -Verbose -Force 4&gt;&1
</pre>

Let&#8217;s examine the first element; we can see that the connection is redirecting to another URI.

```
PS> $uh[0]
VERBOSE: Your connection has been redirected to the following URI:
VERBOSE: "http://download.microsoft.com/download/3/4/C/34C6B4B6-63FC-46BE-9073-FC75EAD5A136/"
```


The second element in the output of _Update-Help_ contains the information we are after. We can see that the _Microsoft.PowerShell.Management_ was updated, and we also get information about the path of the help file, its culture, and the version information.

```
PS> $uh[1]
VERBOSE: Microsoft.PowerShell.Management: Updated
VERBOSE:
C:\Windows\System32\WindowsPowerShell\v1.0\en-US\Microsoft.PowerShell.Commands.Management.dll-hel
p.xml. Culture en-US
VERBOSE: Version 3.1.0.0
```


Let&#8217;s see what _Get-Member_ has to say about it:

```
PS> $uh[1] | Get-Member
   TypeName: System.Management.Automation.VerboseRecord
Name                  MemberType   Definition
----                  ----------   ----------
Equals                Method       bool Equals(System.Object obj)
GetHashCode           Method       int GetHashCode()
GetType               Method       type GetType()
ToString              Method       string ToString()
WriteVerboseStream    NoteProperty System.Boolean WriteVerboseStream=True
InvocationInfo        Property     System.Management.Automation.InvocationInfo InvocationInfo {get;}
Message               Property     string Message {get;set;}
PipelineIterationInfo Property     System.Collections.ObjectModel.ReadOnlyCollection[int] PipelineIterationInfo {get;}
```

We get a _System.Management.Automation.VerboseRecord_ object which describes a verbose message sent to the verbose stream, and information about the command that sent the message (_InvocationInfo_). The _Message_ property contains the actual message we see in the console. If you need to identify verbose messages written to the success stream, you can safely rely on this type and filter objects accordingly.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $uh[1].Message
Microsoft.PowerShell.Management: Updated C:\Windows\System32\WindowsPowerShell\v1.0\en-US\Microsoft.PowerShell.Commands
.Management.dll-help.xml. Culture en-US Version 3.1.0.0
</pre>

We can split the message (by default the _Split_ method breaks the string on the space character) and get access to the array elements we get back:

```
PS> $uh[1].Message.split()
Microsoft.PowerShell.Management:
Updated
C:\Windows\System32\WindowsPowerShell\v1.0\en-US\Microsoft.PowerShell.Commands.Management.dll-help.xml.
Culture
en-US
Version
3.1.0.0
```


Based on the output of the _Split_ operation, we want to extract the relevant pieces of information, create a new custom object and write it back to the pipeline, one object at a time.

The information we want is:

  1. The module name&#8211;it's the first item (index 0). The value is followed by a colon, we'll remove it later on.
  2. The file path that&#8217;s being updated (third line, index 2).
  3. The culture of the help file (fifth line, index 4).
  4. The version of the new help file (last line, we can refer to it as index -1, which in PowerShell gives back the last array element).

We can now construct a new custom object. We'll start by creating a custom object for the first array element:

```
$message = $uh[1].Message.split()[0,2,4,-1]

[PSCustomObject]@{
	Module = $message[0] -replace ':$'
	FileName = (Split-Path $message[1] -Leaf).Trim('.')
	Culture = $message[2]
	Version  = $message[-1]
}

Module                        FileName                      Culture                       Version
------                        --------                      -------                       -------
Microsoft.PowerShell.Manag... Microsoft.PowerShell.Comma... en-US                         3.1.0.0
```

As soon as each object (verbose message) is processed, the object goes out to the console. As you can see, the output of the _Module_ and _FileName_ properties is truncated.

We could use the _-AutoSize_ of the _Format-Table_ cmdlet to adjust the columns size, but doing so will block output of objects to the console until all objects were processed.

When processing the verbose stream we also want to avoid processing unnecessary messages, we want to skip any messages that contains URI redirections, so we process only messages that contains the word &#8220;Updated&#8221;. We pipe the custom objects to the _Tee-Object_ cmdlet, to send output to the console as soon as it flows in, and also save the output to a variable that we can format the way we want it to.

Here&#8217;s the full snippet. Output shown on screen is also saved in the _UpdatedHelp_ variable. When the script finished executing we can investigate and format it as we like.

```
Update-Help -Force -Verbose 4>&1 |
Where-Object {$_.Message -like '*: Updated*'} |
ForEach-Object {
$message = $_.Message.Split()[0,2,4,-1]

[PSCustomObject]@{
    Module = $message[0] -replace ':$'
    FileName = (Split-Path $message[1] -Leaf).Trim('.')
    Culture = $message[2]
    Version  = $message[-1]
}
} | Tee-Object -Variable UpdatedHelp

$UpdatedHelp | Format-Table -AutoSize
```

![](/images/updateHelp2.png)

Lastly, if you run the code more than once you&#8217;ll notice that you get the same output over and over again.

I&#8217;m not sure why it happens (and I reported this [as a bug][3]) but _Update-Help_ reports any on-line help file that matches the ones on your local system.

[1]: http://technet.microsoft.com/en-us/library/hh847735.aspx
[2]: http://technet.microsoft.com/en-us/library/hh847746.aspx
[3]: https://connect.microsoft.com/PowerShell/feedback/details/774859/update-help-verbose-messages
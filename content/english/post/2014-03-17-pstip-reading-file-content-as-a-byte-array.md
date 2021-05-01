---
title: '#PSTip Reading file content as a byte array'
author: Shay Levy
type: post
date: 2014-03-17T18:00:17+00:00
url: /2014/03/17/pstip-reading-file-content-as-a-byte-array/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

A few days ago I had to write a script to upload files to a remote FTP server. I needed to read the file (0.7 mb) and store it as a byte array. My first attempt to do this was to use the _Get-Content_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Content c:\test.log -Encoding Byte
</pre>

It works great but there&#8217;s only one downside to it&#8211;it is painfully slow&#8211;and I quickly resorted to an alternative method using a .NET class:

<pre class="brush: powershell; title: ; notranslate" title="">[System.IO.File]::ReadAllBytes('c:\test.log')
</pre>

_ReadAllBytes()_ worked incredibly fast in compare to the cmdlet. I measured how much it took for each command to finish. _Get-Content_ took 18.308045 seconds to complete while _ReadAllBytes()_ took only 0.2811065!

I had a time limit to finish the script so I left it with the .NET method and decided to check later what can be done to make _Get-Content_ perform faster. Later on I came back to it and checked the help of _Get-Content_. The answer was found in the _ReadCount_ parameter. The default behavior is sending one line at a time, in my case it was one byte at a time.

    PS> Get-Help Get-Content -Parameter ReadCount
    
    -ReadCount
        Specifies how many lines of content are sent through the pipeline at a time. The default value is 1. A value of 0
        (zero) sends all of the content at one time.
        This parameter does not change the content displayed, but it does affect the time it takes to display the content.
        As the value of ReadCount increases, the time it takes to return the first line increases, but the total time for
        the operation decreases. This can make a perceptible difference in very large items.
    
        Required?                    false
        Position?                    named
        Default value                1
        Accept pipeline input?       true (ByPropertyName)
        Accept wildcard characters?  false
I changed it to 0 so all content can be read in a single operation and then I measured again its execution time.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Content c:\test.log -Encoding Byte -ReadCount 0
</pre>

At first glance the result looked very similar to the .NET method, but to my big surprise, it was even faster to complete&#8211;only 0.2384541 seconds!
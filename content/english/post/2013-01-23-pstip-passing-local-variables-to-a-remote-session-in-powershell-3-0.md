---
title: '#PSTip Passing local variables to a remote session in PowerShell 3.0'
author: Jan Egil Ring
type: post
date: 2013-01-23T19:01:58+00:00
url: /2013/01/23/pstip-passing-local-variables-to-a-remote-session-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Passing local variables to a remote session has become a lot easier in PowerShell 3.0 compared to previous versions. Let&#8217;s have a look at a few examples. Consider the following variables defined in your local PowerShell session:

<pre class="brush: powershell; title: ; notranslate" title="">$a = 'PowerShell'
$b = 'Rocks'
</pre>

If you want to use these variables in a remote session, you will see that the following will not work:

```
$a = 'PowerShell'
$b = 'Rocks'

Invoke-Command -ComputerName Server01 -ScriptBlock {
   Write-Output The value of variable a is: $a
   Write-Output The value of variable b is: $b
}

The value of $a is:
The value of $b is:
```

In PowerShell 2.0 we had two options in order to pass our local variables to the remote session. The first option was to pass on the variables to the _–ArgumentList_ parameter. We then had to pick them up from the _$args_ array, and keep track of the order we passed the variables in. For example, the first variable specified on the _–ArgumentList_ parameter could be picked up accessing _$args[0]_, the second variable accessing _$args[1]_, and so on. As we can see in the following example:

```
Invoke-Command -ComputerName Server01 -ScriptBlock {
   Write-Output The value of variable a is: $($args[0])
   Write-Output The value of variable b is: $($args[1])
} -ArgumentList $a,$b

The value of $a is: PowerShell
The value of $b is: Rocks
```

The second option was to use the –ArgumentList parameter and a param block:

```
Invoke-Command -ComputerName Server01 -ScriptBlock {
   param ($first,$second)

   Write-Output The value of variable a is: $first
   Write-Output The value of variable b is: $second
} -ArgumentList $a,$b

The value of $a is: PowerShell
The value of $b is: Rocks
```

In PowerShell 3.0 we can simply use the Using scope modifier followed by a colon and the name of the local variable we want to reference:

```
Invoke-Command -ComputerName Server01 -ScriptBlock {
   Write-Output The value of variable a is: $using:a
   Write-Output The value of variable b is: $using:b
}

The value of $a is: PowerShell
The value of $b is: Rocks
```


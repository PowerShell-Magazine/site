---
title: '#PSTip Dynamically hiding a function from the debugger in PowerShell ISE'
author: Shay Levy
type: post
date: 2013-08-23T18:00:59+00:00
url: /2013/08/23/pstip-dynamically-hiding-a-function-from-the-debugger-in-powershell-ise/
categories:
  - Columns
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 4.0 or above.

In [this great post][1], Tobias walked you through the process of hiding functions from the debugger. In this tip, I want to show a dynamic way of doing the same, a way that doesn&#8217;t require access to the function code if for any reason you cannot decorate a function with the mentioned attribute. Here goes&#8230;

Starting in Windows 4.0 script blocks have a new property: _DebuggerHidden_.

```
PS> ${function:test}|gm d*
   TypeName: System.Management.Automation.ScriptBlock
Name           MemberType Definition
----           ---------- ----------
DebuggerHidden Property   bool DebuggerHidden {get;set;}
```

You can get its value to determine if a function is hidden from the debugger but more importantly, you can set its value and instruct the debugger to skip the function. Let&#8217;s create a function and try to debug it. Put the following code in a script file and save it (debugging works only for saved scripts).

![](/images/DebuggerHidden1.png)

Now move your cursor to the function call (line 6) and press F9 to place a new breakpoint. Press F5 to execute the script and then start pressing F11 to step into the code, you&#8217;ll see that you&#8217;re the debugger steps into the function body and it is executed line by line until eventually you get the return value of the function.

Now let&#8217;s set ask debugger to ignore and &#8220;step over&#8221; the function:

![](/images/DebuggerHidden2.png)

Repeat the debugging process described earlier. Notice that now, when you press the  F11 key, the debugger &#8220;ignores&#8221; the function and you immediately get the result.

[1]: http://www.powertheshell.com/hiding-code-from-ise-debugger
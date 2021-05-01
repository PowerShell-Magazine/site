---
title: '#PSTip Tab Completion in PowerShell 3.0'
author: Shay Levy
type: post
date: 2013-08-22T18:00:43+00:00
url: /2013/08/22/pstip-tab-completion-in-powershell-3-0/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

PowerShell&#8217;s tab completion just got better in v3. In addition to all its awesomeness, it is now capable of completing service or process names, event log names, module names and the list is long.

```
# event logs
PS> Get-EventLog -LogName

# services
PS> Get-Service -Name

# processes
PS> Get-Process -Name
```

There are many more new tab completion capabilities and features, and in this tip I would like to share with you two more less known locations where tab completion can help you discover possible values.

The first one is using a synthetic (calculated) property using the _Format-*_ cmdlets. Just press the TAB key after the opening brace and you get completion for the hash table keys:

![](/images/isetc1.png)

The second one also works for hash tables. If you press the tab key in the Property parameter of _New-Object_, you get completion of the properties of the object you&#8217;re creating:

![](/images/isetc2.png)

<div>
  <p>
    Note that, in the ISE, you might need to press <em>Ctrl+Space</em> right after the opening brace to invoke Intellisense.
  </p>
</div>
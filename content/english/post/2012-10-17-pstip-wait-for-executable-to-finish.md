---
title: '#PSTip Wait for executable to finish'
author: Jakub Jareš
type: post
date: 2012-10-17T18:00:00+00:00
url: /2012/10/17/pstip-wait-for-executable-to-finish/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Sometimes you need to wait for an executable to finish its job. The typical case is a Setup.exe, but this behavior is also ideal candidate for cleaning up temporary files after an application has been closed. Powershell makes this easy; just redirect the output to a (valid) pipe.

```
& 'C:\Program Files\Internet Explorer\iexplore.exe' | echo "Waiting."
"Internet Explorer has been closed."
```

Please notice that the invoke operator &#8216;&&#8217; is used, and that the &#8220;Waiting.&#8221; message is not shown in the Powershell output.

The typical case is waiting for setup.exe or msiexec.exe to finish, but the behavior is also useful for tasks like cleaning up temporary files or starting external applications one after another.
---
title: '#PSTip How to determine if a file is 32-bit or 64-bit'
author: Shay Levy
type: post
date: 2013-03-08T19:00:07+00:00
url: /2013/03/08/pstip-how-to-determine-if-a-file-is-32bit-or-64bit/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Someone at the office asked me to check why a specific file was failing to execute. When double-clicking the file, nothing happened and no related process could be found in Task Manager. To cut the long story short, we found that the issue was probably running the file on a 32-bit machine while the file was written for x64 platforms.  Anyway, we wanted to verify that and I was looking for a way to do that with PowerShell. After a short web search I found [this StackOverflow thread][1].

Here&#8217;s the translated version of the suggested C# code. It sure verified our suspicion&#8211;the failing  file was written for x64 platforms. Here&#8217;s a test against notepad.exe.

```
$MACHINE_OFFSET = 4
$PE_POINTER_OFFSET = 60
$MachineType = Write-Output Native I386 Itanium x64

$filePath = "$env:windir\notepad.exe"
$data = New-Object System.Byte[] 4096
$stream = New-Object System.IO.FileStream -ArgumentList $filePath,Open,Read
$stream.Read($data,0,$PE_POINTER_OFFSET) | Out-Null
$PE_HEADER_ADDR = [System.BitConverter]::ToInt32($data, $PE_POINTER_OFFSET)
$machineUint = [System.BitConverter]::ToUInt16($data, $PE_HEADER_ADDR + $MACHINE_OFFSET)
$MachineType[$machineUint]
```

[1]: http://stackoverflow.com/a/885481/9833
---
title: '#PSTip Using UNC paths when working with PowerShell providers'
author: Jaap Brasser
type: post
date: 2012-11-05T19:00:04+00:00
url: /2012/11/05/pstip-using-unc-paths-when-working-with-powershell-providers/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When working with different PowerShell providers (PSProviders), the use of UNC paths can lead to error messages. This tip will show the pitfalls of using UNC paths when working with PowerShell providers other than the file system. For a full list of the PSProviders available on your system use the Get-PSProvider cmdlet:

```powershell
PS C:\> Get-PSProvider
Name                 Capabilities                            Drives
----                 ------------                            ------
Alias                ShouldProcess                           {Alias}
Environment          ShouldProcess                           {Env}
FileSystem           Filter, ShouldProcess, Credentials      {C, D, E, Q...}
Function             ShouldProcess                           {Function}
Registry             ShouldProcess, Transactions             {HKLM, HKCU}
Variable             ShouldProcess                           {Variable}
Certificate          ShouldProcess                           {Cert}
WSMan                Credentials                             {WSMan}
```


The output shows the default PowerShell 2.0 providers. If you use PowerShell 3.0 or have imported, for example, ActiveDirectory or WebAdministration module, an output of your command will be different. PowerShell providers expose the PowerShell drives (PSDrive). We can change the current location to the Alias: PowerShell drive using the following command:

```powershell
PS> Set-Location -Path Alias:
```


Use the Get-ChildItem Cmdlet to list the contents of the root of this drive. We can export the output to CSV file without a problem:

```powershell
PS Alias:\> Get-ChildItem | Export-Csv -Path C:\Test.csv
```

However, when we attempt to do this by using a UNC path the command fails:

```powershell
PS Alias:\> Get-ChildItem | Export-Csv -Path \\Server01\c$\TestUNC.csv

Export-Csv : Cannot open file because the current provider (Microsoft.PowerShell.Core\Alias) cannot open a file.
At line:1 char:30

+ Get-ChildItem -Path Alias: | Export-Csv -Path \\Server01\c$\TestUNC.csv
+ CategoryInfo          : InvalidArgument: (:) [Export-Csv], PSInvalidOperationException
```

There are two solutions to work around this issue. We can create a PSDrive that maps to the UNC path:

```powershell
PS Alias:\> New-PSDrive -Name UNCPath -PSProvider FileSystem -Root \\Server01\c$\
PS Alias:\> Get-ChildItem | Export-Csv -Path UNCPath:\TestPSDrive.csv
```


Or explicitly state the PSProvider as shown in the following example:

```powershell
PS Alias:\> Get-ChildItem | Export-Csv -Path Microsoft.PowerShell.Core\FileSystem::\\Server01\c$\TestUNC.csv
The Microsoft.PowerShell.Core\ part of this command can be omitted, shortening the command to:
PS Alias:\> Get-ChildItem | Export-Csv -Path FileSystem::\\localhost\c$\TestUNC.csv
```


For more information regarding PowerShell provider you can refer to the built-in help topic by running Get-Help about_Providers.
---
title: How Current Path Affects Cmdlets
author: Tobias Weltner
type: post
date: 2014-02-26T17:00:55+00:00
url: /2014/02/26/current-path-affects-cmdlets/
categories:
  - How To
tags:
  - How To

---
Why and how would the current path affect how cmdlets work? What exactly is the current path?

I am sure you'll be up for a bunch of surprises! Let me take you on a tour to the not-so-well-known intrinsics of PowerShell paths and providers. And yes, I'll be covering some basics at the beginning, so PowerShell veterans, please hang in and read on. Off we go!

### Current Path – PowerShell Cares

The current path is the path you are currently in (as you probably figured). But there are actually two current paths. PowerShell maintains one, and Windows does, too. And they behave completely different.

From within PowerShell, you manage the current path with the family of *-Location cmdlets:

```
PS> Set-Location -Path $home
PS> Get-Location
Path                                                                          
––––                                                                          
C:\Users\Tobias                                                               

PS> Set-Location -Path c:\

PS> Get-Location
Path                                                                          
––––                                                                          
C:\
```

Current Path for PowerShell

Never heard of Set-Location? You may know Set-Location under one of its nicknames: the common alias is "cd", and as you can see below, there are a couple more aliases you can use, so changing the current PowerShell folder must be an important thing:

```
PS> Get-Command -Name cd
CommandType     Name                                               ModuleName 
–––––––––––     ––––                                               –––––––––– 
Alias           cd –> Set-Location                                            

PS> Get-Alias –Definition Set-Location

CommandType     Name                                               ModuleName 
–––––––––––     ––––                                               –––––––––– 
Alias           cd –> Set-Location                                            
Alias           chdir –> Set-Location                                         
Alias           sl –> Set-Location 
```

 Set-Location is known under a bunch of alias names

First important thing to note: changing the PowerShell current path does not affect the Windows current path at all, they are not synced, they are absolutely separate.

This one defaults to the working directory of your PowerShell host and does not change unless you change it explicitly:

```
PS> [System.Environment]::CurrentDirectory
C:\Users\Tobias
PS> [System.Environment]::CurrentDirectory = 'c:\admin'

PS> [System.Environment]::CurrentDirectory
c:\admin

PS> Get-Location
Path                                                                          
––––                                                                          
C:\ 
```

 Current Path for Windows

### PowerShell Path: Not Just FileSystem

The PowerShell current path supports all the different PowerShell drives, so it can not only point to filesystem locations. This would set the current path to HKEY_CURRENT_USER in the Registry (again, no surprise to PowerShell veterans, but hold on a second for surprises):

```
PS> Set-Location -Path HKCU:\
PS> dir

    Hive: HKEY_CURRENT_USER

Name                           Property                                       
––––                           ––––––––                                       
AppEvents                                                                     
Console                        HistoryNoDup           : 0                     
                               FullScreen             : 0                     
                               ScrollScale            : 1                     
                               ExtendedEditKeyCustom  : 0                     
                               CursorSize             : 25                    
                               FontFamily             : 0                     
                               ScreenColors           : 7                     
                               TrimLeadingZeros       : 0                     
                               WindowSize             : 1638480               
                               LoadConIme             : 1                     
                               PopupColors            : 245                   
                               QuickEdit              : 0                     
                               WordDelimiters         : 0                     
                               ColorTable10           : 65280                 
                               ColorTable00           : 0                     
                               ColorTable11           : 16776960              
                               ColorTable01           : 8388608               
                               ColorTable12           : 255                   
                               ColorTable02           : 32768                 
                               ColorTable13           : 16711935              
                               ColorTable03           : 8421376               
                               ColorTable14           : 65535                 
                               EnableColorSelection   : 0                     
                               ColorTable04           : 128                   
                               ColorTable15           : 16777215              
                               ExtendedEditKey        : 0                     
                               ColorTable05           : 8388736               
                               ColorTable06           : 32896                 
                               ColorTable07           : 12632256              
                               NumberOfHistoryBuffers : 4                     
                               ScreenBufferSize       : 19660880              
                               ColorTable08           : 8421504               
                               ColorTable09           : 16711680              
                               FontWeight             : 0                     
                               HistoryBufferSize      : 50                    
                               FontSize               : 0                     
                               InsertMode             : 1                     
                               CurrentPage            : 1                     
Control Panel                                                                 
Environment                    TMP  : C:\Users\Tobias\AppData\Local\Temp      
                               TEMP : C:\Users\Tobias\AppData\Local\Temp      
EUDC                                                                          
Identities                                                                    
Keyboard Layout                                                               
Network                                                                       
Printers                                                                      
Software                                                                      
System                                                                        
Volatile Environment           LOGONSERVER               : \\MicrosoftAccount 
                               USERDOMAIN                : TOBI2              
                               USERNAME                  : Tobias             
                               USERPROFILE               : C:\Users\Tobias    
                               HOMEPATH                  : \Users\Tobias      
                               HOMEDRIVE                 : C:                 
                               APPDATA                   :                    
                               C:\Users\Tobias\AppData\Roaming                
                               LOCALAPPDATA              :                    
                               C:\Users\Tobias\AppData\Local                  
                               USERDOMAIN_ROAMINGPROFILE : TOBI2              
```

 PowerShell paths can point to any provider location, not just the filesystem

So you can set the PowerShell current path to any PSDrive that you (or your friendly modules) have set up:

```
Name     Used (GB) Free (GB) Provider    Root               CurrentLocation
––––     ––––––––– ––––––––– ––––––––    ––––               –––––––––––––––
Alias                        Alias                                        
C            74,55    362,30 FileSystem  C:\                              
Cert                         Certificate \                                
D             8,45     16,55 FileSystem  D:\                              
Env                          Environment                                  
Function                     Function                                     
HKCU                         Registry    HKEY_CURRENT_USER                
HKLM                         Registry    HKEY_LOCAL_MACHINE               
Variable                     Variable                                     
WSMan                        WSMan                                        
```

 PSDrives can all be targeted in PowerShell paths

### Behind The Scene: Provider Selector

Each PSDrive gets its information from a "PSProvider", and the name of the provider is listed in the "Provider" column.

The drive HKCU: for example comes from the provider "Registry". And when you add more modules, they may bring additional providers (and PSDrives) – for example the module ActiveDirectory, and the modules for SQL Server.

And now things become a little bit more mind-blowing: The current PowerShell path is actually a provider selector! It tells PowerShell which provider should be used by default if it cannot determine the appropriate provider elsehow.

Test yourself! What smells funny about this code?

```
PS> cd hkcu:
PS> dir HKEY_CURRENT_USER\Software\Microsoft

    Hive: HKEY_CURRENT_USER\Software\Microsoft

Name                           Property                                       
––––                           ––––––––                                       
Active Setup                                                                  
ActiveMovie                                                                   
Advanced INF Setup                                                            
ASF Stream Descriptor File                                                    
ASP.NET                                                                       
Assistance                                                                    
AuthCookies                                                                   
Calc                           Window_Placement : {44, 0, 0, 0…}            
CharMap                        Advanced : 1                                   
                               CodePage : Unicode                             
```

Taking advantage of default provider selection

Right: "dir" (aka Get-ChildItem) is dumping HKCU\Software\Microsoft from the Registry. It is NOT using a PowerShell drive. It is simply using a regular Registry path. So you could now open regedit.exe, right-click a key in HKEY_CURRENT_USER, copy its path name, paste it to PowerShell, and off you go.

Once you switch PowerShell default path, the line no longer works:

```
PS> cd c:\
PS> dir HKEY_CURRENT_USER\Software\Microsoft
dir : Cannot find path 'C:\HKEY_CURRENT_USER\Software\Microsoft' because it
does not exist.
At line:1 char:1
+ dir HKEY_CURRENT_USER\Software\Microsoft
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\HKEY_CURRENT_USER\Software\M
   icrosoft:String) [Get-ChildItem], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetCh
   ildItemCommand
```

 The default provider controls how paths are interpreted

So the bottom line is:

You either use a PSDrive letter to explicitly have a path tied to a specific provider (Get-PSDrive listed the providers that are tied to particular drive letters).

Or, when the path has no PSDrive letter, PowerShell takes the provider from the default PowerShell path.

If you do not care about this, one identical line of code can behave completely differently, depending on the current PowerShell default path.

Check this out:

```
PS> cd c:\
PS> Test-Path -Path \\storage1\serviceshare
True

PS> cd hklm:

PS> Test–Path –Path \\storage1\serviceshare
False
```

 Test-Path takes the provider from the default path when using UNC names

Since a UNC path name starts with "\" and not with a PSDrive, the path cannot be assigned to one specific provider.

Thus, PowerShell takes it from the default current path, and depending where you are, will interpret it completely different. In the first call, it is interpreted as a filesystem location (and since it is an existing UNC path, the result is TRUE).

The second call treats it like a Registry path, and it is not present, returning FALSE.

### Using Unambiguous Paths

One solution is to make paths unambiguous by adding the provider name that should be used. So to make the above example fool-proof, you could do this:

```
PS> cd c:\
PS> Test-Path -Path FileSystem::\\storage1\serviceshare
True

PS> cd HKCU:\

PS> Test-Path -Path FileSystem::\\storage1\serviceshare
True
```

 Adding provider name to a path makes it unambiguous

This is a very powerful technique because it lets you break out of the restrictions imposed by PSDrives, too.

For example, there are only two default drives representing the Registry hives HKLM and HKCU. What if you wanted to navigate a different hive?

Try this:

```
PS> dir Registry::HKEY_CURRENT_USER\Console
    Hive: HKEY_CURRENT_USER\Console

Name                           Property                                                             
––––                           ––––––––                                                             
%SystemRoot%_System32_WindowsP PopupColors      : 243                                               
owerShell_v1.0_powershell.exe  FontFamily       : 54                                                
                               QuickEdit        : 1                                                 
                               ColorTable05     : 5645313                                           
                               ScreenBufferSize : 196608120                                         
                               WindowSize       : 2359416                                           
                               ColorTable06     : 15789550                                          
                               FontWeight       : 400                                               
                               ScreenColors     : 86                                                
                               FaceName         : Consolas                                          
                               FontSize         : 2949120                                           
%SystemRoot%_SysWOW64_WindowsP PopupColors      : 243                                               
owerShell_v1.0_powershell.exe  FontFamily       : 54                                                
                               QuickEdit        : 1                                                 
                               ColorTable05     : 5645313                                           
                               ScreenBufferSize : 196608120                                         
                               WindowSize       : 3276920                                           
                               ColorTable06     : 15789550                                          
                               FontWeight       : 400                                               
                               ScreenColors     : 86                                                
                               FaceName         : Lucida Console                                    

PS> dir Registry::

    Hive:

Name                           Property                                                             
––––                           ––––––––                                                             
HKEY_LOCAL_MACHINE                                                                                  
HKEY_CURRENT_USER                                                                                   
HKEY_CLASSES_ROOT              (default) :                                                          
HKEY_CURRENT_CONFIG                                                                                 
HKEY_USERS                                                                                          
HKEY_PERFORMANCE_DATA          Global : {80, 0, 69, 0…}                                           
                               Costly : {80, 0, 69, 0…}   
```

 Accessing the registry directly using the provider name

By prefixing the path with the provider name and two colons, you tell PowerShell to interpret the path in the context of this provider – without having to use a PSDrive.

This allows you to use native Registry paths (no need to exchange the hive with a PSDrive anymore), and it allows you to navigate anywhere, even the very root of the Registry.

### Dynamic Parameters

And there is more: each cmdlet certainly has its own set of parameters. In addition, though, it can expose additional parameters when a specific provider is targeted.

Check this out:

```
PS> cd c:\
PS> Get-ChildItem –File –ReadOnly
```

 Dynamic parameters are exposed for specific providers only

If the current path is a filesystem drive (or if you have submitted such a path to the -Path parameter of Get-ChildItem), the cmdlet exposes additional parameters such as -File (list files only) or -ReadOnly (list readonly items only, both were introduced in PowerShell 3.0). IntelliSense and tabcompletion will expose these.

When you change default location (or provide a different path to -Path) to another provider, dynamic parameters may change or become unavailable.

So this would look for any codesigning certificates in your cert store that expire in 5 days:

```
PS> cd cert:\
PS> dir –CodeSigningCert –ExpiringInDays 5 –Recurse
```

 Dynamic parameters provided by certificate provider

(the dynamic parameter -ExpiringInDays (and a bunch more) are available only in Windows 8/Server 2012 and above – which comes with an improved certificate provider)

Likewise, to set a Registry value of a given type, you will only get tabcompletion and IntelliSense for the -Type dynamic parameter, when the path provided by you unambiguously targets the Registry provider.

This works (and creates a key HKCU:\Software\Test with a DWORD value named MyValue set to 2):

```
PS> $null = New-Item -Path HKCU:\Software\Test
PS> Set-ItemProperty -Path HKCU:\Software\Test -Name MyValue -Value 2 -Type DWord
```

 Creating Registry Key and Value

While you type this in, you will get tabcompletion and IntelliSense for all parameters, including the -Type dynamic parameter.

Now try and type this:

```
PS> Set-ItemProperty -Path 'HKCU:\Software\Test' -Name MyValue -Value 3 -type dword
```

 No intellisense for -Type?

This time, you do not get tabcompletion for -Type. Funnily, Set-ItemProperty is unable to recognize quoted paths apparently. This smells a bit like a bug.

### Provider Selection: An Important Choice

As you've seen, the current PowerShell path has important implications, and a number of modules make extensive use of this.

For example, ActiveDirectory takes the connection details to ActiveDirectory from the current AD drive you select. So to connect to different domains, DCs, or to use different credentials, you would have to add more PSDrives (New-PSDrive) where you specify the connection details.

Then, by changing the default PowerShell path (Set-Location), you can switch back and forth between different connections and ADs.

Similar logic is found with SCCM and other products.

So the current PowerShell location is by no means just a type safer to allow you to use relative paths. It is also a default provider selector with many implications.

Some of these you can circumvent by adding the provider name to your paths and making them unique and independent of the current PowerShell path.

So, that's it for now, have fun and a great weekend!

Tobias


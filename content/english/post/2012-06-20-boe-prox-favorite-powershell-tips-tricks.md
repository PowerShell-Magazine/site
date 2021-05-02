---
title: 'Boe Prox’ Favorite PowerShell Tips & Tricks'
author: Boe Prox
type: post
date: 2012-06-20T18:00:52+00:00
url: /2012/06/20/boe-prox-favorite-powershell-tips-tricks/
views:
  - 18089
post_views_count:
  - 2377
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I was asked to come up with a list of 3 PowerShell tips that I use. This was a tough task to come up with because there are just so many tips that I would like to share. So after a lot of thinking (and perhaps some use of Get-Random :)) I finally figured out what I would talk about. So with that and in no particular order, here are my 3 tips for PowerShell.

### SupportsShouldProcess to allow –WhatIf to your functions or script

Whenever I am writing a script or function that will perform some sort of action that will make a change, whether it is to write, delete create, update, etc…, I always make a case to put in a little bit of extra precaution to allow the use of –WhatIf whenever someone attempts to run the code and isn’t quite sure of what might happen.  What this means is that if I decide to run my code, I can see what it would do instead of just running it and assuming I know what it will do.

Take this for example:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Process | Stop-Process –WhatIf
</pre>

If the –WhatIf wasn’t specified in the Stop-Process, every process on the system would try to be stopped.

So how can you do this? First you start out by adding the following code to enable the use of –WhatIf.

<pre class="brush: powershell; title: ; notranslate" title="">[CmdletBinding(
	SupportsShouldProcess=$True
)]
</pre>

This doesn’t automatically allow you to use –WhatIf even if it is specified, we still need to add some extra code to handle how the script/function will react if –WhatIf is used. We need to also use

<pre class="brush: powershell; title: ; notranslate" title="">$PSCmdlet.ShouldProcess(Target, Action)
</pre>

This is a simple function to highlight how you would accomplish this:


    Function Invoke-EncryptFile {
        [CmdletBinding(
            SupportsShouldProcess=$True
        )]
    
        Param (
            [parameter(ValueFromPipeLine=$True,ValueFromPipeLineByPropertyName=$True)]
            [string]$File
        )
    
        Process {
            ForEach ($item in $file) {
                If ($PSCmdlet.ShouldProcess("$Item","Encrypt File")) {
                    [IO.File]::Encrypt($Item)
                }
            }
        }
    }
    Invoke-EncryptFile –File 'services.csv' –WhatIf
![](/images/BoeTips1.png)

As you can see, instead of actually performing the action, it tells you what it would do.  Another bonus feature of this is that it also gives you a helpful message if you run the command with the –Verbose switch.

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-EncryptFile 'services.csv' -Verbose
</pre>

![](/images/BoeTips2.png)

How about that? It is like two deals for the price of one!

#### **Splatting**

Splatting is useful as it helps simplify your code to make it look cleaner. Take this simple code snippet for example:

    Param(
        $ComputerName = $env:COMPUTERNAME,
        $Credential
    )
    
    Process {
        If ($Credential) {
            Try {
                Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Class Root\CIMV2 `
                -Filter "IPEnabled='True'" -ComputerName $ComputerName -Credential $Credential -ErrorAction Stop
            } Catch {
                Write-Warning ("{0}: {1}" -f $ComputerName,$_.Exception.Message)
            }
        } Else {
            Try {
                Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Class Root\CIMV2 `
                -Filter "IPEnabled='True'" -ComputerName $ComputerName -ErrorAction Stop
            } Catch {
                Write-Warning ("{0}: {1}" -f $ComputerName,$_.Exception.Message)
            }
        }
    }
Ok, this works, but really isn’t the easiest on the eyes to read. We are also specifying the same command more than once with only a minor change in what parameters are being used. Now let’s take a look at it using splatting.

```
Param(
    $ComputerName = $env:COMPUTERNAME,
    $Credential
)

Begin {
    $WMIParam = @{
        ComputerName = $ComputerName
        Class = 'Win32_NetworkAdapterConfiguration'
        Namespace = 'Root\CIMV2'
        Filter = "IPEnabled='True'"
        ErrorAction = 'Stop'
    }
    If ($PSBoundParameters['Credential']) {
        $WMIParam.Credential = $Credential
    }
}

Process {
    Try {
        Get-WmiObject @WMIParam
    } Catch {
        Write-Warning ("{0}: {1}" -f $ComputerName,$_.Exception.Message)
    }
}
```

With just a little use of splatting, we now off-loaded most of our parameters to a hash table at the beginning of the code in the Begin{} block and only run the command one time in the Process{} block!

#### **Validate an IP address with [ipaddress] type accelerator**

Sometimes I see folks who write a UI or some other script that requests an IP address which is then validated using regular expressions to determine if it can actually be used or not. While I love to work with regular expressions, something like this is probably a little more complicated than it needs to be especially if you are looking to validate both an IPV4 and an IPV6 address.

To get around this we can use the [ipaddress] type accelerator that allows us to validate that the IP address is legitimate.

<pre class="brush: powershell; title: ; notranslate" title="">[ipaddress]"192.168.1.1"
</pre>

![](/images/BoeTips3.png)

<pre class="brush: powershell; title: ; notranslate" title="">[ipaddress]"1::1f"
</pre>
![](/images/BoeTips4.png)

Notice that the AddressFamily property differs between an IPV4 and an IPV6 address.  So, what happens if we supply an invalid address?

<pre class="brush: powershell; title: ; notranslate" title="">[ipaddress]"195.258.1.1"
</pre>

![](/images/BoeTips5.png)

Perfect! The IP address is shown to be invalid.

Lucky for us, this is Try/Catch friendly so I was able to put together a simple function that will return a $True/$False Boolean value that can be handy to use with a number of situations.


    Function Test-IsValidIP {
        [CmdletBinding()]
    
        Param (
            [parameter(ValueFromPipeLine=$True,ValueFromPipeLineByPropertyName=$True)]
            [Alias("IP")]
            [string]$IPAddress
        )
    
        Process {
            Try {
                [ipaddress]$IPAddress | Out-Null
                Write-Output $True
            } Catch {
                Write-Output $False
            }
        }
    }
    
    Test-IsValidIP "192.1.5.16"
    True
Works great on an IPV4 address and returns a Boolean value of $True.

```
Test-IsValidIP "FE80:0000:0000:0000:0202:B3FF:FE1E:8329"
True
```

Also works great on an IPV6 address.

```
Test-IsValidIP "185.459.2.5"
False
```

As expected, this failed the validation and returned a $False value.
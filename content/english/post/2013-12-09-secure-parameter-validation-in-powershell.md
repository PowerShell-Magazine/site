---
title: Secure Parameter Validation in PowerShell
author: Matt Graeber
type: post
date: 2013-12-09T17:00:54+00:00
url: /2013/12/09/secure-parameter-validation-in-powershell/
categories:
  - How To
tags:
  - How To

---
Parameter validation in PowerShell is an extremely useful and easy way to alert a user as early as possible that incorrect input was provided to a function or cmdlet. The following attributes are available to you for performing explicit parameter validation (taken from [MSDN][1]):

### ValidateCount

Specifies the minimum and maximum number of arguments that a parameter can accept. For more information about the syntax used to declare this attribute, see [ValidateCount Attribute Declaration][2].

### ValidateLength

Specifies the minimum and maximum number of characters in the parameter argument. For more information about the syntax used to declare this attribute, see [ValidateLength Attribute Declaration][3].

### ValidatePattern

Specifies a regular expression that validates the parameter argument. For more information about the syntax used to declare this attribute, see [ValidatePattern Attribute Declaration][4].

### ValidateRange

Specifies the minimum and maximum values of the parameter argument. For more information about the syntax used to declare this attribute, see [ValidateRange Attribute Declaration][5].

### ValidateSet

Specifies the valid values for the parameter argument. For more information about the syntax used to declare this attribute, see [ValidateSet Attribute Declaration][6].

When validating parameters, it is important to make sure your validation attributes make sense though. For example, consider the following simple function that returns the status of a TCP connection:


    function Test-TcpConnection
    {
        Param (
            [String]$IPAddress,
        	[ValidateRange(1, 2147483647)]
        	[Int32]$Port
    	)
    
    	$TcpClient = New-Object System.Net.Sockets.TCPClient
    
        try
        {
            $TcpClient.Connect($IPAddress, $Port)
        }
        catch [System.Net.Sockets.SocketException]
        { }
        finally
        {
            $TcpClient.Connected
            $TcpClient.Close()
        }
    }
Those comfortable with networking should recognize that the ValidateRange attribute doesn’t make sense since a port number is an unsigned, 16-bit value – i.e. 0-65535. You would be surprised to see how many built-in PowerShell cmdlets use nonsensical _ValidateRange_ arguments. Naturally, it would make more sense to provide the following _ValidateRange_ attribute:

<pre class="brush: powershell; title: ; notranslate" title="">[ValidateRange(1, 65535)]
</pre>

Now, the validation attributes are nice and they certainly have their place but often times they are not necessary. By choosing proper data types as parameters to your functions, you will often get parameter validation for free (i.e. implicit validation). Consider the following improvement to the _Test-TcpConnection_ function:


    function Test-TcpConnection
    {
        Param (
            [System.Net.IPAddress]$IPAddress,
        	[ValidateRange(1, [UInt16]::MaxValue)]
        	[UInt16]$Port
    	)
    
        $TcpClient = New-Object System.Net.Sockets.TCPClient
    
        try
        {
            $TcpClient.Connect($IPAddress, $Port)
        }
        catch [System.Net.Sockets.SocketException]
        { }
        finally
        {
            $TcpClient.Connected
            $TcpClient.Close()
        }
    }
By making the IPAddress parameter an IPAddress object, it will automatically parse the IP address provided. No complicated regular expressions required! For example, it will throw an error upon attempting to execute the following command:

<pre class="brush: powershell; title: ; notranslate" title="">Test-TcpConnection 0.0.0.300 80
</pre>

It is also worth noting that the _ValidateRange_ attribute is still necessary for the Port parameter because the _“connect”_ method will not accept a port number of 0.

Starting in PowerShell v3, you also get automatic tab completion if you use the _ValidateSet_ attribute or use an enum type as one of your parameters. Consider the following contrived example:


    function Get-Poem
    {
        Param (
            [ConsoleColor]
            $Color1,    
            [ConsoleColor]
            $Color2
    	)
    
    	Write-Host "Roses are $Color1, violets are $Color2, PowerShell is great and it is Blue."
    }
Using the _ConsoleColor_ enum as parameters provides two benefits:

  1. Only colors defined in the _ConsoleColor_ enum will be accepted (in theory… more on this in a moment).
  2. You will get automatic tab completion when typing in the colors in PS v3 and v4- e.g. Get-Poem Re[TAB] Blu[TAB]

Now, there is a subtle characteristic of enums in .NET that can lead to undefined behavior: you can call the _Parse_ method on an enum and have it return to you an undefined enum value. Consider the following example:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Poem ([Enum]::Parse([ConsoleColor], 200)) ([Enum]::Parse([ConsoleColor], 42))
</pre>

Roses are 200, violets are 42, PowerShell is great and it is Blue.

Obviously, I never intended for this behavior to occur. In fact, most people would think that they would be explicitly preventing this sort of thing from happening! If you were truly concerned about ensuring proper parameter validation, you would need to provide the following validation attribute for each parameter:

<pre class="brush: powershell; title: ; notranslate" title="">[ValidateScript({ [Enum]::IsDefined($_.GetType(), $_) })]
</pre>

This ensures that the enum value you provided is an actual, defined value.

Now, to put things into perspective, I’ll provide an example of a built-in cmdlet introduced in PowerShell v3 that exhibits a lack of parameter validation that ultimately leads to undefined behavior – Resolve-DnsName.

_Resolve-DnsName_ allows you to query individual DNS records of various types. You specify the record type through the _“Type”_ parameter which accepts an enum of type _[Microsoft.DnsClient.Commands.RecordType]_. To view all of the defined record types, run the following:

<pre class="brush: powershell; title: ; notranslate" title="">[Microsoft.DnsClient.Commands.RecordType] | Get-Member -Static -MemberType Property
</pre>

When I saw the list of record types, I noticed that a specific type was missing – AXFR (zone transfer). Penetration testers will often attempt to perform a zone transfer on a potentially misconfigured DNS server in order to get a listing of every DNS record in the zone specified. A successful unauthorized zone transfer is a potential treasure trove of actionable intelligence for an attacker. Naturally, I would have liked it if _Resolve-DnsName_ would let you perform zone transfers (like nslookup.exe does) but unfortunately, there is no _“AXFR”_ defined in the _RecordType_ enum. Fortunately, we can use our new trick to force AXFR (0xFC) to be used:

<pre class="brush: powershell; title: ; notranslate" title="">Resolve-DnsName -Name zonetransfer.me -Type ([Enum]::Parse([Microsoft.DnsClient.Commands.RecordType], 0xfc)) -Server '209.62.64.46' -TcpOnly
</pre>

If you ran this command, you would find that it only returns a single SOA entry. If you ran the command with Wireshark running, however, you would see that a zone transfer was actually performed. Unfortunately, Resolve-DnsName wasn’t designed to parse multiple records from a zone transfer but this is undefined behavior, nonetheless.

So, when choosing parameters you should follow this advice:

  1. Choose your parameter types wisely. Often choosing an appropriate type will provide implicit validation.
  2. Use parameter validation attributes as often as possible. Just make sure they make sense in the first place though.
  3. Beware of people like me that will try to exploit undefined behavior in your functions/cmdlets. 🙂

[1]: http://msdn.microsoft.com/en-us/library/ms714432(v=vs.85).aspx
[2]: http://msdn.microsoft.com/en-us/library/ms714435(v=vs.85).aspx
[3]: http://msdn.microsoft.com/en-us/library/ms714452(v=vs.85).aspx
[4]: http://msdn.microsoft.com/en-us/library/ms714454(v=vs.85).aspx
[5]: http://msdn.microsoft.com/en-us/library/ms714421(v=vs.85).aspx
[6]: http://msdn.microsoft.com/en-us/library/ms714434(v=vs.85).aspx


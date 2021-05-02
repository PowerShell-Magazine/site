---
title: Automatic Remote Desktop Connection
author: Tobias Weltner
type: post
date: 2014-04-18T16:00:23+00:00
url: /2014/04/18/automatic-remote-desktop-connection/
categories:
  - How To
tags:
  - How To

---
**Note**: This tip requires PowerShell 2.0 or above.

As a PowerShell user, you probably have a PowerShell console or the ISE editor on standby. Wouldn&#8217;t it be nice to be able to just hack in a “Connect-RDP” and immediately be connected to a remote desktop when you need it? And let PowerShell deal with login credentials? Here’s how.

### Securely Caching Credentials

To securely cache login credentials, you can use the command line utility cmdkey.exe. With this utility, you can save a username and a password for a given remote connection. Windows will then securely cache the information and automatically use it when needed.

### Connect-RDP – Auto-Login for RDP Sessions

Here is a function called Connect-RDP that automates the RDP connection:

```
function Connect-RDP {
    param (
        [Parameter(Mandatory=$true)]
        $ComputerName,

        [System.Management.Automation.Credential()]
        $Credential
    )

    # take each computername and process it individually
    $ComputerName | ForEach-Object {
        # if the user has submitted a credential, store it
        # safely using cmdkey.exe for the given connection

        if ($PSBoundParameters.ContainsKey('Credential'))
        {
            # extract username and password from credential

            $User = $Credential.UserName
            $Password = $Credential.GetNetworkCredential().Password

            # save information using cmdkey.exe
            cmdkey.exe /generic:$_ /user:$User /pass:$Password
        }

        # initiate the RDP connection
        # connection will automatically use cached credentials
        # if there are no cached credentials, you will have to log on
        # manually, so on first use, make sure you use -Credential to submit

        # logon credential
        mstsc.exe /v $_ /f
    }
}
```

### Set or Update Cached Credentials

To cache credentials for a new remote desktop connection, this is how you’d call the function:

```
PS> Connect-RDP 10.20.30.40 -Credential testdomain\Administrator
```


You would then be prompted for the connection password, and the RDP connection gets initiated. Internally, Connect-RDP stores the logon information in your credential cache. So from now on, to connect to the server via RDP, you no longer need the credentials. Next time, this is all you need:

```
Connect-RDP 10.20.30.40
```


### Using Multiple Connections

The function also supports multiple connections. If all of the connections require the same logon information, you can set it in one step:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Connect-RDP 10.20.30.40, 10.20.30.41, 10.20.30.42 -Credential testdomain\Administrator
</pre>

If the connections require different logon credentials, then set the credentials individually:

```
PS> Connect-RDP 10.20.30.40 -Credential testdomain\Administrator
PS> Connect-RDP 10.20.30.41 -Credential testdomain\Testaccount12
PS> Connect-RDP 10.20.30.42 -Credential testdomain\Tobias
```


Once you have set cached credentials for all your RDP servers, you can connect to one or many with just one call:

```
PS> Connect-RDP 10.20.30.40, 10.20.30.41, 10.20.30.42
```


PowerShell will use the appropriate cached credentials for each of these connections, and opens an RDP session for each server.

### Manage Cached Credentials

To manage your cached credentials, use cmdkey.exe:

```
PS> cmdkey
Creates, displays, and deletes stored user names and passwords. The syntax of this command is:
CMDKEY [{/add | /generic}:targetname {/smartcard | /user:username {/pass{:password}}} | /delete{:targetname | /ras} | /list{:targetname}]

Examples:
  To list available credentials:
     cmdkey /list
     cmdkey /list:targetname

  To create domain credentials:
     cmdkey /add:targetname /user:username /pass:password
     cmdkey /add:targetname /user:username /pass
     cmdkey /add:targetname /user:username
     cmdkey /add:targetname /smartcard

  To create generic credentials:
     The /add switch may be replaced by /generic to create generic credentials

  To delete existing credentials:
     cmdkey /delete:targetname

  To delete RAS credentials:
     cmdkey /delete /ras

PS> cmdkey /list:10.16.114.11

Currently stored credentials for 10.16.114.11:
  Target: 10.16.114.11
  Type: Generic
  User: citrixdev\Administrator
```


---
title: 5 Essential PowerShell Skills for Exchange Server Administrators
author: Paul Cunningham
type: post
date: 2011-12-20T19:00:23+00:00
url: /2011/12/20/5-essential-powershell-skills-for-exchange-server-administrators/
views:
  - 155457
post_views_count:
  - 6394
categories:
  - Exchange
tags:
  - Exchange
---
Ask an experienced Exchange administrator and they’ll tell you that learning PowerShell is important to be able to do the job well. Even though most administrative tasks can be performed using the Exchange Management Console it tends to be much slower to navigate, and is less efficient for doing bulk administration. If you’re still holding yourself back from learning PowerShell, then here are five tips to help get you started.

### Discovering Commands

One of the first challenges with using PowerShell for Exchange Server management is knowing which commands to run for the task you want to perform. Fortunately you only need to remember one: [Get-Command][1].

Using Get-Command you can quickly search for cmdlets based on keywords. Let’s say for example you want to administer a mailbox. Using Get-Command you can discover all of the cmdlets that have the noun “mailbox” in them.

```powershell
PS> Get-Command -Noun Mailbox
CommandType     Name
-----------     ----
Cmdlet          Connect-Mailbox
Cmdlet          Disable-Mailbox
Cmdlet          Enable-Mailbox
Cmdlet          Get-Mailbox
Cmdlet          New-Mailbox
Cmdlet          Remove-Mailbox
Cmdlet          Restore-Mailbox
Cmdlet          Search-Mailbox
Cmdlet          Set-Mailbox
```

With PowerShell’s simple “Verb-Noun” cmdlet names you can now quickly see which cmdlet to use if you want to get a list of mailboxes, set a mailbox property, or create a new mailbox.

### Filtering Output

Some PowerShell cmdlets return a lot of data. For example if you run [Get-Mailbox][2] it will return every mailbox in your organization. That may be hundreds or thousands of mailboxes, when all you really wanted is the mailboxes that match a certain criteria. PowerShell lets you filter the data that a cmdlet returns by using the [Where-Object][3] command, or simply “where” as it is often abbreviated. For example, to get the list of mailbox users in the branch office of the organization you can run the following command:

```powershell
PS> Get-Mailbox | Where-Object { $_.Office -eq "Branch Office"}
Name             Alias                ServerName       ProhibitSendQuota
----             -----                ----------       -----------------
Alex.Heyne       Alex.Heyne           br-ex2010-mb     unlimited
John.Williams    John.Williams        br-ex2010-mb     unlimited
Judith.Rodrigues Judith.Rodrigues     br-ex2010-mb     unlimited
Katherine.Phipps Katherine.Phipps     br-ex2010-mb     unlimited
Olive.Weeks      Olive.Weeks          br-ex2010-mb     unlimited
Sonia.Smith      Sonia.Smith          br-ex2010-mb     unlimited
Wendy.Fyson      Wendy.Fyson          br-ex2010-mb     unlimited
```

As you can see by filtering the output of Get-Mailbox to only those where the “Office” attribute equals “Branch Office”, you can retrieve only the data that you are interested in seeing.

### Using the Pipeline

One of the PowerShell techniques you will use regularly is the pipeline. In PowerShell the pipeline is how output from one command is passed to another. You may have noticed in the previous example the | (or pipe) character that was between the Get-Mailbox and Where-Object cmdlets. That is an example of the pipeline in action. Continuing the previous example, let’s say that the business has decided that all branch office mailbox users should have a 3Gb mailbox storage quota. You can use the same command shown above to get the branch office mailboxes, and then pipe that into the [Set-Mailbox][4] cmdlet to make the change to their storage quota settings.

```powershell
Get-Mailbox | Where-Object { $_.Office -eq "Branch Office"} | Set-Mailbox -IssueWarningQuota 2.8Gb -ProhibitSendQuota 3Gb
```


Now when we look at the list of branch office mailboxes again we can see the change to the storage quota settings.

```powershell
Name             Alias              ServerName     ProhibitSendQuota
----             -----              ----------     -----------------
Alex.Heyne       Alex.Heyne         br-ex2010-mb   3 GB (3,221,225,472 bytes)
John.Williams    John.Williams      br-ex2010-mb   3 GB (3,221,225,472 bytes)
Judith.Rodrigues Judith.Rodrigues   br-ex2010-mb   3 GB (3,221,225,472 bytes)
Katherine.Phipps Katherine.Phipps   br-ex2010-mb   3 GB (3,221,225,472 bytes)
Olive.Weeks      Olive.Weeks        br-ex2010-mb   3 GB (3,221,225,472 bytes)
Sonia.Smith      Sonia.Smith        br-ex2010-mb   3 GB (3,221,225,472 bytes)
Wendy.Fyson      Wendy.Fyson        br-ex2010-mb   3 GB (3,221,225,472 bytes)
```

### Discovering Object Properties

PowerShell is an object-oriented language. If you don’t know what that means then don’t worry, I was using PowerShell for almost two years before I began to understand what it meant.

An “object” is simply a collection of data. In Exchange Server 2010 an object could be a mailbox, a distribution group, a database, or one of many other things.

An object has properties. In the example of a mailbox object these include properties such as the name of the database the mailbox resides on, the mailbox storage quota values, or the office location for the mailbox user. In the previous examples we looked at filtering mailboxes using the “Office” attribute, and setting the values for mailbox storage quota attributes. But how do we know which properties a mailbox object has, so that we can perform that filtering or apply changes to those settings?

Again PowerShell gives us the answer thanks to the [Get-Member][5] cmdlet. Get-Member will list all of the properties of an object to help you learn what you can and can’t do with them. For example, to discover which properties the mailbox object has you can run the following command (note once again the use of the pipeline):

```powershell
PS> Get-Mailbox | Get-Member
TypeName: Microsoft.Exchange.Data.Directory.Management.Mailbox

Name                                   MemberType

----                                   ----------

Clone                                  Method
Equals                                 Method
GetHashCode                            Method
GetProperties                          Method
GetType                                Method
ToString                               Method
Validate                               Method
AcceptMessagesOnlyFrom                 Property
AcceptMessagesOnlyFromDLMembers        Property
AcceptMessagesOnlyFromSendersOrMembers Property
AddressBookPolicy                      Property
AddressListMembership                  Property
Alias                                  Property
AntispamBypassEnabled                  Property
ArbitrationMailbox                     Property
ArchiveDatabase                        Property
ArchiveDomain                          Property
ArchiveGuid                            Property
ArchiveName                            Property
ArchiveQuota                           Property
ArchiveStatus                          Property
ArchiveWarningQuota                    Property
AuditAdmin                             Property
AuditDelegate                          Property
AuditEnabled                           Property
AuditLogAgeLimit                       Property
AuditOwner                             Property
```

I’ve had to truncate the output in this example because it returns quite a long list of properties that mailbox objects have. It would be more convenient to filter the list down to just the properties I might be interested in. So let’s say that we just want to see which mailbox object properties relate to quota settings. Once again we can use the pipeline and filtering to achieve that.

```powershell
PS> Get-Mailbox | Get-Member -Name "*quota*"
 TypeName: Microsoft.Exchange.Data.Directory.Management.Mailbox
Name                         MemberType
----                         ----------
ArchiveQuota                 Property
ArchiveWarningQuota          Property
IssueWarningQuota            Property
ProhibitSendQuota            Property
ProhibitSendReceiveQuota     Property
RecoverableItemsQuota        Property
RecoverableItemsWarningQuota Property
RulesQuota                   Property
UseDatabaseQuotaDefaults     Property
```

Pretty neat isn’t it.

### Remoting

The last essential skill that I’ll demonstrate to you in this article is remoting, which is a feature available in PowerShell 2.0 and above. Exchange Server 2010 servers require at least PowerShell 2.0, so this technique will work for Exchange 2010 environments. Remoting is simply the technique of connecting a PowerShell session on the local computer to a remote server. This is useful if you are logged on to a workstation that has PowerShell installed, but doesn’t have the Exchange Server 2010 management tools installed, and you need to run some Exchange management tasks. By remoting from the workstation to an Exchange 2010 server you quickly get access to all of the Exchange cmdlets.

To demonstrate, here is a PowerShell window on a workstation that does not have the Exchange 2010 management tools installed. Notice that the [Get-ExchangeServer][6] cmdlet returns an error because it doesn&#8217;t exist on that workstation.

```powershell
PS> Get-ExchangeServer
The term 'Get-ExchangeServer' is not recognized as the name of a cmdlet, functi
on, script file, or operable program. Check the spelling of the name, or if a p
ath was included, verify that the path is correct and try again.
At line:1 char:19
+ Get-ExchangeServer >>>>
    + CategoryInfo          : ObjectNotFound: (Get-ExchangeServer:String) [],
   CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

Now let's remote to an Exchange 2010 server and try running the cmdlet again.  There are two steps for PowerShell remoting. The first is to create a new PowerShell session object using [New-PSSession][7].

```powershell
$session = New-PSSession -ConfigurationName Microsoft.Exchange `
                         -ConnectionUri http://ho-ex2010-mb1.exchangeserverpro.net/powershell ` 						 -Authentication Kerberos
```

Notice the URI that the session is connecting to? That is the /powershell virtual directory in IIS on the Exchange 2010 server named “ho-ex2010-mb1”. After you have created the new session object you can import it using [Import-PSSession][8].

```
PS> Import-PSSession $session
ModuleType Name                      ExportedCommands
---------- ----                      ----------------
Script     tmp_c536eb19-09db-41c1... Get-ExchangeDiagnosticInfo
```

Now let’s try the Get-ExchangeServer cmdlet again.

```powershell
PS> Get-ExchangeServer
Name                Site                 ServerRole  Edition     AdminDisplayVe
                                                                 rsion
----                ----                 ----------  -------     --------------
BR-EX2010-MB        exchangeserverpro... Mailbox,... Enterprise  Version 14....
HO-EX2010-CAHT1     exchangeserverpro... ClientAc... Enterprise  Version 14....
HO-EX2010-MB1       exchangeserverpro... Mailbox,... Enterprise  Version 14....
HO-EX2010-MB2       exchangeserverpro... Mailbox,... Enterprise  Version 14....
HO-EX2010-EDGE      exchangeserverpro... Edge        Standard... Version 14....
```

This time it works, thanks to PowerShell remoting.

As you can see from this article PowerShell is not a daunting language to learn and to begin using on a daily basis once you understand a few of the fundamental skills involved. Try some of these yourself and let us know in the comments below if you have any questions.

[1]: http://technet.microsoft.com/en-us/library/ee176842.aspx
[2]: http://technet.microsoft.com/en-us/library/bb123685.aspx
[3]: http://technet.microsoft.com/en-us/library/ee177028.aspx
[4]: http://technet.microsoft.com/en-us/library/bb123981.aspx
[5]: http://technet.microsoft.com/en-us/library/ee176854.aspx
[6]: http://technet.microsoft.com/en-us/library/bb123873.aspx
[7]: http://technet.microsoft.com/en-us/library/dd347668.aspx
[8]: http://technet.microsoft.com/en-us/library/dd347575.aspx
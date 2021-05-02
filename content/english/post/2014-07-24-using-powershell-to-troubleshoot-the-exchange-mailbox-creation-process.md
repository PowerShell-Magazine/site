---
title: Using PowerShell to troubleshoot the Exchange mailbox creation process
author: Emin Atac
type: post
date: 2014-07-24T16:00:55+00:00
url: /2014/07/24/using-powershell-to-troubleshoot-the-exchange-mailbox-creation-process/
categories:
  - Exchange
  - How To
tags:
  - Exchange
  - How To

---
I’ve been recently tasked by the team responsible for user account management to investigate why a user mailbox wasn’t created along with its Active Directory account. PowerShell being my favorite tool for troubleshooting, I immediately loaded the Exchange cmdlets into my session.

```
$session = New-PSSession -ConnectionUri http://node01.my.fqdn/PowerShell/ -ConfigurationName Microsoft.Exchange
Import-PSSession -Session $session
```


![](/images/19.png)

My next step was to reproduce manually what the automated script was supposed to achieve.

Naively, I first tried to check if the mailbox associated to the Active Directory account exists.

```
$User = "JDoe"
Get-Mailbox $User
```


The operation couldn't be performed because object JDoe' couldn't be found on 'PDC.my.fqdn'.

This confirms that the mailbox doesn’t exist. It was expected, but doesn’t unfortunately tell me why.

Before creating a mailbox, the user account creation process concatenates the first and last name and fills in the mail attribute of its Active Directory account. The script retrieves it like this

```
$PDC  = (Get-ADDomainController -Service 1 -Discover).Hostname
$HT = @{DomainController = "$PDC"}
$UserMail = (Get-ADUser -Identity $User -Server "$PDC" -Properties *).Mail
```


When I try to use the mail attribute as the external email address, the cmdlet ends with the following error:

```
Set-MailUser -Identity "NetBiosDomainName\$User" -ExternalEmailAddress $UserMail @HT
The proxy address "SMTP:john.doe@my.fqdn" is already being used by "my.fqdn/Data/Contacts/Doe John". Please choose another proxy address.
```


Nice error. The Exchange cmdlet immediately tells me who’s the culprit using this SMTP address. It also tells me exactly why it refuses to use this email address. A contact object in Active Directory already uses this email address that is defined as one of its proxy addresses.

Let’s find this contact. I just need to translate the above path into its LDAP distinguished name form:

```
($cuser = Get-ADObject -SearchBase "OU=Contacts,OU=Data,DC=my,DC=fqdn" -SearchScope  Subtree -Filter "Name -LIKE '*Doe*'" -Properties proxyAddresses)
```


The proxyAddresses is actually a [multivalued attribute][1] and because only one contact is returned,

```
$cuser | Get-Member -Name proxyAddresses | Format-List -Property *
```


![](/images/25.png)

I can remove  the offending email address from the collection of proxyAddresses by doing:

```
Set-ADObject -Identity $cuser.ObjectGUID -Remove @{proxyAddresses='smtp:john.doe@my.fqdn'}
```


And then I can manually finish the mailbox creation process:

```
Set-MailUser -Identity "NetBiosDomainName\$User" -ExternalEmailAddress $UserMail @HT
Enable-Mailbox -Identity "NetBiosDomainName\$User" -Database "My DB Name" @HT
```


The Exchange cmdlets being no longer required, I just close the remote session:

<pre class="brush: powershell; title: ; notranslate" title="">Remove-PSSession -Session $session
</pre>

I started to write a report to the team responsible for user account management telling them why this particular user account failed. I wanted also to alert them that this could happen again as there are other contacts that have proxy addresses attribute matching our domain. I got the exact count like this:

<pre class="brush: powershell; title: ; notranslate" title="">Get-ADObject -SearchBase "OU=Contacts,OU=Data,DC=my,DC=fqdn" -SearchScope  Subtree -Filter "Name -LIKE '*'" -ResultSetSize 10000 -Properties proxyAddresses |
Where proxyAddresses -match "\@my\.fqdn$" |
Measure-Object
</pre>

Even though I’m not an Exchange admin, this story shows that PowerShell is really very useful for troubleshooting&#8211;it helped me quickly analyze what’s going and fix the problem. Nice tool, I love it.

A VBScript or a GUI approach wouldn’t save my time because if I’m tasked to fix all the other contacts later on, they wouldn’t help me quickly automate the resolution process. Only PowerShell does!

```
(Get-ADObject -SearchBase "OU=Contacts,OU=Data,DC=my,DC=fqdn" -SearchScope  Subtree -Filter "Name -LIKE '*'" -ResultSetSize 10000 -Properties proxyAddresses) |
Where proxyAddresses -match "@my\.fqdn$" | ForEach-Object {
     $paddr = $first = $last = $null
     $Name = $_.Name
     $last,$first = $_.Name -split "\s"
     if ($first.Count -eq 1) {
         $paddr = "smtp:{0}.{1}@my.fqdn" -f $first,$last
         try {
              if ($_.proxyAddresses -match $paddr) {
                   Set-ADObject -Identity $_.ObjectGUID -Remove @{proxyAddresses=$paddr} -ErrorAction Stop -Verbose
              } else {
                   Write-Warning -Message "Proxy address $paddr not found for user $Name"
              }
         } catch {
              Write-Warning -Message "Failed for $($Name) because $($_.Exception.Message)"
         }
     } else {
              Write-Warning -Message "Fix $Name manually: more than 1 first or last name"
     }
}
```


PowerShell is a huge time-saver and designed for automation. This is why I really love it.

[1]: http://technet.microsoft.com/library/ee156515.aspx
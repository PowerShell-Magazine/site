---
title: Managing Active Directory using a jumphost and remoting
author: Claus Nielsen
type: post
date: 2012-09-25T13:32:57+00:00
url: /2012/09/25/managing-active-directory-using-a-jumphost-and-remoting/
views:
  - 10309
post_views_count:
  - 2057
categories:
  - Active Directory
tags:
  - Active Directory

---
I’ve stumbled upon a question someone posted in the Minasi forum, about having to create a script for some managers to connect to AD and make some changes on their subordinates.  Company policy did not allow these managers to connect directly to the Domain controllers, so they had to go through a jumphost.

The admin writing the script had installed both the Microsoft AD cmdlets and the Quest ActiveRoles management cmdlets on the jumphost. Everything worked fine when the script was run on the jumphost directly, but when running the script from a client machine using remoting both the Microsoft AD cmdlets and the Quest cmdlets failed with an error.

My initial thought was that this guy must be doing something wrong. How hard can it be to execute Get-ADUser or Get-QADUser to return some information about an AD user and then change a few settings.

So I set out to try it out. I already had a jumphost that my colleagues and our tech support guys are using to run different scripts/tools. The main difference is that my jumphost is also running Remote Desktop services, so people tend to RDP in and execute the scripts on the server, which works perfectly.

Therefore, I’ve tried to list all users using Get-QADUser and Invoke-Command against my jumphost server:

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-Command -ComputerName RemoteMgmt -ScriptBlock {Add-PSSnapin *Quest*; Get-QADUser}
</pre>

![](/images/image002.jpg)

The error message hasn’t said much. My initial though was that there was some issue loading the Quest snap-in in a remoting session.

To test that theory I wanted to try to load the Microsoft AD cmdlets module instead, to see if that would work:

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-Command -ComputerName RemoteMgmt -ScriptBlock {Import-Module *Active*; Get-ADUser -filter *}
</pre>

I’ve got an error message again.

![](/images/image003.png)

(In the above examples I use Import-Module \*active\* and Add-PSSnapin \*quest\* for brevity, and I know there are no modules/snap-ins with similar names on my test systems. The full command in the above examples would be:

<pre class="brush: powershell; title: ; notranslate" title="">Add-PSSnapin -Name Quest.ActiveRoles.ADManagement and Import-Module -Name ActiveDirectory
</pre>

This error message tells me that it is unable to connect to the Active Directory Web Services, and that the server is down or non-existent. I was sure I would have heard if all of our 2008R2 domain controllers suddenly had ceased to exist.

Just to make sure, I connected to the server using RDP and tested both the Quest and Microsoft AD cmdlets directly from the server, and both worked flawlessly. But this also hinted to me that the problem might be the classic double-hop authentication issue.

So,in order to test that out, I decided to try and use CredSSP to connect to the jumphost. Since I did not have CredSSP enabled on my machines per default I had to set it up. On the jumphost I invoked:

<pre class="brush: powershell; title: ; notranslate" title="">Enable-WSManCredSSP -Role Server
</pre>

![](/images/image004.png)

(Remember there is always a security risk when you enable delegation of user credentials.)

On the client I invoked (RemoteMgmt is the name of the jumphost.):

<pre class="brush: powershell; title: ; notranslate" title="">Enable-WSManCredSSP -Role Client -DelegateComputer RemoteMgmt
</pre>
![](/images/image007.jpg)

Now let’s try our command again, but this time using CredSSP:

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-Command -ComputerName RemoteMgmt -ScriptBlock {Add-PSSnapin *quest*;Get-QADUser} -Authentication CredSSP -Credential (Get-Credential)
</pre>

This time the command executes without any problems. When you use CredSSP authentication, you have to explicitly specify credentials.

Another thing to be aware of when enabling CredSSP is that it is possible to use wildcards. Let’s say you have a “client” machine that has to connect to multiple machines in your domain. You can specify a wildcard like so:

<pre class="brush: powershell; title: ; notranslate" title="">Enable-WSManCredSSP -Role Client -DelegateComputer *.bigbusinness.com
</pre>

The client will allow credential delegation to happen when connecting to all machines in the *.bigbusiness.com domain.

Let’s say I tried to run the above example (only using the hostname) after having enabled CredSSP using a wildcard:

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-Command -ComputerName RemoteMgmt -ScriptBlock {Add-PSSnapin *quest*;Get-QADUser} -Authentication Credssp -Credential (Get-Credential)
</pre>

![](/images/image009.jpg)

It fails with an error message that might lead you on a wild goose chase looking through policy settings. The solution is to use the FQDN instead of just a hostname:

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-Command -ComputerName RemoteMgmt.bigbusiness.com -ScriptBlock {Add-PSSnapin *quest*;Get-QADUser} -Authentication Credssp -Credential (Get-Credential)
</pre>

Another way to make it work is to enable Kerberos delegation for the jumphost.This can be done by going to Active Directory Users and Computers, find the jumphost machine, open properties and select “Delegation”

![](/images/image010.png)

(Again here you can choose to lock down delegation even further, but that is not the purpose of this article.)

Then you can run the command like this:

<pre class="brush: powershell; title: ; notranslate" title="">Invoke-Command -ComputerName RemoteMgmt -ScriptBlock {Add-PSSnapin *quest*;Get-QADUser} -Authentication Kerberos
</pre>

Notice that with Kerberos authentication it is not required to specify a credential object.
---
title: '#PSTip Hyper-V VM Connection Authorization'
author: Shay Levy
type: post
date: 2014-02-27T19:00:37+00:00
url: /2014/02/27/pstip-hyper-v-vm-connection-authorization/
categories:
  - Hyper-V
  - Tips and Tricks
tags:
  - Hyper-V
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Virtual Machine Connection Authorization allows users to connect to virtual machines using the VMConnect interface in Hyper-V (and Hyper-V RSAT tools). Virtual Machine Connection authorization is configured by a Hyper-V Administrator using the _Grant-VMConnectAccess_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">Grant-VMConnectAccess -VMName TestVM -UserName domain\user1
</pre>

VMConnect Authorization uses an Access Control List (ACL) that is placed inside the virtual machine configuration file. Access can also be revoked using _Revoke-VMConnectAccess_.

<pre class="brush: powershell; title: ; notranslate" title="">Revoke-VMConnectAccess -VMName TestVM -UserName domain\user1
</pre>

With _Get-VMConnectAccess_ you can determine which users have access to which virtual machines.

<pre></pre>
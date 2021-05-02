---
title: '#PSTip Disabling multiple AD user accounts'
author: Shay Levy
type: post
date: 2013-07-02T18:00:05+00:00
url: /2013/07/02/pstip-disabling-multiple-ad-user-accounts/
post_views_count:
  - 3484
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

The _Disable-ADAccount_ cmdlet disables an Active Directory user, computer, or service account. When you need to disable multiple accounts you might find yourself trying something obvious like:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Disable-ADAccount -Identity user1,user2,user3
</pre>

But that doesn&#8217;t work and yields an error. The Identity parameter doesn&#8217;t accept multiple values.  The typical solution is to use the services of the _ForEach-Object_ cmdlet and pass one account at a time:

<pre class="brush: powershell; title: ; notranslate" title="">echo user1 user2 user3 | ForEach-Object {
   Disable-ADAccount -Identity $_
}
</pre>

Fortunately there&#8217;s a better and easy way to do that by simply piping the values directly to _Disable-ADAccount_. By default, _Disable-ADAccount_ does not generate any output. Add the _-PassThru_ switch if you want to see the modified object(s).

<pre class="brush: powershell; title: ; notranslate" title="">echo user1 user2 user3 | Disable-ADAccount -PassThru
</pre>
---
title: '#PSTip Overriding the ToString() Method'
author: Jakub Jareš
type: post
date: 2013-12-18T19:00:02+00:00
url: /2013/12/18/pstip-overriding-the-tostring-method/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
**Note**: This tip requires PowerShell 2.0 or above.

<span style="line-height: 1.5em;">In </span><a style="line-height: 1.5em;" href="http://104.131.21.239/2013/05/07/pstip-validate-your-custom-objects/">this very helpful tip</a><span style="line-height: 1.5em;">, Bartek shown us how to create a custom object using the <em>New-Module</em> cmdlet and how to constrain the data the object can hold. </span>Bartek &#8216;s object represents a user, and almost every time I consume  a &#8220;user&#8221; object the information I care about the most is the user&#8217;s full name. So I was wondering what would be the most convenient way to get such information and I realized I want to be able to do _&#8220;$User is now logged on.&#8221;_ and get _&#8220;John Doe is now logged on.&#8221;_, but I&#8217;ve tried it and got  _&#8220;@{BirthDate=12/17/1978 00:00:00; FirstName=John; LastName=Doe} is now logged on.&#8221;_.

So I experimented with it and it turns out all I had to do is define (and export) a ToString function which will output the correct data when the variable will be embedded inside a string.


    function New-User {
        param (
            [String]$FirstName,
            [String]$LastName,
            [DateTime]$BirthDate
        )
    
        New-Module {
            param ( $fn, $ln, $bd )
            [String]$FirstName = $fn
            [String]$LastName = $ln
            [DateTime]$BirthDate = $bd
    
            function ToString {"$FirstName $LastName" }
            Export-ModuleMember -Variable FirstName, LastName, BirthDate -Function ToString
    
        } -AsCustomObject -ArgumentList $FirstName,$LastName,$BirthDate
    }

```
#Create a new user...
PS> $User = New-User -FirstName John -LastName Doe -BirthDate 12/17/1978

#...and use it:
PS> "$User is now logged on."
John Doe is now logged on.

#Not surprisingly it works with other ways of converting to String too:
PS> $User.ToString() + " is the best!"
John Doe is the best!

PS> [string]$User
John Doe
```


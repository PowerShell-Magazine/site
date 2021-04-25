---
title: '#PSTip Quickly generate a password'
author: Boe Prox
type: post
date: 2012-10-22T18:00:34+00:00
url: /2012/10/22/pstip-quickly-generate-a-password/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note:** This tip requires PowerShell 2.0 and above

Generating a password usually consists of creating separate collections containing uppercase letters, lowercase letters, numbers and non-alphanumeric characters (@,%,$,^,&,*, etc…) and then randomizing the collections and pulling a specific number from each collection and then randomizing those into a password.

A more simple approach is to use the System.Web assembly that can be loaded using Add-Type.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Add-Type -AssemblyName System.Web
</pre>

Once loaded, you can make use of the GeneratePassword() from the System.Web.Security.Membership class. This method requires 2 parameters in order to properly create a password string to use. The first parameter is an integer to determine the length of the password which can be between 1 and 128 characters. The second parameter that is required is also an integer to determine the number of non-alphanumeric characters that will be used in the password. This number cannot be greater than the first parameter for the password length, otherwise the operation will fail.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [System.Web.Security.Membership]::GeneratePassword(15,4)
ZU-&amp;o}5O-A*|%%)
</pre>

Pretty handy to use if you are provisioning a lot of accounts and need to assign a password for each account. Being that this is something that can be quite useful; I wrapped this up into a simple function called New-Password that can be used to generate a password.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; New-Password -PasswordLength 18 -NumNonAlphaNumeric 9
b^}[*cH&gt;jY!D#NU(n&gt;
    Function New-Password {
    	[cmdletbinding()]
    
        Param (
            [parameter()]
            [ValidateRange(1,128)]
            [Int]$PasswordLength = 15,
            [parameter()]
            [Int]$NumNonAlphaNumeric = 7
        )
    
        If ($NumNonAlphaNumeric -gt $PasswordLength) {
            Write-Warning ("NumNonAlphaNumeric ({0}) cannot be greater than the PasswordLength ({1})!" -f`
                $NumNonAlphaNumeric,$PasswordLength)
            Break
        }
    
        Add-Type -AssemblyName System.Web
        [System.Web.Security.Membership]::GeneratePassword($PasswordLength,$NumNonAlphaNumeric)
    }
---
title: '#PSTip Validate your custom objects'
author: Bartek Bielawski
type: post
date: 2013-05-07T18:00:00+00:00
url: /2013/05/07/pstip-validate-your-custom-objects/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

In PowerShell 3.0 validation attributes work for regular variables in the same way they worked previously for parameters. They give us many options that were not possible previously. A simple example of an integer that will only support values from 1 to 10:

<pre class="brush: powershell; title: ; notranslate" title="">[ValidateRange(1,10)][int]$Note = 1
$Note = 11
The variable cannot be validated because the value 11 is not a valid value for the Note variable.
</pre>

It can be handy for variables defined in your profile, if you want to make sure you will not accidentally change them to a value that simply won’t work.

Today I would like to suggest another use case. There are many ways to define custom objects. But there is only one way that will, among many other useful things, allow us to constrain the values of the properties. In PowerShell 2.0, there was only the type of a property, in 3.0 , we can also use validation attributes. This technique is a very neat way of using _New-Module_ cmdlet. This cmdlet is primarily used to define dynamic modules, but when we specify _-AsCustomObject_ parameter, any exported variables become properties of custom object, and any exported functions – become its methods:

```
$ValidatedObject = New-Module {
    [ValidatePattern('^\w+$')]
    [string]$FirstName = 'John'

    [ValidatePattern('^\w+$')]
    [string]$LastName = 'Doe'

    [ValidateScript({
        if ($_ -lt '1-Jan-1890') {
            throw "This is for humans, not vampires!"
        } else {
            $true
        }
    })]
	[DateTime]$BirthDate = '17-Dec-1978'

	Export-ModuleMember -Variable *
} -AsCustomObject

PS> $ValidatedObject.BirthDate = 'Test'
Cannot convert value "Test" to type "System.DateTime".

PS> $ValidatedObject.BirthDate = '1-1-1600'
This is for humans, not vampires!

PS> $ValidatedObject.FirstName = 'John II'
The variable cannot be validated because the value John II is not a valid value for the FirstName variable.
```

Now not only the type of the variable meet our requirements, but also the value itself can be used to decide if we want the change value of properties or not.
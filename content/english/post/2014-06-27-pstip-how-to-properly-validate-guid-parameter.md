---
title: '#PSTip How to properly validate GUID parameter'
author: Emin Atac
type: post
date: 2014-06-27T18:00:15+00:00
url: /2014/06/27/pstip-how-to-properly-validate-guid-parameter/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Most of you probably know how to create a Globally Unique Identifier (GUID) with the following code:

<pre class="brush: powershell; title: ; notranslate" title="">[System.Guid]::NewGuid()
</pre>

But, let’s say, I have a function that has a GUID parameter. To validate the given GUID as an input, I’d use a regular expression with the _ValidatePattern_ attribute.


    Function Test-Foo {
        [CmdletBinding()]
        Param(
            [Parameter(Mandatory,ValueFromPipeline)]
            [ValidatePattern('(\{|\()?[A-Za-z0-9]{4}([A-Za-z0-9]{4}\-?){4}[A-Za-z0-9]{12}(\}|\()?')]
            [string[]]$GUID
        )
        Begin{}
    
        Process{
            $_
        }
    
        End{}
    }
The regular expression works great but doesn’t fully cover all the existing GUID formats. The _ValidateScript_ attribute combined with the Parse method of the _System.Guid_ object will help improve the validation.

Let’s see this in action:


    Function Test-Bar {
        [CmdletBinding()]
        Param(
            [Parameter(Mandatory,ValueFromPipeline)]
            [ValidateScript({
                try {
                    [System.Guid]::Parse($_) | Out-Null
                    $true
                } catch {
                    $false
                }
            })]
            [string[]]$GUID
        )
        Begin{}
    
        Process{
            $_
        }
    
        End{}
    }
I’ll first create an array of 5 random GUIDs with different formats. The surrounding parenthesis display the content while assigning the output to a variable.

```
($ar = "N","D","B","P","X" | ForEach {
	[System.Guid]::NewGuid().ToString($_)
})
```

![](/images/11.png)

Let’s pass this array of 5 valid GUIDs to the first function that uses the regular expression:

![](/images/guid2.png)

The error above is expected as the regular expression doesn’t cover the 5<sup>th</sup> format. There is a second issue with the regular expression&#8211;it would report an invalid GUID that mixes 2 or 3 different formats as valid.

![](/images/guid3.png)

The _Test-Foo_ function validates the above GUID although it isn’t valid.

Now, let’s try the second function that uses the validation script and parse method:

![](/images/guid4.png)

No error, all GUIDs are valid. The validation script method fixes the second issue as well:

![](/images/guid5.png)
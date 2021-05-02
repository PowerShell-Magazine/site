---
title: '#PSTip Set-StrictMode, why should you care?'
author: Shay Levy
type: post
date: 2012-10-23T18:00:26+00:00
url: /2012/10/23/pstip-set-strictmode-why-should-you-care/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Consider the following function:


    function test
    {
        $myNumbersCollection = 1..5
        if($myNfunction test
        {
            $myNumbersCollection = 1..5umbersColection -contains 3)
        {
            "collection contains 3"
        }
        else
        {
            "collection doesn't contain 3"
        }
    }
By looking at the code it is obvious to the naked eye that the result you should get is: &#8220;collection contains 3&#8221;. Cool, let&#8217;s put this to the test:

```
PS> test
"collection doesn't contain 3"
```

Ha?! What happened? 3 is in the range of 1 to 5! This is where the Set-StrictMode comes handy. When strict mode is turned on, PowerShell generates a terminating error when the content of an expression, script, or script block violates basic best-practice coding rules.

One important thing to keep in mind is that Set-StrictMode affects only the current scope and its child scopes. To debug script blocks, functions or scripts, turn on strict mode at the beginning of each scope.


    function test
    {
        Set-StrictMode -Version Latest
        $myNumbersCollection = 1..5
    
        if($myNumbersColection -contains 3)
        {
            "collection contains 3"
        }
        else
        {
            "collection doesn't contain 3"
        }
    }
    
    PS> test
    
    The variable '$myNumbersColection' cannot be retrieved because it has not been set.
    At line:8 char:8
    + if($myNumbersColection -contains 3)
    + ~~~~~~~~~~~~~~~~~~~
      ~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (myNumbersColection:String) [], RuntimeException
    + FullyQualifiedErrorId : VariableIsUndefined
A-ha! Now you can see that you actually had a typo and you missed one &#8216;l&#8217; in the variable name.

Without Set-StrictMode turned on in complex and long scripts, you could waste hours to find the typo. Now we can fix the typo and try again:




    function test
    {
        Set-StrictMode -Version Latest
        $myNumbersCollection = 1..5
    
        if($myNumbersCollection -contains 3)
        {
            "collection contains 3"
        }
        else
        {
            "collection doesn't contain 3"
        }
    }
    
    PS> test
    "collection contains 3"
Now it is working as expected. As a best practice, when authoring script and functions turn on strict mode at the beginning of the code so you can quickly capture those kind of mistakes. Lastly, to turn it off, type:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Set-StrictMode -Off
</pre>

For more information, type:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Help Set-StrictMode -Full
</pre>
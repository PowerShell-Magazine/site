---
title: 'Pester Explained: Should'
author: Jakub Jareš
type: post
date: 2015-12-02T17:00:32+00:00
url: /2015/12/02/pester-explained-should/
featured_image: /wp-content/uploads/2014/03/pester.png
views:
  - 16658
post_views_count:
  - 3574
categories:
  - DevOps
  - Pester
tags:
  - DevOps
  - Pester

---
This article is a part of a larger series on [Pester](/tags/pester).

Last time we looked at the theory of assertions and what mechanisms they use to fail our tests. We&#8217;ve also written two assertions of our own to become a part of our own test framework. In this article we won&#8217;t be continuing on out own framework though. Instead we will look more closely on the actual implementation of Pester assertions and walk through the process of failing a test in Pester.

In this article we will be looking just at the Should Be family of assertions, but keep in mind that the rest of the assertions work the same way. A condition is evaluated (be it some value comparison, if a file exists, or if an exception was thrown) and if the condition is not satisfied (False) an exception is thrown.

### Digging in

When you look inside of the sources of Pester you will find a whole folder dedicated to assertions. This folder is, unsurprisingly, called &#8220;Assertions&#8221; and resides inside of the &#8220;Function&#8221; folder. Similarly as the assertion keywords are split in two words, Should and Be, the assertion implementation is also split in two kinds of files. The Should.ps1 that defines the shared logic of all Pester assertions and Be.ps1, Throw.ps1, Exist.ps1 etc. which contain logic specific to the respective assertions.

### Be.ps1

Looking inside of [Be.ps1][2], on the top of the file there is the function that we&#8217;ve been looking at in the previous article, the equality condition that determines the result of the test:

```powershell
function PesterBe($value, $expected) {
    return ($expected -eq $value)
}
```


It uses the standard equality (-eq) operator of PowerShell, and returns a Boolean (true or false) result. Throwing the exception is done elsewhere.

You might also notice that the function is called &#8220;PesterBe&#8221; rather than &#8220;Be&#8221;, this naming convention was chosen in the early versions of Pester to avoid conflicts with user defined code. This need was eliminated by putting the whole Pester runtime in different scope (in version 3), effectively hiding the internals of Pester from user code. The details of how that is done will be described in one of the future articles.

Further down the file you may also notice an implementation of another, but very similar, assertion named [PesterBeExactly][3]. This case sensitive version of the Be assertion uses the case sensitive equality operator (-ceq), and so the different behavior only applies to strings.

Both the assertion condition implementations are accompanied with multiple function that produce various failure messages. The ones with Exactly in name are used when the BeExactly assertion is used, and the ones with Not in name are used when a negative version the assertion (e.g. Should Not Be) is called. The functions are also type aware so a message pointing at first different character will be produced when comparing strings.

```powershell
function PesterBeFailureMessage($value, $expected) {
    if (-not (($expected -is [string]) -and ($value -is [string])))
    {
        return "Expected: {$expected}`nBut was:  {$value}"
    }
    ...
    ( Get-CompareStringMessage -Expected $expected -Actual $value ) -join "`n"
}
```


The assertion condition (the PesterBe function) remains the same for the negative and non-negative call of the assertion though, the result is simply negated when a negative assertion is used.

### Should.ps1

The [Should.ps1][4] file holds the shared logic for the assertions. When calling an assertion you are in fact invoking a function named Should that takes pipeline input and an indeterminate amount of arguments (notice the $args variable and no param block).

```powershell
function Should {
    begin {
        Assert-DescribeInProgress -CommandName Should
        $parsedArgs = Parse-ShouldArgs $args
    }
       ...   
}
```


This, in theory would mean that you can have a very rich API for assertions, but in reality parsing a vast amount of different inputs correctly, while keeping intuitive syntax is difficult to do, and so the parsing logic is kept very simple. In general the expected input is assumed to be this:

```powershell
<Expected> | Should (optional)Not <AssertionName> <Value>
```

Any additional arguments are simply ignored.

The processing of the input is done in the [Parse-ShouldArgs][5] where the captured input is processed. Let&#8217;s see how a &#8220;1 | Should Not Be 10&#8221; would be processed:

```powershell
Parse-ShouldArgs Not,Be,10
#output
Name                           Value      
----                           -----
PositiveAssertion              False      
ExpectedValue                  10         
AssertionMethod                PesterBe
```

The &#8220;Be&#8221; assertion method is translated to &#8220;PesterBe&#8221;, referring to the function we saw earlier in the Be.ps1 file. The &#8220;Not&#8221; is captured as &#8220;PositiveAssertion:$False&#8221;, and the expected value obviously became the &#8220;ExpectedValue&#8221;.

Notice that the actual value is not captured in the output. That is because the call to Parse-ShouldArgs is placed in the begin block of the function where the pipeline output is not available. The actual value will be captured later.

_Note: This approach to calling functions is totally incoherent with the rest of PowerShell cmdlets. In your functions you should follow the correct approach of defining named parameters that take single argument value, or in special cases define ValueFromRemainingArguments attribute (see Write-Host). Avoid using the $args for anything else than getting an indeterminate amount of data. The way the $args is used in Pester is the legacy of the early versions of Pester where at first a fluent API like syntax was used, which was later migrated to the current approach in attempt to closely follow the language of RSpec testing framework. We are aware that the current syntax could be improved greatly, but unfortunately it&#8217;s so widely used that it is unlikely that it will go away any time soon._ 

```powershell
function Should {
    begin {
        Assert-DescribeInProgress -CommandName Should
        $parsedArgs = Parse-ShouldArgs $args
    }
    end {
        $input.MoveNext()
        do {
            $value = $input.Current
            $testFailed = Get-TestResult $parsedArgs $value

            if ($testFailed) {
               ...
            }
        } until ($input.MoveNext() -eq $false)
    }
}
```

When the arguments are parsed by the Parse-ShouldArgs and saved in the $parsedArgs, the Should command enters the end block. In this end block the actual value provided through the pipeline (in our case number 1) becomes available. The Should then continues to stepping through its pipeline input, invoking Get-TestResult on each of them.

The [Get-TestResult][6] function, residing in the Should.ps1 file, is rather simple. It takes the parsed should arguments (including the expected value) and the actual value and returns a boolean result. To determine the result it invokes the assertion condition (the PesterBe function) on the expected and actual values.

```powershell
function Get-TestResult($shouldArgs, $value) {
    $assertionMethod = $shouldArgs.AssertionMethod
    $command = Get-Command $assertionMethod -ErrorAction $script:IgnoreErrorPreference
    ...throw on incorrect assertion name


       #for 1 | Should Not Be 10
       #$testResult = (& PesterBe 1 10)
    $testResult = (& $assertionMethod $value $shouldArgs.ExpectedValue)

    if ($shouldArgs.PositiveAssertion) {
        return -not $testResult
    }
    return $testResult
}
```

The invocation of the assertion condition function is done via the &#8216;&&#8217; invocation operator. This works because of the aforementioned naming convention for those functions: Pester + <AssertionName> (e.g. Pester + Be).

At this point we know whether the assertion passed of failed, but we only have a True/False result, no exception was thrown yet. And that is the last thing that happens in the [Should][7] function.

```powershell
function Should {
    begin {
       ...
    }
    end {
        $input.MoveNext()
        do {
            ...
            $testFailed = Get-TestResult $parsedArgs $value
            if ($testFailed) {
                ...

                $failureMessage = Get-FailureMessage $parsedArgs $value
                throw ( New-ShouldErrorRecord -Message $failureMessage -File $file -Line $line -LineText $lineText)
            }
        } until ($input.MoveNext() -eq $false)
    }
}
```

The result of the call to Get-TestResult function is inspected, and if it is False a failure message is obtained. Then a Pester specific exception is thrown. This exception will stop the test from executing and will fail it. Exactly as described in the previous article.

> Note: The failure message is obtained, by a pretty much the same process as getting the result of the test. But instead of invoking the assertion condition function (PesterBe), an assertion message function is called ([NotPesterBeFailureMessage][8]), producing the appropriate message.
>

### Comparing the implementations

In the previous article we created a simple implementation of an assertion that did not take into account any parsing issues, nor different types of input objects as well as pipeline input. This left us with an extremely simple implementation, consisting only of a single &#8220;if&#8221; and &#8220;throw&#8221;:

```powershell
function Assert-Equal ($Expected, $Actual) {
    if ($Actual -ne $Expected)
    {
        throw "Value '$Expected' was expected, but was '$Actual'"
    }
}
```


Getting rid of all the clutter in the Should function, we can see the same basic pattern emerge:

```powershell
function Should {
    begin {
        ...
    }
    end {
        $input.MoveNext()
        do {
            ...
            if ($testFailed) {
                ...
                throw ( New-ShouldErrorRecord -Message $failureMessage -File $file -Line $line -LineText $lineText)
            }
        } until ($input.MoveNext() -eq $false)
    }
}
```


Which confirms that the theory that we learned the last time is applied in the actual code of Pester.

### Summary

In this article we looked closely at the implementation of the Should command in Pester. Described the process needed to fail an unsuccessful test and compared the theory that we learned with the actual implementation.

Next time we will look at the It and Describe blocks, how the tests are actually executed, and how the suite prevents failing on every failed test.

[2]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Be.ps1#L3
[3]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Be.ps1#L26
[4]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Should.ps1#L72
[5]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Should.ps1#L1
[6]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Should.ps1#L35
[7]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Should.ps1#L92
[8]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Be.ps1#L21
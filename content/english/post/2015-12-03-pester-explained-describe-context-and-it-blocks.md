---
title: 'Pester Explained: Describe, Context, and It Blocks'
author: Jakub Jareš
type: post
date: 2015-12-03T17:00:12+00:00
url: /2015/12/03/pester-explained-describe-context-and-it-blocks/
post_views_count:
  - 6834
categories:
  - DevOps
  - Pester
tags:
  - DevOps
  - Pester

---
This article is a part of a larger series on [Pester](/tags/pester).

Last time, we looked at how assertions work in theory, and how they are implemented in Pester. This gave us the foundation to understand how tests are failed, but in order to fail a test we first need to run it. So this time we will have a closer look at It, Context, and Describe and will create our own Test runner.

### Poor man’s test runner

In the simplest case, you do not need much to run test script code. Actually, we already did it in the first article where we tested our Assert-Equal assertion.

```powershell
function Assert-Equal ($Expected, $Actual) {
   if ($Actual -ne $Expected)
   {  
      throw "Value '$Expected' was expected, but was '$Actual'"
   }
}

$expected = 8
$actual = 4
Assert-Equal -actual $actual -expected $expected
```

We simply take a piece of code and wait for it to fail or pass. So technically, we do not need a test runner, but it makes our lives a lot easier. It takes care of looking up all tests in a test suite, shows a nicely colored and formatted output, and enables us to organize our tests a little better.

### Making our own test runner

The main reason to have a test runner, though, is to be able to run all tests in the test suite even if some of them fail. To be able to do that we need to catch any exception and translate it to textual output or some other harmless type of output.

In the Assert-Throw assertion, we already did something very similar. We captured an exception so the user could not see it. To do that we used a try-catch block and wrapped it in a function like this:

```powershell
function Assert-Throw ([ScriptBlock]$ScriptBlock) {
   $exceptionWasThrown = $False
   try
   {
       &$ScriptBlock
   }
   catch
   {
       $exceptionWasThrown = $True
   }

   if (-not $exceptionWasThrown)
   {
       throw 'Expected an exception to be thrown but no exception was thrown.'
   }
}
```

We are going to use a very similar approach for our test runner. Only this time we will output the exception that we captured, and we will output name of the current test as well:

```powershell
function Test-Case ([String]$Name, [ScriptBlock]$ScriptBlock) {
    try 
    {
       &$ScriptBlock
       Write-Host -ForegroundColor Green "${Name}: Test passed."
    }
    catch [Exception]
    {
        Write-Host -ForegroundColor Red "${Name}: Test failed because $_"
    }
}
```


In the Test-Case function we are taking a piece of code wrapped in a ScriptBlock. We execute this code using the & invocation operator and wait for it to either succeed or fail. By failing, we specifically mean that an exception was thrown. When an exception is thrown, for example by an assertion function, the code jumps inside of the catch block and outputs a red message to screen to notify us that the test code failed.

If no exception is thrown the code will write a green message, that lets us know that our code did not fail, which means that our test passed.

### Create a suite of tests

Now we are ready to take what we learned so far and create a suite of two tests:

```powershell
function Assert-Equal ($Expected, $Actual) {
    if ($Actual -ne $Expected) 
    {
        throw "Value '$Expected' was expected, but was '$Actual'"
    }
}

function Test-Case ([String]$Name, [ScriptBlock]$ScriptBlock) {
    try 
    {
       &$ScriptBlock
       Write-Host -ForegroundColor Green "${Name}: Test passed."
    }
    catch [Exception]
    {
        Write-Host -ForegroundColor Red "${Name}: Test failed because $_"
    }
}

Test-Case "Eight is four" { 
    $expected = 8
    $actual = 4
    Assert-Equal -actual $actual -expected $expected
}

Test-Case "Ten is ten" { 
    $expected = 10
    $actual = 10

    Assert-Equal -actual $actual -expected $expected
}
```

Which outputs:

```powershell
Eight is four: Test failed because Value '8' was expected, but was '4'
Ten is ten: Test passed.
```

Pay attention to the output. You can see that both tests run even though the first test failed. We just created our own test runner!

### It

The Test-Case function is roughly equivalent to the &#8220;It&#8221; function of Pester. &#8220;It&#8221; hosts a single test and prevents any failed test from failing the whole suite.

The [actual implementation of It][2] is riddled with input validation, testing the framework state, skipping tests, making them pending and so on, but the basic idea is still the same. Look at the implementation of [Invoke-Test][3] function and you will find the familiar pattern.

```powershell
$errorRecord = $null
try
{
    Invoke-TestCaseSetupBlocks
    do
    {
        $null = & $ScriptBlock @Parameters
    } until ($true)
}
catch
{
    $errorRecord = $_
}
finally
{
    ...
}

$result = Get-PesterResult -ErrorRecord $errorRecord
```

### More interesting bits of It

There are few more interesting bits in Pester&#8217;s It function. Those are not important to understanding the framework as a whole, but the It function implements so much that I feel obligated to describe at least some of those.

  * Any output of the ScriptBlock is [assigned to $null][4], which simply means that the output is discarded.
  * The script invocation is placed inside of a [single-iteration &#8220;do until&#8221; loop][5]. The code in this loop will only run once, and so the loop looks useless, but the opposite is true. When the &#8220;break&#8221; keyword is used in test code without any surrounding loop, it would jump outside of the &#8220;It&#8221; block and would make the test suite fail unexpectedly. To prevent this a single-iteration-loop is added around every test case.
  * The [setup][6] and [teardown][7] functionality enables you run code before and after every test case. Using a combination of try-catch-finally and try-catch blocks it guarantees that the teardown code will run even if the test fails, but exception in the teardown code will not fail the whole suite.
  * [Skip][8] and [Pending][9] parameters enable you to force the test into states different from Pass and Fail. Skip and Pending states are useful for temporarily putting some tests on hold and for notifying you of empty tests.
  * [TestCases][10] parameter enables you to define examples of input and expected values, which will result in the test being run once for each of the examples.

### Context and Describe

The Context and Describe are mainly the so-called syntactic sugar, a language construct that helps us explain our intentions better. They also let us organize the code in per-feature and per-use-case groups. Context and Describe differ slightly in Pester, but we will disregard those minor differences and will implement both groups by a single function.

```powershell
function Test-Block ([String]$Name, [ScriptBlock]$ScriptBlock) {
    try 
    {
        $Global:IndentationLevel++
        Write-Host -ForegroundColor Magenta (Indent-Line "Block $Name")
        &$ScriptBlock
    }
    catch [Exception]
    {
        Write-Host -ForegroundColor Red (Indent-Line "${Name}: Run failed in block because $_")
    }
    finally
    {
        $Global:IndentationLevel--
    }
}
```


The Test-Block again uses the familiar try-catch pattern. This is not by accident. Any code can be provided in the $ScriptBlock. This means that the code in Test-Block might fail, and we need to protect our test runner from that. We do not want exceptions to prevent us from running all test blocks.

Apart from that, we also thrown in some nice indentation. The indentation level is stored in a global variable and simply translated to tabulator spacing.

```powershell
$Global:IndentationLevel = 0

function Assert-Equal ($Expected, $Actual) {
   if ($Actual -ne $Expected)
   {
       throw "Value '$Expected' was expected, but was '$Actual'"
   }
}

function Test-Case ([String]$Name, [ScriptBlock]$ScriptBlock) {
   try
   {
       $Global:IndentationLevel++
       &$ScriptBlock
      Write-Host -ForegroundColor Green (Indent-Line "${Name}: Test passed.")
   }

   catch [Exception]
   {
       Write-Host -ForegroundColor Red (Indent-Line "${Name}: Test failed because $_")
   }

   finally
   {
       $Global:IndentationLevel--
   }
}

function Test-Block ([String]$Name, [ScriptBlock]$ScriptBlock) {
   try
   {
       $Global:IndentationLevel++
       Write-Host -ForegroundColor Magenta (Indent-Line "Block $Name")
       &$ScriptBlock
   }

   catch [Exception]
   {
       Write-Host -ForegroundColor Red (Indent-Line "${Name}: Run failed in block because $_")
   }

   finally
   {
       $Global:IndentationLevel--
   }
}

function Indent-Line
{
   param (
       [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
       [String[]]
       $Line
   )

   begin {
       $indent = $Global:IndentationLevel - 1
   }

   process {
       foreach($l in $Line) {  
           "`t" * $indent + $l
       }
   }
}

Test-Block "Describe" {
   Test-Block "Context" {
       Test-Case "Eight is four" {
          $expected = 8
          $actual = 4
          Assert-Equal -actual $actual -expected $expected
       }

       Test-Case "Ten is ten" {
          $expected = 10
          $actual = 10
          Assert-Equal -actual $actual -expected $expected
       }
   }

   Test-Block "Context 2" {
       Test-Case "I will fail" {
           throw
       }
   }
}
```

![](/images/pesterit.png)

### Summary

This concludes our review of Describe, Context and It. Hopefully you saw that a test runner is very simple at its core. Both Test-Case and Test-Block use very similar code. They execute input ScriptBlock and handle every possible exception.

Do not worry if the implementation of Test-Case seems too simplified in comparison to It. The It function handles more concerns than just error handling and output that we described in this article. We will look at those concerns in some of the upcoming articles.

[1]: http://powershellmagazine.com/tag/pester/
[2]: https://github.com/pester/Pester/blob/master/Functions/It.ps1
[3]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L247
[4]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L253
[5]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L251
[6]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L249
[7]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L267
[8]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L234
[9]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L238
[10]: https://github.com/pester/Pester/blob/master/Functions/It.ps1#L176
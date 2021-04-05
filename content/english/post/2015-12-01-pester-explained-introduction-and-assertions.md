---
title: 'Pester Explained: Introduction and Assertions'
author: Jakub Jareš
type: post
date: 2015-12-01T17:00:41+00:00
url: /2015/12/01/pester-explained-introduction-and-assertions/
views:
  - 19275
post_views_count:
  - 3909
categories:
  - DevOps
  - Pester
tags:
  - Pester
  - DevOps

---
This article is a part of a larger series on [Pester](/tags/pester).

I always found the word framework intimidating. It&#8217;s probably because my first encounter with the word was in .NET Framework, which at that point in time was total magic to me. There were tons of classes, keywords, and other things, and everybody except me, seemed to know the secret formula to connect the pieces together to make them do awesome stuff. And I was sitting there copying code from a book, being unable to make it work most of the time. And when it worked I was just waiting for the moment when the magic stops working and my code will break again. All in all, it took me a lot of time to understand that a framework is just code. I am not saying that I am a .NET expert now, but it feels liberating to know that no magic is likely happening in the core of my program.

This brings me to Pester. Pester is also called a framework, and what&#8217;s more a TESTING framework. That&#8217;s like double magic, or at least it was for me when I started learning about testing. And I am afraid a lot of you are in the same position. Wanting to learn how to test, but always fearing that the magical stuff that&#8217;s happening inside of the framework will stop working.

But you can&#8217;t be further from the truth, Pester is just code, and in the basic form quite simple code. And that&#8217;s why I am writing this series of posts that will cover the basic building blocks of Pester, and where you will get to write your own simpler version of Pester.

In the end you will hopefully be convinced that a framework is not much more than a pile of code that does exactly what it&#8217;s told.

## Assertion theory

Let&#8217;s start from the end, that is from the assertions, and work our way from the inside out.

The assertions are what decides whether the test will pass or fail. An assertion in Pester is represented by word Should and a second word such as Be or Exist, that determines what kind of assertion should be used. The most used combination of those words is Should Be, that tests whether the actual and expected values are equal, so we will use that for our first example.

```powershell
$expected = 8
$actual = 4
$actual | Should Be $expected
```


The $actual value typically wouldn&#8217;t be hardcoded in the test; rather it would be a result a some function call, such as Get-ProcessorCoreCount. The test would actually look more like this:

```powershell
$expected = 8
$actual = Get-ProcessorCoreCount
$actual | Should Be $expected
```


We can express the same comparison as a simple if statement:

```powershell
if ($actual -eq $expected)
{
   $didTheTestPass = $True
}
else
{
   $didTheTestPass = $False
}
```


We can also express the condition in an opposite way:

```powershell
if ($actual -ne $expected) 
{
    $didTheTestPass = $False
}
else 
{
    $didTheTestPass = $True
}
```


We could further get rid of the else clause, by assuming every test starts as a passing test.

```powershell
$didTheTestPass = $True
if ($actual -ne $expected) 
{
    $didTheTestPass = $False
}
```


This brings one problem though&#8211;if we put two such assertions in a row, it does not matter if the first one fails or passes, because only the last assertion would determine the outcome of the test.

That is definitely not correct. We want the first assertion to fail to stop the execution of the test. This can likely be done in many ways, but all the testing frameworks I know, throw an exception to do that. Our assertion would look like this:

```powershell
$didTheTestPass = $True
if ($actual -ne $expected) 
{
    throw "The test failed!"
}
```


At this point the $didTheTestPass does not bring any added value, because we know that any test that did not throw an exception is a passing test. The core of our assertion would look like this:

```powershell
if ($actual -ne $expected) 
{
   throw "The test failed!"
}
```


Which is simpler, but not oversimplified version of what is happening inside of Pester, after you remove all the parsing and fancy messages.

If you do not believe me, go to [line 92 in Should.ps1][2] and see for yourself. The comparison is also very easy to find, it&#8217;s on [line 3 in the Be.ps1][3] file.

### Writing your own assertion

I promised that we will write a testing framework of our own, so let&#8217;s start. It won&#8217;t be exactly like Pester, but I will be pretty close. Our own framework will definitely need to have assertions, but we will avoid all the unnecessary parsing Pester does and we will create a function called Assert-Equal which will be very simple:

```powershell
function Assert-Equal ($expected, $actual) {
    if ($actual -ne $expected) 
    {
        throw "Value '$expected' was expected, but was '$actual'"
    }
}
```


And we can write our first test (don&#8217;t forget to keep the Assert-Equal function in the same file and make sure you define it before you use it):

```powershell
function Assert-Equal ($Expected, $Actual) {
    if ($Actual -ne $Expected) 
    {
        throw "Value '$Expected' was expected, but was '$Actual'"
    }
}

$expected = 8
$actual = 4

Assert-Equal -actual $actual -expected  $expected
```

Which will output:

```powershell
Value '8' was expected, but was '4'
At line:5 char:3

+         throw "Value '$Expected' was expected, but was '$Actual'"
+         CategoryInfo          : OperationStopped: (Value '8' was expected, but was '4':String) [], RuntimeException
+         FullyQualifiedErrorId : Value '8' was expected, but was '4'
```


Yaay! You just wrote your first piece of a testing framework.

### Expecting exceptions

As we saw earlier using exceptions to fail tests is great because the test will stop executing as soon as any assertion fails. There is also other reason why exceptions are used in most of testing frameworks: When code fails, it usually throws an exception which in turn will make your test fail without using any assertion.

But what if throwing an exception is the expected outcome of the test? What if we want to test that our code throws FileNotFoundException when we try to read from a file that does not exist?

```powershell
function Read-File ($Path) {
    if (-not (Test-Path $Path))
    {
        throw [IO.FileNotFoundException]"The file '$Path' was not found."
    }
    Get-Content $Path
}
Read-File C:\NotExistingFile.txt
```

We need a way to check if a given piece of code thrown an exception, and prevent this exception from failing the test. The try/catch statement turns out to be a perfect candidate for that:

```powershell
function Read-File ($Path) {
    if (-not (Test-Path $Path))
    {
        throw [IO.FileNotFoundException]"The file '$Path' was not found."
    }

	Get-Content $Path
}

$exceptionWasThrown = $False
try 
{
   # Read-File C:\NotExistingFile.txt
}
catch 
{
    $exceptionWasThrown = $True
}

if (-not $exceptionWasThrown)
{
    throw 'Expected an exception to be thrown but no exception was thrown.'
}
```

Notice that the call to the Read-File is commented out, so by default no code will be executed in the try block, and hence no exception will be thrown inside of the try catch block. This means that the $exceptionWasThrown will remain $False, and so the last “if” will throw an exception saying &#8216;Expected an exception to be thrown but no exception was thrown.&#8217;, which will fail the test.

Now try to uncomment the call to the Read-File. The read file will throw an exception (unless you actually have a file called NotExistingFile.txt on your C: drive, of course). This exception will be swallowed by the catch block, preventing our test from failing. The catch block will also set the  $exceptionWasThrown variable to $True, the condition of the last &#8220;if&#8221; won&#8217;t be satisfied, and as a result the whole script won&#8217;t produce any output, meaning that our test passed.

### Assert-Throw

We could definitely use such useful assertion in our own framework, so let&#8217;s create a function named Assert-Throw, which will look like this:

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

As you can see the body of the function is almost the same as the code in the previous script, the only challenge that we needed to solve was getting the piece of code that should throw an exception as a parameter. We used a script block for that. A script block is a piece of code that we can execute at the right time and place using the &#8216;&&#8217; operator.

You can try using our new assertion, but expect no output because the Read-File throws an exception, which is what we want to happen and so our test passes:

```powershell
function Read-File ($Path) {
    if (-not (Test-Path $Path))
    {
        throw [IO.FileNotFoundException]"The file '$Path' was not found."
    }
	Get-Content $Path
}

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

Assert-Throw -ScriptBlock { Read-File C:\NotExistingFile.txt }
```

### Is there more to it?

Now that we covered two different assertions, you might ask: Is there more to it? And the answer would be: No, not really.

Every other assertion in Pester is just a variation on the Should Be (or in our case Assert-Equal). The difference between Should Be, Should Match, or Should Exist is only in the condition where in the &#8220;if&#8221; condition they use -eq, -match and Test-Item respectively, but the mechanism remains the same.

Single exception of this rule is the Should Throw assertion which on the outside acts same as the other assertions, but internally uses slightly different code to throw assertion when none was thrown, and do nothing when any assertion was thrown. If you’d like to compare the actual implementation in Pester with our own simplified assertion, and I encourage you to do so, please go to line [11 in PesterThrow.ps1.][4]

### Summary

So far you learned how assertions work internally and how they make tests fail. Next time we will look at the insides of Pester, and walk through the actual assertion implementation.

[2]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Should.ps1#L92
[3]: https://github.com/pester/Pester/blob/master/Functions/Assertions/Be.ps1#L3
[4]: https://github.com/pester/Pester/blob/master/Functions/Assertions/PesterThrow.ps1#L11
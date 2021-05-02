---
title: 'Testing your PowerShell scripts with Pester: Assertions and more'
author: Jakub Jareš
type: post
date: 2014-03-27T16:34:18+00:00
url: /2014/03/27/testing-your-powershell-scripts-with-pester-assertions-and-more/
categories:
  - DevOps
  - Pester
tags:
  - DevOps
  - Pester
---
<span style="line-height: 1.5em;">This is part two of Pester series of articles. In this article you will learn about the </span><i style="line-height: 1.5em;">Context</i> <span style="line-height: 1.5em;">keyword, how to use all the assertions available in Pester and a bit about best practices.</span>

**Note:** [The previous article][1] showed how to install Pester, established some of the basic terms and a _“Hello World”_ example. I recommend reading it before you proceed with this article.

To support you in using natural language, Pester uses a set of keywords that were carefully chosen to form a fluent domain-specific-language (DSL). You start by using the _Describe_ block to define what you are testing. Then you use _Context_ to explain the circumstances. And then you use _It_ and _Should_ to define the tests themselves. You already learned how to use _Describe_ and _It_. Let’s take a look at the rest.

### Context

_Context_ has a very similar function to the _Describe_ keyword. It lets you create groups of tests inside _Describe_ blocks. These blocks usually define the “context” in which the tests are run. As with _Describe,_ _Context_ is followed by a description and a script block.

<pre class="brush: powershell; title: ; notranslate" title="">Describe "Get-Item" {
	Context "no parameter is provided" {
		It "fails" {
			{ Get-Item } | Should Throw
		}
	}
	Context "path is provided" {
		It "returns the correct item" {
			( Get-Item -Path C:\Windows ).FullName | Should Be 'C:\Windows'
		}
		It "works when Path parameter is specified by position" {
			{ Get-Item C:\ } | Should Not Throw
		}
	}
}
</pre>

_Context_ also forms a separate scope in _TestDrive_. We will cover the details about that in the next article.

#### Placing a ScriptBlock

Although I call the _Describe_, the _Context_ and _It_ keywords, they are just a mere functions exported by the Pester module. This limits the way you can provide script blocks to their parameters. You have to place the opening brace on the same line as the keyword or use a backtick (\`) to escape the end of the line. Placing the braces as in the following example will fail:

<pre class="brush: powershell; title: ; notranslate" title="">Context "defines script block incorrectly"
{
   #some tests
}
</pre>

This limitation is the same for every function or cmdlet taking the -ScriptBlock parameter, even the ForEach-Object cmdlet which is natively provided with PowerShell has this limitation.

#### Assertions

An assertion determines if your test passes or fails by taking actual and expected values and comparing them in some way. In the example shown in the previous article, you used the _Should Be_ assertion as such:

<pre class="brush: powershell; title: ; notranslate" title="">Get-HelloWorld | Should Be 'Hello world!'
</pre>

The actual value in this case was the output of the _Get-HelloWorld_ function. The expected value was the _‘__Hello world!__’_ string. If these two values were equal the assertion passed, if they weren&#8217;t the assertion failed. As you can see, the actual value is provided by pipeline and the expected value is placed after the _Should Be_ keyword. All the other Pester assertions are used in the same manner&#8211;only the way the values are compared is different.

The assertions are:

  * Should Be
  * Should BeExactly
  * Should BeNullOrEmpty
  * Should Match
  * Should MatchExactly
  * Should Exist
  * Should Contain
  * Should ContainExactly
  * Should Throw

All of these assertions also have a negative version that passes where the positive assertion fails. (And the other way around of course.) Let’s use the negative version of the _Should Be_ assertion. We already know how to make sure the version of PowerShell we are using is not 1:

<pre class="brush: powershell; title: ; notranslate" title="">$PSVersionTable.PSVersion.Major | Should Not Be 1
</pre>

By placing _Not_ between the _Should_ keyword and the name of the assertion, I negated the result of the assertion. So instead of passing &#8216;if the input was equal to 1&#8217; it will fail. This is not particularly useful for the _Should Be_ assertion, but it is the only assertion we know so far. So let’s broaden our knowledge and take a look on each of Pester assertions (All the examples shown form a passing assertion).

#### Should Be

A _Should Be_ assertion is the most versatile assertion you can find in Pester and you will likely use it the most often. In general it is used like this:

```
<Actual value> | Should Be <Expected value>
```

The words wrapped in <> are just placeholders for the actual values. A notation I will use throughout this article to not overwhelm you with details. In reality you of course do not use the <> to enclose your actual and expected values.

Inside the assertion the standard _-eq_ operator is used to determine if the actual value and the expected value are equal. This has two implications: the assertion is case insensitive and references are compared when objects are compared. Keep that in mind when you write your tests.

Let&#8217;s see another example usage:

<pre class="brush: powershell; title: ; notranslate" title="">$Process = Get-Process -Name Idle
$Process.Name | Should Be Idle
</pre>
#### Should BeNullOrEmpty

This assertion tests if the value provided is null or empty and is used as such:

```
<Value> | Should BeNullOrEmpty
```

In my opinion the negative version of this assertion is more useful than the positive one. You will usually use it to test if your function returns any output.

<pre class="brush: powershell; title: ; notranslate" title="">Get-Help | Should Not BeNullOrEmpty
</pre>
#### Should Match

This assertion tests if a given string matches a regular expression.

```
<String> | Should Match <Pattern>
```

<pre class="brush: powershell; title: ; notranslate" title="">"PowershellMagazine.com" | Should Match "\.com$"
</pre>
#### Should Exist

This assertion tests if an item (file, folder, registry path…) exists.

```
<Path> | Should Exist
```

You will likely use it to test if file or folder exists, but it can test also Registry and other paths. Inside this assertion the _Test-Path_ cmdlet is used, so you can test any path _Test-Path_ can handle.

<pre class="brush: powershell; title: ; notranslate" title="">"C:\Windows" | Should Exist
</pre>
#### Should Contain

This assertion tests if a file contains a given string.

```
<Path> | Should Contain <Pattern>
```

Inside this assertion the _-match_ operator is used, so you use regular expressions to define the string you are looking for. This gives you a lot of power to make advanced tests, but on the other hand it makes the simple tests a bit more complicated. You have to look out for any characters that have special meaning in RegEx and escape them by “\”. If you are as lazy as I am or you determine the value on runtime so you don’t know the value beforehand, you can use a trick to force the assertion do “simplematch” (as-is match):

```
<Path> | Should Contain ([regex]::Escape(<Pattern>))
```

#### Should Throw

This assertion tests if a given script block throws an exception (a terminating error).

```
{ <Code> } | Should Throw
```

Notice that the code is enclosed in braces and as a result you are passing a script block to the assertion. This assertion is one of my favorites, but unfortunately it does not support testing for specific exceptions at the moment.

<pre class="brush: powershell; title: ; notranslate" title="">{ Get-Process -Name "!@#$%&" -ErrorAction Stop } | Should Throw
</pre>
#### Case sensitive assertions

The _Should BeExactly_, _Should MatchExactly_ and _Should ContainExactly_ are case sensitive versions of the _Should Be_, _Should Match_ and _Should Contain_ assertions. Doing case sensitive comparison makes sense only when you compare strings or characters. If you compare types like Integers, Booleans and others, using the case sensitive assertion has no effect on the result. In such cases I recommend using the standard assertions instead.

#### Multiple assertions

You can place more assertions inside a single _It_ block, but keep in mind that each test should have only a small span of concern. Ideally it should depend on a single condition. But if it makes sense to use multiple assertions, nothing is stopping you from doing that. In such case the first assertion to fail, fails the whole test.

#### Custom assertions

If you cannot find an assertion that suits your problem, you can always use the _Should Be_ and test for a _Boolean_. For example like this:

<pre class="brush: powershell; title: ; notranslate" title="">( "String" -is [String] ) | Should Be $true
</pre>

Or you can skip the assertion entirely and just _throw_ when the test should fail.

<pre class="brush: powershell; title: ; notranslate" title="">if ( "String" -isnot [String] )
{
	throw "The 'String' is not a String."
}
</pre>

Although this costs you the nicely formatted output Pester provides when the test fails.

You can also go to the [Pester issue page][2] and report a new issue there.

### PowerShell 2.0 compatibility

Pester is PowerShell 2.0 compatible and as long as PowerShell 2.0 will be relevant Pester will support it. But there is a really specific problem in PowerShell 2.0 that might make your tests act strangely. Let’s say for example you use a Get-DatabaseData function to get in user information and you want to fail the test if no data are returned:

```
function Get-DatabaseData ($User) {}

function HasValue {
	param (
		[Parameter(ValueFromPipeline=$true)]
		$Value
	)
	process {
		$allValues += $Value
	}
	end
	{
		($allValues -ne $null) -and (-not [string]::IsNullOrEmpty($AllValues))
	}
}
```

Two functions are defined in this script: Get-DatabaseData function that produces no output, because the user was not found in the db. And a simplified version of Should Not BeNullOrEmpty assertion I named HasValue to make the examples easier to understand. Let’s check if the “assertion” works:

<pre class="brush: powershell; title: ; notranslate" title="">Get-DatabaseData –User Jakub | HasValue
</pre>

As expected the “assertion” produces false because the Get-DatabaseData produced no output.

Now let’s make a small, seemingly harmless change and wrap the function into brackets and assert again:

<pre class="brush: powershell; title: ; notranslate" title="">( Get-DatabaseData –User Jakub ) | HasValue
</pre>

In PowerShell 3.0 or later this produces false again. But running the same script in PowerShell 2.0 produces no output! The command in brackets is evaluated, but for some reason the rest of the pipeline is not processed.

Trying the same but using a sub-expression instead of brackets:

<pre class="brush: powershell; title: ; notranslate" title="">$( Get-DatabaseData –User Jakub ) | HasValue
</pre>

Again this produces false in PowerShell 3.0 and no output in PowerShell 2.0.

This is pretty serious problem for Pester, if there is no failed assertion the test passes. So a test containing two negative assertions passes even though it should fail:

<pre class="brush: powershell; title: ; notranslate" title="">It "Passes in PowerShell 2.0" {
	( Get-DatabaseData –User Jakub ) | Should BeNullOrEmpty
	( Get-DatabaseData –User Jakub ) | Should Not BeNullOrEmpty
}
</pre>

To avoid this issue do not surround your functions with brackets or sub-expressions. Even if you are not using PowerShell 2.0. You’ll save someone from lot of debugging if he runs your tests on PowerShell 2.0.

Strangely enough both of these produce the correct output (false) in PowerShell 2.0 and later:

<pre class="brush: powershell; title: ; notranslate" title="">$() | HasValue
("") | HasValue
</pre>
### Writing your tests

All of the good books on programming I&#8217;ve ever read contained a recommendation that went along those lines: write your code like you should maintain it for the rest of your life. For me this is a very important advice. It does not mean making the code perfect. It rather means you should do your best to make your code readable, easy to understand, and easy to debug. The same holds true for your tests. Write tests that you’d love to inherit when taking over a project. Write tests that you’d love to read.

Keeping this rather difficult goal in mind I always start with thinking about _what_ the code should do. Instead of thinking about _how_ the code should do it. The more of the technical details I hide from myself the better. I use descriptions, comments, functions and nice variable names to abstract the problem I am solving.

I briefly mentioned this in the previous article. The _what_ is the specification and the _how_ is the implementation. If the difference between _what_ and _how_ is not clear I hope comparing these two example tests will help:

```
#test one
It "Additional info set to 'm'" {
	$user.AdditionalInfo[3] | should be 'm'
}

#test two
It "User is manager" {
	$isManager = $User.AdditionalInfo[3] -eq 'm'
	$isManager | Should Be $true
}
```

The first test starts from _how_ (the fourth character of the additional info is set to ‘m’) and leaves _what (or why)_ for you to figure out. This is highly impractical if you have a large test-base and look for tests testing a specific feature, not to mention your code is tested but the tests do not serve as easy-to-read specification.

The second test starts from _what_ (is the user a manager?), giving you the choice to find out _how_ (again fourth character is &#8216;m&#8217;) __if you need.

That example may look bit stretched or obvious but I&#8217;ve seen so many tests and scripts that only said _how_ without even mentioning _why,_ to feel obligated to point this out.

You also shouldn&#8217;t re-implement your tests in your functions and keep your tests independent on the environment. Mocking and _TestDrive_ helps a lot with the latter, and we will cover these in the next article.

[1]: /2014/03/12/get-started-with-pester-powershell-unit-testing-framework/
[2]: https://github.com/pester/Pester/issues
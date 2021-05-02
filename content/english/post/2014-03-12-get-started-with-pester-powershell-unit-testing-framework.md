---
title: Get started with Pester (PowerShell unit testing framework)
author: Jakub Jareš
type: post
date: 2014-03-12T17:38:47+00:00
url: /2014/03/12/get-started-with-pester-powershell-unit-testing-framework/
categories:
  - DevOps
  - Pester
tags:
  - Pester
  - DevOps

---
<span style="line-height: 1.5em;">Some time ago I stumbled upon the Pester framework that promised I would be able to test my scripts. This seemed to be super-useful for my day-to-day scripting, but unfortunately the learning curve was a bit steeper than I thought it would be. It took me reading a book on Test-Driven-Development (TDD) to understand what the ideas behind Pester are, and that they are actually pretty simple. Armed with this knowledge I was finally able to put Pester in use, and found it useful through the whole life cycle of my scripts. To help you with the first steps, I wrote a short series of articles that describe the basics of Pester.</span>

### What is Pester?

Pester is a unit testing framework for PowerShell. It provides a few simple-to-use keywords that let you create tests for your scripts. Pester implements a test drive to isolate your test files, and it can replace almost any command in PowerShell with your own implementation. This makes it great for both [black-box][1] and [white-box][2] testing. Pester is best used with [TDD][3] approach to development.

For those of you who are not familiar with TDD, let me sum it up briefly: TDD is the opposite of the traditional approach to development. Traditionally you write your functional code first and then you write tests for it. You are progressing from implementation (how the code is written) to specification (how it should work). TDD is turning this around, so you progress from specification to implementation. First you define how the code should work and then you write the code. This is a pretty powerful concept.

In TDD you write the specification in the form of tests. Before you add any new feature you write a set of tests for it. You run your test suite and make sure all the new tests fail. This shows that there is a feature missing. You then develop the feature, testing it frequently. Once all your tests (including the new ones) pass you know the new feature is finished. You also know you did not break anything else while adding it.

Do this for every feature, patch or any other change and you are always one test suite run from seeing if everything works as requested. That is the biggest benefit of TDD.

There are three more you might like:

  * TDD forces you think before you start scripting. If you are unable to write the tests, chances are you don’t fully understand the problem you are trying to solve.
  * Switching between projects and patching bugs becomes easier. With tests in place you don’t have to remember how exactly your components interact. You can focus only on the change you are making and validate the rest by running your tests.
  * Bugs do not reappear. Once write test for a bug and squash it will never get to your production code again.

Pester brings these benefits to PowerShell and that is why I love using it.

### First steps

#### Downloading and installing Pester

[_Pester_][4] is a PowerShell module authored by [Scott Muc][5] and improved by the community. It&#8217;s available for free on GitHub and installation is pretty easy. You just [download][4] it, extract it to your Modules folder, and then import it to your PowerShell session.

Let&#8217;s start by downloading it from the GitHub and extracting the archive into your Modules directory:

![](/images/pester1.png)

If you used Internet Explorer to download the archive, you need to unblock the archive before extraction, otherwise PowerShell will complain when you import the module. If you are using PowerShell 3.0 or newer you can use the Unblock-File cmdlet to do that:

```
Unblock-File -Path "$env:UserProfile\Downloads\Pester-master.zip"
```


If you are using an older version of PowerShell you will have to unblock the file manually. Go to your Downloads folder and right-click _Pester-master.zip_. On the general tab click _Unblock_ and then click OK to close the dialog.

![](/images/pester2.png)

Open your Modules directory and create a new folder called Pester. You can use this script to open the correct folder effortlessly:


    function Get-UserModulePath {
        $Path = $env:PSModulePath -split ";" -match $env:USERNAME
        if (-not (Test-Path -Path $Path))
        {
        	New-Item -Path $Path -ItemType Container | Out-Null
        }
        $Path
    }
    Invoke-Item (Get-UserModulePath)
Extract the archive to the Pester folder. When you are done you should have all these files in your Pester directory:

![](/images/pester3.png)

Start a new PowerShell session and import the Pester module using the commands below:

```
Get-Module -ListAvailable -Name Pester
Import-Module Pester
Get-Module -Name Pester | Select -ExpandProperty ExportedCommands
```

![](/images/pester4.png)

Next you will need a folder to play with. On my computer I created _&#8216;C:\Pester&#8217;_ for this purpose and you should do the same. If you choose another working directory, make sure you change the paths in the examples accordingly.

#### Creating and failing our first test

If you got this far I am sure you can’t wait to start testing. For the first example we’ll start with something extremely simple and create a function called _Get-HelloWorld_. The function will output a ‘Hello world!’ string when invoked and nothing more.

But before we start writing any code we should have a test in place. Creating a new test in Pester is easy. It contains a utility function _New-Fixture_ to create the basic &#8220;scaffolding&#8221; for a test. Optionally it can create a separate folder to keep our working folder well-organized.

```
cd C:\Pester
New-Fixture -Path HelloWorldExample -Name Get-HelloWorld
cd .\HelloWorldExample
Dir
```


![](/images/pester5.png)

The &#8220;scaffolding&#8221; created by the _New-Fixture_ function consists of two files, and auto-generated code that links them. The first file _Get-HelloWorld.ps1_ is the file where the production code is placed. The second file, _Get-HelloWorld.**Tests**.ps1_, is where the tests are placed. First take a look at the content of the _Get-HelloWorld.ps1_ file.

```
function Get-HelloWorld {

}
```

Nothing surprising except an empty function definition.

The content of the _Get-HelloWorld.Tests.ps1_ file is way more interesting, a default test is waiting there:

```
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
$sut = (Split-Path -Leaf $MyInvocation.MyCommand.Path).Replace(".Tests.", ".")
. "$here\$sut"

Describe “Get-HelloWorld" {
    It "does something useful" {
        $true | Should Be $false
    }
}
```

Before explaining how everything works, let’s do some testing first. First you need to change the test to reflect what the _Get-HelloWorld_ function should do. Take the whole content of the tests file and replace it with this:

```
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
$sut = (Split-Path -Leaf $MyInvocation.MyCommand.Path).Replace(".Tests.", ".")
. "$here\$sut"

Describe "Get-HelloWorld" {
    It "outputs 'Hello world!'" {
        Get-HelloWorld | Should Be 'Hello world!'
    }
}
```

![](/images/pester6.png)

Now it is time to run the test and see if it passes. To do so, you will use another Pester function called _Invoke-Pester_. By default,_Invoke-Pester_ runs all the tests in all the files in the current directory and its sub- <br clear="ALL" />directories, and that is exactly what we need:

```
cd C:\Pester\HelloWorldExample
Invoke-Pester
```


The output is red, and red means that the test failed. Don’t worry, remember what I said about TDD, you should always start with a failing test. Before we move on to implementing the function we should confirm the test failed for the right reason. In this case we are testing for specific output but the Get-HelloWorld returns nothing. Personally I would expect a message saying that there was no output but output was expected. And fortunately that is what the error message says:_&#8220;Expected: {Hello world!}, But was {}&#8221;_. The test fails correctly and we can finally implement the funciton. Open the _Get-HelloWorld.ps1_ file and replace the empty definition with this:

```
function Get-HelloWorld {
	'Hello world!'
}
```

![](/images/pester7.png)

Make sure you saved the file and run the test again:

```
cd C:\Pester\HelloWorldExample
Invoke-Pester
```


The tests passed this time and that means the function is working! Congratulations, you just created, failed and passed your first Pester test!

#### So what happened?

Let’s start over and explain what actually happened step-by-step.

First you took the default test and replaced it with another one. Two things were different in these tests. The description of the _It_ block and the code in the _It_ block.

In Pester the _It_ block represents one test. The description of the _It_ block sums up what is tested, and the code inside the script block determines whether the test passes or fails. If you’d keep the script block empty the test would always pass. Such test is useless so you need a way to make the test pass or fail depending on a given condition. To do that Pester implements a set of keywords called _assertions_.

The assertion used in the default and the new test is _Should Be_. This assertion takes input from the pipeline and compares it with the expected value. You provide that value after the _Should Be_ keywords. In the default test the _$true_ is compared to _$false_ and such test always fails.

In our test the output of the _Get-HelloWorld_ function is compared to the expected value _&#8216;Hello world!&#8217;_. So the result of the tests depends on the output of the _Get-HelloWorld_ function. When we invoked the test for the first time _Get-HelloWorld_ produced no output and hence the assertion failed with: _&#8220;Expected: {Hello world!}, But was {}&#8221;_ message. We then changed the function implementation to produce the correct output and the test passed.

#### Rest of the tests file

In the tests file there are few more things worth noticing. As you can see the _It_ block is placed inside a _Describe_ block. The _Describe_ block represents a group of tests and helps you keep your test file well-organized. It may seem useless now but it will prove more useful as your tests suite will grow. You can change the description of the _Describe_ to whatever you like, but do not remove the _Describe_ block entirely. Every tests file has to have at least one.

The _Describe_ keyword, has more to it than grouping your tests. You use it to separate scopes for [_TestDrive_][6], [_Mock_][7] and you can even use its description to run a set of tests. All these capabilities will be covered later. Now it is enough to remember that every test file has to have at least one _Describe_ block and that the _It_ blocks are placed inside the _Describe_ block.

The last thing in the tests file we did not cover are the top three lines of it. These lines link your tests file to your production code file. That is why the _Get-HelloWorld_ function can be used in your tests file even though it is defined elsewhere.

The first line sets the _$here_ variable to the path where the tests file is placed. The second line takes the name of the tests file, and by removing the &#8220;.Tests.&#8221; from the name it gets the name of the code file. The third line joins these two pieces of information together to form a full path to the code file and dot-sources it. Dot-sourcing the code file acts as if you copied the whole content of it and pasted it to the test file. As a result the _Get-HelloWorld_ function can be used inside the tests file.

#### Summary

This covers the basics of testing with Pester. In the next article we will take a detailed look on more Pester keywords and all the assertions available. Using _TestDrive_ and Mock will follow. If you found this article interesting, you missed something, or you disagree, please share your opinion in the comment section.

[1]: http://en.wikipedia.org/wiki/Black-box_testing
[2]: http://en.wikipedia.org/wiki/White-box_testing
[3]: http://en.wikipedia.org/wiki/Test-driven_development
[4]: https://github.com/pester/Pester
[5]: http://scottmuc.com/
[6]: https://github.com/pester/Pester/wiki/TestDrive
[7]: https://github.com/pester/Pester/wiki/Mock
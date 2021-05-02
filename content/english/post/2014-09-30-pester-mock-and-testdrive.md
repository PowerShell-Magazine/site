---
title: Pester Mock and TestDrive
author: Jakub Jareš
type: post
date: 2014-09-30T10:00:07+00:00
url: /2014/09/30/pester-mock-and-testdrive/
categories:
  - Pester
  - DevOps
tags:
  - Pester
  - DevOps

---
In this part of Pester basics series, I will cover the most powerful tool from the whole framework, the _Mock_ function. This function lets you hide any function with a fake implementation of your choosing, count how many times it was called and filter on parameters of the call. This comes in handy when you need to force the code under test to a stable predefined state. In other words detach it from real world resources.

Before we have a deeper look at how exactly the _Mock_ is used there are two things you should know: The version of Pester used in this article is the stable 2.0.1+ version you can find in the master branch on Github. There is also a new 3.0.2 version available in the master branch, packing a lot of new exciting features, but bringing few breaking changes at the same time. To keep all the information in the previous two articles relevant I decided to use the same major version as with the previous two articles.

The second thing is: Testing your code with PowerShell is difficult, because PowerShell is all about dealing with real world resources. Such resources are files for example. A file can be deleted, renamed, moved, locked for access or made unavailable in a different way. Not being certain about the environment in which our tests run is a huge problem. After all, we want our test suite to be as stable and as independent from the real world as possible. Actually each of our tests should test only one aspect of the problem and all the other aspects should be stable. The _Mock_ function helps us with this to some extent but you should always keep in mind that every use of _Mock_ puts you in danger of replacing the real command with an inaccurate representation of it. Make sure you recognize the boundaries between your script and the real-world and make sure you replace and test these connections super-carefully.

Now before boring you to death let’s see some code:

<pre class="brush: powershell; title: ; notranslate" title="">function Restart-InactiveComputer
{
    if (-not ( Get-Process explorer -ErrorAction SilentlyContinue ) )
    {
        Restart-Computer -Force
    }
}
</pre>

This function does pretty much what its name says&#8211;it restarts the computer when there is no user logged on. To be exact, it uses Explorer.exe process to determine if any user is logged on. If there is no Explorer.exe running there is no user logged on. This function is saved in Restart-InactiveComputer.ps1 file.

For the end-user this function is super-easy to use as there are no parameters. For us testers it is impossible to test with our current skill set. The reason why, is that we can’t easily control the number of Explorer.exe processes running, and even if we somehow executed the test without having the Explorer process running it would only result in restarting the station. Not exactly what you want to do every time you run your test suite.

So now that we identified the boundaries, we have to replace them with appropriate mocks. The Restart-Computer should ideally do nothing, but we still need a way to figure out if it was called so for the time being we make it to return “Restarting!” when called:

<pre class="brush: powershell; title: ; notranslate" title="">Mock Restart-Computer { “Restarting!” }
</pre>

Don’t run the code just yet it will fail if you run it outside a Describe block.

The Get-Process is used in an if condition so to test it we should have at least two different versions of the command: one that returns nothing and one that returns something.

<pre class="brush: powershell; title: ; notranslate" title="">Mock Get-Process {}
Mock Get-Process { $true }
</pre>

Please notice that I am not creating a new System.Diagnostics.Process object in the mocks, nor any special kind of object. I am just returning the plain minimum to control the result of the if condition, that is returning nothing and returning $true. And by the way you don’t have to use the curly brackets if you define empty mock, but I do because it makes the _Mock_ easier to identify when you skim the code.

Now that we have all the basic building blocks let’s put them in a fixture and save it to a file called Restart-InactiveComputer.Tests.ps1:

<pre class="brush: powershell; title: ; notranslate" title="">$here = Split-Path -Parent $MyInvocation.MyCommand.Path
$sut = (Split-Path -Leaf $MyInvocation.MyCommand.Path).Replace(".Tests.", ".")
. "$here\$sut"
Describe "Restart-InactiveComputer" {
    Mock Restart-Computer { "Restarting!" }
    It "Restarts the computer" {
       Mock Get-Process {}
       Restart-InactiveComputer | Should be “Restarting!”
    }
    It "Does not restart the computer if user is logged on" {
       Mock Get-Process { $true }
       Restart-InactiveComputer | Should BeNullOrEmpty
    }
}
</pre>

As you can see both of the tests are green and that means they passed.

## Assert-MockCalled

In the previous example we used the output of the mocked Restart-Computer function to check if the function was called or not. This is possible in the simplest of cases but if the output of Restart-Computer was piped to Out-Null, and so the output was discarded this would not be possible. Fortunately Pester provides Assert-MockCalled function that helps you count how many times a _Mock_ was called.

```
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
$sut = (Split-Path -Leaf $MyInvocation.MyCommand.Path).Replace(".Tests.", ".")
. "$here\$sut"

Describe "Restart-InactiveComputer" {
    Mock Restart-Computer { "Restarting!" }
    It "Restarts the computer" {
        Mock Get-Process {}
        Restart-InactiveComputer | Out-Null
        Assert-MockCalled Restart-Computer -Exactly 1
    }

    It "Does not restart the computer if user is logged on" {
        Mock Get-Process { $true }
        Restart-InactiveComputer | Out-Null
        Assert-MockCalled Restart-Computer -Exactly 0
    }
}
```

Which yields these results:

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\temp\example1&gt; Invoke-Pester
Executing all tests in C:\temp\example1
Describing Restart-InactiveComputer
[+] Restarts the computer 6ms
[-] Does not restart the computer if user is logged on 12ms
Expected Restart-Computer to be called 0 times exactly but was called 1 times
at line: 393 in C:\temp\example1\Restart-InactiveComputer.Tests.ps1
Tests completed in 19ms
Passed: 1 Failed: 1
</pre>

As you can see the last tests failed, because the Restart-Computer command was called once but it should not been called at all. There isn’t any problem in the test case itself. We are hitting Pester 2.0 limitation here. The _Mock_ call history is shared through the Context and there is no way to change it. So the only way around this is to define a Context for each of the tests:

```
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
$sut = (Split-Path -Leaf $MyInvocation.MyCommand.Path).Replace(".Tests.", ".")
. "$here\$sut"
Describe "Restart-InactiveComputer" {
   Mock Restart-Computer { "Restarting!" }
   Context "Computer should restart" {
      It "Restarts the computer" {
         Mock Get-Process {}
         Restart-InactiveComputer | Out-Null
         Assert-MockCalled Restart-Computer -Exactly 1
      }
   }
   Context "Computer should not restart" {
      It "Does not restart the computer if user is logged on" {
          Mock Get-Process { $true }
          Restart-InactiveComputer | Out-Null
          Assert-MockCalled Restart-Computer -Exactly 0
      }
   }
}


PS C:\temp\example1> Invoke-Pester
Executing all tests in C:\temp\example1
Describing Restart-InactiveComputer
Context Computer should restart
[+] Restarts the computer 6ms
Context Computer should not restart
[+] Does not restart the computer if user is logged on 5ms
Tests completed in 11ms
Passed: 2 Failed: 0
```

And finally both of the tests pass.

## Default and filtered mocks

So far we only used default mocks. Any call to the Get-Process cmdlet was replaced by the call to the mocked version of the cmdlet. This is enough for testing our idealized example function but in real life you need more control. For this reason there is -ParameterFilter parameter for the Mock and the Assert-MockCalled functions. This parameter lets you select the appropriate mock based on the parameters used when calling the command. The usage is the following:

```
Mock Get-Process { "default" }
Mock Get-Process { "filtered" } -ParameterFilter { $Name -eq "Explorer" }
```


and here are tests using the mocks, verifying the results of the calls:

```
Describe "MultipleMocks" {
    Mock Get-Process { “default” }
    Mock Get-Process { “filtered” } -ParameterFilter { $Name -eq "Explorer" }
    It "Calls the default mock" {
        Get-Process | Should Be "default"
    }
    
    It "Also calls the default mock" {
        Get-Process -Name Idle | Should Be "default"
    }

    It "Calls the filtered mock" {
        Get-Process -Name Explorer | Should Be "filtered"
    }

}

PS C:\temp\example2> Invoke-Pester
Executing all tests in C:\temp\example2
Describing MultipleMocks
[+] Calls the default mock 8ms
[+] Also calls the default mock 2ms
[+] Calls the filtered mock 5ms
Tests completed in 15ms
Passed: 3 Failed: 0
```

This way you can simply define a one default “fallback” mock that keeps you safe if something goes wrong. (Like keeping your station from being restarted.) And then create multiple less general mocks that target each call to the command specifically. But you don’t have to create a default mock at all, if you don’t need to.

This brings us to describing in which order the filtered and default mocks are evaluated and what scope rules are used.

## Mock evaluation order

There are basically two stacks of Mocks in Pester for each mocked command. One stacking the default mocks and one stacking the filtered mocks. In reality this means that the last filtered _Mock_ to be defined is the first to be evaluated. And the same holds true for the default mocks, but the filtered mocks are always evaluated first. If a mocked command is called, Pester goes through all the filtered mocks and checks if any of the filters returns true and uses that _Mock_. If no matching filtered _Mock_ is found it looks for a default Mock to use. If there is none the original command is used.

## Mock scoping

The last set of rules you should keep in mind are the scoping rules that apply to mocks and mock call assertions. They complicated our lives already when we tried to assert on a mock call count in the first example, and they may produce all kinds of unexpected results if you are not aware of them:

  * _Mock_ defined in Describe is available in the whole Describe.</p> 
  * _Mock_ defined in Context is available in the whole Context.

  * _Mock_ defined in It is available in its parent scope. That is in whole Context if the It is placed in a Context, and in whole describe if the It is placed in a Describe.

Now for our final example let’s see another version of the first example which targets the mocks and assertions more precisely, making sure the commands are called with the correct parameters:

```
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
$sut = (Split-Path -Leaf $MyInvocation.MyCommand.Path).Replace(".Tests.", ".")
. "$here\$sut"

Describe "Restart-InactiveComputer" {
   Mock Restart-Computer { "Restarting!" }
   Context "Computer should restart" {
      It "Restarts the computer" {
         Mock Get-Process {} -ParameterFilter { $Name -eq "Explorer" }
         Restart-InactiveComputer | Out-Null
         Assert-MockCalled Restart-Computer -Exactly 1 -parameterFilter { $Force }
      }
   }

   Context "Computer should not restart" {
      It "Does not restart the computer if user is logged on" {
          Mock Get-Process { $true } -ParameterFilter { $Name -eq "Explorer" }
          Restart-InactiveComputer | Out-Null
          Assert-MockCalled Restart-Computer -Exactly 0 -parameterFilter { $Force }
      }
   }
}

PS C:\temp\example3&gt; Invoke-Pester
Executing all tests in C:\temp\example3
Describing Restart-InactiveComputer
Context Computer should restart
[+] Restarts the computer 5ms
Context Computer should not restart
[+] Does not restart the computer if user is logged on 6ms
Tests completed in 11ms
Passed: 2 Failed: 0
```

## TestDrive

Working with temporary files is always a hassle, you have to create a temporary file storage, resolve any naming conflicts and clean up when you are done. Fortunately all this is done automatically by Pester and exposed as a PSDrive called _TestDrive_. A storage that you can use to isolate your test files from the environment.

<pre class="brush: powershell; title: ; notranslate" title="">TestDrive:\test_file.txt | Should Exist
</pre>

A basic scoping rules are implemented for the _TestDrive_. A clean _TestDrive_ is created for every Describe and all the files created are available in the whole Describe scope. If the Context keyword is also used the state of the _TestDrive_ is recorded before moving into the Context block. Inside the Context block the files from the Describe scope are available for reading and modification. You can move them around and create new ones as well.

Once the Context block is finished all the files created inside that block are deleted, leaving only the files created in the Describe block. When the Describe block is finished all contents of the _TestDrive_ are discarded.

Recording the state of the drive is done by saving a list of the files and folders present on the drive. No snapshots or any other magic is done. In practice this means that if you create a file in the Describe block and then change its content inside the Context block, the modifications are preserved even after you left the Context block.

Internally the _TestDrive_ creates a randomly named folder placed in $env:Temp for every Describe and links it to the _TestDrive_ PSDrive. Making the folder names random enables you to run multiple instances of Pester in parallel, as long as they are running as separate processes. That means running in different PowerShell.exe sessions or running using PowerShell jobs.
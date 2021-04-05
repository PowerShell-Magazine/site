---
title: 'Pester: Triangulation and reusing test cases'
author: Jakub Jareš
type: post
date: 2015-06-04T16:00:19+00:00
url: /2015/06/04/pester-triangulation-and-reusing-test-cases/
views:
  - 12429
post_views_count:
  - 3129
categories:
  - Pester
tags:
  - Pester

---
Pester has a great feature called TestCases. This feature enables you to easily call the same test code multiple times, but with different data.

This comes handy in two cases&#8211;triangulation and reusing test code.

### Triangulation

Triangulation is one of the base principles of automated testing. You provide two or more examples of input & output and run it through the tested function. When done properly it will force you to come up with general implementation of the tested function.

In the following example we verify the behavior of the [modulo operation][1]. This operation returns the remainder of whole number division and is denoted as % in PowerShell.

![](/images/pestertri.png)

Providing just the first two test cases, we could simply fool the test and return the value of the a. But the more test cases we provide, the more certain we are that the operation in fact behaves correctly.

Adding more test cases without the TestCases functionality would require a lot of copy-pasting and modifying the values. This is very error prone and would likely cause a lot of problems.

With TestCases it is as simple as this:


```powershell
Describe 'Test reuse' {
    It 'Prerequisite file <file> is present' `
    -TestCases `
        @{file = '.\Script1.ps1'},
        @{file = '.\Script2.ps1'},
        @{file = '.\Script3.ps1'} `
    -Test {
        param ([string]$file)
	$file | Should Exist
}
}
```
![](/images/pestertri2.png)

As you can see the test was executed once for each test case and produced a very clear output without barely any effort. You should also notice that you can use tokens named as the parameter names to show the input and output values for the given test case. In our example the parameters are $a, $b and $c and so the tokens are <a>, <b>, and <c>.

### Test case reuse

Another scenario where the TestCases feature is very useful is for testing prerequisites.

![](/images/pestertri3.png)

In the above picture you can see that the test takes a path to a script and tests if that script file exist. On the first look it might seem pretty similar to triangulation, but the test does not point to a single function. Instead it points to multiple places which are _determined_ by the input data.

In code it would look like this:

```powershell
Describe 'Test reuse' {
    It 'Prerequisite file is present' -TestCases
    @{file = '.\Script1.ps1'},
    @{file = '.\Script2.ps1'},
    @{file = '.\Script3.ps1'} -Test {
    param ([string]$file)

    $file | Should Exist
    }
}
```

![](/images/pestertri4.png)

[1]: http://en.wikipedia.org/wiki/Modulo_operation
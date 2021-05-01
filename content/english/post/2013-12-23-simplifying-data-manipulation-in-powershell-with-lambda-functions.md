---
title: Simplifying Data Manipulation in PowerShell with Lambda Functions
author: Matt Graeber
type: post
date: 2013-12-23T17:00:29+00:00
url: /2013/12/23/simplifying-data-manipulation-in-powershell-with-lambda-functions/
categories:
  - How To
tags:
  - How To

---
Scenario: You have a large dataset and need to perform various transformations on the data. For example, let’s say you need to multiply a sequence by a variable constant and you need to add a sequence by a variable constant. If you’re going to perform this task many times, naturally, it would make sense to write a function that performs these operations. For example, these functions might manifest themselves in the following way:


    function Add
    {
        param (
            [Int] $Constant,
            [ValidateNotNullOrEmpty()]
            [Int[]] $Sequence
        )
        foreach ($Object in $Sequence)
        {
            Write-Output ($Object + $Constant)
        }
    }
    
    function Multiply
    {
        param (
            [Int] $Constant,
            [ValidateNotNullOrEmpty()]
            [Int[]] $Sequence
        )
        foreach ($Object in $Sequence)
        {
            Write-Output ($Object * $Constant)
        }
    }
    
    Add 2 @(1, 2, 3, 4, 5)
    Multiply 5 @(1, 2, 3, 4, 5)
Sure, defining these functions is easy enough, but what if hundreds of transformations are necessary? Defining hundreds of helper functions would obviously be tedious and would clutter your script. Rather than declaring a function that describes how to transform each element, what if we could just apply a simple transformation to each element that didn’t require a function definition? This can be accomplished with lambda functions.

A lambda function is a nameless (i.e. anonymous) function that accepts arguments and returns the result of a simple operation that doesn&#8217;t affect variables outside of its scope. In PowerShell lingo, a lambda function is simply a script block with a _‘param’_ declaration.

Now that I&#8217;ve confused you with this abstract concept, allow me to show you some examples to put things into perspective. Here’s a sample lambda function in PowerShell that simply multiplies a number by two:

<pre class="brush: powershell; title: ; notranslate" title="">$Double = { param($x) $x * 2 }
</pre>

To invoke the lambda function, you simply prefix it with the call operator _(&)_ or the _Invoke_ method.

<pre class="brush: powershell; title: ; notranslate" title="">&$Double 2
$Double.Invoke(2)
</pre>

So now let’s apply arbitrary lambda functions to a dataset. As an example, I’m going to replicate the _‘map’, ‘reduce’_, and _‘filter’_ functions in Python. The _‘map’_ function applies a function to each element of a sequence. The _‘reduce’_ function applies a function with two arguments cumulatively to a sequence of objects, hence _‘reducing’_ the dataset to a single object. Finally, the _‘filter’_ function returns a subset of objects from a sequence when a function evaluates to true. Here is a simple PowerShell implementation of each of these functions:

    #requires -Version 3
    # Ast parameter validation is used to ensure that the lambda
    # function passed in has either one or two parameters.
    
    function Map-Sequence
    {
        param (
            [Parameter(Mandatory)]
            [ValidateScript({ $_.Ast.ParamBlock.Parameters.Count -eq 1 })]
            [Scriptblock] $Expression,
            [Parameter(Mandatory)]
            [ValidateNotNullOrEmpty()]
            [Object[]] $Sequence
    	)
    
    	$Sequence | % { &$Expression $_ }
    }
    
    function Reduce-Sequence
    {
        param (
            [Parameter(Mandatory)]
            [ValidateScript({ $_.Ast.ParamBlock.Parameters.Count -eq 2 })]
            [Scriptblock] $Expression,
            [Parameter(Mandatory)]
            [ValidateNotNullOrEmpty()]
            [Object[]] $Sequence
    	)
    
    	$AccumulatedValue = $Sequence[0]
    
        if ($Sequence.Length -gt 1)
        {
            $Sequence[1..($Sequence.Length - 1)] | % {
                $AccumulatedValue = &$Expression $AccumulatedValue $_
            }
        }
    
        $AccumulatedValue
    }
    
    function Filter-Sequence
    {
        param (
            [Parameter(Mandatory)]
            [ValidateScript({ $_.Ast.ParamBlock.Parameters.Count -eq 1 })]
            [Scriptblock] $Expression,
            [Parameter(Mandatory)]
            [ValidateNotNullOrEmpty()]
            [Object[]] $Sequence
    	)
    
    	$Sequence | ? { &$Expression $_ -eq $True }
    }
Now that we have these helper functions in place, we can easily apply transformations to sets of data without needing to define traditional (i.e. imperative) functions. Consider the following examples:

```
$IntArray = @(1, 2, 3, 4, 5, 6)
$Double = { param($x) $x * 2 }
$Sum = { param($x, $y) $x + $y }
$Product = { param($x, $y) $x * $y }
$IsEven = { param($x) $x % 2 -eq 0 }

Map-Sequence $Double $IntArray
Reduce-Sequence $Sum $IntArray
Reduce-Sequence $Product $IntArray
Filter-Sequence $IsEven $IntArray
```

The _‘Map-Sequence’_ example is a one-liner that doubles each element of an array. The _‘Reduce-Sequence’_ one-liner calculates the sum of the elements of an array. It then calculates the product of an array. Finally, the _‘Filter-Sequence’_ one-liner returns only even numbers from the array.

Hopefully, now you can see just how easy it can be to perform simple transformations to data without needing to define functions for each transformation. By using lambda functions in your scripts, you can greatly simplify your code!
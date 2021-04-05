---
title: Advanced Script Profiling
author: Adam Driscoll
type: post
date: 2011-11-15T07:27:15+00:00
url: /2011/11/15/advanced-script-profiling/
views:
  - 33694
post_views_count:
  - 3092
categories:
  - Performance
tags:
  - Performance

---
### Introducing Measure-Block

Sometimes things just don’t work as expected. When scripts get really complicated, performance bottlenecks can be hard to find. A lot of times you can nail down the issue by wrapping a Measure-Command around questionable portions of the script. This can be time consuming. Because of this I started developing a new advanced function: Measure-Block. Measure-Block can be placed anywhere with your PowerShell script. Wrapping a script block with Measure-Block statements integrated into it with another Measure-Block statement and utilizing the –Process parameter will yield a detailed profiling report of the different sub-sections within the script that you choose to profile. Sounds complicated so here’s an example:

```powershell
PS> New-Alias -Name MB -Value Measure-Block            

function TestScript()
{
 MB {
  $variable = MB { Get-Process } -PassThru
  MB {
   foreach($var in $variable)
   {
    MB {
     $var | Format-Table
    }
   }
  }
 }
}            

PS> Measure-Block -Process -ScriptBlock { TestScript } | Format-MeasureBlock
```

The above script snippet defines a function with a few `Measure-Block` calls distributed amongst the code. Each of the calls to Measure-Block profiles the script block that is provided with the call to the function. The resulting object model is hierarchal and allows for more detailed analysis. Imagine a tree where the durations of the children make up the duration of their parent.

  * Root – 10 Seconds 
      * Loop 1 – 5 Seconds 
          * Get-Process – 3 Seconds
          * Get-Service – 2 Seconds
      * Loop 2 – 5 Seconds 
          * Start-Service – 5 Seconds

There is also a `Format-MeasureBlock` command that flattens and outputs the object for a quick glance on profiling. Here’s some example output. Note that the name can be customized but by default is set to the location in the script.

```powershell
Name                 Duration         Scope
----                 --------         -----
MEASURE_BLOCK:285:4  00:00:01.8197878     1  #Inside TestScript
MEASURE_BLOCK:286:17 00:00:00.0075114     2  #Get-Process
MEASURE_BLOCK:287:5  00:00:01.7896269     2  #The Total Loop
MEASURE_BLOCK:290:7  00:00:00.0087703     3  #Each Loop Iteration
MEASURE_BLOCK:290:7  00:00:00.0042948     3  #...
MEASURE_BLOCK:290:7  00:00:00.0157303     3
MEASURE_BLOCK:290:7  00:00:00.0161733     3
```


When the outer `Measure-Block` command is not present the inner non-Process commands will simply behave as a pass-through, effectively turning off profiling. Like any good profiler, the Measure-Block cmdlet will add overhead to a script because of the extra processing that is going on to collect the data.

### Inside Measure-Block

Measure-Block works by tracking command duration with a Stopwatch. This is the same mechanism that Measure-Command uses. Start and Stop are called during the Begin and End of the Measure-Block function.

The command also utilizes the Stack class to keep track of the different Measure-Block calls. During the Begin method in the Measure-Block function, a new “MeasureBlock” object is pushed onto the Stack and the Scope depth variable is increased. The object is also added to the previous object’s Children collection. This is all done using the Push-MeasureBlockScope function.

```powershell
function Push-MeasureBlockScope()
{
 [CmdletBinding()]
 param(
  [Parameter()]
  $MeasureBlock
 )            

 Begin
 {
  if ($Global:MeasureBlockScope -ne )
  {
   $Global:MeasureBlockStack.Peek().Children += $MeasureBlock
   $MeasureBlock.Scope = $Global:MeasureBlockScope
   $Global:MeasureBlockScope++
  }            

  $Global:MeasureBlockStack.Push($MeasureBlock)
 }
}
```

During the End method, the `Pop-MeasureBlockScope` is called to move back up the stack. The Process method of the function works in three different ways. If there is input on the pipeline but no script block the input is simply pushed through the cmdlet. If there is input on the pipeline and a script block, the input is piped to the script block. Finally, if there is only a script block, it is simply invoked.

This allows the Measure-Block function to not only utilize script blocks, like Measure-Command, but also allows it to be integrated into the pipeline. One thing to note is that if you integrate it into the pipeline the Begin and End portions of the previous command will not be taken into consideration when calculating the Duration.

Another interesting caveat of having the Measure-Block function as part of the pipeline is the order in which parts of commands are processed. For example, with a pipeline as follows:

```powershell
PS> Measure-Block { Get-Process } | Measure-Block { Format-Table}
```

The processing of command would be in this sequence:

  1. Get-Process:Begin
  2. Get-Process:Process
  3. Format-Table:Begin
  4. Format-Table:Process
  5. …
  6. Format-Table:Process
  7. Format-Table:End
  8. Get-Process:End

The second MeasureBlock object would be the child of the first due to the way these commands are processed! `Measure-Block` may be a function that you would only use in very dire circumstances but it is an interesting experiment into the pipeline, function processing and script blocks.

Download the scripts for this article [here](/downloads/advancedscriptprofiling.zip).

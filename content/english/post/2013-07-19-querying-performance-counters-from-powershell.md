---
title: Querying performance counters from PowerShell
author: Tobias Weltner
type: post
date: 2013-07-19T16:00:20+00:00
url: /2013/07/19/querying-performance-counters-from-powershell/
categories:
  - How To
tags:
  - How To

---
Whether you want to know the CPU load of a system or how busy your network is, querying performance counters provides reliable and fast answers. In fact, PowerShell&#8217;s _Get-Counter_ cmdlet makes this task almost trivial. To find out the current CPU load in the past 3 seconds (average), for example, this is all you need:

```
# Sample interval 3 seconds
$load = (Get-Counter "\Processor(_total)\% Processor Time" -SampleInterval 3).CounterSamples.CookedValue

"Average CPU load in the past 3 seconds was $load%"
```

All you need to know is the name of the performance counter, and we&#8217;ll cover that in a second.

### Challenge: Non-US systems

The sample script above will either work beautifully or fail miserably. It simply won&#8217;t run on non-US systems. That&#8217;s right: performance counter names are localized, so they are differently named, depending on the language your computer uses. Crab but reality.

This is a huge strain. A German script will not run in New York, and a French script fails anywhere outside France.

### Replacing performance counter names with ID numbers

There is a clever workaround, though: you can turn language-specific performance counter names into language-neutral ID numbers, then convert these numbers on the target system back into names. This way, you always get the names in the correct language.

Here&#8217;s a  function called _Get-PerformanceCounterLocalName_. When you submit the correct ID number, it returns the localized performance counter name for your system. While the sample code above worked only on US systems, the next sample code will work on any system, regardless of locale:

```
Function Get-PerformanceCounterLocalName
{
  param
  (
    [UInt32]
    $ID,
	$ComputerName = $env:COMPUTERNAME
  )

  $code = '[DllImport("pdh.dll", SetLastError=true, CharSet=CharSet.Unicode)] public static extern UInt32 PdhLookupPerfNameByIndex(string szMachineName, uint dwNameIndex, System.Text.StringBuilder szNameBuffer, ref uint pcchNameBufferSize);'

  $Buffer = New-Object System.Text.StringBuilder(1024)
  [UInt32]$BufferSize = $Buffer.Capacity

  $t = Add-Type -MemberDefinition $code -PassThru -Name PerfCounter -Namespace Utility
  $rv = $t::PdhLookupPerfNameByIndex($ComputerName, $id, $Buffer, [Ref]$BufferSize)

  if ($rv -eq 0)
  {
    $Buffer.ToString().Substring(0, $BufferSize-1)
  }
  else
  {
    Throw 'Get-PerformanceCounterLocalName : Unable to retrieve localized name. Check computer name and performance counter ID.'
  }
}

$processor = Get-PerformanceCounterLocalName 238
$percentProcessorTime = Get-PerformanceCounterLocalName 6

(Get-Counter "\$processor(_total)\$percentProcessorTime" -SampleInterval 1).CounterSamples.CookedValue
```

As you can see, instead of hard-coding localized performance counter names, the code uses _Get-PerformanceCounterLocalName_ with ID numbers 238 and 6 to get the localized names, then uses those to get the performance data.

Which raises the question: Where do these numbers come from? And what other performance counters of interest are there?

### Finding other performance counter names

Let&#8217;s first see how you can find all the other performance counter names, and then, how you can turn them into ID numbers. To see all the performance counters available, you can use this line:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Counter -ListSet * | Select-Object -ExpandProperty Counter
\TBS counters\CurrentResources
\TBS counters\CurrentContexts
\WSMan Quota Statistics(*)\Process ID
\WSMan Quota Statistics(*)\Active Users
\WSMan Quota Statistics(*)\Active Operations
\WSMan Quota Statistics(*)\Active Shells
(...)
</pre>

But beware: it is a huge list (and it may look different on your side, again, depending on your language settings). A better way may be to filter the general category. This would list all performance counters related to _&#8220;processor&#8221;_:

<pre class="brush: powershell; title: ; notranslate" title="">Get-Counter -ListSet *processor* | Select-Object -ExpandProperty Counter
\Processor Information(*)\Processor State Flags
\Processor Information(*)\% of Maximum Frequency
\Processor Information(*)\Processor Frequency
\Processor Information(*)\Parking Status
\Processor Information(*)\% Priority Time
\Processor Information(*)\C3 Transitions/sec
\Processor Information(*)\C2 Transitions/sec
\Processor Information(*)\C1 Transitions/sec
\Processor Information(*)\% C3 Time
\Processor Information(*)\% C2 Time
\Processor Information(*)\% C1 Time
\Processor Information(*)\% Idle Time
\Processor Information(*)\DPC Rate
\Processor Information(*)\DPCs Queued/sec
\Processor Information(*)\% Interrupt Time
\Processor Information(*)\% DPC Time
\Processor Information(*)\Interrupts/sec
\Processor Information(*)\% Privileged Time
\Processor Information(*)\% User Time
(...)
</pre>

Let&#8217;s pick _\Processor Information(*)\Processor Frequency_ counter: on a US system, you could easily query the processor frequency like this:

    PS> Get-Counter "\Processor Information(*)\Processor Frequency" -SampleInterval 1
    Timestamp                 CounterSamples
    ---------                 --------------
    17.07.2013 22:35:00       \\tobiasair1\processor information(_total)\processor frequency
                              :
                              0
    	\\tobiasair1\processor information(0,_total)\processor
          frequency :
          0
    
          \\tobiasair1\processor information(0,3)\processor frequency :
          800
    
          \\tobiasair1\processor information(0,2)\processor frequency :
          800
    
          \\tobiasair1\processor information(0,1)\processor frequency :
          800
    
          \\tobiasair1\processor information(0,0)\processor frequency :
          800

So in your counter name, &#8220;(*)&#8221; stands for &#8220;all counter instances&#8221;, and as you can see, there are two total counts and four additional counts for my four-core-machine, representing each core. If I wanted to monitor a specific core, I could have written:

```
# Sample interval 1 seconds

$freq = (Get-Counter "\Processor Information(0,0)\Processor Frequency" -SampleInterval 1).CounterSamples.CookedValue

"Average CPU frequency in CPU Core 0 in the past second was $freq MHz"
```

Note how I replaced &#8220;(*)&#8221; with &#8220;(0,0)&#8221; in the counter name to monitor my first core CPU.

### Translating performance counter names

In order for the script code to run on any machine, not just US systems, we now need to translate the performance counter names to their corresponding ID numbers. Here&#8217;s how:


    function Get-PerformanceCounterID
    {
        param
        (
            [Parameter(Mandatory=$true)]
            $Name
        )
        if ($script:perfHash -eq $null)
        {
            Write-Progress -Activity 'Retrieving PerfIDs' -Status 'Working'
    
            $key = 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Perflib\CurrentLanguage'
            $counters = (Get-ItemProperty -Path $key -Name Counter).Counter
            $script:perfHash = @{}
            $all = $counters.Count
    
            for($i = 0; $i -lt $all; $i+=2)
            {
               Write-Progress -Activity 'Retrieving PerfIDs' -Status 'Working' -PercentComplete ($i*100/$all)
               $script:perfHash.$($counters[$i+1]) = $counters[$i]
            }
        }
    
        $script:perfHash.$Name
    }
    
    Get-PerformanceCounterID -Name 'Processor Information'
    Get-PerformanceCounterID -Name 'Processor Frequency'
The function _Get-PerformanceCounterID_ can translate the localized name part to a generic ID number. So the two localized strings correspond to ID numbers 1848 and 1884:

```
PS> Get-PerformanceCounterLocalName 1848
Processor Information

PS> Get-PerformanceCounterLocalName 1884
Processor Frequency
```

Now, you can turn the script into a globalized version that runs on any locale:

```
$ProcessorInformation = Get-PerformanceCounterLocalName 1848
$ProcessorFrequency = Get-PerformanceCounterLocalName 1884

$freq = (Get-Counter "\$ProcessorInformation(0,0)\$ProcessorFrequency" -SampleInterval 1).CounterSamples.CookedValue
"Average CPU frequency in CPU Core 0 in the past second was $freq MHz"
```

### Summary

By turning localized performance counter names into ID numbers, working with performance counters finally is possible across different cultures, producing truly language-independent scripts. The trick is to convert the performance counter name into a culture-neutral ID number, then to use this number on the target system to get the correct performance counter name.
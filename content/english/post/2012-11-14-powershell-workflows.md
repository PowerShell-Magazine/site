---
title: PowerShell Workflows
author: Aleksandar Nikolic
type: post
date: 2012-11-14T17:43:51+00:00
url: /2012/11/14/powershell-workflows/
categories:
  - News
tags:
  - News

---
Today we have a real treat for you–an excerpt about the Windows PowerShell Workflow feature from the upcoming book [Learn PowerShell Toolmaking in a Month of Lunches](http://www.manning.com/jones4/)written by Don Jones and Jeffery Hicks.*Workflows are a type of PowerShell command, just as cmdlets and functions are types of commands. One of the easiest ways to understand workflows is to contrast them with their closest cousin: functions. This article, based on chapter 18 of [Learn PowerShell Toolmaking in a Month of Lunches](http://www.manning.com/jones4/), discusses the facets and design guidelines of workflows in PowerShell 3.0.*

[You may also be interested in…](https://www.powershellmagazine.com/2012/11/14/powershell-workflows/#Related)

## PowerShell 3 Workflows

Functions are declared with the *function* keyword; workflows are declared with the *workflow* keyword. Functions are executed by PowerShell itself; workflows are translated to the .NET Framework’s Windows Workflow Foundation (WF) and executed by WF external to PowerShell. Both functions and workflows execute a given set of commands in a specific sequence, but workflows—thanks to WF—include detailed logging and tracking of each and include the ability to retry steps that fail because of, for example, an intermittent network hiccup or other transitory issue. Functions do one thing at a time; workflows can do one thing at multiple times—parallel multitasking. Functions start, run, and finish; a workflow can pause, stop, and restart. If you turn off your computer in the middle of a function, the function is lost; if you do so while a workflow is running, the workflow can potentially be recovered and resumed automatically.

Table 1 illustrates some of the differences between a function and a workflow.

Table 1 Function or workflow

| **Function**                                          | **Workflow**                                           |
| ----------------------------------------------------- | ------------------------------------------------------ |
| Executed by PowerShell                                | Executed by workflow engine                            |
| Logging and retry attempts through complicated coding | Logging and retry attempts part of the workflow engine |
| Single-action processing                              | Supports parallelism                                   |
| Runs to completion                                    | Can run, pause, and restart                            |
| Data loss possible during network problems            | Data can persist during network problems               |
| Full language set and syntax                          | Limited language set and syntax                        |
| Runs cmdlets                                          | Runs activities                                        |

Workflow is incorporated into the shell by running *Import-Module PSWorkflow*; that module extends PowerShell to understand workflows and to execute them properly. Workflows are exposed as commands, meaning you execute them just like commands. For example, if you created a workflow named Do-Something, you’d just run Do-Something to execute it or run *Do-Something –AsJob* to run it in PowerShell’s background job system. Executing a workflow as a job is cool, because you can then use the standard *–Job* cmdlets (like *Get-Job* and *Receive-Job*) to manage them. There are also *Suspend-Job* and *Resume-Job* commands to pause and resume a workflow job.

### Common parameters for workflows

Just by using the *workflow* keyword, you give your workflow command a pretty large set of built-in common parameters. We’re not going to provide an extensive list, but here are some of the more interesting ones (and you can consult PowerShell’s documentation for the complete list):

- –*PSComputerName*—A list of computers to execute the workflow on
- –*PSParameterCollection*—A list of hash tables that specify different parameter values for each target computer, enabling the workflow to have variable behavior on a per-machine basis
- –*PSCredential*—The credential to be used to execute the workflow
- –*PSPersist*—Force the workflow to save (checkpoint) the workflow data and state after executing each step (we’ll show you how you can also do this manually)

There are also a variety of parameters that let you specify remote connectivity options, such as –*PSPort*, –*PSUseSSL*, –PSSessionOption, and so on; these correspond to the similarly named parameters of Remoting commands like *Invoke-Command* and *New-PSSession*.

The values passed to these parameters are accessible as values within the workflow. For example, a workflow can access *$PSComputerName* to get the name of the computer that particular instance of the workflow is executing against right then.

### Activities and stateless execution

Workflow is built around the concept of activities. Each PowerShell command that you run within a workflow is a single, standalone activity.

The big thing to get used to in workflow is that each command, or activity, executes entirely on its own. Because a workflow can be interrupted and later resumed, each command has to assume that it’s running in a completely fresh, brand-new environment. Variables created by one command can’t be used by the next command, which can get a bit weird. Workflow does support an *InlineScript* block, which will execute all commands inside the block within a single PowerShell session. Everything within the block is a standalone script.

Now, this isn’t to say that variables don’t work at all; that would be pretty pointless. For example, consider the script in the following listing (we’ve included this as a numbered listing so that you can run it for yourself in the PowerShell ISE, if you like).

Listing 1 Example workflow with variables

| 12345678910111213141516 | Import-Module PSWorkflow workflow Test-Workflow {   $a = 1  $a   $a++  $a   $b = $a + 2  $b } Test-Workflow |
| ----------------------- | ------------------------------------------------------------ |
|                         |                                                              |

**Try it Now** Run this, and you should see the output 1, 2, and 4, with each number on its own line. That’s the expected output, and seeing that will help you verify that workflow is operating on your system.

Now try the example in this listing.

Listing 2 Example workflow that won’t work properly

| 123456789101112 | Import-Module PSWorkflow workflow Test-Workflow {   $obj = New-Object -TypeName PSObject  $obj \| Add-Member -MemberType NoteProperty `           -Name ExampleProperty `           -Value 'Hello!'  $obj \| Get-Member} Test-Workflow |
| --------------- | ------------------------------------------------------------ |
|                 |                                                              |

This doesn’t produce the intended results, in that the object in *$obj* won’t have an *ExampleProperty* property containing “Hello!” That’s because *Add-Member* runs in its own space, and its modification to *$obj* doesn’t persist to the third command in the workflow. To make this work, we could wrap the entire set of commands as an *InlineScript*, forcing them to all execute at the same time, within a single PowerShell instance. The following listing shows this example.

Listing 3 Example workflow using InlineScript

| 1234567891011121314 | Import-Module PSWorkflow workflow Test-Workflow {   InlineScript {    $obj = New-Object -TypeName PSObject    $obj \| Add-Member -MemberType NoteProperty `             -Name ExampleProperty `             -Value 'Hello!'    $obj \| Get-Member  }} Test-Workflow |
| ------------------- | ------------------------------------------------------------ |
|                     |                                                              |

**Try it Now** Try each of these three examples and compare their results. Workflows do take a big of getting used to, and these simple examples will help you to start understanding workflow’s key differences.

### Persisting state

The state of a workflow consists of its current output, the task that it’s currently executing, and other information. It’s important that you help the workflow maintain this state, especially when kicking off a long-running command that might be executed. To do so, run the *Checkpoint-Workflow* command (or the *Persist* workflow activity). You can force this to happen after every single command is executed by running the workflow with the –*PSPersist* switch.

### Suspending and resuming workflows

A workflow can suspend itself if you run *Suspend-Workflow* within the workflow. You might do this, for example, if you’re about to run some high-workload command that can only be run during a maintenance window. Before running the command, you check the time, and if you’re not in the window, you suspend the workflow. Someone would need to manually resume the workflow (or schedule it in Task Scheduler) by running *Resume-Job* and providing the necessary job ID.

### Inherently remotable

Workflows are designed from the ground up to be remoted, which is why all workflow commands get a *–PSComputerName* parameter automatically. If you run a workflow with one or more computer names, PowerShell connects to the remote computers via Remoting (which must be enabled) and has those computers run the workflow using their local resources. This means the remote computers must also be running PowerShell 3.0. But the following core PowerShell commands always run locally on the machine where the workflow was initiated:

- Add-Member
- Compare-Object
- ConvertFrom-Csv, ConvertFtom-Json, ConvertFrom-StringData
- Convert-Path
- ConvertTo-Csv, ConvertTo-Html, ConvertTo-Xml
- ForEach-Object
- Get-Host
- Get-Member
- Get-Random
- Get-Unique
- Group-Object
- Measure-Command
- Measure-Object
- New-PSSessionOption, New-PSTransportOption
- New-TimeSpan
- Out-Default, Out-Host, Out-Null, Out-String
- Select-Object
- Sort-Object
- Update-List
- Where-Object
- Write-Debug, Write-Error, Write-Host, Write-Output, Write-Progress, Write-Verbose, Write-Warning

These are run locally mainly for performance reasons; if you need one of these to run on a targeted remote computer, wrap them in an *InlineScript{}* block.

### Parallelism

Windows workflow is designed to execute tasks in parallel, and PowerShell exposes that capability through a modified *ForEach* scripting construct and a new *Parallel* construct. They work a bit differently.

With *Parallel*, the commands inside the construct can run in any order. Within the *Parallel* block, you can use the *Sequence* keyword to surround a set of commands that must be executed in order; that batch of commands may begin executing at any point, for example:

| 12345678910111213 | Workflow Test-Workflow {  "This will run first"   parallel {    "Command 1"    "Command 2"     sequence {      "Command A"      "Command B"    }  }} |
| ----------------- | ------------------------------------------------------------ |
|                   |                                                              |

The output here might be

| 1234 | Command 1Command ACommand BCommand 2 |
| ---- | ------------------------------------ |
|      |                                      |

*Command B* will always come after *Command A*, but *Command A* might come first, second, or last—there’s no guarantee. The commands actually execute at the same time, meaning *Command 1*, *Command 2*, and the sequence may all kick off at once, which is what makes the output somewhat nondeterministic. This is useful for when you have several tasks to complete, don’t care about the order in which they run, and want them to finish as quickly as possible.

The parallelized *ForEach* is somewhat different:

| 12345 | Workflow Test-Workflow {  Foreach –parallel ($computer in $computerName) {    Do-Something –computerName $computer  }} |
| ----- | ------------------------------------------------------------ |
|       |                                                              |

Here, WF may launch multiple simultaneous *Do-Something* commands, each targeting a different computer. Execution should be roughly in whatever order the computers are stored in *$ComputerName*, although because of varying execution times the order of the results is nondeterministic.

### General workflow design strategy

It’s important to understand that the entire contents of the workflow get translated into WF’s own language, which only understands activities. With the exception of a few commands, Microsoft has provided WF activities that correspond to most of the core PowerShell cmdlets. That means most of PowerShell’s built-in commands—the ones available before any modules have been imported—work fine.

That isn’t the case with add-in modules, though. Further, because each workflow activity executes in a self-contained space, you can’t even use *Import-Module* by itself in a workflow. You’d basically import a module, but it would then go away by the time you tried to run any of the module’s commands.

The solution is to think of a workflow as a high-level task coordination mechanism. You’re likely to have a number of *InlineScript{}* blocks within a workflow because the contents of those blocks execute as a single unit, in a single PowerShell session. Within an *InlineScript{}*, you can import a module and then run its commands. Each *InlineScript{}* block that you include runs independently, so think of each one as a standalone script file of sorts: Each should perform whatever setup tasks are necessary for it to run successfully.

### Summary

Workflows are an important new feature of PowerShell v3. They’re an incredibly rich, complex technology and a type of tool you can create and make great use of. We discussed the workflow facets and general design strategy.
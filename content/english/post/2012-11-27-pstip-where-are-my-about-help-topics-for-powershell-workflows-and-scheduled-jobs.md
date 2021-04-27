---
title: '#PSTip Where are my About help topics for PowerShell workflows and scheduled jobs?'
author: Aleksandar Nikolic
type: post
date: 2012-11-27T19:00:32+00:00
url: /2012/11/27/pstip-where-are-my-about-help-topics-for-powershell-workflows-and-scheduled-jobs/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

You&#8217;ve updated your PowerShell help using the Update-Help cmdlet and you are pretty sure that you&#8217;ve got help files for PSWorkflow and PSScheduledJob modules as well. If you look into their folders, you can see the help files:

```
PS> echo PSworkflow,PSScheduledJob | foreach { dir $pshome\Modules\$_\en-US }
Directory: C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSworkflow\en-US
Mode                LastWriteTime     Length Name
----                -------------     ------ ----

-a---        10/31/2012  11:23 AM      28323 about_ActivityCommonParameters.help.txt
-a---        10/31/2012  11:23 AM       3221 about_Checkpoint-Workflow.help.txt
-a---        10/31/2012  11:23 AM       3199 about_Foreach-Parallel.help.txt
-a---        10/31/2012  11:23 AM       4655 about_InlineScript.help.txt
-a---        10/31/2012  11:23 AM       1706 about_Parallel.help.txt
-a---        10/31/2012  11:23 AM       3315 about_Sequence.help.txt
-a---        10/31/2012  11:23 AM       4464 about_Suspend-Workflow.help.txt
-a---        10/31/2012  11:23 AM      16166 about_WorkflowCommonParameters.help.txt
-a---        10/31/2012  11:23 AM      12046 about_Workflows.help.txt
-a---         11/6/2012  10:18 AM      61184 Microsoft.PowerShell.Workflow.ServiceCore.dll-help.xml

Directory: C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSScheduledJob\en-US
Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---         9/27/2012  10:44 AM      11695 about_Scheduled_Jobs.help.txt
-a---         9/27/2012  10:44 AM       9913 about_Scheduled_Jobs_Advanced.help.txt
-a---         9/27/2012  10:44 AM      11214 about_Scheduled_Jobs_Basics.help.txt
-a---         9/27/2012  10:44 AM      21071 about_Scheduled_Jobs_Troubleshooting.help.txt
-a---         10/1/2012  12:33 AM     390801 Microsoft.PowerShell.ScheduledJob.dll-help.xml
```

But when you try to output about_WorkflowCommonParameters help topic, for example, you get an error:

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; Get-Help about_WorkflowCommonParameters
Get-Help : Get-Help could not find about_WorkflowCommonParameters in a help file in this session. To download updated help topics type: "Update-Help". To get help online, search for the help topic in the TechNet library at http://go.microsoft.com/fwlink/?LinkID=107116.
</pre>

Why? You are bitten by a bug in the module auto-loading feature implementation. The work around is to import PSWorkflow (or PSScheduledJob) module first:

```
PS> Import-Module PSWorkflow
PS> Get-Help about_*
Name                              Category  Module          Synopsis
----                              --------  ------          --------
...                               ...       ...             ...
...                               ...       ...             ...
about_ActivityCommonParameters    HelpFile                  Describes the parameters that Windows PowerShell
about_Checkpoint-Workflow         HelpFile                  Describes the Checkpoint-Workflow activity, which
about_Foreach-Parallel            HelpFile                  Describes the ForEach -Parallel language construct in
about_InlineScript                HelpFile                  Describes the InlineScript activity, which runs Windows
about_Parallel                    HelpFile                  Describes the Parallel keyword, which runs the
about_Sequence                    HelpFile                  Describes the Sequence keyword, which runs selected
about_Suspend-Workflow            HelpFile                  Describes the Suspend-Workflow activity, which suspends
about_WorkflowCommonParameters    HelpFile                  This topic describes the parameters that are valid on all Windows
about_Workflows                   HelpFile                  Provides a brief introduction to the Windows
```

Voilà! You can now enjoy all the knowledge packed in these About help topics.
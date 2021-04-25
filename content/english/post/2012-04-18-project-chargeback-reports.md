---
title: Project Chargeback Reports
author: Luc Dekens
type: post
date: 2012-04-18T18:00:50+00:00
url: /2012/04/18/project-chargeback-reports/
views:
  - 7989
post_views_count:
  - 1347
categories:
  - PowerCLI
  - VMware
tags:
  - VMware
  - PowerCLI

---
The following is a typical scenario that you are bound to see in your environment. Your organization has several projects running, and each of these projects uses a number of virtual machines in your VMware vSphere environment. Management wants you to send a daily informational email to each project leader, with an overview consumed resources for his project(s).

PowerShell to the rescue!

### Getting the stats

The [PowerCLI][1] snap-in provides a cmdlet called [Get-Stat][2] that will provide us with the statistical data from the vCenter database. Based on a _Custom Attribute_, called _Project_, that is present on each machine that belong to a specific project, the script will first select all the Virtual Machines.

![](/images/Luc_01.png)

Once the selection of the Virtual Machines is made, the script will retrieve the statistical data. To limit the load on the vCenter, we only use one _Get-Stat_ call in the script. With some nested _Group-Object_ calls, the script produces a report per project owner, detailing all virtual machines in each of his projects.

To keep the example script simple, we only retrieve two metrics. But it should be obvious that you can easily expand the script to retrieve and calculate numerous other metrics.

### Who to send it to?

The _ProjectOwner_ custom attribute contains an Active Directory _Display Name_ value. With an AdsiSearcher call, the script can easily find the email address of the user and send him the report by email.

```powershell
$dcName = "TEST"
$displayName = $projectOwner
$adDomain = [adsi]("LDAP://" + $dcName + ":389/dc=mycorp,dc=com")
$adSearch = [adsisearcher]$adDomain
$adSearch.Filter = '(&(objectClass=User)(name=' + $displayName + '))'
$result = $adSearch.FindOne()
$mailAddress = $result.Properties["mail"][0]
```

With the [Quest’s AD snapin][3] the code required to find the email address becomes a lot easier.

```powershell
$user = Get-QADUser -DisplayName $projectOwner
$emailAddress = $user.Email
```

Management wants the report , that is send to the Project Owner, to look something like this

![](/images/Luc_02.png)

To avoid cluttering the script, the report shows the data in its simplest form. You can of course go out of your way and send the report as an HTML mail, using style sheets and all.

Find the complete reporting script as file [_vcProjectStats.ps1_][4].

### Schedule the task

We want to schedule the report task to run _every day_ at _02:00 AM_. This can be accomplished quite easily with the _TaskScheduler_ module that comes with the [PowerShellPack][5] .

Before installing the PowerShellPack make sure to “Unblock” the MSI file.  And make sure to [correct the typo][6] in the Register-ScheduleTask code before you use the module.

The available functions in the TaskScheduler module give access to most of the [Task Scheduler 2.0][7] features. But for our purposes, one important feature was missing, the ability to run the scheduled task under a well-defined account.

To solve that problem I created a function, called _Set-TaskPrincipal_. This function will fill in the required [Principal][8]  properties. The password for the account is passed to the Task Scheduler during the registration of the task, in the Register-ScheduledTask function.

![](/images/Luc_03.png)

The script to create the Scheduled Task can be found as attachment [_Create-ScheduledTask.ps1_][4]

### Conclusion

For those that are not yet convinced, I think that these small scripts show, once again, how easy it is to use PowerShell to solve your day-to-day task as an administrator.

The scripts demonstrate

  * the flexibility to use 3-party snapins in your scripts
  * the availability of numerous modules to solve your problems
  * the ease to expand on existing modules
  * the richness of the core PowerShell language

And as always, there is no single, correct script when using PowerShell. Feel  free to send in your improvement suggestions or different methods to tackle the problem.

Happy scripting !

&nbsp;

[1]: http://communities.vmware.com/community/vmtn/server/vsphere/automationtools/powercli?view=overview
[2]: http://www.vmware.com/support/developer/PowerCLI/PowerCLI50/html/Get-Stat.html
[3]: http://www.quest.com/powershell/activeroles-server.aspx
[4]: http://104.131.21.239/wp-content/scripts/ProjectChargebackReports.zip
[5]: http://archive.msdn.microsoft.com/PowerShellPack
[6]: http://archive.msdn.microsoft.com/PowerShellPack/Thread/View.aspx?ThreadId=3524
[7]: http://msdn.microsoft.com/en-us/library/aa383614%28v=VS.85%29.aspx
[8]: http://msdn.microsoft.com/en-us/library/aa382071%28v=VS.85%29.aspx
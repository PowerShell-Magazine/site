---
title: How PowerShell saved the day – Jan Egil Ring
author: Jan Egil Ring
type: post
date: 2015-03-10T16:00:36+00:00
url: /2015/03/10/how-powershell-saved-the-day-jan-egil-ring/
views:
  - 9896
post_views_count:
  - 1249
categories:
  - How To
tags:
  - How To

---
We&#8217;re now kicking off a new series on PowerShell Magazine called “How PowerShell saved the day”, where the intent is to share success stories where PowerShell was a major factor when solving a problem.

My story is about a file cluster, which was modified by a virus that encrypted files due to a user opening an e-mail attachment. Of course there was a virus-protection in place both on the e-mail server, the client computer, and the file servers, but the virus definition files didn't include this specific virus yet.

Shortly after the incident was reported, we gathered some facts:

  * The file types encrypted were Microsoft Office documents (typically .docx and .xlsx)
  * The encrypted documents was renamed with the original filename + a 7 digit random extension, such as Report.docx.dwqskfj

The encrypted files was quickly identified using regular PowerShell filtering:

```powershell
dir '\\domain.local\Public' -Filter "*dwqskfj" -Recurse -File | Select-Object -Property name,directory,fullname | Export-Csv -Path C:\temp\infected.csv -NoTypeInformation -Encoding Unicode -Delimiter ';'
```


At this point we didn&#8217;t know whether more files than those we had identified were encrypted, so we looked for other files with an extension greater than 4 with a high count:

```powershell
dir '\\domain.local\Public' -Recurse -File | Where-Object {$_.Extension.Length -gt 4} | Group-Object -Property extension | sort count -Descending
```


Most files returned from the results were verified as normal files, but we did identify one more file extension which was identified as encrypted files. When contacting the user who owned these files, the user acknowledged having opened a similar e-mail attachment as well.

Now that the files were identified (about 5-6000), the next task was to recover them. Restoring such a large number of files manually was not an option. Restoring all files back to the last recovery point was not desired either since that would mean a lot of work users had performed that day would get lost.

Restoring the entire file cluster data to an alternate location in order to script the recovery of the encrypted files from there would work, but that would be too time consuming when there was many TBs of data.

Luckily, the backup software being used&#8211;Microsoft System Center DPM&#8211;has full PowerShell support. Creating a PowerShell script to perform the recovery was an affordable task, which was completed against a number of test-files in a reasonable amount of time:

![](/images/How_PS_saved_the_day_JER.png)

This script is also available on [GitHub][1].

Note that there is room for improvement in this script. But the point in this case is that you can get the job done in an efficient manner when “your hair is on fire”. Improving the script with things such as removing aliases, adding error handling, logging and so on is something that can be added later in order to reuse it in case of a similar event.

The restore operation was fully completed after a few hours and the encrypted files was moved off to a temporary location until the recovery operation was verified to be 100% successful.

If you have a story to share on how PowerShell saved your day, please [submit an article pitch][2] to us.

[1]: https://github.com/janegilring/PSCommunity/blob/master/Microsoft%20System%20Center/Data%20Protection%20Manager/Restore-DPMRecoverableItem.ps1
[2]: /write-for-us/
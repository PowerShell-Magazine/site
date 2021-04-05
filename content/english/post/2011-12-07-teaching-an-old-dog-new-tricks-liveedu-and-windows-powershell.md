---
title: Teaching an old dog new tricks – Live@Edu and Windows PowerShell
author: Daniel Soares
type: post
date: 2011-12-07T19:00:12+00:00
url: /2011/12/07/teaching-an-old-dog-new-tricks-liveedu-and-windows-powershell/
views:
  - 8005
post_views_count:
  - 1569
categories:
  - Learning PowerShell
tags:
  - Learning PowerShell

---
By way of introduction, I am the Software Services Manager at Patrick Henry College, a Christian liberal arts school in Virginia, USA. I have about 25 years in IT, the last 15 years primarily spent developing collaborative applications in IBM Lotus Notes. Over the last 6 years, I expanded my skill set to include web technologies, MS SQL, SSIS and SSRS and now spend a good deal of my time on database administration.

Our student email used to run on a Mirapoint appliance until about two years ago when we moved to a hosted solution with Live@Edu. I was tasked with automating the account creation and synchronization of data between our MSSQL based Student Information System and Live@Edu. This was my introduction to PowerShell. Being familiar with scripting languages, it was a question of learning the syntax.

Since I had already deployed a few solutions using SQL Server Integration Services, I set out to create an SSIS project that would create the .csv file from the Student Information System (SIS) and use that file as the input for the PowerShell script. Microsoft provides a sample script (CSV_Parser.ps1) and a sample .csv file that can be downloaded from <http://go.microsoft.com/fwlink/?LinkID=142060>. Using this sample script, with some slight modifications, I was able to automate the account creation and also set up a PowerShell script job that ran on a periodic basis to synchronize account information between the SIS and Live@Edu. I also wrote PowerShell scripts to create and populate distribution groups each semester based on enrollment, gender and dorm. Another script was to import the faculty/staff Directory from IBM Lotus Domino into the Live@Edu External Contacts. As our staff are added or deleted in Lotus, the external contacts on the Live@Edu get updated by a scheduled SQL server job which calls a corresponding PowerShell script.

More recently, we had the need to create an alumni domain, set up an alumni alias for the student account if the student record in the SIS had the student as having graduated from Patrick Henry College and finally make that alias the Primary SMTP address for the student (while preserving any other proxy addresses that might have existed). Most of you PowerShell gurus would probably find this quite elementary, but for the newbies, this is what I ended up doing.

At first I was attempting to use the -PrimarySMTPAddress like this:

```powershell
$Temp = Get-Mailbox -Identity testuser
$Temp.EmailAddresses += 'testuser@alumni.phc.edu'
Set-Mailbox $Temp.Name -EmailAddresses
$Temp.EmailAddresses
Set-Mailbox $Temp.Name -PrimarySmtpAddress 'testuser@alumni.phc.edu'
```


Shay Levy, who is a PowerShell MVP was kind enough to help me troubleshoot the error I kept getting when I used this script and we ended up determining that -PrimarySMTPAddress is not a supported cmdlet parameter. To determine the list of supported cmdlet parameters, I ran the following command:

```powershell
PS> (Get-Command Set-Mailbox).Parameters.Keys
```

Microsoft gives this example on how to change the primary email address of an existing user:

```powershell
PS> Set-Mailbox <Identity> -EmailAddresses SMTP:<new primary e-mail address>,<user ID>,<existing proxy address 1>,<existing proxy address 2>
```

So I modified the original script to:

```powershell
$Temp = Get-Mailbox -Identity testuser
$Temp.EmailAddresses += 'SMTP:testuser@alumni.phc.edu'
Set-Mailbox $Temp.Name -EmailAddresses $Temp.EmailAddresses
```


This set the alumni address as the primary, retained the proxy address that was in place and added the old student.phc.edu address to the proxy address list. The final change I had to make was to set up the script to use my .csv file as input and dynamically pass values.

```powershell
foreach($line in $csv)
{
    $EmailAddresses = @{Name='EmailAddresses';Expression={($_.EmailAddresses -cmatch 'smtp') -join ';'}}
    $Emailadd = Get-Mailbox $Line.UserID | Select-Object Name, Alias, PrimarySmtpAddress, $EmailAddresses            
    if ($Emailadd.PrimarySmtpAddress -notlike '*@alumni.phc.edu')
    {
        # add a second email address (alias) for these students in Outlook who
        # have graduated from PHC and do not have an alumni.phc.edu alias
        # and make the alumni.phc.edu account the primary email address, 
        # while retaining any proxy addresses that might exist.
        $AlumniAddress = $line.NewAddress
        $NewPrimaryAddress = 'SMTP:'+$line.NewAddress
        $StudentAddress=$line.UserID
        $Temp = Get-Mailbox -Identity $Line.Name
        $Temp.EmailAddresses += $NewPrimaryAddress
        Set-Mailbox $Temp.Name -EmailAddresses $Temp.EmailAddresses
    }
}
```

Not too hard once you figure out the right parameters to use.

I am very grateful to Karl Mitschke, whose blog at <http://unlockpowershell.wordpress.com> is extremely useful and also to Shay Levy who has his blog at <http://PowerShay.com>

More resources can be found at:

<http://social.technet.microsoft.com/wiki/contents/articles/windows-powershell-survival-guide.aspx>

<http://technet.microsoft.com/en-us/scriptcenter/dd742419>

If your student email is being hosted at Live@Edu, this resource will be very useful to you:

<http://blogs.technet.com/b/educloud/archive/tags/powershell/>

Since I had a background programming with LotusScript and JavaScript, it was not that hard to learn PowerShell. With the help of the numerous examples available on the web, I was able to quickly come up with functional PowerShell code. Perhaps it would have been even easier if Live@Edu provided more resources on using PowerShell that are specific to their Exchange environment. I encourage those of you who are new to PowerShell to look into using it. It is extremely powerful and easy to deploy, especially in conjunction with SQL Server. I hope you have benefited from this article and that it encourages you to give PowerShell a test drive.

Would you like to share your story? Win one of [TrainSignal’s][1] PowerShell training courses? Learn more about the “[How I Learned to Stop Worrying and Love Windows PowerShell][2]” contest.

[1]: http://www.trainsignal.com/default.aspx
[2]: /2011/11/29/call-for-writers-share-your-experiences-and-help-new-users/
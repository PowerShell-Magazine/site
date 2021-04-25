---
title: Using SkyDrive to sync your WindowsPowerShell folder
author: Claus Nielsen
type: post
date: 2012-10-16T16:00:54+00:00
url: /2012/10/16/using-skydrive-to-sync-your-windowspowershell-folder/
categories:
  - How To
tags:
  - How To

---
I while back I was asked if it is possible to use SkyDrive to sync the WindowsPowerShell folder between multiple machines. Not using SkyDrive at that time myself I did not know if it was possible, but I assumed it was.

I set out to test it, I’ve installed SkyDrive on my main Windows 7 workstation and a test machine, and then logged in with my LiveID on both machines. After installation, SkyDrive created a SkyDrive folder under C:\Users\%UserName%\. My initial thought was to go into that folder and create a symlink pointing to C:\Users\%UserName%\Documents\WindowsPowerShell.

I started an elevated command prompt, navigated to C:\Users\%UserName%\SkyDrive and did:

![](/images/SkyDriveSync1.png)

<pre class="brush: powershell; title: ; notranslate" title="">mklink /d "WindowsPowerShell" C:\Users\%Username%\Documents\WindowsPowershell
</pre>

![](/images/SkyDriveSync2.png)

That created a symlink in the SkyDrive folder that pointed to C:\Users\%UserName%\Documents\WindowsPowerShell. I went to the test machine, but nothing was  <br clear="ALL" />synced over.  From this test I could deduce that SkyDrive did not follow symlinks to sync data.

My next test was to “reverse” my steps and try to create a “WindowsPowerShell” symlink in C:\Users\%Username%\Documents\ and point that to the SkyDrive folder.

In my C:\Users\%UserName%\SkyDrive folder I created two folders \Documents\WindowsPowerShell, so I ended up with C:\Users\%UserName%\SkyDrive\Documents\WindowsPowerShell

![](/images/SkyDriveSync3.png)

In the elevated cmd prompt I navigated to C:\Users\%Username%\Documents\ and tried to do:

<pre class="brush: powershell; title: ; notranslate" title="">mklink /d "WindowsPowerShell" C:\Users\%UserName%\skydrive\Documents\WindowsPowershell
</pre>

That failed with an error saying: Folder already exists.

![](/images/SkyDriveSync4.png)

Which makes sense, since you cannot have a symlink and a folder with the same name, so I copied the contents from C:\Users\%Username%\Documents\WindowsPowershell to C:\Users\%UserName%\skydrive\Documents\WindowsPowerShell

I renamed the WindowsPowerShell folder in C:\Users\%Username%\Documents\WindowsPowerShell to WindowsPowerShellOld, and ran the following command again. It completed without any errors.

![](/images/SkyDriveSync5.png)

<pre class="brush: powershell; title: ; notranslate" title="">mklink /d "WindowsPowerShell" C:\Users\%UserName%\skydrive\Documents\WindowsPowerShell
</pre>

Now in my Documents folder I have a symlink named WindowsPowerShell pointing to C:\Users\%UserName%\SkyDrive\Documents\WindowsPowerShell where the actual WindowsPowerShell folder containing my profile.ps1 and my modules is located.

### So to summarize

  * Install SkyDrive on both machines.

  * On your main machine, create two folders in your SkyDrive folder, one called Documents and beneath that one called WindowsPowerShell, you should see an empty folder synced to the second machine. (You could name it anything you like)

  ```
  Mkdir C:\Users\%UserName%\SkyDrive\Documents\WindowsPowerShell
  ```

  * Trick Windows to go look for profiles/modules etc, in %Username%\SkyDrive\Documents\WindowsPowerShell instead of %username%\Documents\WindowsPowerShell. To do that, create a symlink called &#8220;WindowsPowerShell&#8221; in %username\Documents pointing to %Username%\SkyDrive\Documents\WindowsPowerShell.But since you already run Powershell and have a folder called   %Username%\Documents\WindowsPowerShell it will fail. So you have to rename that folder, and let mklink create the symlink called &#8220;WindowsPowerShell"

  ```
  robocopy C:\Users\%UserName%\Documents\WindowsPowerShell C:\Users\%UserName%\SkyDrive\Documents\WindowsPowerShell  /MIR
  
  ren C:\Users\%UserName%\Documents\WindowsPowerShell  C:\Users\%UserName%\Documents\ WindowsPowerShellOLD
  
  mklink /d "C:\Users\%UserName%\Documents\WindowsPowerShell\WindowsPowerShell1" C:\Users\%UserName%\skydrive\Documents\WindowsPowerShell
  ```

* Now you have what appears to be a folder in %Username%\Documents called WindowsPowerShell, but is actually a symlink redirecting you to the Documents\WindowsPowerShell folder in you SkyDrive folder.

* Test it. Create a folder/file on one machine and you should see it synced on the other machine.

I tested that PowerShell worked, by seeing that my profile got loaded, and I had access to my modules. Then I did the same on the test machine, and voila I had set up sync between my two machines. To test that it actually worked, I tried altering a few lines in my profiles.ps1 on the test machine, it was almost instantaneously synced to my primary machine.

A few words of caution if you are doing this as well, if you delete a file on one machine, it will be deleted everywhere, if you change a file it will changed everywhere.. So before you go and sync your folders, make sure you continuously backup your WindowsPowerShell folder somewhere outside of SkyDrive’s reach.
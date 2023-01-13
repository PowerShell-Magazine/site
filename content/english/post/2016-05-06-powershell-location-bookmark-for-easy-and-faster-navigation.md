---
title: PowerShell Location Bookmark for Easy and Faster Navigation
author: Ravikanth C
type: post
date: 2016-05-06T16:00:16+00:00
url: /2016/05/06/powershell-location-bookmark-for-easy-and-faster-navigation/
views:
  - 14725
post_views_count:
  - 4560
categories:
  - Module Spotlight
tags:
  - Modules

---
I work at the command line most of the time and sometimes I find moving to certain folders very tedious and requires lot of typing (even with tab-completion). For example, one of the locations (may not be very frequent) that I move to when working at PowerShell console or ISE is my local GitHub folder where all my repositories are stored. Another example could be the configuration folder in C:\Windows\System32. At times, this is frustrating. So, I wanted something simple but useful for me to just type as few characters as possible but still let me navigate to the place I want to.

So, this is when I started writing out something quick and added a few functions to my Profile script. And, then I tweeted that! 🙂

{{< tweet user="ravikanth" id="728151607762817024" >}}

I saw a few suggestions as a response to this.

### PSDrives

One suggestion was to use the _New-PSDrive_ cmdlet and add one for each location I want to quickly move to. This is not very intuitive to me. I can map something like _Conf:_ to go to C:\Windows\System32\Configuration but I don&#8217;t get tab-completion. I always have to type the full drive name like _cd conf:_. You can tab-complete within the drive but not the drive itself.

### Jump.Location or ZLocation modules

These modules are very good. It learns your usage and then auto-completes the path based on the history! This is good if you are frequently accessing a few folders. However, this is not my case, exactly. I wanted an easier way to navigate to a longer path and of course, at the same time make it easy for me to get into some of the folders that I use frequently.

So, here it is: [PSBookmark][3]!

### PSBookmark

This module is a very simple module that provides four commands to manage location bookmarks in PowerShell. This has been very useful for me and while I don&#8217;t see me investing lot of time in this, I will certainly implement any feedback or suggestions you may have.

### How do you get this?

Simple! Either clone my Github repo or get it from [PowerShell Gallery][5]!

```powershell
Install-Module -Name PSBookmark
```

### How to use this?

Since the idea is to make it easy to navigate and avoid lot of typing, I will use aliases instead of the full function names. I had a hard time coming up with the function names but aliases were very easy.

```powershell
PS C:\> Get-Command -Module PSBookmark
CommandType     Name                                               Version    Source                 -----------     ----                                               -------    ------
Function        Get-LocationBookmark                               1.0.0      PSBookmark             Function        Remove-LocationBookmark                            1.0.0      PSBookmark             Function        Save-LocationBookmark                              1.0.0      PSBookmark             Function        Set-LocationBookmarkAsPWD                          1.0.0      PSBookmark             

PS C:\> Get-Command -Module PSBookmark -CommandType Alias
CommandType     Name                                               Version    Source                 
-----------     ----                                               -------    ------
Alias           glb -> Get-LocationBookmark                        1.0.0      PSBookmark           
Alias           goto -> Set-LocationBookmarkAsPWD                  1.0.0      PSBookmark
Alias           rlb -> Remove-LocationBookmark                     1.0.0      PSBookmark           
Alias           save -> Save-LocationBookmark                      1.0.0      PSBookmark
```

### Create a New Location Alias

You can use the _save_ command to save alias for either the PWD or a specific path.

```powershell
#This will save $PWD as scripts
save scripts 

#This will save C:\Documents as docs
save docs C:\Documents
```

### Jump or Goto a Saved Location Alias

```powershell
#You don't have to type the alias name. Instead, you can just tab complete. This function uses dynamic parameters.
goto docs
```


### Get All Saved Alias Locations

```powershell
PS C:\> glb
Name                           Value                                                                 ----                           -----
docs                           C:\Documents                                                           
oss                            C:\Documents\Github
```

### Remove a Saved Alias Location

```powershell
#You don't have to type the alias name. Instead, you can just tab complete. This function uses dynamic parameters.
rlb docs
```


### TODO

  * This was written in just a few minutes and did not spend any time polishing it. So, there is certainly scope for improvement.
  * At the moment, the aliases are stored as a hash table in a .PS1 file in $env:UserProfile. This may change in future.

[1]: https://github.com/rchaganti/PSBookmark#psdrives
[2]: https://github.com/rchaganti/PSBookmark#jumplocation-or-zlocation-modules
[3]: https://github.com/rchaganti/PSBookmark
[4]: https://github.com/rchaganti/PSBookmark#how-do-you-get-this
[5]: http://www.powershellgallery.com/packages/PSBookmark
[6]: https://github.com/rchaganti/PSBookmark#how-to-use-this
[7]: https://github.com/rchaganti/PSBookmark#create-a-new-location-alias
[8]: https://github.com/rchaganti/PSBookmark#jump-or-goto-a-saved-location-alias
[9]: https://github.com/rchaganti/PSBookmark#get-all-saved-alias-locations
[10]: https://github.com/rchaganti/PSBookmark#removed-a-saved-alias-location
[11]: https://github.com/rchaganti/PSBookmark#todo
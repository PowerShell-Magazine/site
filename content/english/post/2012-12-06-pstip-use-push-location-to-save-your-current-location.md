---
title: '#PSTip Use Push-Location to save your current location'
author: Jakub Jareš
type: post
date: 2012-12-06T19:00:40+00:00
url: /2012/12/06/pstip-use-push-location-to-save-your-current-location/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Going through folders, you sometimes find yourself in some strangely named ones you are sure you&#8217;ll have to revisit again. To avoid searching for the folders after some digging around,  use the pushd (Push-Location) command to save your current location to stack.

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\Windows\System32\DriverStore\FileRepository\brmfcmdm.inf_x86_neutral_3b38c2e8e6f06c1b&gt; pushd
</pre>

Or to save the location and go one folder up in the folder structure do:

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\Windows\System32\DriverStore\FileRepository\brmfcmdm.inf_x86_neutral_3b38c2e8e6f06c1b&gt; pushd ..
</pre>

Now you can dig around how you want. For example go check the driver setup logs in the inf folder. After you are finished, use popd (Pop-Location) to go back to where you were.

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\windows\inf&gt; popd
PS C:\Windows\System32\DriverStore\FileRepository\brmfcmdm.inf_x86_neutral_3b38c2e8e6f06c1b&gt;
</pre>
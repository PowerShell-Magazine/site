---
title: '#PSTip Explore all possible -Verb values for registered file extensions'
author: David Moravec
type: post
date: 2012-12-26T19:00:56+00:00
url: /2012/12/26/pstip-explore-all-possible-verb-values-for-registered-file-extensions/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In previous tips we saw how to [find possible values of -Verb parameter][1] and [how to use it][2]. What about to check all possible _-Verb_ values for all extensions?

First, let’s investigate all registered (associated) extensions in the system. There is a native tool named [assoc][3]. As it’s a part of cmd.exe, we have to call it inside command line. If it’s used without a parameter, it will display all associations.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; cmd /c assoc | Select-Object -First 10
.323=h323file
.386=vxdfile
.3g2=VLC.3g2
.3gp=VLC.3gp
.3gp2=VLC.3gp2
.3gpp=VLC.3gpp
.5vw=wireshark-capture-file
.7Z=WinZip
.aac=aacfile
.aca=Agent.Character.2
</pre>

Let’s test the whole process with the first extension returned. We need to split input line to receive just that extension.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; cmd /c assoc | Select-Object -First 1 | ForEach { ($_ -split '=')[0] }
.323
</pre>

We used [Split operator][4] to obtain just the text before the equal sign. Now we’ll use [previous tip][1] to see all possible _-Verb_ values.

```
cmd /c assoc |
Select-Object -First 1 |
ForEach { ($_ -split '=')[0] } |
ForEach { (New-Object System.Diagnostics.ProcessStartInfo -ArgumentList "test$_").Verbs }
open
```

We created a new _ProcessStartInfo_ object and passed a dummy file name to it – in this case named _test.323_. From the object we created, we extracted only its _Verbs_ property. As we don’t need to create an object from the output, it’s sufficient to send the output to _Out-GridView_ cmdlet. But before that, let’s remove double call of _ForEach-Object_ cmdlet.

```
cmd /c assoc |
Select-Object -First 10 |
ForEach { $ext = ($_ -split '=')[0]; "{0}: {1}" -f $ext, ((New-Object System.Diagnostics.ProcessStartInfo -ArgumentList "test$ext").Verbs -join ', ') }

.323: open
.386:
.3g2: AddToPlaylistVLC, Open, PlayWithVLC
.3gp: AddToPlaylistVLC, Open, PlayWithVLC
.3gp2: AddToPlaylistVLC, Open, PlayWithVLC
.3gpp: AddToPlaylistVLC, Open, PlayWithVLC
.5vw: open
.7Z: open, print
.aac: Batch Convert with WavePad Sound Editor, Convert sound file, Edit sound file, Edit with WavePad Sound Editor
.aca:
```

We used PowerShell’s _format_ operator (_-f_) to have output formatted like: ext: _Verb1, Verb2,_ … Now we can remove _Select-Object_ and send all data to the _Out-GridView_:

<pre class="brush: powershell; title: ; notranslate" title="">cmd /c assoc |
ForEach { $ext = ($_ -split '=')[0]; "{0}: {1}" -f $ext, ((New-Object System.Diagnostics.ProcessStartInfo -ArgumentList "test$ext").Verbs -join ', ') } | Out-GridView -Title 'Verb values for associated extensions'
</pre>

![](/images/psassoc.png)

[1]: /2012/12/12/pstip-explore-values-that-can-be-used-with-start-processs-verb-parameter/
[2]: /2012/12/13/pstip-print-a-word-document-using-start-process-cmdlet/
[3]: http://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/assoc.mspx?mfr=true
[4]: http://technet.microsoft.com/en-us/library/hh847811.aspx
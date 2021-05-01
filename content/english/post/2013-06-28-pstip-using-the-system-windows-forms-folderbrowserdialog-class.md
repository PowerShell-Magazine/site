---
title: '#PSTip: Using the System.Windows.Forms.FolderBrowserDialog Class'
author: Jaap Brasser
type: post
date: 2013-06-28T20:00:58+00:00
url: /2013/06/28/pstip-using-the-system-windows-forms-folderbrowserdialog-class/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In PowerShell, it is possible to use GUI elements to allow for user input during scripts. Although it is possible to create your own forms from scratch, there are also many useful pre-built dialogs available. In this tip, we will see how the _System.Windows.Forms.FolderBrowserDialog_ can be used to select a folder or path that can be used within a script.

On the first line we add the System.Forms class. This is to ensure the assembly is loaded. The _New-Object_ Cmdlet creates the actual form and _ShowDialog()_ Method on the object invokes the folder browser dialog.

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName System.Windows.Forms
$FolderBrowser = New-Object System.Windows.Forms.FolderBrowserDialog
[void]$FolderBrowser.ShowDialog()
$FolderBrowser.SelectedPath
</pre>

![](/images/BrowseForFolder.png)

It is also possible to specify in which folder the folder browser starts. By default this is the user’s desktop but this can be changed by modifying the SelectedPath property:

```
Add-Type -AssemblyName System.Windows.Forms
$FolderBrowser = New-Object System.Windows.Forms.FolderBrowserDialog -Property @{
    SelectedPath = 'C:\Temp’
}

[void]$FolderBrowser.ShowDialog()
$FolderBrowser.SelectedPath
```

To limit the folders the RootFolder property can be utilized, it should be noted that only special folders can be entered here, a folder of the [System.Environment+SpecialFolder] type. This means a custom path cannot be entered here. In the next code sample I have also disable the New Folder property, this is to ensure a user cannot create a folder and then select that folder:

```
Add-Type -AssemblyName System.Windows.Forms

$FolderBrowser = New-Object System.Windows.Forms.FolderBrowserDialog -Property @{
    RootFolder = 'MyDocuments'
    ShowNewFolderButton = $false
}

[void]$FolderBrowser.ShowDialog()
$FolderBrowser.SelectedPath
```

To get a full list of the available special folders the [enum] accelerator can be used to enumerate these folders. This code will generate list of the available values for the RootFolder property:

<pre class="brush: powershell; title: ; notranslate" title="">[Enum]::GetNames([System.Environment+SpecialFolder])
</pre>

To learn more about this class or the values in the RootFolder property the following two MSDN articles can be used:

<http://msdn.microsoft.com/en-us/library/system.windows.forms.folderbrowserdialog.aspx>

<http://msdn.microsoft.com/en-us/library/system.environment.specialfolder(v=vs.95).aspx>
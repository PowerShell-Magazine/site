---
title: '#PSTip Using the System.Windows.Forms.OpenFileDialog Class'
author: Jaap Brasser
type: post
date: 2013-07-01T18:00:31+00:00
url: /2013/07/01/pstip-using-the-system-windows-forms-openfiledialog-class/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

In PowerShell, it is possible to use GUI elements to request user input. Although it is possible to create your own forms from scratch, there are also many useful pre-built dialogs available. In this tip, I will show you how to use the _System.Windows.Forms.OpenFileDialog_ to select one or multiple files.

The following code will open a window that will prompt the user to select a single file. By setting the InitialDirectory property, the starting directory will be set to the current user’s desktop. This is done by using the [Environment] Desktop special folder:

```
Add-Type -AssemblyName System.Windows.Forms
$FileBrowser = New-Object System.Windows.Forms.OpenFileDialog -Property @{
    InitialDirectory = [Environment]::GetFolderPath('Desktop')
}

[void]$FileBrowser.ShowDialog()
$FileBrowser.FileNames
```

![](/images/FileBrowser.png)

If documents need to be selected, it can be useful to set the starting folder to the documents folder. By setting a filter, we can ensure that only a certain type of file is selected. The next code sample will allow users to select .docx files. The filter can be changed by the user to also select an xlsx file:

<pre class="brush: powershell; title: ; notranslate" title="">Add-Type -AssemblyName System.Windows.Forms
$FileBrowser = New-Object System.Windows.Forms.OpenFileDialog -Property @{
    InitialDirectory = [Environment]::GetFolderPath('MyDocuments')
    Filter = 'Documents (*.docx)|*.docx|SpreadSheet (*.xlsx)|*.xlsx'
}
[void]$FileBrowser.ShowDialog()
$FileBrowser.FileNames
</pre>

To select multiple files the MultiSelect property should be set to True.

```
Add-Type -AssemblyName System.Windows.Forms

$FileBrowser = New-Object System.Windows.Forms.OpenFileDialog -Property @{
    Multiselect = $true
}

[void]$FileBrowser.ShowDialog()
$FileBrowser.FileNames
```

For more information about this class the following MSDN article can be used:

<http://msdn.microsoft.com/en-us/library/system.windows.forms.openfiledialog.aspx>
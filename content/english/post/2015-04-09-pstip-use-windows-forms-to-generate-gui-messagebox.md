---
title: 'PSTip: Use Windows Forms to generate GUI messagebox'
author: Jaap Brasser
type: post
date: 2015-04-09T18:00:56+00:00
url: /2015/04/09/pstip-use-windows-forms-to-generate-gui-messagebox/
views:
  - 25009
post_views_count:
  - 5195
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
While it is great that PowerShell can do so many things in the background without having to interact with it, occasionally it might be useful to have a graphical notification. For example, when a job finishes or when input is required in order for the script to be able to continue. For this purpose a popup window can be displayed; this is possible using the Windows.Forms namespace and in particular the MessageBox class.

The following example will display a MessageBox form with an OK button and the ‘Hello World’ text:

```powershell
Add-Type -AssemblyName System.Windows.Forms | Out-Null
[System.Windows.Forms.MessageBox]::Show("Hello World")
```


The MessageBox class can accept more arguments than just a single string. To explore the possible constructors the Show method can be called without any parameters displaying all OverloadDefinitions. In the following code the OverloadDefinitions and the total count of possible definitions will be displayed:

```powershell
[System.Windows.Forms.MessageBox]::Show
[System.Windows.Forms.MessageBox]::Show.OverloadDefinitions.Count
```


The output from the previous command shows that it is possible to change the MessageBoxButtons and that it is possible to add a MessageBoxIcon. The GetNames method of the Enum class can be called to enumerate the possible entries for these options:

```powershell
[Enum]::GetNames([System.Windows.Forms.MessageBoxButtons])
[Enum]::GetNames([System.Windows.Forms.MessageBoxIcon])
```


The parameters for Title, Caption, MessageBoxButtons and MessageBoxIcon will be specified to create a new MessageBox. The output will be stored in the $MessageBox variable and can be used for further execution in the script:

```powershell
$MessageBox = [System.Windows.Forms.MessageBox]::Show(
    'Text','Caption','YesNoCancel','Question'
)
```


For more information about the MessageBox class and to view all the definitions have a look at its MSDN entry:

[MessageBox Class][1]

[1]: https://msdn.microsoft.com/en-us/library/system.windows.forms.messagebox
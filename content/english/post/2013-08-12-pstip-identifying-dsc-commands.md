---
title: '#PSTip Identifying DSC commands'
author: Shay Levy
type: post
date: 2013-08-12T18:00:25+00:00
url: /2013/08/12/pstip-identifying-dsc-commands/
categories:
  - Columns
  - Tips and Tricks
tags:
  - PowerShell DSC

---
**Note**: This tip requires PowerShell 4.0 or above.

Starting in Windows PowerShell 4.0 with the new Desired State Configuration (DSC) feature, we now have one more command type added to the list of command types: Configuration.

To get a list of Configuration commands only from your PowerShell session, type:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Command -CommandType Configuration
</pre>

To tell if a command in hand is a Configuration command, check the command _ScriptBlock.IsConfiguration_ property

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $command.ScriptBlock.IsConfiguration
</pre>
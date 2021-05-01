---
title: '#PSTip Ejecting and closing CDROM drive–the PowerShell way!'
author: Ravikanth C
type: post
date: 2013-11-12T19:00:38+00:00
url: /2013/11/12/pstip-ejecting-and-closing-cdrom-drive-the-powershell-way/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

I came across a project on [Github][1] (nothing related to PowerShell though) that provides a web-based UI for running system commands. This is based on [Node.js][2] implementation and does not support Windows platform yet. So, I was thinking about adding some PowerShell support.

I started discussing this with a few folks and they suggested an [alternative for running system commands][3] on Windows. While looking at that, I stumbled upon the CDROM eject and close commands supported by this 3rd-party tool. I wanted to see how easy that is to do in PowerShell. So, here is what I came up with.

    Add-Type -TypeDefinition  @'
    using System;
    using System.Runtime.InteropServices;
    using System.ComponentModel;
    
    namespace CDROM
    {
        public class Commands
        {
            [DllImport("winmm.dll")]
            static extern Int32 mciSendString(string command, string buffer, int bufferSize, IntPtr hwndCallback);
            public static void Eject()
            {
                 string rt = "";
                 mciSendString("set CDAudio door open", rt, 127, IntPtr.Zero);
            }
    
            public static void Close()
            {
                 string rt = "";
                 mciSendString("set CDAudio door closed", rt, 127, IntPtr.Zero);
            }
        }
    }
    '@
    
    [CDROM.Commands]::Eject()
    [CDROM.Commands]::Close()
The above snippet uses the .NET interop namespace to invoke a Win32 API for working with multimedia devices. Btw, I came across several blog posts on using WScript Shell and Windows Media Player OCX to achieve this task. But, like I mentioned, this is more PowerShell way of doing it. Drop a comment if you are aware of any alternative methods.

[1]: https://github.com/benkaiser/desktop-command-remote/
[2]: http://nodejs.org/
[3]: http://www.nirsoft.net/utils/nircmd.html
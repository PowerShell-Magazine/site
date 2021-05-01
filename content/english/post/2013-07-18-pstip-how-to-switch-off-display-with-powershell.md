---
title: '#PSTip How to switch off display with PowerShell'
author: Jakub Jareš
type: post
date: 2013-07-18T18:00:53+00:00
url: /2013/07/18/pstip-how-to-switch-off-display-with-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

This week on a PowerShell forum, I stumbled upon user request to turn off his PC’s display on demand. I did not find what was the reason, but I guess he was trying to watch a horror movie and the shining display was ruining the experience. Or, more likely, he wanted to maximize his laptop’s battery life. Either way, after a few web searches I was able to come up with this:

```
# Turn display off by calling WindowsAPI.
# SendMessage(HWND_BROADCAST,WM_SYSCOMMAND, SC_MONITORPOWER, POWER_OFF)
# HWND_BROADCAST	0xffff
# WM_SYSCOMMAND	0x0112
# SC_MONITORPOWER	0xf170
# POWER_OFF	      0x0002

Add-Type -TypeDefinition '
using System;
using System.Runtime.InteropServices;

namespace Utilities {
	public static class Display
	{
		[DllImport("user32.dll", CharSet = CharSet.Auto)]
		private static extern IntPtr SendMessage(
			IntPtr hWnd,
			UInt32 Msg,
			IntPtr wParam,
			IntPtr lParam
		);

        public static void PowerOff ()
        {
            SendMessage(
                (IntPtr)0xffff, // HWND_BROADCAST
                0x0112, 	       // WM_SYSCOMMAND
                (IntPtr)0xf170, // SC_MONITORPOWER
                (IntPtr)0x0002  // POWER_OFF
            );
        }
	}
}
'
```

The code uses the _Add-Type_ cmdlet to define a new static type Utilities.Display. The type defines public method _PowerOff()_ which you call to power off the display. To try it out run the code above (there will be no output) to define the type and then use this call to power off your display:

<pre class="brush: powershell; title: ; notranslate" title="">[Utilities.Display]::PowerOff()
</pre>

If it worked for you wrap the method call to _Switch-DisplayOff_ PowerShell function, to make it easier to call and discover.

<pre class="brush: powershell; title: ; notranslate" title="">function Switch-DisplayOff
{
	[Utilities.Display]::PowerOff()
}
</pre>

Now it is ready to use in your current PowerShell session. If you decide to place it in your _$profile_ for later use make sure you include the whole type definition as well as the function definition.
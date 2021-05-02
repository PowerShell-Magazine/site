---
title: 'PowerShell Demo Extravaganza: Lego Mindstorms Robot, Part 1'
author: Krunoslav Mihalic
type: post
date: 2014-03-21T18:00:03+00:00
url: /2014/03/21/powershell-demo-extravaganza-lego-mindstorms-robot-part-1/
categories:
  - Demo
tags:
  - Demo

---
One of PowerShell Magazine’s editors is asking me for quite some time to write an article about how to move Lego “bricks” from PowerShell. This demo is part of PowerShell Demo Extravaganza session that I&#8217;ve presented at different Microsoft conferences (TechEd Australia 2013 session is recorded [here][1]). This will be the first part in PowerShell demo extravaganza series of posts, so let’s start.

To move Lego from command line you need one special Lego “brick”. It comes with only few sets (8547 or [31313][2]), and it’s called “intelligent brick”. This is the newest model (version 3).

![](/images/image0011.jpg)

In the past I used previous model to move different sets (like [Crane][3] and [R2D2][4] (at 1:14)). To move them I used .Net library from [MindSqualls site][5]. Last week I finally received new set but when I tried to use the same approach it didn’t work. Luckily, a new .NET library was published last year on CodePlex ([LEGO MINDSTORMS EV3 API][6]). Great things about this new .NET library is that it comes with documentation too.

First, we need to [download][7] .NET library, and expand files (don’t forget to unblock the archive before unpacking). It comes with three flavors:

  * Desktop
  * Phone
  * WinRT

We will concentrate on the Desktop version. The steps are similar for other two platforms.

### Step 1. Load DLL into your PowerShell session

<pre class="brush: powershell; title: ; notranslate" title="">[Reflection.Assembly]::LoadFile('C:\Lego.Ev3.1.0.0\Lego.Ev3.Desktop.dll')
</pre>

### Step 2. Create a “brick” object

Before we continue, we need to connect “brick” and a laptop. The following connection types are available:

  * USB
  * Bluetooth
  * WiFi (it requires USB WiFi adapter to be inserted into “brick” )

In this example we will use Bluetooth. Please ensure that Bluetooth is enabled on “brick” and iPhone/iPad/iPod is not selected, and then pair it with a computer and note outbound COM port.

![](/images/image0022.jpg)

<pre class="brush: powershell; title: ; notranslate" title="">$btc = New-Object -TypeName Lego.Ev3.Desktop.BluetoothCommunication COM3
$brick = New-Object -TypeName Lego.Ev3.Core.Brick $btc
</pre>

### Step 3. Connect a laptop and a “brick”

<pre class="brush: powershell; title: ; notranslate" title="">$brick.ConnectAsync()
</pre>

And that’s it&#8211;the connection is established with just three commands. Now we have full control of our Lego Mindstorms robot!

We can connect motors and sensors to the “brick”. Motors are connected to ports named with letters (“A”, “B”, “C”, “D”) and sensors that are named with numbers (“1”, “2”, “3”, “4”). Brick object that we have created have the “Ports” property. It allows us to get information about what is connected to each port.

Display motor connected on port “B”

<pre class="brush: powershell; title: ; notranslate" title="">$brick.Ports["B"]
</pre>

![](/images/image0033.jpg)

**Note:** When motor is connected to a port, *Value properties are number of steps (full circle is 360 steps) that motor has done.

Display sensor connected on port “3”:

<pre class="brush: powershell; title: ; notranslate" title="">$brick.Ports["3"]
</pre>

![](/images/image0044.jpg)

**Note:** In case of infrared sensor *Value properties represent a distance between sensor and nearest obstacle. To move the “brick” we need first to assemble an EV3 robot that has some motors. For start we will use the [TRACK3R][8]. It has three motors and one IR sensor that we can use to measure distance.

To move motors we can use different methods “under” _$brick.DirectCommand_ property:

<pre class="brush: powershell; title: ; notranslate" title="">$brick.DirectCommand.StepMotorAtPowerAsync(@("B","C"), 50, 360, $false)
</pre>

  * @(&#8220;B&#8221;,&#8221;C&#8221;)- turn on two motors
  * 50 – represtens Power (1 &#8211; 100)
  * 360 &#8211; number of steps
  * $false &#8211; engage (parking) brake after command is completed

This is the complete code to move motors for one full circle:

```
[Reflection.Assembly]::LoadFile('C:\Lego.Ev3.1.0.0\Lego.Ev3.Desktop.dll')
$btc = New-Object -TypeName Lego.Ev3.Desktop.BluetoothCommunication COM3
$brick = New-Object -TypeName Lego.Ev3.Core.Brick $btc

$brick.ConnectAsync()
$brick.DirectCommand.StepMotorAtSpeedAsync(@("B","C"), 50, 360, $false)
```

How hard is to move Lego “Brick” using PowerShell? Well, just four lines of code! I really like PowerShell!

In the next post we will take care that this robot avoids obstacles. Stay tuned!

[1]: http://channel9.msdn.com/Events/TechEd/Australia/2013/MDC323
[2]: http://brickset.com/sets/45500-1/EV3-Intelligent-Brick
[3]: http://www.youtube.com/watch?v=23zU8RlZh1s
[4]: http://www.youtube.com/watch?v=HNTiIDn9Ui0#t=75
[5]: http://www.mindsqualls.net/
[6]: http://legoev3.codeplex.com/
[7]: http://legoev3.codeplex.com/releases/view/114257
[8]: http://www.lego.com/en-us/mindstorms/?domainredir=mindstorms.lego.com
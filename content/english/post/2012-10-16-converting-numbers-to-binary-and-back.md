---
title: '#PSTip Converting numbers to binary and back'
author: Shay Levy
type: post
date: 2012-10-16T18:00:33+00:00
url: /2012/10/16/converting-numbers-to-binary-and-back/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
To convert a number to its equivalent binary string representation, use the Convert.ToString method with a base of 2.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Convert]::ToString(192,2)
11000000
</pre>

To convert a binary number into its decimal representation, use the Convert.ToInt32 method with a base of 2 :

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Convert]::ToInt32('11000000',2)
192
</pre>
Binary conversions are usually used in IP addressing and subnetting calculations. Here&#8217;s an example of converting an IP address to its binary representation and back.

```
$ip = '192.168.10.1'

# convert to binary form
$bin = $ip -split '\.' | ForEach-Object {
    [System.Convert]::ToString($_,2).PadLeft(8,'0')
}

# print result
$bin
11000000
10101000
00001010
00000001

# join the objects
$bin -join '.'
11000000.10101000.00001010.00000001

# convert the result back to decimal
$dec = $bin | ForEach-Object {
    [System.Convert]::ToByte($_,2)
}

# print result
$dec
192
168
10
1

# join the result to form a valid IP Address
$dec -join '.'
192.168.10.1
```
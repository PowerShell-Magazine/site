---
title: '#PSTip Creating flagged enumerations in PowerShell'
author: Ravikanth C
type: post
date: 2012-10-01T14:28:07+00:00
url: /2012/10/01/pstip-creating-flagged-enumerations-in-powershell/
views:
  - 10534
post_views_count:
  - 1340
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
At times, we may want to create an enumeration object in which a combination of different constants means different values or descriptions. For example, look at this scripting skill set enumeration:

<pre class="brush: powershell; title: ; notranslate" title="">namespace Scripting
{
    public enum Skill
    {
       Batch = 1,
       VBScript = 2,
       Perl = 4,
       Python = 8,
       PowerShell = 16
    }
}
</pre>

From the above enumeration, we can derive the skill set value for a person. But, what if we want to derive a set of all scripting skills he/she holds? or what if we want to assign a value to a person to denote that he knows all these scripting languages or skills?

The general enumeration constants won’t let us do this. This is where the [FlagsAttribute][1] class will help us. Here is how the MSDN article distinguishes regular enumeration constants from those defined with FlagsAttribute:

> Bit fields are generally used for lists of elements that might occur in combination, whereas enumeration constants are generally used for lists of mutually exclusive elements. Therefore, bit fields are designed to be combined with a bitwise OR operation to generate unnamed values, whereas enumerated constants are not. Languages vary in their use of bit fields compared to enumeration constants.

Now that we understand why we need FlagsAttribute, let us see a quick example on how to build these enum objects:

```
$enum = "
using System;

namespace Scripting
{
     [FlagsAttribute]
     public enum Skill
     {
         Batch = 1,
         VBScript = 2,
         Perl = 4,
         Python = 8,
         PowerShell = 16
     }
}
"
Add-Type -TypeDefinition $enum -Language CSharpVersion3
```

We can access the enumeration we just created by using the namespace [Scripting.Skill]

![](/images/enum1.png)

This is it. Now, a person who knows all these scripting languages will have an enumeration value of 31.

How do I know that? Let us see that in a subsequent #[PSTip][2]. 🙂

[1]: http://msdn.microsoft.com/en-us/library/system.flagsattribute.aspx
[2]: /category/columns/tipsandtricks/
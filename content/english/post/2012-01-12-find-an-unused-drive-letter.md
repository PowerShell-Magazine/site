---
title: Find an unused drive letter
author: Shay Levy
type: post
date: 2012-01-12T19:00:15+00:00
url: /2012/01/12/find-an-unused-drive-letter/
views:
  - 84715
post_views_count:
  - 5593
categories:
  - Brainteaser
tags:
  - Brainteaser

---
### Problem

Your task is to write the shortest code to find one of the unused drive letters (excluding a,b and c). PowerShell&#8217;s default aliases are allowed. You have one week.  Answers should be posted as a comment to this brain teaser. That&#8217;s it.

[<img class="alignnone size-full wp-image-1823" style="margin: 6px; display: inline; float: left;" title="PowerCLIBook" src="http://104.131.21.239/wp-content/uploads/2011/12/PowerCLIBook.jpg" alt="" width="80" height="99" />][1]

The winner gets a copy of  the &#8220;[VMware vSphere PowerCLI Reference][2]&#8221; from [Sybex][3]!

We'll post clues over the next couple days, stay tuned.

Good Luck!

### Solution

We have a winner: P!

Your code was 36 characters long:

```powershell
for($j=67;gdr($d=[char]++$j)2>0){}$d
```

This is the breakdown of our solution (47 characters):

```powershell
ls function:[d-z]: -n|?{!(test-path $_)} | random
```

We took a different approach (though more lengthy) to get a list of available by listing the default A-Z functions and using the Name switch to get the names only.

We then tested if a drive with the incoming drive letter name was available, if so, it was written back to the pipeline.

Finally, we used the `Get-Random` cmdlet to choose for us an available letter. As you can see we called `Get-Random` by its verb only. You may suspect that &#8216;random&#8217; is an alias for that command but it isn't. We used a known trick to shorten `Get-Random` &#8211; when PowerShell cannot resolve a command it tries to resolve it again by prepending the `Get` verb.

We hope you had fun and that you enjoyed participating and was able to learn a thing or two. See you in the next brain teaser.

We also would like to thanks our sponsors, [Sybex][3], for giving away the book for the Winner.

[1]: http://www.powerclibook.com/ "My Inspiration"
[2]: http://www.powerclibook.com/
[3]: http://eu.wiley.com/WileyCDA/Section/id-420431.html
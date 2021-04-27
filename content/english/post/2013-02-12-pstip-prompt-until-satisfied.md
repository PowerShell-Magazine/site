---
title: '#PSTip Prompt until satisfied'
author: Shay Levy
type: post
date: 2013-02-12T19:13:31+00:00
url: /2013/02/12/pstip-prompt-until-satisfied/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Sometimes you need to prompt a user for input so you can base your script upon it. The _Read-Host_ cmdlet is made just for that.

```
PS> $result = Read-Host 'Enter a number between 1 and 5'
Enter a number between 1 and 5: 3

PS> $result
3
```

What if the user entered something you didn&#8217;t expect? Your script fails to execute it. For example, the user supplied a higher or a lower  number or even a non-digit character.

With the following loop, a do-while loop, you can force the user to enter a valid number. A prompt will appear as long as the result doesn&#8217;t meet your criteria.

```
do{
    $result = Read-Host 'Enter a number between 1 and 5'
}
while(1..5 -notcontains $result)


Enter a number between 1 and 5: 0
Enter a number between 1 and 5: 6
Enter a number between 1 and 5: a
Enter a number between 1 and 5: 3

PS> $result
3
```

As you can see all three attempts to assign a non valid value resulted in a re-prompt. It will continue to execute until the user enters a value your script is expecting to work with.
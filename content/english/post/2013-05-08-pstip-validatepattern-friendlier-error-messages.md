---
title: '#PSTip ValidatePattern – Friendlier error messages'
author: Bartek Bielawski
type: post
date: 2013-05-08T18:00:00+00:00
url: /2013/05/08/pstip-validatepattern-friendlier-error-messages/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

PowerShell has a very nice list of built-in parameter validation attributes. They help validating parameter values before any code runs and give end-user consistent pre-configured error messages:

```
function Test-ValidateSet {
	param (
	    [ValidateSet(
		'Left',
		'Center',
		'Right'
	    )]
	    [string]$Alignment
	)

"Selected alignment: $Alignment"

}

Test-ValidateSet -Alignment Justify

Test-ValidateSet : Cannot validate argument on parameter 'Alignment'. 
The argument &amp;quot;Justify&amp;quot; does not belong to the set &amp;quot;Left,Center,Right&amp;quot; specified by the ValidateSet attribute. 
Supply an argument that is in the set and then try the command again.
```

As you can see – I haven’t wrote a single sentence, yet my error message is pretty clear. This is the case for most validators. There is, however, one that produces an error message that for most of us won’t be helpful at all&#8211;ValidatePattern:

```
function Show-UglyError {
	param (
	    [ValidatePattern(
		'^\d{4}-\d{4}$'
	    )]
	    [string]$Puzzle
	)

	"You solved 4-digits hyphen 4-digits puzzle with: $Puzzle!"
}

Show-UglyError -Puzzle Not-A-Solution

Show-UglyError : Cannot validate argument on parameter 'Puzzle'.
The argument &amp;quot;Not-A-Solution&amp;quot; does not match the &amp;quot;^\d{4}-\d{4}$&amp;quot; pattern. 
Supply an argument that matches &amp;quot;^\d{4}-\d{4}$&amp;quot; and try the command again.
```

As Joel _“Jaykul”_ Bennett pointed out in his [blog post on the same topic][1], for a “normal” user (a person who is not fluent in regular expressions) this error message looks like “Cannot validate argument on parameter &#8216;Puzzle&#8217;. The argument &#8220;Not-A-Solution&#8221; does not match &#8230; yadda, yadda.” Joel’s solution is to build your own, custom _ValidatePatternEx_ class, that derives from _ValidateEnumeratedArgumentsAttribute_ and overrides _ValidateElement_ method. As much as I love this idea, I can see situations when someone may look for something easier and I would like to suggest a different approach: using regular expression comments:

```
function Show-FriendlyError {
	param (
    		[ValidatePattern(
        		'(?# 4 digits hyphen 4 digits)^\d{4}-\d{4}$'
    		)]
    		[string]$AlmostEasy
	)

	"You guessed it right: $AlmostEasy"

}

Show-FriendlyError -AlmostEasy Wrong!

Show-FriendlyError : Cannot validate argument on parameter 'AlmostEasy'. 
The argument &amp;quot;Wrong!&amp;quot; does not match the &amp;quot;(?# 4 digits hyphen 4 digits)^\d{4}-\d{4}$&amp;quot; pattern. 
Supply an argument that matches &amp;quot;(?# 4 digits hyphen 4 digits)^\d{4}-\d{4}$&amp;quot; and try the command again.
```

The used pattern has two parts: first part is our comment that may give end-user more clue on how good pattern would look like: _‘(?# Any description you want to give to user)’_. Second part (after closing bracket) is pattern that PowerShell will use to validate input, so it contains actual regular expression.

Maybe not perfect, but still easier than original and the implementation cost is very low.

[1]: http://huddledmasses.org/better-error-messages-for-powershell-validatepattern/
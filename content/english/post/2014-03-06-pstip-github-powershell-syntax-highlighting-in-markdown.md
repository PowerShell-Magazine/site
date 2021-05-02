---
title: '#PSTip GitHub PowerShell syntax highlighting in Markdown'
author: Carlos Perez
type: post
date: 2014-03-06T19:00:42+00:00
url: /2014/03/06/pstip-github-powershell-syntax-highlighting-in-markdown/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
<a style="line-height: 1.5em;" href="http://github.com/">GitHub</a><span style="line-height: 1.5em;">, a service for hosting development projects that can be written in many programing and scripting languages has its own extensions to the popular </span><a style="line-height: 1.5em;" href="https://help.github.com/articles/github-flavored-markdown">Markdown style</a><span style="line-height: 1.5em;">. This allows us to add syntax highlighting in our Markdown files inside of our repositories (like the README.md file). Also, when opening  Issues, Pull Request and even on the  Wiki we can use it in code blocks to syntax highlight code following one of the supported language types.</span>

The way we specify what type of syntax highlighting should be used for processing the code block is by adding at the front of the triple backtick marker used by Github the name of the language. In the case of PowerShell it would be:


```
<#
.Synopsis
   Short description
.DESCRIPTION
   Long description
.EXAMPLE
   Example of how to use this cmdlet
.EXAMPLE
   Another example of how to use this cmdlet
#>
function Verb-Noun
{
    [CmdletBinding()]

    [OutputType([int])]

    Param
    (
        # Param1 help description
        [Parameter(Mandatory=$true,
                   ValueFromPipelineByPropertyName=$true,
                   Position=0)]
        $Param1,

        # Param2 help description
        [int]
        $Param2
    )

    Begin
    {
    }

    Process
    {
    }

    End
    {
    }
}
```
GitHub would render this in the following manner:

![](/images/GitExample.png)

If you plan to use Markdown files in your project to document it or to generate documentation you will later export to HTML or PDF, you can also use a program called [MarkdownPad 2 Pro][1] . The Pro version allows use of the GitHub Markdown Engine in its Options.

![](/images/github.png)

This works great when one wants to edit the Markdown files locally and see a preview of how they would look rendered.

![](/images/githubmd.png)

[1]: http://markdownpad.com/
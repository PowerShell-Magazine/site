---
title: '#PSTip Using XPath in PowerShell, Part 4'
author: Bartek Bielawski
type: post
date: 2014-06-30T18:00:24+00:00
url: /2014/06/30/pstip-using-xpath-in-powershell-part-4/
categories:
  - XML
  - Tips and Tricks
tags:
  - XML
  - Tips and Tricks

---
Filtering using XPath is not limited to queries that make sure that existing value is equal to value that we are interested in. If the data within XML is numeric, we can use typical mathematical comparison operators:

```
Select-Xml -Path .\Test.xml -XPath "//subNode[@Number > 1]" | 
    ForEach-Object { $_.Node } 

Number
------
2
```

**Note**: You can find the code and input file <a href="https://gist.github.com/bielawb/2cb86a8abd01bae47ada" target="_blank">here</a>.

It gets even more interesting once we start using functions available in XPath. First of all: contains() function. This is the function that we can use to do &#8220;wildcard&#8221; filters. Unlike PowerShell –contains operator this function works on strings. The first argument is used as a base and the second one as a substring that we want to find. For example, if we are interested in any XHTML element with id containing string &#8220;test&#8221;:

```
Select-Xml -Path .\TestPage.xhtml -XPath "//*[contains(@id,'test')]" |
    ForEach-Object { $_.Node.id }

testing
othertest
```

There are also two other functions useful in XPath filters. First can be used if we want to make sure that node name contains certain substring: name(). It may also happen, that we will need to check value of a given node: that will require use of text() function. We can combine the two to get all nodes with name containing &#8216;Password&#8217; or with value containing &#8216;contoso&#8217;:

```
$xpath = @'
    //*[
        contains(
            name(),
            'Password'
        ) or
        contains(
            text(),
            'contoso'
        )    
    ]
'@

Select-Xml -Path .\ValueAndName.xml -XPath $xpath |
    Format-Table -AutoSize Node, @{ 
        Name = 'Value'
        Expression = { $_.Node.InnerXml }
    }

Node           Value      
----           -----
localPassword  P@ssw0rd   
domainName     contoso.com
domainPassword P@$$word 
```

Finally: we can reverse any query using not() function to return only nodes that do not meet our criteria:

```
Select-Xml -Path .\Test.xml -XPath "//subNode[not(@Number > 1)]" | 
    ForEach-Object { $_.Node } 

Number
------
1
```

This way we can sometimes get away with simpler filter that would normally give us all nodes we are not interested in, and reverse it to get the one we really need.
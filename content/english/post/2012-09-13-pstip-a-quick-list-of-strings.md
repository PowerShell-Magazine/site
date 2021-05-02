---
title: '#PSTip A quick list of strings'
author: Shay Levy
type: post
date: 2012-09-13T18:00:02+00:00
url: /2012/09/13/pstip-a-quick-list-of-strings/
views:
  - 7401
post_views_count:
  - 1345
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When you need to create a collection of strings and operate on each of them you&#8217;d go with something like:

<pre class="brush: powershell; title: ; notranslate" title="">'one','two','three','four','five' | ForEach-Object { $_ }
</pre>

As you can see, and your fingers probably feel, those strings needs to be quoted and comma delimited before they can be written to the pipeline. One way to avoid this is to create a helper function (coming from Perl), Quote-List (or ql for short):

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; function ql { $args }
PS&gt; ql one two three four five | ForEach-Object { $_ }
</pre>

The function takes a list of unquoted arguments and returns a collection of strings. That&#8217;s cool but it requires you to create the ql function or make sure it&#8217;s present on all systems you intend to use it on. Actually, there&#8217;s a better, built-in way, of doing this without having to create helper function. The answer is the Write-Output cmdlet. In the following example, I&#8217;m using Write-Output&#8217;s &#8216;echo&#8217; alias:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; echo one two three four five | ForEach-Object { $_ }
</pre>

If you can&#8217;t live without commas, this is also valid:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; echo one,two,three,four,five | ForEach-Object { $_ }
</pre>
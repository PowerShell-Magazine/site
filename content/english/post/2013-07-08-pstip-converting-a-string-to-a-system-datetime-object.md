---
title: '#PSTip Converting a String to a System.DateTime object'
author: Jakub Jareš
type: post
date: 2013-07-08T18:00:24+00:00
url: /2013/07/08/pstip-converting-a-string-to-a-system-datetime-object/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In a perfect world all date and time data available are represented by _System.DateTime_ objects. In the real world we are usually not that lucky. You get lot of obscurely formatted dates that you have to convert yourself. The .NET Framework offers a special method just for that, [_TryParseExact()_][1]. Calling the method is not particularly easy so I&#8217;ve created a short function to make it easier to work with:


	function Convert-DateString ([String]$Date, [String[]]$Format)
	{
		$result = New-Object DateTime
	    $convertible = [DateTime]::TryParseExact(
	        $Date,
	        $Format,
	        [System.Globalization.CultureInfo]::InvariantCulture,
	        [System.Globalization.DateTimeStyles]::None,
	        [ref]$result)
	
	    if ($convertible) { $result }
	}
Let&#8217;s call it with a date string in an uncommon format and see how the function handles different date and time separators:

<pre class="brush: powershell; title: ; notranslate" title="">Convert-DateString -Date '12/10\2013 13:26-34' -Format 'dd/MM\\yyyy HH:mm-ss'
Saturday, October 12, 2013 1:26:34 PM
</pre>

The function successfully converts the date and returns a _DateTime_ object. If the operation was not successful nothing would have been returned.

Also notice the Format parameter accepts array of strings, allowing you to specify more than one format of the input.

```
Convert-DateString -Date '12:26:34' -Format 'HH:mm:ss','HH-mm-ss'
Convert-DateString -Date '12-26-34' -Format 'HH:mm:ss','HH-mm-ss'

Thursday, July 4, 2013 12:26:34 PM
Thursday, July 4, 2013 12:26:34 PM
```

The string is converted successfully, and because only time was provided, today’s date is used to complete the other date parts of the object.

### Building a format string

Providing correct string to the Format parameter is essential for successfully using the function. Let’s have a quick look on how to create one yourself: The string uses the following characters _&#8220;d&#8221;, &#8220;f&#8221;, &#8220;F&#8221;, &#8220;g&#8221;, &#8220;h&#8221;, &#8220;H&#8221;, &#8220;K&#8221;, &#8220;m&#8221;, &#8220;M&#8221;, &#8220;s&#8221;, &#8220;t&#8221;, &#8220;y&#8221;, &#8220;z&#8221;_ to define type, position and format of the input values. The type of the input value (day, month, minute etc.) is defined by choosing the correct letter to represent the value (day _d_, month _M_, minute _m_ etc.), case matters here. The position is defined by placing the character on the correct place in the string. The format is defined by how many times the character is repeated (_d_ to represent 1-31, _dd_ for 01-31, _dddd_ for Monday).

```
Convert-DateString -Date 'Thursday, July 4, 2013 12:26:34 PM' `
-Format 'dddd, MMMM d, yyyy hh:mm:ss tt'

Thursday, July 4, 2013 12:26:34 PM
```

If your string contains any of the listed characters or the backslash (“\”) character you have to escape it by preceding it with a backslash:

```
Convert-DateString -Date "d: 01&#92;&#48;1\2013" -Format '\d: dd\\MM\\yyyy'
Tuesday, January 1, 2013 12:00:00 AM
```


The complete reference to creating custom date and time format strings, as well as many examples may be found [here][2].

[1]: http://msdn.microsoft.com/en-us/library/h9b85w22.aspx
[2]: http://msdn.microsoft.com/en-us/library/8kb3ddd4.aspx
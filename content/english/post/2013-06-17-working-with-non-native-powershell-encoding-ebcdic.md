---
title: Working with non-native PowerShell encoding (EBCDIC)
author: Josh Miller
type: post
date: 2013-06-17T16:00:00+00:00
url: /2013/06/17/working-with-non-native-powershell-encoding-ebcdic/
categories:
  - How To
tags:
  - How To

---
The _-Encoding_ parameter in many native I/O cmdlets uses the [Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding] enum.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [System.Enum]::GetNames([Microsoft.PowerShell.Commands.FileSystemCmdletProviderEncoding])
Unknown
String
Unicode
Byte
BigEndianUnicode
UTF8
UTF7
UTF32
Ascii
Default
Oem
</pre>

Files that come from an IBM mainframe such as the AS/400 are often exported to ASCII or Unicode when a PC is the intended consumer. Sometimes, however, we need to be able to work with these files that are formatted in IBM’s native EBCDIC encoding.  “Extended Binary Coded Decimal Interchange Code (EBCDIC)” is an 8-[bit][1] [character encoding][2] that is extended from the 6-bit encoding used on punch cards. See <http://en.wikipedia.org/wiki/EBCDIC> for more detail.  Fortunately the .NET Framework provides about 140 native encoding types.

To get the list run _[System.Text.Encoding]::GetEncodings()_. Many of the items in the _[FileSystemCmdletProviderEncoding]_ enum are included in this list. You may also notice that about 35 of these encodings contain either IBM or EBCDIC in their _DisplayName_.  The most common or generic of the EBCDIC formats are under the IBM037 name.

An example content of an EBCDIC file opened in Notepad looks like this; not that readable:

![](/images/ebcdic.png)

To read the data and use the native commands, we can load the file as bytes then convert those bytes to a .NET string that we can work with.

<pre class="brush: powershell; title: ; notranslate" title="">$Buffer = Get-Content $SourceFileName -Encoding byte
$Encoding = [System.Text.Encoding]::GetEncoding("IBM037")
$String = $Encoding.GetString($Buffer)
</pre>

Magically the $String variable now contains text we can read.  Now another interesting thing about some IBM data files is that they are character length-specific data fields, and do not necessarily contain any EOL (End-Of-Line) characters. Depending on the type of file you are looking at, you may have a NEL (NExt-Line 0x15) character mark; if so, the Encoding class should have resolved that for you.

If you didn’t and you don’t want to play all day with line wrap in your editor, then we need to insert \`n. A regular expression using the all-encompassing “.” and a set length will easily give us the field records that we need. They can then be left in an array or joined back together again with \`n.

Most line default records should be 80 characters long from punch card legacies, but in the financial analysis file that I used 107 characters were used to define the line records.


	Function Decode-EBCDIC {
	    Param (
	     [io.fileinfo]$SourceFileName,
	     [int]$LineLength = -1,
	     $Encoding = "IBM037"
	    )
	
	    #Based on http://www.codeproject.com/Articles/31720/How-to-Read-an-EBCDIC-File-in-VB-NET
	    $Buffer = Get-Content $SourceFileName -encoding byte
	    $Encoding = [System.Text.Encoding]::GetEncoding("IBM037")
	    $String = $Encoding.GetString($Buffer)
	
	    if ($LineLength -gt 1) {
	        $Regex = [regex]".{$LineLength}"
	        $OutString = ($Regex.Matches($String) | Select-Object -ExpandProperty Value) -join "`r`n"
	        # Join won't add a CR at the end of the string, so we manually append it.
	        $OutString += "`r`n"
	    } else {
	        $OutString = $String
	    }
	    $OutString
	}
	
	PS> Decode-EBCDIC .\EBCDIC_wiki.txt
	http://en.wikipedia.org/wiki/EBCDIC#History
	EBCDIC /'?bs?d?k/ was devised in 1963 and 1964 by IBM and was announced with
	the release of the IBM System/360 line of mainframe computers. It is an 8-bit
	character encoding, in contrast to, and developed separately from, the 7-bit
	ASCII encoding scheme. It was created to extend the existing binary-coded
	decimal (BCD) interchange code, or BCDIC, which itself was devised as an
	efficient means of encoding the two zone and number punches on punched cards
	into 6 bits.
	(…)
	
	PS> Decode-EBCDIC .\EBCDIC_wiki_noNEL.txt -LineLength 79
	http://en.wikipedia.org/wiki/EBCDIC#History
	EBCDIC /'?bs?d?k/ was devised in 1963 and 1964 by IBM and was announced with
	the release of the IBM System/360 line of mainframe computers. It is an 8-bit
	character encoding, in contrast to, and developed separately from, the 7-bit
	ASCII encoding scheme. It was created to extend the existing binary-coded
	decimal (BCD) interchange code, or BCDIC, which itself was devised as an
	efficient means of encoding the two zone and number punches on punched cards
	into 6 bits.
	(…)
Sample file: [EBCDIC\_wiki\_noNEL][3], [EBCDIC_wiki][4]

[1]: http://en.wikipedia.org/wiki/Bit "Bit"
[2]: http://en.wikipedia.org/wiki/Character_encoding "Character encoding"
[3]: /images/EBCDIC_wiki_noNEL.txt
[4]: /images/EBCDIC_wiki.txt
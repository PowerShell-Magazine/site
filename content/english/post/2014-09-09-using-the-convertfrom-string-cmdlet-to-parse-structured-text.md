---
title: Using the ConvertFrom-String cmdlet to parse structured text
author: Bartek Bielawski
type: post
date: 2014-09-09T23:29:25+00:00
url: /2014/09/09/using-the-convertfrom-string-cmdlet-to-parse-structured-text/
categories:
  - How To
tags:
  - How To

---
One of PowerShell strengths has always been string manipulation. PowerShell has very good support for regular expressions&#8211;using both cmdlets and operators. There are cmdlets to import and export several text file formats: XML, JSON, CSV. I use regular expressions in PowerShell almost every day. There is one problem with regular expressions though: it&#8217;s a language that is easier to write than read. And, it can be very complex when you try to process structured text files, especially if structure is not limited to single line and is not in one of natively supported formats.

### New kid on the block

On September 4, a new version of <a href="http://104.131.21.239/2014/09/05/windows-management-framework-5-0-september-2014-preview-is-available-for-download/" target="_blank">Windows Management Framework 5.0 Preview was announced</a>. This release comes with a cmdlet, _ConvertFrom-String_, that simplifies processing of any structured text. It does this in a relatively straightforward fashion. You don&#8217;t have to be a regex expert to get what you want, you just need to understand output of your command, a content of your log file, a structure of data exported from your database. As long as you can see a pattern and &#8216;explain it&#8217; to PowerShell, PowerShell will turn a structured text into objects.

_ConvertFrom-String_ has two modes. In the first one (basic delimited parsing) it&#8217;s not much different from _ConvertFrom-Csv_. Processing is performed line by line, and there is no way to identify input header other than using _Select-Object -Skip_ to ignore the first few lines. You can specify delimiter, names of properties, and you are good to go. But that&#8217;s not scenario in which this new cmdlets shines. I would argue that using _ConvertFrom-Csv_ is as easy, and is possible in PowerShell 2.0.

Second mode (auto generated example-driven parsing) is much more interesting. Release notes mention FlashExtract is used to get the results.

**Note**: You can find more on FlashExtract in publication &#8220;FlashExtract: A Framework for Data Extraction by Examples&#8221;, PLDI 2014, Vu Le, Sumit Gulwani, that can be found <a href="http://research.microsoft.com/en-us/um/people/sumitg/publications.html" target="_blank">here</a>.

In this mode, we hand _ConvertFrom-String_ a template of data that we want to process. We need to follow a few rules when we prepare the template, but once we are done, PowerShell will look at any structured data as one of the well known formats. This article focuses entirely on this mode. We will take a look at real-world example of processing structured file (export from Opera mail client address book), how the workflow is simplified by using this new cmdlet in contrast with the one that depends on regular expressions. We will also take a look at _ConvertFrom-String_ limitations and &#8216;gotchas&#8217;. And last but not least, we will use the new cmdlet to process more complex input data to create complex objects.

### Story of one address book transfer

Almost two years ago, I had to help my wife to move her address book from Opera mail client to Outlook. After testing different options, I ended up with &#8216;Opera Hotlist version 2.0&#8217; file as my input and needed to figure out a way to convert it to CSV, format that Outlook was able to &#8216;consume&#8217;. Luckily, file retrieved from Opera was human-readable. And after looking at the content I immediately came up with idea how to get what I wanted. Here is an example data file.

```
Opera Hotlist version 2.0
Options: encoding = utf8, version=3

#CONTACT
	ID=11
	NAME=Justynka
	CREATED=1195505237
	MAIL=JUSTYNA66@gmail.com
	ICON=Contact0

#CONTACT
	ID=12
	NAME=Leszek
	CREATED=1195677687
	MAIL=Leszek@domena.pl
	ICON=Contact0

#CONTACT
	ID=13
	NAME=Iwona Kwiatkowska
	CREATED=1196277590
	MAIL=iwon.kwiat@op.pl
	ICON=Contact0

#CONTACT
	ID=14
	NAME=JUSTYNA66@gmail.com
	CREATED=1347061687
	MAIL=JUSTYNA66@gmail.com
	ICON=Contact0

#FOLDER
	ID=15
	NAME=Kosz
	CREATED=1195505227
	TRASH FOLDER=YES
	UNIQUEID=EAF22324295C86499476802CC76DE41E

-

#CONTACT
	ID=16
	NAME=Ania
	CREATED=1195505237
	MAIL=Ania.Nowak@poczta.com
	ICON=Contact0

#CONTACT
	ID=58
	NAME=Bartek Bielawski
	CREATED=1381258759
	MAIL=bartek.bielawski@live.com
	ICON=Contact0

#CONTACT
	ID=20
	NAME=Poczta Grupowa
	URL=
	CREATED=1347221208
	DESCRIPTION=
	ACTIVE=YES
	MAIL=xyz.abc.grupa.foo@gmail.com
	PHONE=
	FAX=
	POSTALADDRESS=
	PICTUREURL=
	ICON=Contact0
```

How did I solve this problem back then? I&#8217;ve used regular expressions and cmdlet with name that is similar to the one I would use today&#8211;_ConvertFrom-StringData_. You can see that each entry starts with the same string (#CONTACT). So there it was: easy way to _-split_ records. Next thing I noticed&#8211;each record defines properties using &#8216;key=value&#8217; syntax, something that _ConvertFrom-StringData_ can translate into a hash table. When we have the hash table, creating an object is very easy. My work here was finished or so I thought. The actual workflow from idea to actual implementation is:

  * Read file as single string (-Raw), split on &#8216;#CONTACT&#8217;, process each record with _ConvertFrom-StringData_
  * OK, there is header, so let&#8217;s replace it
  * Replace with &#8216;.\*?&#8217; failed&#8230; need to use &#8216;[\s\S]\*?&#8217; instead to cover multiline pattern
  * Converting to hash table still fails&#8211;some records are prefixed with #FOLDER rather than #CONTACT
  * Works for the most part, except there is &#8216;-&#8216; somewhere that causing it to fail, need to replace it
  * All hash tables look OK, time to convert them to objects using _New-Object_

Final code looks simple!

```
(Get-Content .\opera.adr -Raw) -replace '^Opera[\s\S]*?#CONTACT' -split '#CONTACT|#FOLDER' |
    ForEach-Object {
        $Props = ConvertFrom-StringData -StringData ($_ -replace '\n-\s+')
        New-Object PSOBject -Property $Props | Select-Object Name, Mail
    }
```


How workflow changes when we start to use _ConvertFrom-String_ cmdlet? First of all, we don&#8217;t have to use regular expressions. Even better, we don&#8217;t need to know anything about it. All we need to do is describe structure of our file. We can either use a file, or string (preferably here-string) to define our template. To sum it up:

  * Copy few records from file to here-string that we will use as a template
  * Add curly brackets to identify name and mail of contact
  * Make sure that you highlight property that starts new &#8216;set&#8217; with &#8216;*&#8217; suffix

Code is longer (mainly because of template definition), but it&#8217;s also easier to write. I didn&#8217;t have to extract data on my own, everything happened &#8216;automagically&#8217;.

Here is the final code:

```
$TemplateAdr = @'
#CONTACT
	ID=11
	NAME={Name*:Justynka}
	CREATED=1195505237
	MAIL={Mail:JUSTYNA66@gmail.com}
	ICON=Contact0

#CONTACT
	ID=20
	NAME={Name*:Poczta Grupowa}
	URL=
	CREATED=1347221208
	DESCRIPTION=
	ACTIVE=YES
	MAIL={Mail:xyz.abc.grupa.foo@gmail.com}
	PHONE=
	FAX=
	POSTALADDRESS=
	PICTUREURL=
	ICON=Contact0

'@

Get-Content .\opera.adr | ConvertFrom-String -TemplateContent $TemplateAdr |
    Format-Table -AutoSize Name, Mail

Name              Mail

----              ----

Justynka          JUSTYNA66@gmail.com
Leszek            Leszek@domena.pl
Iwona Kwiatkowska iwon.kwiat@op.pl
Ewa Nowak         EWA22@gmail.com
Kosz
Ania              Ania.Nowak@poczta.com
Bartek Bielawski  bartek.bielawski@live.com
Poczta Grupowa    xyz.abc.grupa.foo@gmail.com
```

It looks fine, but you have to be aware of limitations and &#8216;gotchas&#8217;.

### There is always &#8216;but&#8217;&#8230;

_ConvertFrom-String_ cmdlet can do wonders for us but you can walk into problems if you are not careful.

First and foremost, examples! You have to see patterns and make sure that PowerShell is aware of all possibilities. If your template won&#8217;t cover certain scenario you can end up with partial results (worse, as you may not see that something is missing at first) or with following error message:

_[28,27: ConvertFrom-String] ConvertFrom-String appears to be having trouble parsing your data using the template you&#8217;ve provided. We’d love to take a look at what went wrong, if you&#8217;d li

ke to share the data and template used to parse it. We&#8217;ve saved these files to C:\Users\Bartek\AppData\Local\Temp\smt5asdi.1&#215;1.input.txt and C:\Users\Bartek\AppData\Local\Temp\smt5asdi.1

x1.template.txt &#8211; feel free to attach them in a mail to psdmfb@microsoft.com. We will review all submissions, although we can&#8217;t guarantee a response._

Partial results can have two flavours. Either an entire item is missing or a property of the item is missing. The former is very hard to spot. Both are result of the fact that examples you provided in your template are too specific. For example, if I have items in address book with the name that is same as e-mail address and both examples in my template suggest &#8216;FirstName LastName&#8217; or &#8216;Name&#8217; format, this record will be ignored. If an e-mail is provided in a format different than the one seen in examples, we may end up with records that don&#8217;t have &#8216;Mail&#8217; property. Last but not least, at times you can get very unpredictable results. For example, when I worked on this article I &#8216;cleaned&#8217; address book entries and my template. I noticed that in one case &#8216;Mail&#8217; property was there but value contains only part of e-mail address.

Here is the sample data, template, and results I retrieved.

```
$otherTemplate = @'
{First*:Jan} {Last:Fasola}
MAIL={Mail:Jan@Fasola.com}

{First*:Not} {Last:Real}
MAIL={Mail:Just@Similar.to}
'@

$otherData = @'
Jan Fasola
MAIL=Jan@Fasola.com

not there
MAIL=all@lower.case

Ewa Kowalska
MAIL='Ewa' &lt;Ewa@Kowalska.com&gt;

Missing2 Cause
MAIL=used@number

Silly Cut
MAIL=causeIt@Expects.up
'@

$otherData | ConvertFrom-String -TemplateContent $otherTemplate |
    Format-Table -AutoSize First, Last, Mail

First Last     Mail
----- ----     ----
Jan   Fasola   Jan@Fasola.com
Ewa   Kowalska
Silly Cut      It@Expects.up
```

As you can see, examples in my template are very specific. Both the first and last names start with upper case letter. In both records &#8216;Mail&#8217; starts with upper case too. There are no numbers in name. Another thing that you probably noticed is that it does not really have to be actual example from input data, just need to follow same pattern.

In the output two items are missing. One has full name in all lower case and the second contains a number in the &#8216;First&#8217; field. Our template doesn&#8217;t &#8216;allow&#8217; such items, so both were discarded. Next, _Mail_ for &#8216;Ewa Kowalska&#8217; doesn&#8217;t follow the pattern we see in template. So, it&#8217;s not visible on this object. And, last but not least, in the last record _Mail_ is &#8217;causeIt@Expects.up&#8217; but in our output first part (starting with lower case) was removed. How to fix it? Just modify one of examples:

```
$fixedTemplate = @'
{First*:Jan} {Last:Fasola}
MAIL={Mail:Jan@Fasola.com}

{First*:not2} {Last:real}
MAIL={Mail:'Just' &lt;similar@to.ewa&gt;}
'@

$otherData | ConvertFrom-String -TemplateContent $fixedTemplate
```

Alternatively, we could just add records that were missing/incomplete to the template. In my opinion, this method is following the &#8216;KISS&#8217; principle. It may take more time but we are not forced to figure out what went wrong.

There are other potential issues, e.g. <a href="http://www.lazywinadmin.com/2014/09/powershell-convertfrom-string-and.html" target="_blank">here</a> François-Xavier Cat identified a problem when you want to capture _netstat -na_ output and &#8216;Status&#8217; property is not present for UDP connections (and suggested a nice workaround). Also, he shows another real-life example of _ConvertFrom-String_ use case, so it&#8217;s definitely worth reading.

### Complex is possible

So far, we worked on the flat data. _ConvertFrom-String_ however is able to process more complex data too. We can have nested properties, that are collections of objects. In other words: any file (or other input string data) can now be seen as XML/JSON. We just need to identify nested properties and cover them in our template. We will use output from Sysinternals _handle -u_ command. You can download it <a href="http://technet.microsoft.com/en-us/sysinternals/bb896655.aspx" target="_blank">here</a>. Example results:

![](/images/SysInternals-Handle-output.png)

In PowerShell it would map to single object for each process, with property that would hold all handles, each being object with three properties. With that in mind we can start building our template. Because we already know that data does not have to be &#8220;real&#8221; we will try to understand output and create template based on that information, leaving only lines that seem necessary for PowerShell to understand all input (and process it correctly):

```
$template = @'
------------------------------------------------------------------------------
{ProcessName*:Testing} pid: {PID:4} {User:\<unable to open process>}
   {Id*:18}: {Type:File}  (R--)   {Name:E:\$Extend\$RmMetadata\$TxfLog\$TxfLogContainer00000000000000000001}

   {Id*:90}: {Type:File}  (R--)   {Name:\clfs}
------------------------------------------------------------------------------
{ProcessName*:someprocess.exe} pid: {PID:6} {User:NT AUTHORITY\SYSTEM}
    {Id*:C}: {Type:File}  (---)   {Name:C:\Windows\System32}

   {Id*:A4}: {Type:Section}       {Name:\Sessions\1\Windows\SharedSection}
------------------------------------------------------------------------------
{ProcessName*:Extend64.exe} pid: {PID:2512} {User:EMIS\Bartek}
    {Id*:C}: {Type:File}  (RW-)   {Name:C:\Windows}
   {Id*:28}: {Type:File}  (R-D)   {Name:C:\Windows\System32\en-US\conhost.exe.mui}
'@
```


None of these processes is running on my system. I copied lines with handles information, but changed them to make sure I cover all scenarios. With this template I could parse output from _handle_ and convert it to complex objects. The only problem is that PowerShell adds ExtentText to all objects generated (including nested objects). And even though for testing/ debugging it may be handy, it&#8217;s usually not needed in the &#8216;final product&#8217;. Another problem I had was the fact that my handles were stored in automatically named property &#8216;Items&#8217;. I didn&#8217;t manage to find a way to change this name within template, so I&#8217;ve used _Select-Object_ to do that for me.

```
handle -u | ConvertFrom-String -TemplateContent $Template |
    Select-Object ProcessName, PID, User, @{
        Name = 'Handles'
        Expression = {
            $_.Items | Select-Object * -ExcludeProperty ExtentText
        }
    }
```


I would love to have parameter that would disable adding &#8216;ExtentText&#8217; property (or better yet: leave it out as a default behaviour, and adding it only if user requests it with switch parameter). Another problem that I had when I tried to figure out correct template was lack of verbose messages. The only way to get some feedback on &#8216;how&#8217; is to use _-Debug_ parameter, and even than we only get output if operation succeeds:

```
DEBUG: Property: ProcessName
Program: ESSL((PrecedingEndsWith(Hyphen(\-), Hyphen(\-), Hyphen(\-))): 0, 1, ...: ε...ε, 1 + Alphabet([\p{Lu}\p{Ll}\-.]+)...Dynamic Token(\ pid:\ )(\ pid:\ ), Number([0-9]+(\,[0-9]{3})*(

\.[0-9]+)?), WhiteSpace(( )+), 1)
-------------------------------------------------
Property: PID
Program: ESSL((Contains(Dynamic Token(\ pid:\ )(\ pid:\ ), Number([0-9]+(\,[0-9]{3})*(\.[0-9]+)?), WhiteSpace(( )+), 1)): 0, 1, ...: Alphabet([\p{Lu}\p{Ll}\-.]+), Dynamic Token(\ pid:\ )
(\ pid:\ )...Number([0-9]+(\,[0-9]{3})*(\.[0-9]+)?), WhiteSpace(( )+), 1 + Alphabet([\p{Lu}\p{Ll}\-.]+), Dynamic Token(\ pid:\ )(\ pid:\ ), Number([0-9]+(\,[0-9]{3})*(\.[0-9]+)?)...White

Space(( )+), 1)
-------------------------------------------------
Property: User
Program: ESSL((Contains(all lower((?&lt;![\p{Lu}\p{Ll}])(\p{Ll})+), Colon(\:), WhiteSpace(( )+), 1)): 0, 1, ...: WhiteSpace(( )+), Number([0-9]+(\,[0-9]{3})*(\.[0-9]+)?), WhiteSpace(( )+)..

.ε, 1 + ε...ε, 0)
-------------------------------------------------
Property: Id
Program: ESSL((Contains(Colon(\:), WhiteSpace(( )+), Camel Case(\p{Lu}(\p{Ll})+), 1)): 0, 1, ...: WhiteSpace(( )+)...ε, 1 + ε...Colon(\:), WhiteSpace(( )+), Camel Case(\p{Lu}(\p{Ll})+),

1)
-------------------------------------------------
Property: Type
Program: ESSL((Contains(Colon(\:), WhiteSpace(( )+), Camel Case(\p{Lu}(\p{Ll})+), 1)): 0, 1, ...: Colon(\:), WhiteSpace(( )+)...Camel Case(\p{Lu}(\p{Ll})+), WhiteSpace(( )+), 1 + Colon(\

:), WhiteSpace(( )+), Camel Case(\p{Lu}(\p{Ll})+)...WhiteSpace(( )+), 1)
-------------------------------------------------
Property: Name
Program: ESSL((Contains(Colon(\:), WhiteSpace(( )+), Camel Case(\p{Lu}(\p{Ll})+), 1)): 0, 1, ...: WhiteSpace(( )+)...ε, -1 + ε...ε, 0)
-------------------------------------------------
```


It would be great to get some feedback when parsing fails, for example which part of file/rule was responsible for it. Sending input and template to authors every time it fails seems like overkill. On the other hand: fixing these errors looks like shooting in the dark. In my opinion this cmdlet should help a lot of administrators to parse even most complex command results/log files without deep knowledge of regular expressions. And even though I&#8217;m huge regex fan, I&#8217;m happy that we got yet another tool to parse text in PowerShell.
---
title: Mastering everyday XML tasks in PowerShell
author: Tobias Weltner
type: trending
date: 2013-08-19T16:00:48+00:00
url: /2013/08/19/mastering-everyday-xml-tasks-in-powershell/
categories:
  - How To
  - XML
tags:
  - How To
  - XML

---
PowerShell has awesome XML support. It is not obvious at first, but with a little help from your friends here at PowerShellMagazine.com, you&#8217;ll soon solve every-day XML tasks &#8211; even pretty complex ones &#8211; in no time.

So let&#8217;s check out how you put very simple PowerShell code to work to get the things done that used to be so mind-blowingly complex in the pre-PowerShell era.

Let&#8217;s create an XML document from scratch, add new data sets, change pieces of information, add new data, remove data, and save an updated version of it to a new well-formed XML file.

### Creating New XML Documents

Creating completely fresh XML documents from scratch used to be a tedious task. Many scripters resorted to creating XML files as a plain text. While that&#8217;s OK, it is error prone. Chances are that typos and case issues sneak in, and you may find yourself in an unfriendly world of malformed and dysfunctional XML.

No more, because there&#8217;s a buddy that can help you create XML documents: the _XMLTextWriter_ object. It shields the complexity of dealing with the raw XML object model, and instead assists you in writing your pieces of information to an XML file.

To begin this story, let&#8217;s create a fairly complex XML document that the upcoming examples can use to play with. The goal is to create an XML document that has all the typical things in it: nodes, attributes, data sections, and comments.

```
# this is where the document will be saved:
$Path = "$env:temp\inventory.xml"

# get an XMLTextWriter to create the XML
$XmlWriter = New-Object System.XMl.XmlTextWriter($Path,$Null)

# choose a pretty formatting:
$xmlWriter.Formatting = 'Indented'
$xmlWriter.Indentation = 1
$XmlWriter.IndentChar = "`t"

# write the header
$xmlWriter.WriteStartDocument()

# set XSL statements
$xmlWriter.WriteProcessingInstruction("xml-stylesheet", "type='text/xsl' href='style.xsl'")

# create root element "machines" and add some attributes to it
$XmlWriter.WriteComment('List of machines')
$xmlWriter.WriteStartElement('Machines')
$XmlWriter.WriteAttributeString('current', $true)
$XmlWriter.WriteAttributeString('manager', 'Tobias')

# add a couple of random entries
for($x=1; $x -le 10; $x++)
{
    $server = 'Server{0:0000}' -f $x
    $ip = '{0}.{1}.{2}.{3}' -f  (0..256 | Get-Random -Count 4)

    $guid = [System.GUID]::NewGuid().ToString()

    # each data set is called "machine", add a random attribute to it:
    $XmlWriter.WriteComment("$x. machine details")
    $xmlWriter.WriteStartElement('Machine')
    $XmlWriter.WriteAttributeString('test', (Get-Random))

    # add three pieces of information:
    $xmlWriter.WriteElementString('Name',$server)
    $xmlWriter.WriteElementString('IP',$ip)
    $xmlWriter.WriteElementString('GUID',$guid)

    # add a node with attributes and content:
    $XmlWriter.WriteStartElement('Information')
    $XmlWriter.WriteAttributeString('info1', 'some info')
    $XmlWriter.WriteAttributeString('info2', 'more info')
    $XmlWriter.WriteRaw('RawContent')
    $xmlWriter.WriteEndElement()

    # add a node with CDATA section:
    $XmlWriter.WriteStartElement('CodeSegment')
    $XmlWriter.WriteAttributeString('info3', 'another attribute')
    $XmlWriter.WriteCData('this is untouched code and can contain special characters /\@&lt;&gt;')
    $xmlWriter.WriteEndElement()

    # close the "machine" node:
    $xmlWriter.WriteEndElement()
}

# close the "machines" node:
$xmlWriter.WriteEndElement()

# finalize the document:
$xmlWriter.WriteEndDocument()
$xmlWriter.Flush()
$xmlWriter.Close()

notepad $path
```

This script generates a fake server inventory with a lot of random information. The result is opened in notepad and will look similar to this:

<?xml version="1.0"?>
<?xml-stylesheet type='text/xsl' href='style.xsl'?>
<!--List of machines-->
<Machines current="True" manager="Tobias">
 <!--1. machine details-->
 <Machine test="578163632">
  <Name>Server0001</Name>
  <IP>31.248.95.170</IP>
  <GUID>51cb0dfb-75ed-4967-8392-47d87596c73c</GUID>
  <Information info1="some info" info2="more info">RawContent</Information>
  <CodeSegment info3="another attribute"><![CDATA[this is untouched code and can contain special characters /\@<>]]></CodeSegment>
 </Machine>
 <!--2. machine details-->
 <Machine test="124214010">
  <Name>Server0002</Name>
  <IP>33.60.233.89</IP>
  <GUID>9618b8bc-c200-46ce-b423-ee030555242d</GUID>
  <Information info1="some info" info2="more info">RawContent</Information>
  <CodeSegment info3="another attribute"><![CDATA[this is untouched code and can contain special characters /\@<>]]></CodeSegment>
 </Machine>
(...)
</Machines>


The purpose of this XML document is two-fold: it serves as an example how you can create XML files from scratch, and it serves as sample data for the following exercises.

Just assume this was an XML file with relevant information. You can apply the tactics you are about to learn to any well-formed XML file.

**Attention:** XMLTextWriter does a lot of magic for you, but you are responsible for creating meaningful content. One of the issues that can easily burn your feet is a malformed node name. Node names must not contain spaces.

So while &#8220;CodeSegment&#8221; is OK, &#8220;Code Segment&#8221; would not be OK. XML would try and name your node &#8220;Code&#8221;, then add an attribute named &#8220;Segment&#8221;, and finally choke on the fact that you never assigned a value to the attribute.

### Finding Information in XML Files

One common task is to extract information from an XML file. Let&#8217;s assume you need a list of machines and their IP addresses. Provided you have generated the sample XML file above, then this is all it takes to create the report:

```
# this is where the XML sample file was saved:
$Path = "$env:temp\inventory.xml"

# load it into an XML object:
$xml = New-Object -TypeName XML
$xml.Load($Path)

# note: if your XML is malformed, you will get an exception here
# always make sure your node names do not contain spaces
# simply traverse the nodes and select the information you want:
$Xml.Machines.Machine | Select-Object -Property Name, IP
```

The result will look similar to this:

```
Name          IP
----          --
Server0001    31.248.95.170
Server0002    33.60.233.89
Server0003    226.6.1.30
Server0004    139.30.8.110
Server0005    94.104.253.8
Server0006    202.80.178.61
Server0007    22.217.227.159
Server0008    253.72.25.212
Server0009    233.147.116.60
Server0010    41.173.220.129
```


Note: Some of you may wonder why I used an XML object in the first place. Often you find code like this:

```
# this is where the xml sample file was saved:
$Path = "$env:temp\inventory.xml"


# load it into an XML object:
[XML]$xml = Get-Content $Path
```

The simple reason is performance. Reading in the XML file as a plain text file via _Get-Content_ and then casting it to XML in a second step is a very expensive approach. Even though our XML file isn&#8217;t that large, the latter solution takes almost 7 times more time than the first one, and this will add up with even larger XML files.

So whenever you want to load an XML file, make sure you get an XML object and use its _Load()_ method. This method is versatile enought by the way to also accept URLs, so you can use an URL to your favorite RSS feed as well &#8211; provided you have direct Internet access and no proxy settings to configure.

### Picking Particular Instances

Let&#8217;s assume you do not want a list of all servers, but instead just want to look up the IP address and the information attribute _info1_ for a specific server in your list. You could use the same approach like this:

<pre class="brush: powershell; title: ; notranslate" title="">$Xml.Machines.Machine |
Where-Object { $_.Name -eq 'Server0009' } |
Select-Object -Property IP, {$_.Information.info1}
</pre>

This would get you the IP address for _&#8220;server0009&#8221;_ plus the _info1_ attribute. Instead of querying all elements and then picking the one you are after on the client side, you can also use XPath, a XML query language:

<pre class="brush: powershell; title: ; notranslate" title="">$item = Select-XML -Xml $xml -XPath '//Machine[Name="Server0009"]'
$item.Node | Select-Object -Property IP, {$_.Information.Info1}
</pre>

The XPath query _&#8220;//Machine[Name=&#8221;Server0009&#8243;]&#8221;_ looks for all &#8220;Machine&#8221; nodes that have a sub-node called _&#8220;Name&#8221;_ with a value of _&#8220;Server0009&#8221;_.

**Important**: XPath is case-sensitive, so if the node name is _&#8220;Machine&#8221;_, then you cannot query for _&#8220;machine&#8221;_.

As a side note, in both approaches you need a script block to access attributes because the attribute _&#8220;info1&#8221;_ is part of a sub-node _&#8220;Information&#8221;_. As always in these scenarios, you can use a hash table to assign a better name to that piece of information:

<pre class="brush: powershell; title: ; notranslate" title="">$info1 = @{Name='AdditionalInfo'; Expression={$_.Information.Info1}}
$item = Select-XML -Xml $xml -XPath '//Machine[Name="Server0009"]'
$item.Node | Select-Object -Property IP, $info1
</pre>

The result will look similar to this:

<pre class="brush: powershell; title: ; notranslate" title="">IP              AdditionalInfo
--              --------------
97.196.140.12   some info
</pre>

XPath is an extremely powerful XML query language. You can find information on its syntax all over the place in the Internet (check these links for example: <http://www.w3schools.com/xpath/> and <http://go.microsoft.com/fwlink/?LinkId=143609>). When you read these documents, you will find that XPath can also use so-called &#8220;user-defined functions&#8221; like _last()_ or _lowercase()_. These functions are not supported here.

#### Changing XML Content

Often, you will want to update information in an XML document. Rather than parsing the XML yourself, simply stick to the techniques you just learned.

So if you wanted to update _Server0006_ and assign it a new name and a different IP address, this is what you would do:

```
$item = Select-XML -Xml $xml -XPath '//Machine[Name="Server0006"]'
$item.node.Name = "NewServer0006"
$item.node.IP = "10.10.10.12"
$item.node.Information.Info1 = 'new attribute info'

$NewPath = "$env:temp\inventory2.xml"
$xml.Save($NewPath)
notepad $NewPath
```

As you can see, updating information is simple, and all changes you make are applied automatically to the underlying XML object. All you need to do is to save the changed XML object to file to make your changes permanent. The result is displayed in the Notepad editor and will look similar to this:

```
<!--6. machine details-->
  <Machine test="559669990">
    <Name>NewServer0006</Name>
    <IP>10.10.10.12</IP>
    <GUID>cca8df99-78e1-48e0-8c4d-193c6d4acbd2</GUID>
    <Information info1="new attribute info" info2="more info">RawContent</Information>
    <CodeSegment info3="another attribute"><![CDATA[this is untouched code and can contain special characters /\@<>]]></CodeSegment>
  </Machine>
```


You have just made changes to an existing XML document in no time, without tricky parsing, and without risking to break XML structure.

In the same way, you can make bulk adjustments. Let&#8217;s assume all the servers are to get brand new names. Instead of _&#8220;ServerXXXX&#8221;_, the machines now need to be named like _&#8220;Prod_ServerXXXX&#8221;_. Here&#8217;s the solution:

```
Foreach ($item in (Select-XML -Xml $xml -XPath '//Machine'))
{
    $item.node.Name = 'Prod_' + $item.node.Name
}

$NewPath = "$env:temp\inventory2.xml"
$xml.Save($NewPath)
notepad $NewPath
```

Note how all server names in the XML document have been updated. _Select-XML_ this time won&#8217;t return just one object but many, one for each server. This is because XPath this time selects all &#8220;Machine&#8221; nodes without special filtering. That&#8217;s why all of these nodes need to be processed in a foreach loop.

Inside of the loop, the node _&#8220;Name&#8221;_ is assigned a new value, and once all &#8220;Machine&#8221; nodes are updated, the XML document is saved and opened in Notepad.

You may argue that in this example, prepending the server name with _&#8220;Prod_&#8221;_ is really a trivial change, and that is true. There may be more complex requirements. However, the focus here is to show how you fundamentally change XML data, not how you do sophisticated string operations.

Still, if you ask yourself how you would, for example, replace _&#8220;ServerXXXX&#8221;_ with _&#8220;PCXX&#8221;_ (including turning a 4-digit number into a 2-digit number, so this definitely is _not_ a trivial change), here is a solution as well:

<pre class="brush: powershell; title: ; notranslate" title="">foreach($item in (Select-XML -Xml $xml -XPath '//Machine'))
{
    if ($item.node.Name -match 'Server(\d{4})')
    {
      $item.node.Name = 'PC{0:00}' -f [Int]$matches[1]
    }
}
$NewPath = "$env:temp\inventory2.xml"
$xml.Save($NewPath)
notepad $NewPath
</pre>

This time, a regular expression extracts the numeric part of the original server name, then the -f operator reformats the number and adds it to the new server prefix.

Neither regular expressions nor number formatting are in the focus of this article. The important part is to see that you are free to use whatever technique you like to construct the new server name. At the end of the day, changing the XML content always sticks to the same rules, though.

### Adding New Data

Occasionally, updating data is not enough. You may want to add a new computer to the list. Again, this is straightforward. You simply pick an existing node, clone it, then update its content and append it to the parent of your liking. This way, you do not have to create the complex node structure yourself and can be certain that the new node is structured just like any of the existing nodes.

This will add a new machine to the list of machines:

```
# clone an existing node structure
$item = Select-XML -Xml $xml -XPath '//Machine[1]'
$newnode = $item.Node.CloneNode($true)

# update the information as needed
# all other information is defaulted to the values from the original node
$newnode.Name = 'NewServer'
$newnode.IP = '1.2.3.4'

# get the node you want the new node to be appended to:
$machines = Select-XML -Xml $xml -XPath '//Machines'
$machines.Node.AppendChild($newnode)

$NewPath = "$env:temp\inventory2.xml"
$xml.Save($NewPath)
notepad $NewPath
```

Since the node you are adding is cloned from an existing node, all information in this new node is copied from the existing node. Information that you do not update will keep the old values.

And what if you wanted to add the new node to the top of the list? Simply use _InsertBefore()_ instead of _AppendChild()_:

<pre class="brush: powershell; title: ; notranslate" title=""># add it to the top of the list:
$machines.Node.InsertBefore($newnode, $item.node)
</pre>

Likewise, you can basically insert the new node anywhere. This would insert it right after _Server0007_:

<pre class="brush: powershell; title: ; notranslate" title=""># add it after "Server0007":
$parent = Select-XML -Xml $xml -XPath '//Machine[Name="Server0007"]'
$machines.Node.InsertAfter($newnode, $parent.node)
</pre>

### Removing XML Content

Deleting data entirely from your XML file is just as easy. If you wanted to remove _Server0007_ from your list, here&#8217;s how:

<pre class="brush: powershell; title: ; notranslate" title=""># remove "Server0007":
$item = Select-XML -Xml $xml -XPath '//Machine[Name="Server0007"]'
$null = $item.Node.ParentNode.RemoveChild($item.node)
</pre>

### Enormous Power at Your Fingertips

With the examples presented, you can now manage the most commonly needed XML manipulations in just a couple of lines of code. It is well worth investing some time into improving your XML and XPath proficiency &#8211; you can do amazing things with them.

And for those of you that have sticked with me this long, I have a little present for you: a great little tool I use very often that can be very helpful for you, too, I am sure. It uses the exact same tactics you just heard about. Here&#8217;s the story:

_ConvertTo-XML_ can convert any object into XML, and since XML is a hierarchical data format, preserving structure up to a given depth, it is an excellent way of examining nested object properties. So you can &#8220;unfold&#8221; an object structure and look at all of its properties, even the deeply nested ones.

Without XML and XPath, all you could do is look at plain XML and search for information yourself. For example, if you wanted to find out where exactly the $host object stores PowerShell’s color information, you could do this (which might be not such a good idea after all because you get flooded with raw XML information):

<pre class="brush: powershell; title: ; notranslate" title="">$host | ConvertTo-XML -Depth 5 | Select-Object -ExpandProperty outerXML
</pre>

With the knowledge just presented, you could now take the raw XML and extract and filter the object properties.

So here&#8217;s the promised function called _Get-ObjectProperty_ which works a little bit like _Get-Member_ on steroids. It can tell you which property inside an object holds the value you are after. Have a look:

```
PS> $host | Get-ObjectProperty -Depth 2 -Name *color*
Name                    Value                   Path                    Type
----                    -----                   ----                    ----
TokenColors                                     $obj1.PrivateData.To... Microsoft.PowerShel...
ConsoleTokenColors                              $obj1.PrivateData.Co... Microsoft.PowerShel...
XmlTokenColors                                  $obj1.PrivateData.Xm... Microsoft.PowerShel...
ErrorForegroundColor    #FFFF0000               $obj1.PrivateData.Er... System.Windows.Medi...
ErrorBackgroundColor    #FFFFFFFF               $obj1.PrivateData.Er... System.Windows.Medi...
WarningForegroundColor  #FFFF8C00               $obj1.PrivateData.Wa... System.Windows.Medi...
WarningBackgroundColor  #00FFFFFF               $obj1.PrivateData.Wa... System.Windows.Medi...
VerboseForegroundColor  #FF00FFFF               $obj1.PrivateData.Ve... System.Windows.Medi...
VerboseBackgroundColor  #00FFFFFF               $obj1.PrivateData.Ve... System.Windows.Medi...
DebugForegroundColor    #FF00FFFF               $obj1.PrivateData.De... System.Windows.Medi...
DebugBackgroundColor    #00FFFFFF               $obj1.PrivateData.De... System.Windows.Medi...
ConsolePaneBackgroun... #FF012456               $obj1.PrivateData.Co... System.Windows.Medi...
ConsolePaneTextBackg... #FF012456               $obj1.PrivateData.Co... System.Windows.Medi...
ConsolePaneForegroun... #FFF5F5F5               $obj1.PrivateData.Co... System.Windows.Medi...
ScriptPaneBackground... #FFFFFFFF               $obj1.PrivateData.Sc... System.Windows.Medi...
ScriptPaneForeground... #FF000000               $obj1.PrivateData.Sc... System.Windows.Medi...
```

This will return all nested properties inside of $host that have _&#8220;Color&#8221;_ in its name. Console output most likely is truncated, so you are better off displaying the information in a grid view window:

<pre class="brush: powershell; title: ; notranslate" title="">$host | Get-ObjectProperty -Depth 2 -Name *color* | Out-GridView
</pre>

Note the column _&#8220;Path&#8221;_: this property specifies exactly how you would access a given nested property. In the example, _Get-ObjectProperty_ walks two levels deep inside the object hierarchy. Greater depths will unfold even more information but will also pollute the results with more irrelevant noise information.

While you can pipe in multiple objects, it is best to pipe only one object due to the large amount of resulting data. This line would list all nested properties in a process object, five levels deep, that have a numeric value:

```
PS> Get-Process -id $pid | Get-ObjectProperty -Depth 5 -IsNumeric
Name                    Value                   Path                    Type
----                    -----                   ----                    ----
Handles                 684                     $obj1.Handles           System.Int32
VM                      1010708480              $obj1.VM                System.Int32
WS                      291446784               $obj1.WS                System.Int32
PM                      251645952               $obj1.PM                System.Int32
NPM                     71468                   $obj1.NPM               System.Int32
CPU                     161,0398323             $obj1.CPU               System.Double
BasePriority            8                       $obj1.BasePriority      System.Int32
HandleCount             684                     $obj1.HandleCount       System.Int32
Id                      4560                    $obj1.Id                System.Int32
Size                    264                     $obj1.MainModule.Size   System.Int32
ModuleMemorySize        270336                  $obj1.MainModule.Mod... System.Int32
FileBuildPart           9421                    $obj1.MainModule.Fil... System.Int32
FileMajorPart           6                       $obj1.MainModule.Fil... System.Int32
FileMinorPart           3                       $obj1.MainModule.Fil... System.Int32
ProductBuildPart        9421                    $obj1.MainModule.Fil... System.Int32
ProductMajorPart        6                       $obj1.MainModule.Fil... System.Int32
ProductMinorPart        3                       $obj1.MainModule.Fil... System.Int32
Size                    264                     $obj1.Modules[0].Size   System.Int32
ModuleMemorySize        270336                  $obj1.Modules[0].Mod... System.Int32
(...)
```

And this line would return all nested properties of the spooler service object that is of type _&#8220;String&#8221;_:

```
PS> Get-Service -Name spooler | Get-ObjectProperty -Type System.String
Name                    Value                   Path                    Type
----                    -----                   ----                    ----
Name                    spooler                 $obj1.Name              System.String
Name                    RPCSS                   $obj1.RequiredServic... System.String
Name                    DcomLaunch              $obj1.RequiredServic... System.String
DisplayName             DCOM Server Process ... $obj1.RequiredServic... System.String
MachineName             .                       $obj1.RequiredServic... System.String
ServiceName             DcomLaunch              $obj1.RequiredServic... System.String
Name                    RpcEptMapper            $obj1.RequiredServic... System.String
DisplayName             RPC Endpoint Mapper     $obj1.RequiredServic... System.String
(...)
```

And here&#8217;s the source code for _Get-ObjectProperty_. It is slightly more complex than just a couple of lines but still amazingly short, given the job it does for you.

It utilizes the exact same techniques that were just explained, so once you feel comfortable with the simple examples above, you can try and digest this one as well &#8211; or simply use it as a tool and not worry about its XML magic:


    Function Get-ObjectProperty
    {
        param
        (
            $Name = '*',
            $Value = '*',
            $Type = '*',
            [Switch]$IsNumeric,
            [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
            [Object[]]$InputObject,
            $Depth = 4,
            $Prefix = '$obj'
        )
       
    	Begin
      	{
        	$x = 0
        	Function Get-Property
        	{
                param
                (
                	$Node,
                	[String[]]$Prefix
                )
    
          		$Value = @{Name='Value'; Expression={$_.'#text' }}
         		Select-Xml -Xml $Node -XPath 'Property' | ForEach-Object {$i=0} {
            		$rv = $_.Node | Select-Object -Property Name, $Value, Path, Type
            		$isCollection = $rv.Name -eq 'Property'
        
            		if ($isCollection)
            		{
              			$CollectionItem = "[$i]"
              			$i++
              			$rv.Path = (($Prefix) -join '.') + $CollectionItem
            		}
            		else
            		{
              			$rv.Path = ($Prefix + $rv.Name) -join '.'
            		}
        
            		$rv
        
            		if (Select-Xml -Xml $_.Node -XPath 'Property')
            		{
              			if ($isCollection)
              			{
                            $PrefixNew = $Prefix.Clone()
                            $PrefixNew[-1] += $CollectionItem
                            Get-Property -Node $_.Node -Prefix ($PrefixNew )
                          }
                          else
                          {
                			Get-Property -Node $_.Node -Prefix ($Prefix + $_.Node.Name )
              			 }
            		}
          		}
        	}
          	$Value = @{Name='Value'; Expression={$_.'#text' }}
    
      		Select-Xml -Xml $Node -XPath 'Property' | ForEach-Object {$i=0} {
    			$rv = $_.Node | Select-Object -Property Name, $Value, Path, Type
    			$isCollection = $rv.Name -eq 'Property'
                if ($isCollection)
                {
    				$CollectionItem = "[$i]"
    				$i++
    				$rv.Path = (($Prefix) -join '.') + $CollectionItem
    			}
    			else
    			{
    				$rv.Path = ($Prefix + $rv.Name) -join '.'
                 }
                 $rv
    			if (Select-Xml -Xml $_.Node -XPath 'Property')
    			{
    				if ($isCollection)
    				{
    					$PrefixNew = $Prefix.Clone()
    					$PrefixNew[-1] += $CollectionItem
    					Get-Property -Node $_.Node -Prefix ($PrefixNew )
                     }
                     else
                     {
                      	Get-Property -Node $_.Node -Prefix ($Prefix + $_.Node.Name )
                     }
    			}
             }
    	}
    }
    
        Process
        {
            $x++
            $InputObject |
            ConvertTo-Xml -Depth $Depth |
            ForEach-Object { $_.Objects } |
            ForEach-Object { Get-Property $_.Object -Prefix $Prefix$x  } |
            Where-Object { $_.Name -like "$Name" } |
            Where-Object { $_.Value -like $Value } |
            Where-Object { $_.Type -like $Type } |
            Where-Object { $IsNumeric.IsPresent -eq $false -or $_.Value -as [Double] }
        }
    }


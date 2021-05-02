---
title: A JSON primer for administrators
author: Ravikanth C
type: post
date: 2014-12-01T17:00:47+00:00
url: /2014/12/01/a-json-primer-for-administrators/
categories:
  - How To
  - JSON
tags:
  - How To
  - JSON
---
JavaScript Object Notation ([JSON][1]) is quickly becoming the most used data-interchange format. XML enjoyed the top spot for a long time but slowly people have been moving towards JSON. JSON is a first-class citizen in Windows PowerShell. With many automation frameworks and software products adopting JSON, it is important for system administrators to understand what is JSON and how to use it. This article is a JSON primer to give you an overview of this data-interchange format and how to use it.

JSON facilitates structured data-interchange between programming languages. [ECMA-404 specification][2] describes the JSON data-interchange format. Although JSON carries JavaScript in its name, you don&#8217;t need to know [JavaScripting][3] to be able to learn and use JSON. Its name comes from the fact that the JSON format is derived from the JavaScript object literal syntax.

<pre class="brush: jscript; title: ; notranslate" title="">var car = {
   type:  "Fiat",
   model: 500,
   color: "white"
};
</pre>

As you see in the above example, the JavaScript object literal is nothing but a key-value pair. The JSON data-interchange format is similar to this with a few differences. Let&#8217;s explore!

### Introducing JSON data-interchange format

If you have worked with different programming or scripting languages, each of them have their own data representation. In this era of cloud and mobile services, the services built using one programming language are generally consumed by a myriad of clients built in various languages. Take an example of services offered by Microsoft Azure. Most or all of them are written in C# and offer APIs with JSON as the output data format. We can consume these services in PowerShell, Python, and many more languages with libraries to translate the JSON data into a native data representation within those languages. This is how we have been using XML for a long time. So, this should be straightforward to understand. Then, why JSON? This topic is highly debatable. However, I like to quote just one example. You can read [rest here][4].

XML is quite verbose. Just think about representing the above JavaScript object literal in XML.

<pre class="brush: xml; title: ; notranslate" title="">&lt;Car&gt;
   &lt;Type&gt;Fiat&lt;/Type&gt;
   &lt;Model&gt;500&lt;/Model&gt;
   &lt;Color&gt;White&lt;/Color&gt;
&lt;/Car&gt;
</pre>

Now, let us see the same representation in JSON. Don&#8217;t worry about the details of the JSON representation but just see how clean it is compared to XML.

<pre class="brush: jscript; title: ; notranslate" title="">{
"Car" : {
   "Type"  : "Fiat"
   "Model" : 500
   "Color" : "White"
   }
}
</pre>

What we have seen above is a very simple example. Imagine the verbosity of XML when the data that needs to be represented is huge. This is certainly one of the reasons for me to choose between XML and JSON. With this background, let us go into the details of JSON data format.

To start with, the general syntax for writing JSON objects is as follows:

<pre class="brush: jscript; title: ; notranslate" title="">{
   "Data1" : "Value1",
   "Data2" : "Value2"
}
</pre>

As you see in this example, the key-value pair is enclosed within curly brackets. The keys, _Data1_ and _Data2_ as shown in the example, must always be enclosed within double quotes. This is one key difference from the JavaScript object literal syntax. The key and value are separated by a colon &#8220;:&#8221;. Also, observe that the key-value pairs (or members) must be separated by commas.

You can nest objects in JSON. Here is a generic example.

<pre class="brush: jscript; title: ; notranslate" title="">{
   "Data1" : "Value1",
   "Data2" : {
        "Data21" : "Value21"
   }
}
</pre>

**Whether we need to enclose the value within double quotes or not depends on the type of the data**.

### Data Types in JSON

A JSON value can be  of a basic data type such as string, number, and boolean. You can create ordered list of values such as arrays. And, as shown above, you can use unordered key-value pairs within JSON. JSON also supports empty values denoted by null.

Here are some examples of data types and how they are used in JSON.

#### Numbers

In JSON data format, numbers represent a sequence of digits. There is no differentiation between floats, doubles, and so on. **JSON values cannot be hexadecimal or octal.** Here is an example of representing numbers in JSON.

<pre class="brush: powershell; title: ; notranslate" title="">{
   "age" : 30,
   "speed" : 10.12,
   "somenumber" : 1.778979793e+23,
   "negativenumber" : -1212
}
</pre>

As you see in the above example, we can represent different types of numbers including exponential and negative numbers. **When the value you want to store in JSON format is a number, do not enclose it within the quotes.**

#### Strings

Like every other programming or scripting language, a string represents a sequence of characters.

<pre class="brush: jscript; title: ; notranslate" title="">{
   "Name" : "Windows PowerShell",
   "Tab" : "Windows\tPowerShell",
   "NewLine" : "Windows\nPowerShell",
   "Backspace" : "Windows\bPowerShell",
   "CarriageReturn" : "Windows\rPowerShell",
   "WithQuotes" : "Windows \"PowerShell\"",
   "HexCharacters" : "Windows PowerShell \u1F44D"
}
</pre>

I have included several ways of representing string values in the above example. You see that all the values are enclosed within double quotes. **Using single quotes is not allowed in JSON**. When you need to use double quotes within the value string, you need to escape the same using the escape sequences. There are other escape sequences also shown in the above example. If you need to include special Unicode characters derived from hex representation, the same can be done using _\u_ escape sequence.

#### Booleans

Boolean values can be represented in JSON using _true_ or _false_.

<pre class="brush: jscript; title: ; notranslate" title="">{
    "Test1" : true,
    "Test2" : false
}
</pre>

Remember that the values representing Boolean values should be always in lower case. Using _True_ and _False_ is not valid.

#### Null

You can also create empty values by assigning null to a key.

<pre class="brush: powershell; title: ; notranslate" title="">{
"Name" : null
}
</pre>

Make a note that the value you assign must be null in lowercase. **Using _Null_ (uppercase) results in an error.**

#### Arrays

Arrays are an ordered list of values. Within the JSON data format, these values are enclosed within square brackets ([ and ]).

<pre class="brush: powershell; title: ; notranslate" title="">{
 	"Array1" : ["PowerShell", "Python", "Perl"],
 	"Array2" : [10, 11, 12],
 	"Array3" : ["Ravi",10,true]
}
</pre>


As you see in the example, you can assign the ordered lists or arrays to JSON keys. It is possible to mix data types (see _Array3_) within the arrays. However, this is not recommended as not all programming languages support mixed data types in arrays. So, when the JSON data is passed on to another language, there may be errors. Avoid mixing data types within in arrays in JSON.

#### Objects

Finally, objects within JSON data are the unordered set of key-value pairs. Essentially, by using objects within JSON data, you create nested JSON data representation.

<pre class="brush: jscript; title: ; notranslate" title="">{
   "Name" : "Windows PowerShell",
   "Version" : "4.0.30319.18444",
   "IsStable" : true,
   "PreviousVersions" : [1.0,2.0,3.0],
   "Future" : {
      "Version" : 5.0,
      "IsReady" : false
   }
}
</pre>

The above example is a comprehensive one. It shows all data types along with nested objects. This ends our discussion on the data types in JSON. Now, how do you validate whether your JSON format is valid or not? I simply use the _ConvertFrom-Json_ cmdlet.

<pre class="brush: powershell; title: ; notranslate" title="">$json = @'
{
   "Name" : "Windows PowerShell",
   "Version" : "4.0.30319.18444",
   "IsStable" : true,
   "PreviousVersions" : [1.0,2.0,3.0],
   "Future" : {
      "Version" : 5.0,
      "IsReady" : false
   }
}
'@
$Object = ConvertFrom-Json -InputObject $json

![](/images/json1.png)

This is just a primer like I mentioned earlier. I suggest that you start looking at using JSON wherever possible. It will be an essential skill to have. I will show you why in my future posts here! 🙂

[1]: http://json.org/
[2]: http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf
[3]: http://en.wikipedia.org/wiki/JavaScript
[4]: https://www.google.co.in/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#sourceid=chrome-psyapi2&ie=UTF-8&q=JSON%20vs%20XML
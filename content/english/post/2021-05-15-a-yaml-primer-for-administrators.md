---
title: A YAML primer for administrators
author: Ravikanth C
images:
  - '/images/yaml.png'
type: epic
date: 2021-05-15
url: /2021/05/15/a-yaml-primer-for-administrators/
categories:
- YAML
tags:
- YAML
---

In today's cloud and agile infrastructure world, JSON and YAML play a major role in describing the infrastructure and data exchange. In many situations, the choice between JSON or YAML really depends on the application or infrastructure you are dealing with. For example, [Ansible playbooks](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) use YAML, [Azure Resource Manager (ARM) templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/) are written in JSON, [AWS Cloudformation](https://aws.amazon.com/cloudformation/) supports both YAML and JSON formats, [GitHub actions workflows](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions) are written in YAML, and [Kubernetes uses YAML](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) for almost everything.

If you are integrating two different products or services, you would choose a data representation format that is common to these applications or services as a data exchange format. I have, in my personal projects, mostly used JSON for data representation since handling JSON is built into PowerShell and I do not need an external module to handle that data. However, given the number of other things that I am working on, I bump into YAML once in a while. Initially, I found it difficult to switch the context. I made some notes as I started using and writing YAML. This article is coming directly from those notes. If you are new to both JSON and YAML, you may also want to read a [JSON primer for administrators](/2014/12/01/a-json-primer-for-administrators/). If you are still stuck to XML for whatever reasons, [Tobias](/author/tobias-weltner/) wrote a great article on working with [XML in PowerShell](/2013/08/19/mastering-everyday-xml-tasks-in-powershell/).

### What is YAML?

[YAML](https://yaml.org/) -- YAML Ain't Markup Language -- is a human-friendly data-serialization / data-representation language. Compared to JSON and XML, YAML is less verbose. Instead of brackets, YAML depends on indentation and that stumped me multiple times when I was starting. In YAML, you always use spaces for indentation and not tabs. For someone coming from PowerShell background and who religiously used tabs in the code, it took a while to get used to spaces. But, believe me, YAML is very easy to write and read.

> Examples in this article were validated / parsed using the [PowerShell-Yaml module](https://www.powershellgallery.com/packages/powershell-yaml). 

There are three basic data types that YAML supports -- lists, maps, and scalars.

#### Lists

The most basic type of data structure in YAML representation is a list. You can also call this a sequence.

```yaml
- PowerShell
- Python
- Go
- Rust
```

This is a simple list of items where each item starts with a `-`. Using the inline format, this can be folded into a single line

```yaml
[PowerShell, Python, Go, Rust]
```

In either case when you parse this using `ConvertFrom-Yaml`, you get a generic collection.

```powershell
PS> $list = ConvertFrom-Yaml -Yaml (Get-Content -Path .\list.yaml -Raw)
PS> $list
PowerShell
Python
Go
Rust
```

You can create a list of lists or a sequence of sequences.

```yaml
- 
 - PowerShell
 - Python
 - Rust
 - Ruby
-
 - C
 - C++
 - C#
 - Go
```

#### Maps

Maps are key-value pairs. Maps use a `:` and space to separate key and its value. Items in a map are scalars which you will learn later in this article.

```yaml
name: Ravikanth C
url: '/author/ravikanth-c'
```

You can map combine maps and lists together.

```yaml
name: Ravikanth C
url: '/author/ravikanth-c'
languages:
 - PowerShell
 - Python
 - Go
 - Rust
```

When you parse this using `ConvertFrom-Yaml`, you get an ordered dictionary.

```powershell
PS> $map = ConvertFrom-Yaml -Yaml (Get-Content -Path .\list.yaml -Raw)
PS> $map                  

Name                           Value
----                           -----
name                           Ravikanth C
url                            /author/ravikanth-c
languages                      {PowerShell, Python, Go, Rust}
```

You can create a list (sequence) of maps as well.

```yaml
-
 name: 'Ravikanth C'
 url: '/author/ravikanth-c'
 languages: [PowerShell, Python, Go]
-
 name: 'Alexandair'
 url: '/author/aleksandar-nikolic'
 languages: [PowerShell, Python, Go]
```

Notice the indentation after the each `-`. This is mandatory.

You can create map of maps as well. This is done by simply specifying as key-value pairs.

```yaml
ravikanth: 
 name: Ravikanth C
 url: '/author/ravikanth-c'
 languages:
  - PowerShell
  - Python
  - Go
alex:
 name: Alexandair
 url: '/author/aleksandar-nikolic'
 languages:
  - PowerShell
  - Python
  - Go
```

Here is how `ConvertFrom-Yaml` translates this into PowerShell ordered dictionary.

```powershell
PS C:\sandbox\yaml> $map = ConvertFrom-Yaml -Yaml (Get-Content -Path .\list.yaml -Raw)
PS C:\sandbox\yaml> $map[0]

Name                           Value
----                           -----
name                           Ravikanth C
url                            /author/ravikanth-c     
languages                      {PowerShell, Python, Go}

PS C:\sandbox\yaml> $map.ravikanth

Name                           Value
----                           -----
name                           Ravikanth C
url                            /author/ravikanth-c     
languages                      {PowerShell, Python, Go}
```

In the above example, the list of languages is same for both nodes -- ravikanth and alex. So, is there a way you can just reference the same list at both places? Yes. YAML provides anchors and aliases for this purpose. Take a look at this updated example.

```yaml
ravikanth: 
 name: Ravikanth C
 url: '/author/ravikanth-c'
 languages: &LANG
  - PowerShell
  - Python
  - Go
alex:
 name: Alexandair
 url: '/author/aleksandar-nikolic'
 languages:
  - *LANG
```

In the first map, the `&LANG` identifies an anchor. The literal text LANG after the & symbol can be anything of your choice. Wherever you want to repeat the same list within the YAML document, you can simply use `*LANG`.

```powershell
PS C:\sandbox\yaml> $map = ConvertFrom-Yaml -Yaml (Get-Content -Path .\list.yaml -Raw)
PS C:\sandbox\yaml> $map[1]

Name                           Value
----                           -----
name                           Alexandair
url                            /author/aleksandar-nikolic      
languages                      {PowerShell, Python, Go}
```

#### Scalars

Strings, numbers, bools, and null are considered scalar data types. Here is a quick example that shows all these data types in YAML representation.

```yaml
name: Ravikanth C
url: '/author/ravikanth-c'
tagline: "Cloud and Infrastructure Technologist, Author, Microsoft MVP, Public Speaker, and Innovator"
description: null
numPosts: 300
isEditor: true
```

In this example of a YAML map, `numPosts` represents a number data type. The value assigned to a key like this can be an integer or a float or a hex or an octal or even an exponential. 

The `description` property is set to `null` in the above example. `null` is another allowed scalar type.

Similar to JSON, boolean values are represented using `true` and `false`. In the above example, the `isEditor` property is set to a boolean value `true`.

The properties `name`, `url`, and `tagline` are all strings. YAML supports both double and single quotes unlike JSON where only double quotes are allowed. If you notice, the value for property `name` is not enclosed in any quotes. This is totally valid in YAML. Quotes may become necessary only when there are special characters (such as `:`,`{`,`}`,`[`,`]`,`,`,`&`,`*`,`#`,`?`,`|`,`-`,`<`,`>`,`=`,`!`,`%`,`@`,`\`) in the value assigned to a key. Also, when you want a number type to be considered as string, you have to enclose the value in quotes. For example, `numPosts: '300'`. For string values, it is always a good practice to use quotes.

##### Flow Style

The `name`, `url`, and `tagline` properties in the above example are of [flow scalar type](https://yaml.org/spec/1.2/spec.html#id2786942) in YAML. `name` is an unquoted or plain style scalar, `url` is a single-quoted style scalar, and `tagline` is a double-quoted style scalar. This type of scalars support limited escaping. You can understand this easily with an example.

```yaml
name: Ravikanth C
url: '/author/ravikanth-c'
tagline: 'Cloud and Infrastructure Technologist, Author, Microsoft MVP, Public Speaker, and Innovator'
description: 'Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on
  software-defined Storage and hybrid cloud architectures.
  He is a ''published'' author and an innovator with many granted patents 
  and pending applications. He is a multi-year recipient of Microsoft''s Most Valuable Professional (MVP) award in Cloud and Data Center Management. 
  
    Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro
  and Bangalore PowerShell User Groups.'
numPosts: 300
isEditor: true
```

In the above example, the description is updated from a null value to a multi-line string. YAML supports multi-line strings as values. If you notice, this mulit-line string is enclosed in single-quotes and escaped single-quotes at two places. There is also `\n` as the newline character.

Save the above example as authors.yaml and run the following commands.

```powershell
$yaml = Get-content -Path .\authors.yaml -Raw
$author = ConvertFrom-Yaml -Yaml $yaml
$author.description
```

This will produce output as shown below.

```powershell
PS C:\sandbox\yaml> $author.description
Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on software-defined Storage and hybrid cloud architectures. He is a 'published' author and an innovator with many granted patents and pending applications. He is a multi-year recipient of Microsoft's Most Valuable Professional (MVP) award in Cloud and Data Center Management.
Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro and Bangalore PowerShell User Groups.
```

The single-quote escaping worked as expected. However, although the second line in the original value started on a newline, the parsed output does not honor that. The blank line before the last line resulted in a newline. This demonstrates how the escaping is limited in flow style scalars. If you want to control escaping in the multi-line string and preserve formatting, you need to use block style scalars.

##### Block Style

The [block scalar](https://yaml.org/spec/1.2/spec.html#id2793652) type in YAML gives better control over how escaping in a scalar is interpreted. Block style has three different parts. First, one of the two block style indicators to specify how newline characters should be treated. The literal block style, specified using `|` character, will preserve newlines as is.

```yaml
name: Ravikanth C
url: '/author/ravikanth-c'
tagline: 'Cloud and Infrastructure Technologist, Author, Microsoft MVP, Public Speaker, and Innovator'
description: | 
  'Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on
  software-defined Storage and hybrid cloud architectures.
  He is a ''published'' author and an innovator with many granted patents 
  and pending applications. He is a multi-year recipient of Microsoft''s 
  Most Valuable Professional (MVP) award in Cloud and Data Center Management.
  Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro
  and Bangalore PowerShell User Groups.'
numPosts: 300
isEditor: true
```

If you parse this using PowerShell-Yaml, you will see:

```powershell
PS C:\sandbox\yaml> $author.description                        
'Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on
software-defined Storage and hybrid cloud architectures.
He is a ''published'' author and an innovator with many granted patents    
and pending applications. He is a multi-year recipient of Microsoft''s     
Most Valuable Professional (MVP) award in Cloud and Data Center Management.
Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro 
and Bangalore PowerShell User Groups.'

PS C:\sandbox\yaml> ($author.description  -split '\r?\n').count
8
```

The difference between flow and block style should already be clear. The newlines are all preserved.

In the folded block style, you use the angular bracket `>` at the beginning of the scalar value.

```yaml
name: Ravikanth C
url: '/author/ravikanth-c'
tagline: 'Cloud and Infrastructure Technologist, Author, Microsoft MVP, Public Speaker, and Innovator'
description: > 
  'Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on
  software-defined Storage and hybrid cloud architectures.
  He is a ''published'' author and an innovator with many granted patents 
  and pending applications. He is a multi-year recipient of Microsoft''s 
  Most Valuable Professional (MVP) award in Cloud and Data Center Management.
  Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro
  and Bangalore PowerShell User Groups.'
numPosts: 300
isEditor: true
```

The folded style replaces all newline characters within the string with a space. 

```powershell
PS C:\sandbox\yaml> $author.description
'Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on software-defined Storage and hybrid cloud architectures. 
He is a ''published'' author and an innovator with many granted patents  and pending applications. He is a multi-year recipient of Microsoft''s  Most Valuable Professional (MVP) award in Cloud and Data Center Management. Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro and Bangalore PowerShell User Groups.'

PS C:\sandbox\yaml> ($author.description  -split '\r?\n').count
2
```

> As shown in the above example, when using either literal or folded block styles, you must give a line break after the style indicator. 

By default, when using folded style, the trailing newline is preserved and any trailing empty lines are all removed. This behavior can be modified using the chomping indicators. There are two [chomping operators](https://yaml.org/spec/1.2/spec.html#id2794534) within the YAML syntax.

The `-` indicator is used to ***strip*** the final line break and any trailing empty lines.

```yaml
description: >-
  'Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on
  software-defined Storage and hybrid cloud architectures.
  He is a ''published'' author and an innovator with many granted patents 
  and pending applications. He is a multi-year recipient of Microsoft''s 
  Most Valuable Professional (MVP) award in Cloud and Data Center Management.

  Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro
  and Bangalore PowerShell User Groups.

  '
```

The `+` indictor is used to ***keep*** the final line break and any trailing empty lines.

```yaml
description: >+
  'Ravikanth is a Distinguished Member Technical Staff at Dell EMC working on
  software-defined Storage and hybrid cloud architectures.
  He is a ''published'' author and an innovator with many granted patents 
  and pending applications. He is a multi-year recipient of Microsoft''s 
  Most Valuable Professional (MVP) award in Cloud and Data Center Management.

  Ravikanth is the founder of PowerShell Magazine and leads Bangalore IT Pro
  and Bangalore PowerShell User Groups.

  '
```

I have not come across scenarios (yet!) where I need to use chomping indicators. So, whatever is the default behavior of handling multi-line strings seems to be good so far.

This is what I have learned in my experiments with YAML so far. Have you figured out any tips and tricks? Post here as comments.
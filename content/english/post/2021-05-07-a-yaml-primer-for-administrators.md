---
title: A YAML primer for administrators
author: Ravikanth C
type: epic
date: 2021-05-07
url: /2021/05/07/a-yaml-primer-for-administrators/
categories:
- YAML
tags:
- YAML
---

In today's cloud and agile infrastructure world, JSON and YAML play a major role in describing the infrastructure and data exchange. In many situations, the choice between JSON or YAML really depends on the application or infrastructure you are dealing with. For example, [Ansible playbooks](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) use YAML, [Azure Resource Manager (ARM) templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/) are written in JSON, and [AWS Cloudformation](https://aws.amazon.com/cloudformation/) supports both YAML and JSON formats. If you are integrating two different products or services, you would choose a data representation format that is common to these applications or services as a data exchange format. I have, in my personal projects, mostly used JSON for data representation since handling JSON is built into PowerShell and I do not need an external module to handle that data. However, given the number of other things that I am working on, I bump into YAML once in a while. Initially, I found it difficult to switch the context. I made some notes as I started using and writing YAML. This article is coming directly from those notes. If you are new to both JSON and YAML, you may want to start with a [JSON primer for administrators](/2014/12/01/a-json-primer-for-administrators/). If you are still stuck to XML for whatever reasons, [Tobias](/author/tobias-weltner/) wrote a great article on working with [XML in PowerShell](/2013/08/19/mastering-everyday-xml-tasks-in-powershell/).

### What is YAML?

[YAML](https://yaml.org/) -- YAML Ain't Markup Language -- is a human friendly data-serialization / data-representation language. Compared to JSON and XML, YAML is less verbose. Instead of brackets, YAML depends on indentation and that stumped me multiple times. In YAML, you always use spaces for indentation and not tabs. For someone coming from PowerShell background and who religiously used tabs in the code, it took a while to get used to spaces. But, believe me, YAML is very easy to write and read.

YAML data representation uses key-value pairs. There are three basic data types that YAML supports -- scalars, lists, and maps. 

### Scalars

Strings, numbers, booleans, and null are considered scalar data types. Here is a quick example that shows all these data types in YAML representation.

```yaml
name: Ravikanth C
url: '/author/ravikanth-c'
tagline: "Cloud and Infrastructure Technologist, Author, Microsoft MVP, Public Speaker, and Innovator"
description: null
numPosts: 300
isEditor: true
```

In this example, `numPosts` represents a number data type. The value assigned to key like this can be an integer or float or hexa or octal or even an exponential.

The properties `name`, `url`, and `tagline` are all strings. YAML supports both double and single quotes unlike JSON where only double quotes are supported. If you notice, the value for property `name` is not enclosed in any quotes. This is totally valid in YAML. Quotes may become mandatory only when there are special characters (such as `:`,`{`,`}`,`[`,`]`,`,`,`&`,`*`,`#`,`?`,`|`,`-`,`<`,`>`,`=`,`!`,`%`,`@`,`\`) in the value assigned to a key. Also, when you want a number type to be considered as string, you have to enclose the value in quotes. For example, `numPosts: '300'`. For string values. it is always a good practice to use quotes.

The description property is set to `null` in the above example.


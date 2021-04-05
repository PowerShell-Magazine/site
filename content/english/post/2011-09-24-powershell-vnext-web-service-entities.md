---
title: PowerShell vNext – Web Service Entities
author: Doug Finke
type: post
date: 2011-09-24T12:40:31+00:00
url: /2011/09/24/powershell-vnext-web-service-entities/
featured_image: /wp-content/uploads/2011/09/odata.png
views:
  - 16750
post_views_count:
  - 1639
categories:
  - News
  - oData API
tags:
  - News
  - API

---
With the release of [Windows 8][1] and [PowerShell V3 CTP1][2], Microsoft added [Management ODATA Web Services Dev. Tools][3] which lets you create PowerShell web services.

### What is Management OData?

Management OData is an infrastructure for creating a web service endpoint that exposes your PowerShell cmdlets and scripts, as OData Web service entities.

OData ([Open Data Protocol][4]) is a Web protocol for querying and updating data that provides a way to unlock your data and free it from silos that exist in applications today. OData presents resources as a set of database-like tables with a definite schema. OData does this by applying and building upon Web technologies such as HTTP, Atom Publishing Protocol (AtomPub) and JSON to provide access to information from a variety of applications, services, and stores.

Good news, [PowerShell V3 supports consuming JSON][5] out of the box.

### Examples

The tools provide a walkthrough creating two endpoints, one for Processes and the other for Services. They use Get-Process, Get-Service, and Set-Service under the covers. Once setup, you can access it this way:

http://localhost:7000/MODataSvc/Microsoft.Management.Odata.svc/Processes

For this to work you need to install Windows Server vNext.

### More Technical Details

A Management OData server uses two types of schema files to define its resources: a public schema and a private schema. It defines its resources in terms of the CIM model using the MOF file format. Internally translates these resource definitions into the CSDL format that is used by the ODATA protocol.

The Management ODATA feature exposes a structured, data-oriented schema on top of PowerShell artifacts.

The tools supplied assists in the creation of these files.

### Conclusion

Being able to whip up scripts and expose their results to the web as Urls supplying a JSON format is a big deal. For example, Twitter provides search results as JSON as does Facebook.

Now it is possible to deliver data from existing sources or repurpose it by combining sources and presenting over the Web in a well-known format.

In turn it can be used by applications like:

  * Other PowerShell scripts
  * Windows Phone 7 (Mango)
  * PHP (WordPress Blogs)
  * Java and Andriod applications
  * iPad/iPhone applications
  * Drupal
  * Joomla

PowerShell keeps getting better and better.

[1]: http://msdn.microsoft.com/en-us/windows/apps/br229516
[2]: http://www.microsoft.com/download/en/details.aspx?id=27548&utm_source=feedburner&utm_medium=twitter&utm_campaign=Feed:+MicrosoftDownloadCenter+$Microsoft+Download+Center$
[3]: http://archive.msdn.microsoft.com/mgmtODataWebServ
[4]: http://www.odata.org/
[5]: http://www.dougfinke.com/blog/index.php/2011/09/15/powershell-v3-and-json/
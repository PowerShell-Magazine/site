---
title: The Case of Unknown Custom Object Property
author: Ravikanth C
type: post
date: 2020-02-10T15:30:16+00:00
url: /2020/02/10/the-case-of-unknown-custom-object-property/
post_views_count:
  - 12621
views:
  - 12687
categories:
  - Debugging
tags:
  - Debugging

---
In the last two or three months, I have been busy working on a complete end to end deployment automation solution and been writing lot of PowerShell code. Literally, lot of code. A simple to use deployment automation solution always has complex implementation beneath. I was dealing with several configuration files, dynamically deciding what configuration to fetch and update, and dealing with hundreds of lines of dynamically generated JSON for progress tracking and so on. 

While working with one such JSON (shown below), I was seeing a strange error when trying to update a specific property within the JSON.

```powershell
{
     "id" : "6894cb4e-907c-43d0-b79d-c4fb8ef422eb",
     "description" : "Just another manifest for deployment",
     "version" : "1.0.0.0",
     "systems" : [
         {
             "serialNumber" : "123abc",
             "ipAddress" : "8.9.10.11",
             "status" : "pending"
         },
         {
             "serialNumber" : "456def",
             "ipAddress" : "8.9.10.12",
             "status" : "pending"
         },
         {
             "serialNumber" : "789ghi",
             "ipAddress" : "8.9.10.13",
             "status" : "pending"
         }
     ]
 }
```


This is a very minimal and simplified version of JSON that I have in the automation framework. In this JSON, based on the status of deployment, I need to update the status property of each system in the manifest. 

Here is what I tried.

{{< figure src="/images/customobject1.png" >}} {{< load-photoswipe >}}

Now, that error is pretty strange. To investigate this, I looked at the type of object that was getting returned from the Where() method.

{{< figure src="/images/customobject2.png" >}}

**Update (2/11):** Prasoon commented on this post and mentioned that the Item() method on this collection to update the status property. Here is how we do it based on his suggestion

```powershell
$manifest.systems.Where({$_.serialNumber -eq '123abc'}).item(0).Status = 'Complete'
```

What you see below is my investigation before Prasoon commented on this post!

It should ideally be a PS custom object. The where() method is therefore doing something to the custom object. I tried, then, using the index of an object within the systems collection.

This is good. So, I can work around the issue with Where() method by using the index but the only catch here is that I need to dynamically determine the index of a specific object instance during the orchestration. I tried a couple of methods.

{{< figure src="/images/customobject4.png" >}}

I was skeptical about the above method of using `where()` again. And, it does fail. The index of the object instance returned using this method is always -1. 

In the second method, I resorted to using a simple for loop to gather the index of the node.

```powershell
$serialNumber = '456def'
for ($currentIndex = 0; $currentIndex -lt $manifest.systems.Count; $currentIndex++)
 {
     if ($manifest.systems[$currentIndex].serialNumber -eq $serialNumber)
     {
         break
     }
     else
     {
         continue
     }
 }
 $manifest.systems[$currentIndex].status = 'complete'
```

The above snippet does not look super optimal to me but works as expected.

{{< figure src="/images/customobject5.png" >}}

I have not figured out any other optimal way of handling this but have you come across something like this? Do you see a better way to handle this?
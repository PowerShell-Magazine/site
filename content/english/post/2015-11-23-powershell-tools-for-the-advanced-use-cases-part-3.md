---
title: PowerShell Tools For The Advanced Use Cases – part 3
author: "David O'Brien"
type: post
date: 2015-11-23T17:00:00+00:00
url: /2015/11/23/powershell-tools-for-the-advanced-use-cases-part-3/
views:
  - 12410
post_views_count:
  - 2112
categories:
  - How to
  - DevOps
tags:
  - How to
  - DevOps

---
### Posts in this series

  1. [PowerShell Tools for the Advanced Use Cases, part 1][1]
  2. [PowerShell tools for the advanced use cases – part 2][2]
  3. PowerShell tools for the advanced use cases &#8211; part 3 (this article)

I am currently on board flight Qantas 423 from Sydney to Melbourne, the last hop on this very long way back home from the Midwest Management Summit in Minneapolis/Minnesota USA.

It was requested of me to maybe show the integration of part 1 and part 2 into the Continuous Integration (CI) server that I used in my demos.

I have so far mostly used TeamCity and Jenkins, some customers use GO.CD or other products. For this demo I will stuck to my home lab CI Server TeamCity.

### TeamCity

You can install TeamCity on Linux and on Windows. It really depends on what you will be doing with it whether it’ll be wise (or easier) to install it on Linux or Windows.

I do a lot of cross-platform management / automation, so Linux as the hosting platform made sense, it also has tools like git already installed.

If all you do is Windows, then install it on Windows, works just as well.

TeamCity is usually used as a Continuous Integration or Build server to automate builds or packaging of source / application code into deployable artefacts like MSI files, NuGet packages or plain zip files, depending on your platform. These are just the most common for Windows.

### Installation

The installation of TeamCity is very straight forward, both on Linux and Windows. Especially for lab environments you can pretty much get away with a “Next-Next-Done” installation.

Download the TeamCity source files from here:

After installation TeamCity will have installed the TeamCity agent on the local server so that we can dive pretty much straight in.

### High Level build workflow

Pictures are easier understood than words, so that’s why I created a little diagram to explain what we are trying to accomplish here.

### Continuous Integration

Having Pester and PSScriptAnalyzer in your environment means that you will most likely have higher code quality than without. Every time you save your file, because you just finished a new function or fixed a bug maybe, you should run your Pester tests and check for any Style atrocities.

I am lazy in that regards, I would also probably forget to do it after a while, or, because I’m human like you, become too confident with myself and just think “pff, I wrote that code, it’s awesome!” and don’t test because of that.

The trick is to build a workflow that picks up your code after you saved it (checked it in) and automatically runs the tests for you.

We also want code that passes these tests to automatically be packaged into something that we could pick up and deploy, via PowerShell, DSC, System Center Configuration Manager or, if you must, manually even.

This is all part of CI, makes everybody’s life easier and brings up code quality at least by a factor of 42. 😉

### From Code to Artefact

What we are trying to achieve with Continuous Integration can be told with this short storyline:

A Developer writes code that he/she saves and commits to Source Control. The CI Server polls the Version and Source Control (VSC) repository regularly and upon being notified that there is a change in the codebase the CI server downloads the new code to a Build Agent. The Build process is executed, our Pester tests run and PSScriptAnalyzer rules get tested. Only if all tests pass will the CI server pack the code into the desired format and upload them to the repository management server.

### How to integrate Pester into TeamCity

Pester is great, it’s so easy to handle and even easier to integrate into basically any CI Server as it can output the test results in the NUnit 2.5 format as an XML file which TeamCity knows how to interpret.

The Pester Github wiki even has a short explanation on how to make test results show up in TeamCity. Here’s a quick overview.

I assume that you have already created a build project in TeamCity and added some Pester tests to your build pipeline.

![](/images/advtools1.png)

Enter your Build Configuration settings.

Select “Build Features”.

![](/images/advtools2.png)

Select “Add Build Feature”.

Fill in the following values. You might need to change the path to Pester’s XML file.

  * Type: XML report processing
  * Report Type: NUnit
  * Monitoring rules: 
      * %teamcity.build.checkoutDir%\pester_xml.xml

![](/images/advtools3.png)

Save the changes.

Make sure that you have one step in your build stage that executes Pester and outputs the XML to the path configured above. I’m using the following PowerShell code to ensure this:

```powershell
try {
    Import-Module -Name Pester -ErrorAction Stop
    $checkoutdir = "%system.teamcity.build.checkoutDir%"
    $pester_xml = Join-Path $checkoutdir pester_xml.xml
    $result = Invoke-Pester -OutputFile $pester_xml -OutputFormat NUnitXml -PassThru -Strict `
    -ErrorAction Stop

    if ($result.FailedCount -gt 0) {
       throw "{0} tests did not pass" -f $result.FailedCount
    }
}

catch {
    $msg = $_
    Write-Error -ErrorRecord $msg
    exit 1
}
```

With these settings you will be able to view the following information on every build summary:

![](/images/advtools4.png)

TeamCity can even tell if tests failed because of new errors or if that error has already been encountered before. Very powerful.

I demoed that integration at MMS 2015 in Minneapolis and said that this test runs every time I commit new code to my Git repository, automatically. I don’t have to do anything else now, if all the tests pass (there are some more), then, and only then, will TeamCity pack my code and upload it to the PowerShell Gallery.

In the next part of this series I will show how to, with these artefacts now available, achieve Continuous Deployment.

[1]: /2015/10/12/powershell-tools-for-the-advanced-use-cases-part-1/ "PowerShell Tools for the Advanced Use Cases, part 1"
[2]: /2015/11/02/powershell-tools-for-the-advanced-use-cases-part-2/
---
title: Pester Result Reporting With Suggestions And XSL Support
author: Prasoon Karunan v
type: popular
date: 2019-07-25T16:00:07+00:00
url: /2019/07/25/pester-result-reporting-with-suggestions-and-xsl-support/
post_views_count:
  - 5640
views:
  - 5594
categories:
  - DevOps
  - Pester
tags:
  - DevOps
  - Pester

---
I believe there is no introduction required for pester in PowerShell community. If you have never heard of Pester, [this](https://github.com/pester/pester/wiki) is the place to go first. 

This article uses a custom version of Pester which is not an official version yet. This custom version of Pester with these changes is currently available [here](https://github.com/kvprasoon/Pester/tree/remark).


Pester is used both as an unit testing framework as well as an operational validation tool. In both the use cases, output report plays an important role in presenting the test results. This article is about the output report with and added capability. 

Below is a Pester test script with few tests.

```powershell
param(
    [Parameter(Mandatory = $True)]
    [string]$ConfigPath
)

$Config = Get-Content -Path $ConfigPath | ConvertFrom-Json
Describe "Describing validation tests post deployment" {
    Context "Post deployment validation tests for services" {
        BeforeAll {
            $Config.service.expectedconfiguration | ForEach-Object -Process {
                $Name = $_.Name
                $Status = $_.Status
                $StartMode = $_.StartMode
                $Service = Get-Service -Name $Name
    
                it "Service $Name status should be $Status" {
                    $Service.Status | Should -Be $Status
                } 
    
                it "Service $Name startmode should be $StartMode" {
                    $Service.StartType | Should -Be $StartMode
                } 
            }
        }
    }
    
    Context "Post deployment validation tests for folder permission" {
        $Config.folderpermission.expectedconfiguration | ForEach-Object -Process {
            $User = $_.user
            $Permission = $_.permission
            $Path = $_.path
            it "user $User should have $Permission permission on path $Path" {
                $Access = (Get-Acl -Path $Path).Access | Where-Object -FilterScript { $_.IdentityReference -eq $User }
                $Access.FileSystemRights | Should -Contain $Permission
            } 
        }
    }
    
    Context "Post deployment validation tests for firewall rule" {
        $Config.firewallrule.expectedconfiguration | ForEach-Object -Process {
            $Rulename = $_.rulename
            $Direction = $_.direction
            $Rule = Get-NetFirewallRule -Name $RuleName -ErrorAction SilentlyContinue
    
            it "A Firewall rule with name $RuleName should be available" {
                $Rule | Should -Not -BeNullOrEmpty
            } 
    
            it "Firewall rule $RuleName should be allowed for $Direction connection" {
                $Rule.Direction | Should -Not $Direction
            } 
        }
    }
}
```

Data for above test script is shown below.

```powershell
{
    "service": {
        "suggestion": {
            "startmode": "Open PowerShell as administrator and run 'Set-Service -Name {0} -StartType {1}'",
            "status": "Make the service '{0}' in {1} state."
        },
        "expectedconfiguration": [
            {
                "name": "BITS",
                "status": "running",
                "startmode": "automatic"
            },
            {
                "name": "wuauserv",
                "status": "running",
                "startmode": "automatic"
            }
        ]
    },
    "folderpermission": {
        "suggestion": {
            "message": "Give {0} permission for {1} user on {2} folder."
        },
        "expectedconfiguration": [
            {
                "path": "c:\\Deployment\\config",
                "user": "RDFC\\Test",
                "permission": "FullControl"
            },
            {
                "path": "c:\\Deployment\\files",
                "user": "RDFC\\kvprasoon",
                "permission": "FullControl"
            }
        ]
    },
    "firewallrule": {
        "suggestion": {
            "rulename": "Open wf.msc and create an {0} rule with name '{1}'.",
            "direction": "Open wf.msc and create the firewall rule '{0}' for {1} connection."
        },
        "expectedconfiguration": [
            {
                "rulename": "Rule1",
                "direction": "Inbound"
            },
            {
                "rulename": "Rule2",
                "direction": "Outbound"
            }
        ]
    }
}
```


In a nutshell, the above pester test script will test Service status, file system permissions, and firewall rules by reading the data from the JSON configuration file.

You can execute the test as shown below.

```powershell
Invoke-Pester -Script @{Path = 'C:\temp\OpsValidation.tests.ps1' ; Parameters = @{ConfigPath = 'c:\temp\OpsValidationConfig.json'}}
```

This will show the test result in the console with summary.

![image](/images/remark1.png)

When comes to the reporting side, Invoke-Pester has parameters which will create a [nunit][1] based XML test result file. Nunit XML reports are mostly rendered using a report reader or by converting it to html format. There are many tools in the market that convert Nunit XML to nice UI reports. [Reportunit][2] (now [extent-reports][3]) is my favorite. But this tool uses jquery and CSS which are referenced online, hence made me to think about adding this capability to Pester. Since the report is in XML format, my choice was to go with [XSLT][4]. In short, _XSL is a steroid for XML_. This article doesn’t cover anything on how it transforms XML to HTML. [Here][5] is a simple example for creating an XSL targeting an XML. Now we know that XSL can transform an XML to HTML but how to add this in the Nunit report generated by Pester. This is done by adding a new parameter to accept the XSL path and putting it as a stylesheet reference in XML generated by pester. I’ve named the parameter as **–XSLPath.** Below is an example execution with the XSL path. 

```powershell
Invoke-Pester -Script @{Path = 'C:\temp\OpsValidation.tests.ps1' ; Parameters = @{ConfigPath = 'c:\temp\OpsValidationConfig.json'}} -OutputFile c:\temp\OpsValidation.xml -OutputFormat NUnitXml -XSLPath c:\temp\OpsValidation.xsl
```

Once executed, open the XML report using a web browser to see the magic !

So far so good! But here comes the interesting part of this article.

We are now able to have a report in html, hence it is easy to see the test failures (human beings are interested in analyzing failures than success !) in a web browser. How about adding some suggestions/remarks for the end user/support engineers to fix the issue and make the test pass?

Well then that has to be done for each testcase. Yes for each test cases. This is done by adding a new parameter to the **it** function in Pester. 

Below is an example with the suggestion feature.

```powershell
it "Service BITS status should be Running" {
     $Service.Status | Should -Be "Running"
 } -Remark "Open services.msc and start BITS service"
```


But there is a caveat, we can add a parameter to the It function, but how to add this remark in the report XML. The report XML is in nunit format and it has predefined layout. The test result layout has to be followed in the report and therefore any Nunit report readers can parse the result.

Well, Pester report doesn’t use all the attributes of the nunit test result layout and I found the **Label** attribute as a candidate for adding remarks. So with the remark support for testcases, below is the final pester test script.

Script: <https://gist.github.com/kvprasoon/bec40fa50d6975fcdafa6536b61cf1aa>

Test Configuration: <https://gist.github.com/kvprasoon/2dd5fc64eec0653e4bdde6a18da526ff>

Lets execute and see the report with suggestions. 

```powershell
Invoke-Pester -Script @{Path = 'C:\temp\OpsValidation.tests.ps1' ; Parameters = @{ConfigPath = 'c:\temp\OpsValidationConfig.json'}} -OutputFile c:\temp\OpsValidation.xml -OutputFormat NUnitXml -XSLPath c:\temp\OpsValidation.xsl
```

After opening the generated report in a web browser.

![image](/images/remark2.png)

If you want to explore and see the code changes, Pester with these changes is currently available [here](https://github.com/kvprasoon/Pester/tree/remark).

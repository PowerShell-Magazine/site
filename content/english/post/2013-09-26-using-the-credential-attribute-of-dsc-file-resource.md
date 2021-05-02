---
title: Using the Credential attribute of DSC File Resource
author: Ravikanth C
type: post
date: 2013-09-26T14:12:50+00:00
url: /2013/09/26/using-the-credential-attribute-of-dsc-file-resource/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
In an article about [copying DSC custom resources to remote systems][1], I talked about setting computer account permissions. Many of us, including me, may not always want to do that given the number of computers to be managed or various other security concerns. As I’d mentioned at the end of that article, there is a _Credential_ attribute which can be used to specify the credentials to access the network share.

Let us start with an example showing how this attribute can be used.


    Configuration CopyDSCResource {
        param (
            [Parameter(Mandatory=$true)]
            [ValidateNotNullOrEmpty()]
            [String[]]$NodeName,
            [Parameter(Mandatory=$true)]
            [String]$SourcePath,
    
            [Parameter(Mandatory=$false)]
            [PSCredential]$Credential,
    
            [Parameter(Mandatory=$false)]
            [String]$ModulePath = "$PSHOME\modules\PSDesiredStateConfiguration\PSProviders"
        )
    
        Node $NodeName {
            File DSCResourceFolder {
                SourcePath = $SourcePath
                DestinationPath = $ModulePath
                Recurse = $true
                Type = "Directory"
                Credential = $Credential
            }
        }
    }
    
    CopyDSCResource -NodeName SRV2-WS2012R2,SRV3-WS2012R2 -SourcePath "\\10.10.10.101\DSCResources" -Credential (Get-Credential)
When we dot-source this configuration, we expect to see the MOF files created for the server nodes specified. But, this is what we would end up seeing.

```
PS C:\demo> .\Demo1.ps1
cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
ConvertTo-MOFInstance : System.InvalidOperationException error processing property 'Credential' OF TYPE 'File': Converting and storing an encrypted password
as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
At C:\demo\Demo1.ps1:19 char:9
+   File
At line:164 char:16
+     $aliasId = ConvertTo-MOFInstance $keywordName $canonicalizedValue
+                ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : InvalidOperation: (:) [Write-Error], InvalidOperationException
+ FullyQualifiedErrorId : FailToProcessProperty,ConvertTo-MOFInstance
ConvertTo-MOFInstance : System.InvalidOperationException error processing property 'Credential' OF TYPE 'File': Converting and storing an encrypted password
as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
At C:\demo\Demo1.ps1:19 char:9
+   File
At line:164 char:16
+     $aliasId = ConvertTo-MOFInstance $keywordName $canonicalizedValue
+                ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : InvalidOperation: (:) [Write-Error], InvalidOperationException
+ FullyQualifiedErrorId : FailToProcessProperty,ConvertTo-MOFInstance
Errors occurred while processing configuration 'CopyDSCResource'.
At C:\Windows\system32\WindowsPowerShell\v1.0\Modules\PSDesiredStateConfiguration\PSDesiredStateConfiguration.psm1:1991 char:5
+     throw $errorRecord
+     ~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : InvalidOperation: (CopyDSCResource:String) [], InvalidOperationException
+ FullyQualifiedErrorId : FailToProcessConfiguration
```

If you observe the output, it talks about _PSDscAllowPlainTextPassword_ for converting the encrypted password acquired using _Get-Credential_ cmdlet. But, where do we place this attribute and set it to True? We need to do that as a part of node configuration. To make this work, we need to use the [configuration data][2] for the node attributes. Simply put, DSC configuration data is a hash table that contains an array of hash table entries for each node. Here is a sample.

```
$ConfigurationData = @{
    AllNodes = @(
        @{
            NodeName="*"
            PSDscAllowPlainTextPassword=$true
         }
        @{
            NodeName="SRV2-WS2012R2"
         }
        @{
            NodeName="SRV3-WS2012R2"
         }
   )
}
```


Make a note of the hash table key names that are used in _$ConfigurationData_. The top-level hash table should have _AllNodes_ key whose value is an array of hash tables. Each hash table inside this array must have a key named _NodeName_ whose value can either be “*” or the target system name. The _PSDscAllowPlainTextPassword_ attribute needs to be added to any of the hash tables inside the array. Since this attribute applies to all the nodes, we can put it inside the hash table which has _NodeName_ set to “*”. The attributes we specify within this hash table apply to all nodes in the configuration data.

Let us get back to the original subject.

The following configuration document demonstrates how we use the configuration data along with _Credential_ attribute to authenticate to the network path.

```
$ConfigurationData = @{
    AllNodes = @(
        @{
            NodeName="*"
            PSDscAllowPlainTextPassword=$true
         }
        @{
            NodeName="SRV2-WS2012R2"
         }
        @{
            NodeName="SRV3-WS2012R2"
         }
    )
}

Configuration CopyDSCResource {
    param (
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [String]$SourcePath,

        [Parameter(Mandatory=$false)]
        [PSCredential]$Credential,

        [Parameter(Mandatory=$false)]
        [String]$ModulePath = "${PSHOME}\modules\PSDesiredStateConfiguration\PSProviders"
    )

Node $AllNodes.NodeName {
    File DSCResourceFolder {
        SourcePath = $SourcePath
        DestinationPath = $ModulePath
        Recurse = $true
        Type = "Directory"
        Credential = $Credential
        MatchSource = $true
    }
}

}

CopyDSCResource -ConfigurationData $configurationData -SourcePath "\\10.10.10.101\DSCResources" -Credential (Get-Credential)
```

After we dot-source this configuration document, we can use _Start-DscConfiguration_ cmdlet to apply the configuration. At this point, you will see that the remote server is able to connect to the share using given credentials and perform the configuration change.

[1]: /2013/09/02/copying-powershell-modules-and-custom-dsc-resources-using-dsc/
[2]: http://technet.microsoft.com/en-us/library/dn249925.aspx
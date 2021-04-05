---
title: '#PSDSC FailoverClusterDSC – Deploy a Storage Spaces Direct Cluster'
author: Ravikanth C
type: post
date: 2017-12-19T17:00:56+00:00
url: /2017/12/19/psdsc-failoverclusterdsc-deploy-a-storage-spaces-direct-cluster/
views:
  - 9150
post_views_count:
  - 4999
categories:
  - PowerShell DSC
  - Module Spotlight
tags:
  - PowerShell DSC
  - Modules

---
I have been working on the [_FailoverClusterDsc_][1] resource module and finally had the chance to add some examples and make the repository public.

> This is not a fork of the [xFailoverCluster][2] module. I am adding only the resources that I am developing from scratch to this module. These resources will follow the HQRM guidelines.
>

You can take a look at each of these resources to check what different configuration options are supported as of today.

Here is an example of creating and configuring a Storage Spaces Direct cluster.

```powershell
$configData = @{
	AllNodes = @(
		@{
    	    NodeName = 'localhost'
           	thumbprint = '25A1359A27FB3F2D562D7508D98E7189F2A1F1B0'
        	CertificateFile = 'C:\PublicKeys\S2D4N01.cer'
        	PsDscAllowDomainUser = $true
    	}
    )
}

Configuration CreateS2DCluster
{
    param
    (
        [Parameter(Mandatory = $true)]
        [pscredential]
        $Credential,
    	
    	[Parameter(Mandatory = $true)]
    	[String[]]
    	$ParticipantNodes,

    	[Parameter(Mandatory = $true)]
    	[String]
    	$ClusterName,

    	[Parameter(Mandatory = $true)]
    	[String]
    	$StaticAddress,

    	[Parameter(Mandatory = $true)]
    	[String[]]
    	$IgnoreNetworks,

    	[Parameter(Mandatory = $true)]
    	[String]
    	$QuorumResource,

    	[Parameter(Mandatory = $true)]
    	[String]
    	$QuorumType 
	)

	Import-DscResource -ModuleName FailoverClusterDsc

	Node $AllNodes.NodeName
	{
    	FailoverCluster CreateCluster
    	{
        	ClusterName = $ClusterName
        	StaticAddress = $StaticAddress
        	NoStorage = $true
        	IgnoreNetwork = $IgnoreNetworks
        	Ensure = 'Present'
        	PsDscRunAsCredential = $Credential
    	}

    	WaitForFailoverCluster WaitForCluster
    	{
        	ClusterName = $ClusterName
        	PsDscRunAsCredential = $Credential
    	}

    	Foreach ($node in $ParticipantNodes)
    	{
        	FailoverClusterNode $node
        	{
            	NodeName = $node
            	ClusterName = $ClusterName
            	PsDscRunAsCredential = $Credential
            	Ensure = 'Present'
        	}
    	}

    	FailoverClusterQuorum FileShareQuorum
    	{
        	IsSingleInstance = 'Yes'
        	QuorumType = $QuorumType
        	Resource = $QuorumResource
    	}

    	FailoverClusterS2D EnableS2D
    	{
        	IsSingleInstance = 'yes'
        	Ensure = 'Present'
    	}
	}
}

CreateS2DCluster -Credential (Get-Credential) -ConfigurationData $configData `
                                           -QuorumType 'NodeAndFileShareMajority' `
                                           -QuorumResource '\\sofs\share' `
                                           -ClusterName 'S2D4NCluster' `
                                           -StaticAddress '172.16.102.45' `
                                           -IgnoreNetworks @('172.16.103.0/24','172.16.104.0/24') `
                                           -ParticipantNodes @('S2D4N02','S2D4N03','S2D4N04')
```
In the above pattern, I am creating a failover cluster and then adding the remaining nodes using the same configuration document. You can, however, have the node addition configuration using the _FailoverClusterNode_ resource as a separate configuration document that gets enacted on the participant node.

The failover cluster configuration requires administrator privileges and these resources do not have a Credential parameter of their own and depend on _PSDscRunAsCredential_. Therefore, you need at least PowerShell 5.0 to use these resources.

I am looking at expanding the resource modules to beyond what is there at the moment. If you see any issues or have feedback, feel free to create an issue in my repository. These resources lack tests today. I would be glad to accept any PRs for tests.

[1]: https://github.com/rchaganti/FailoverClusterDSC
[2]: http://github.com/powershell/xFailoverCluster
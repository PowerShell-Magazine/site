---
title: Reporting Cluster Shared Volume (CSV) disk space utilization
author: Ravikanth C
type: post
date: 2014-02-28T17:00:16+00:00
url: /2014/02/28/reporting-cluster-shared-volume-csv-disk-space-utilization/
categories:
  - How To
  - Hyper-V
tags:
  - How To
  - Hyper-V

---
The Failover Cluster cmdlets can be used to get information about Cluster Shared Volumes (CSV). The _Get-ClusterSharedVolume_ cmdlet when run on a cluster node, gives us the information about all the CSVs.

```
PS> Get-ClusterSharedVolume
Name        State        Node
----        -----        ----
VM-CSV      Online       SERVER-2
```

We can dig deep into the object returned by this cmdlet to find the total space allocated to the CSV and the freespace available.

```
PS> $csv = Get-ClusterSharedVolume

PS>  $csv.SharedVolumeInfo.Partition
Name                    : \\?\Volume{6688b177-8da7-45ed-93d1-04d649925cd0}\
DriveLetter             :
DriveLetterMask         : 0
FileSystem              : CSVFS
FreeSpace               : 1055810584576
MountPoints             : {}
PartitionNumber         : 2
PercentFree             : 48.01538
Size                    : 2198900506624
UsedSpace               : 1143089922048
HasDriveLetter          : False
IsCompressed            : False
IsDirty                 : Unknown
IsFormatted             : True
IsNtfs                  : False
IsPartitionNumberValid  : True
IsPartitionSizeValid    : True
```

If you have noticed, the _Get-ClusterSharedVolume_ cmdlet gives us the ownernode for the disk but it does not tell us what is the disk number. For reporting purposes, it is important for me to be able to accurately identify the physical disk number on the ownernode. This mapping is not straightforward. We can use a combination of Storage cmdlets and CIM cmdlets to create this mapping.


    Function Get-CSVtoPhysicalDiskMapping {
        param (
            [string]$clustername = &quot;.&quot;
        )
    
        $clusterSharedVolume = Get-ClusterSharedVolume -Cluster $clusterName
    
        foreach ($volume in $clusterSharedVolume) {
            $volumeowner = $volume.OwnerNode.Name
            $csvVolume = $volume.SharedVolumeInfo.Partition.Name
            $cimSession = New-CimSession -ComputerName $volumeowner
            $volumeInfo = Get-Disk -CimSession $cimSession | Get-Partition | Select DiskNumber, @{
                    Name=&quot;Volume&quot;;Expression={Get-Volume -Partition $_ | Select -ExpandProperty ObjectId}
            }
    
            $csvdisknumber = ($volumeinfo | where { $_.Volume -eq $csvVolume}).Disknumber
            $csvtophysicaldisk = New-Object -TypeName PSObject -Property @{
                    &quot;CSVName&quot; = $volume.Name
                    &quot;CSVVolumePath&quot; = $volume.SharedVolumeInfo.FriendlyVolumeName
                    &quot;CSVOwnerNode&quot;= $volumeowner
                    &quot;CSVPhysicalDiskNumber&quot; = $csvdisknumber
                    &quot;CSVPartitionNumber&quot; = $volume.SharedVolumeInfo.PartitionNumber
                    &quot;Size (GB)&quot; = [int]($volume.SharedVolumeInfo.Partition.Size/1GB)
                    &quot;FreeSpace (GB)&quot; = [int]($volume.SharedVolumeInfo.Partition.Freespace/1GB)
            }
    
            $csvtophysicaldisk
        }
    }
As you see above, we are using the partition information in the CSV object to map it to a volume&#8217;s disk number on the physical host. For this, we are first retrieving all the disk information from the ownernode of the CSV volume. Once we have the disk information, we retrieve a list of partition and then map them to the volumes. What we are interested in here is really the _ObjectId_ property of each volume.

In the CSV object&#8217;s partition information, the name of the partition is written in the form of a physical device path. In this example, _&#92;?\Volume{6688b177-8da7-45ed-93d1-04d649925cd0}\_ represents the volume&#8217;s _ObjectId_. All we need to do now is to compare the _ObjectId_s we retrieved to the CSV partitions volume name.

```
PS> Get-CSVtoPhysicalDiskMapping

FreeSpace (GB)        : 983
CSVVolumePath         : C:\ClusterStorage\Volume1
CSVOwnerNode          : STOMP-SERVER-2
Size (GB)             : 2048
CSVName               : VM-CSV
CSVPartitionNumber    : 2
CSVPhysicalDiskNumber : 2
```

Once you have this information as a custom object, you can simply send this to the ConvertTo-Html cmdlet to create a HTML file. We can beautify the HTML formatting using custom CSS. More on that later!
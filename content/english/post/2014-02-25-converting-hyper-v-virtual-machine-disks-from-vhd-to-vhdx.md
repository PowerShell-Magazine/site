---
title: Converting Hyper-V virtual machine disks from VHD to VHDX
author: Shay Levy
type: post
date: 2014-02-25T17:00:57+00:00
url: /2014/02/25/converting-hyper-v-virtual-machine-disks-from-vhd-to-vhdx/
categories:
  - How To
  - Hyper-V
tags:
  - How To
  - Hyper-V

---
<span style="line-height: 1.5em;">I&#8217;ve read recently a <a href="http://www.aidanfinn.com/?p=16007)">post by Hyper-V MVP Aidan Finn</a> about a script he wrote to convert Hyper-V virtual machine VHDs to the new VHDX disk format. </span>I liked what I saw and I gave the script a few spins in my lab. It worked flawlessly, but I wanted to create a more robust version&#8211;one that supports conversions of one or more local or remote virtual machine disks.

<span style="line-height: 1.5em;">The <em>ConvertTo-Vhdx</em> advanced function is the result. It adds the missing functionality I was looking for as well as fixes a gotcha where a VM would shut down </span>even if it had no disks to convert. The function accepts pipeline input of virtual machine objects as well as virtual machine names. It also adds support to run the function in _WhatIf_ mode. When _-WhatIf_ is specified, virtual machines are checked and messages are written to the console. The _-DeleteSource_ switch parameter allows you to delete the source virtual hard disk(s) after the conversion, and _-StartVM_ will turn on the virtual machine when conversion has finished.


    function ConvertTo-Vhdx
    {
        [CmdletBinding(DefaultParameterSetName='name',SupportsShouldProcess=$true)]
    	param(
      [Parameter(Position=0,Mandatory,ValueFromPipelineByPropertyName,ValueFromPipeline,ParameterSetName='name')]
        [Alias('VMName')]
        [System.String[]]$Name,
    
        [Parameter(Position=0,Mandatory,ValueFromPipeline,ParameterSetName='inputObject')]
        [Microsoft.HyperV.PowerShell.VirtualMachine[]]$InputObject,
    
        [Parameter(Position=0,Mandatory,ValueFromPipelineByPropertyName)]
        [Alias('cn')]
        [string]$ComputerName=$env:COMPUTERNAME,
    
        [switch]$DeleteSource,
        [switch]$Force,
        [switch]$StartVM
    )
    
    begin
    {
        $bp = $PSCmdlet.MyInvocation.BoundParameters
    
        $ds = [bool]$bp['DeleteSource']
        $vb = [bool]$bp['Verbose']
        $cfrm = [bool]$bp['Confirm']
        $frc = [bool]$bp['Force']
        $wi = [bool]$bp['WhatIf']
    }
    
    process
    {
        try
        {
            $vms = if($PSCmdlet.ParameterSetName -eq 'name')
            {
                Get-VM -Name $Name -ComputerName $ComputerName
            }
            else
            {
                $InputObject
            }
    
            foreach($vm in $vms)
            {
                $vhd = $vm.HardDrives | Where-Object { $_.Path -like '*.vhd'}
    
                if(!$vhd)
                {
                    Write-Warning "No VHD disks detected on VM: $($vm.Name)"
                    continue
                }
                else
                {
                    Write-Verbose "VHD disks were detected on VM: $($vm.Name)"
                }
    
                if($vm.State -eq 'Running')
                {
                    Write-Verbose "Shutting down $($vm.Name)"
                    Stop-VM $vm.Name -ComputerName $ComputerName -Confirm:$cfrm -Force:$frc -ErrorAction Stop -WhatIf:$wi
                }
    
                $vhd | ForEach-Object{
    
                    $drv = $_ | Select-Object *
                    $vhdx = [System.IO.Path]::ChangeExtension($_.Path,'vhdx')
    
                    Convert-VHD –Path $_.Path –DestinationPath $vhdx -ComputerName $ComputerName -DeleteSource:$ds -Confirm:$cfrm -Verbose:$vb -WhatIf:$wi
                    Set-VHD –Path $vhdx -PhysicalSectorSizeBytes 4kb -ComputerName $ComputerName -Confirm:$cfrm -Verbose:$vb -WhatIf:$wi
                    Remove-VMHardDiskDrive $_ -WhatIf:$wi -Verbose:$vb | Out-Null
                    Add-VMHardDiskDrive -VMName $vm.Name -ComputerName $ComputerName -Path $vhdx -ControllerType $drv.ControllerType -ControllerLocation $drv.ControllerLocation -ControllerNumber $drv.ControllerNumber -Confirm:$cfrm -Verbose:$vb -WhatIf:$wi
                }
    
                if($StartVM)
                {
                    Start-VM $vm.Name -ComputerName $ComputerName -Verbose:$vb -WhatIf:$wi
                }
            }
        }
        catch
        {
          Write-Error $_
        }
    }
    }
The variables in the _Begin_ block are used to determine the values of switches regardless if they were present on the command line or not. The values are then passed to the corresponding parameters of each command used inside the function.
---
title: 'Adam Driscoll’s Favorite PowerShell Tips & Tricks'
author: Adam Driscoll
type: post
date: 2012-06-11T18:00:30+00:00
url: /2012/06/11/adam-driscolls-favorite-powershell-tips-tricks/
views:
  - 25764
post_views_count:
  - 2205
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
I am a developer by trade and primarily use PowerShell to aid in my development practices. The majority of the time I’m a .NET developer, which means I can use PowerShell to quickly prototype what I’m working on. This often requires me to run PowerShell as an administrator.

### Running PowerShell as an Administrator

You can easily right click on the PowerShell icon and select ‘Run as Administrator’ and the User Account Control (UAC) dialog will be presented. This method works OK, but it requires me to use my mouse and navigate a context menu. It’s time consuming. Instead you can easily run PowerShell as administrator right from another PowerShell window with the following command line.

<pre class="brush: powershell; title: ; notranslate" title="">Start-Process PowerShell –Verb RunAs</pre>

The RunAs verb is a special verb in Windows that enables processes to request elevation. Once we accept the UAC prompt we will have an elevated command prompt running and we don’t have to use a mouse.

### Running PowerShell under .NET 4.0

A lot of the things I develop are written in .NET 4.0. PowerShell v2.0, natively, runs under the .NET 2.0 Framework. There are a couple of tricks that can be used to run PowerShell under .NET 4.0. The first is to use the UseOnlyLastestClr Registry value. Never do this! This is especially true for a developer. It is a system wide setting that will cause everything to run under .NET 4.0 and can cause a lot of compatibility problems. Instead, an app.config is the recommended practice. An app.config can direct a particular application to run under a specific .NET version. By placing a configuration file in the $PSHome directory, with the name powershell.exe.config and the following contents, we can get PowerShell to run under .NET 4.0.

<pre class="brush: xml; title: ; notranslate" title="">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;configuration&gt;
   &lt;startup useLegacyV2RuntimeActivationPolicy="true"&gt;
     &lt;supportedRuntime version="v4.0.30319 " /&gt;
   &lt;/startup&gt;
&lt;/configuration&gt;
</pre>

The problem with the app.config file is that it is buried in the system and requires an elevated text editor to be opened to make modifications. Instead, I created an advanced function to allow me to specify the .NET version I’d like to run under. It backs up the configuration file if one is there already.

<pre class="brush: powershell; title: ; notranslate" title="">if(Test-Path $PowerShellConfigPath)
{
   $PreviousContents = Get-Content -Path $PowerShellConfigPath
   Remove-Item $PowerShellConfigPath
}
</pre>

It writes out the configuration file for the specified .NET version.

<pre class="brush: powershell; title: ; notranslate" title="">Set-Content -Path $PowerShellConfigPath -Value $ConfigFileContents</pre>

Then it starts PowerShell.

<pre class="brush: powershell; title: ; notranslate" title="">$process = Start-Process PowerShell -ArgumentList "-noexit" –PassThru</pre>

And finally, it waits until .NET is loaded into the process before removing or replacing the file that was created. If we were to remove the file too quickly it won’t have time to read the contents and will run under the default or previously configured .NET version.

<pre class="brush: powershell; title: ; notranslate" title="">while((Get-Process -Id $process.Id | Select -Expand Modules | Where { $_.ModuleName -match $mscor }| Measure-Object | Select-Object -ExpandProperty count) -eq 0)
{
     Start-Sleep -Seconds 1
}
</pre>

All of this is wrapped in a Start-PowerShell function that I can call like so.

<pre class="brush: powershell; title: ; notranslate" title="">Start-PowerShell –DotNetVersion v4</pre>

Remember that because this is modifying a file in the system directory, the PowerShell command window will need to be started as administrator. I added an AsAdmin switch for this purpose.

<pre class="brush: powershell; title: ; notranslate" title="">Start-PowerShell –AsAdmin</pre>

As an aside, this will not be an issue in PowerShell 3.0 because the PowerShell command prompt will always run under .NET 4.0. It may become an issue in the future when .NET 5.0 is released. 😉

### Finding COM Object Information

COM interop in PowerShell is great. The ability to quickly load up, discover and utilize COM objects is really helpful. A lot of times I need to know exactly where a COM object has been loaded from and what version is registered. With other .NET assemblies we can use the Assembly parameter to find out this information. Since COM Interop is generated on the fly, this is the result of the same strategy on a COM object.

```
$ie = New-Object –ComObject InternetExplorer.Application
$ie.GetType().Assembly

GAC Version Location
--- ------- --------
True v4.0.30319 c:\Windows\Microsoft.NET\...\mscorlib.dll
```

Since the type was auto-generated, the assembly reports as mscorlib.dll. We can see that the PSTypeNames reports a similarly, interminable list of types.

<pre class="brush: powershell; title: ; notranslate" title="">$ie.PSTypeNames
System.__ComObject#{d30c1661-cdaf-11d0-8a3e-00c04fc9e26e}
System.__ComObject
System.MarshalByRefObject
System.Object
</pre>

To get around this I created a function that uses the Registry provider to look up the registration and version information for a COM object. We just need to specify the ProgId and the information will be returned.

```
PS> Get-ComInformation InternetExplorer.Application
Path              				Version
----             				------
"C:\Program Files\Internet Explorer\ie...       9.00.8112.16421...
```

This is accomplished by creating a new drive to the HKEY classes root Registry hive. This drive does not exist by default.

<pre class="brush: powershell; title: ; notranslate" title="">New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT | Out-Null
</pre>

Then we look up the class ID.

<pre class="brush: powershell; title: ; notranslate" title="">$clsid = Get-ItemProperty -Path "HKCR:\$Name\CLSID"
</pre>

Next, we can query the Local or InProc COM server information.

<pre class="brush: powershell; title: ; notranslate" title="">$serverInfo = Get-ItemProperty -Path "HKCR:\CLSID\$($clsid.'(default)')\InProcServer32"
</pre>

Finally, we can use the .NET FileVersionInfo class to query the version information of the EXE or DLL.

<pre class="brush: powershell; title: ; notranslate" title="">[Diagnostics.FileVersionInfo]::GetVersionInfo($serverInfo.'(default)'.Replace('"', ''))</pre>

It’s then written out as a custom PSObject to the pipeline for consumption. This is really helpful when a script requires a particular version of a COM object. It’s also nice to quickly identify when a rogue COM DLL gets registered in a place other than the one we are expecting.

Here&#8217;s are the functions mentioned in the post:


	function global:Start-PowerShell {
	[CmdletBinding()]
	param(
		[ValidateSet("v2","v4")]
		[Parameter()]
		$DotNetVersion = "v2",
	
		[Parameter()]
		[Switch]$AsAdmin
	)
	
	Begin
	{
		$ConfigFileContents = "&lt;?xml version=`"1.0`" encoding=`"utf-8`"?&gt;
						&lt;configuration&gt;
						    &lt;startup useLegacyV2RuntimeActivationPolicy=`"true`"&gt;
						      &lt;supportedRuntime version=`"{0}`" /&gt;
						    &lt;/startup&gt;
						&lt;/configuration&gt;"
	
		$PowerShellConfigPath = Join-Path $PSHOME "powershell.exe.config"
	
		Write-Verbose "Replacing $PowerShellConfigPath..."
	
		if ($DotNetVersion -eq "v2")
		{
			$ConfigFileContents = $ConfigFileContents -f "v2.0.50727"
			$mscor = "mscorwks"
		}
		elseif ($DotNetVersion -eq "v4")
		{
			$ConfigFileContents = $ConfigFileContents -f "v4.0.30319"
			$mscor = "mscoree"
		}
	
		Write-Verbose $ConfigFileContents
	
		if (Test-Path $PowerShellConfigPath)
		{
			$PreviousContents = Get-Content -Path $PowerShellConfigPath
			Remove-Item $PowerShellConfigPath
		}
	
		Set-Content -Path $PowerShellConfigPath -Value $ConfigFileContents
	
		Start-Sleep -Seconds 1
	
		if ($AsAdmin)
		{
			$process = Start-Process PowerShell -Verb RunAs -ArgumentList "-noexit" -PassThru
		}
		else
		{
			$process = Start-Process PowerShell -ArgumentList "-noexit" -PassThru
		}
	
		while((Get-Process -Id $process.Id | Select -Expand Modules | Where { $_.ModuleName -match $mscor }|Measure-Object | Select -ExpandProperty count) -eq 0)
		{
			Start-Sleep -SEconds 1
		}
	
		Write-Verbose "$($process.Modules | Where { $_.ModuleName -match $mscor })"
	
		if ($PreviousContents -ne $null)
		{
			Set-Content -Path $PowerShellConfigPath -Value $PreviousContents
		}
		else
		{
			Remove-Item -Path $PowerShellConfigPath
		}
	}
	}
	
	function global:Get-ComInformation
	{
		[CmdletBinding()]
		param(
		[Parameter(Mandatory=$true)]
		[string]$Name
	)
	
	begin
	{
		if ((Get-PSDrive HKCR -ErrorAction SilentlyContinue) -eq $null)
		{
			New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT | Out-Null
		}
	}
	
	process
	{
		if (!(Test-Path "HKCR:\$Name\CLSID"))
		{
			Write-Error "ProgId [$Name] was not found."
			return
		}
	
		$clsid = Get-ItemProperty -Path "HKCR:\$Name\CLSID"
	
		if (Test-Path -Path "HKCR:\CLSID\$($clsid.'(default)')\InProcServer32")
		{
			$serverInfo = Get-ItemProperty -Path "HKCR:\CLSID\$($clsid.'(default)')\InProcServer32"
		}
		elseif (Test-Path -Path "HKCR:\CLSID\$($clsid.'(default)')\LocalServer32")
		{
			$serverInfo = Get-ItemProperty -Path "HKCR:\CLSID\$($clsid.'(default)')\LocalServer32"
		}
		else
		{
			Write-Error "Failed to locate the server information for COM object [$Name]."
			return
		}
	
		if ($serverInfo.Version -ne $null)
		{
			$version = $serverInfo.Version
		}
		elseif ($serverInfo.'(default)' -ne $null)
		{
			$version = [Diagnostics.FileVersionInfo]::GetVersionInfo($serverInfo.'(default)'.Replace('"', '')).FileVersion
		}
	
		if ($serverInfo.CodeBase -ne $null)
		{
			$path = $serverInfo.CodeBase
		}
		elseif ($serverInfo.Assembly -ne $null)
		{
			$path = $serverInfo.Assembly
		}
		elseif ($serverInfo.'(default)' -ne $null)
		{
			$path = $serverInfo.'(default)'
		}
	
		$properties = @{
			Path = $path
			ThreadingModel = $serverInfo.'ThreadingModel'
			Version = $version
			}
	
		New-Object PSObject -Property $properties
	}
	}
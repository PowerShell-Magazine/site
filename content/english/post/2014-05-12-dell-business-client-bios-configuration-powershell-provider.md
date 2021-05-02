---
title: Dell Business Client BIOS Configuration PowerShell Provider
author: Vibha Garg
type: post
date: 2014-05-12T16:00:52+00:00
url: /2014/05/12/dell-business-client-bios-configuration-powershell-provider/
categories:
  - Dell
tags:
  - Dell

---
Familiar with a PowerShell provider?  If yes, you must have appreciated the fact that once you learn to use a provider then it is almost effortless to learn and use any other built-in or 3<sup>rd</sup>-party provider. The PowerShell provider framework has a fixed number of cmdlets and a provider exposes all or subset of them depending on its implementation. And as always, there is the `Get-Help <provider name>` cmdlet which saves you from remembering everything about using the provider.

One of the latest offerings from Dell’s Enterprise Solutions Group is a [PowerShell provider][1] which can configure BIOS settings of the Latitude, Optiplex, Precision and the Venue 11 systems. If you are familiar with CCTK command line utility for managing Dell Business client systems, you can perform the same set of operations as the CCTK utility using this new PowerShell provider.

[Update: Download the beta provider at <http://en.community.dell.com/techcenter/enterprise-client/w/wiki/6901.dell-client-powershell-provider.aspx>]

You can manage the BIOS settings on these client systems using the Dell OMCI WMI provider too. Then, why we chose to come up with a provider and not a binary module in PowerShell space? The answer is simple. If you have already invested in learning PowerShell and understand the basics of using the cmdlets such as Get-ChildItem, Get-Item , and so on, you should be able to apply that knowledge to manage the Dell Business client system BIOS settings. Also, going through the WMI classes and writing WMI queries is probably not a happy choice for administrators.

To add to that, here are a list of additional advantages with a PowerShell provider implementation.

  1. Number of cmdlets exposed by a provider framework is fixed  and all of them are known to users today. And this set is not huge, merely < 20.  All provider cmdlets are standard in nature.
  2. Users don’t have to learn about any new name or syntax of a custom cmdlet as is the case with a module.
  3. You start with root item and  can navigate (cd , dir) the entire tree of underlying objects which are BIOS attributes for Dell SMBIOS provider.  No need to know any strings in advance.
  4. Even with subsequent releases, number of interfaces or cmdlets is not going to be swollen. They will remain exactly the same. You will get a product with more features (minus the hassle of learning new cmdlets). This is in contrast with a module, which mostly exposes a new set of cmdlets with a newly added feature.

For example, had we come up with a module instead of a PowerShell provider, number of cmdlets that we may have to expose could be directly proportional to number of BIOS attributes. And that’s a significantly large number.

Let’s take a quick look of the capabilities of Dell Client PowerShell Provider:

  1. It represents the SMBIOS attributes in the same fashion as in Setup Menu (F2).
  2. Strings for attribute names and their settings and the categories are largely same as in Setup Menu (F2).
  3. You can navigate through the BIOS attributes using cd and dir commands. Start with the root node which is a _DellSmbios: drive:_

    cd Dellsmbios:

<ol start="4">
  <li>
    You can configure a BIOS attribute:<br /> <em>Set-Item Dellsmbios:\PostBehaviour\NumLock Enabled</em>
  </li>
</ol>

<ol start="5">
  <li>
    You can set, change or clear the Admin password and System password
  </li>
  <li>
    You can configure the TPM security settings
  </li>
  <li>
    You can change the legacy or UEFI boot sequence
  </li>
  <li>
    You can check the service tag and change the asset tag
  </li>
  <li>
    Provider can be integrated in the Windows PE .wim image
  </li>
</ol>

Generic syntax of the cmdlets:

  * `cd –Path <Path to Drive/Category>`
  * `dir -Path <Path to Drive/Category/Attribute>`
  * `Set-Item –Path <Path to Attribue> -Value <Value To Set> -Password <password value>`

Note 1 – If Admin or System password is installed, Use the _–Password_ parameter to provide the password with _Set-Item_. If password is not installed, the _-Password_ parameter is not processed.

Note 2 – Since _–Path_ and _–Value_ are positional parameters, you can omit writing them.

Note 3 – To find a valid `<Value To Set>`, please check _PossibleValues_ property of an attribute.

Note 4 – Strings for category, attribute and value are case-insensitive.

Note 5 &#8211; _-Password_ parameter accepts password as a plain string as of now.

```
#Set BIOS Admin Password
Set-Item -Path Dellsmbios\Security\AdminPassword –Value dell123

#Change BIOS Admin Password
Set-Item -Path Dellsmbios\Security\AdminPassword –Value dell1234 –Password dell123

#Clear BIOS Admin Password
Set-Item -Path Dellsmbios\Security\AdminPassword –Value “” –Password dell123

#Disable Chassis Intrusion alert
Set-Item -Path Dellsmbios\Security\ChassisIntrusion -Value Disabled

#Set Wake On Lancd
Set-Item -Path Dellsmbios:\PowerManagement\WakeOnLANorWLAN -Value "LANorWLAN"

#Change Asset Tag
Set-Item –Path DellSmbios:\SystemInformation\AssetTag MyAssetTag -Password dell123

#Set WWAN Connection AutoSense
Set-Item -Path Dellsmbios:\PowerManagement\ControlWWANRadio -Value Enabled

#Get Service Tag
Get-ChildItem DellSmbios:\SystemInformation\ServiceTag

#Get Boot Sequence
Get-ChildItem DellSmbios:\BootSequence\Bootsequence
```

![](/images/image001dell.png)

<pre class="brush: powershell; title: ; notranslate" title="">#Enable PXE boot
Set-Item -Path Dellsmbios:\SystemConfiguration\"Integrated NIC" -Value "Enabled w PXE"
</pre>

![](/images/image002dell.png)

Dell Client PowerShell Provider is easy to use and commands are intuitive. Give it a try and share your feedback at <http://en.community.dell.com/techcenter/enterprise-client/f/4887.aspx>

Test-Dellprovider!

[1]: http://en.community.dell.com/techcenter/enterprise-client/f/4887.aspx
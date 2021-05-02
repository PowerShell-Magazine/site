---
title: '#PSTip Detecting Wi-Fi adapters'
author: Shay Levy
type: post
date: 2013-04-04T18:00:12+00:00
url: /2013/04/04/pstip-detecting-wi-fi-adapters/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

Using WMI we can get a list of Wi-Fi adapters with the following command:

```
Get-WmiObject -Namespace root\wmi -Class MSNdis_PhysicalMediumType -Filter "NdisPhysicalMediumType=9 OR NdisPhysicalMediumType=1"
```

I could not find any documentation on the MSNdis_PhysicalMediumType class, but the values of the NdisPhysicalMediumType property maps onto OID_GEN_PHYSICAL_MEDIUM documented <a href="http://msdn.microsoft.com/en-us/library/windows/hardware/ff569621(v=vs.85).aspx">here</a>. The integer values of the NdisPhysicalMediumType enum are missing but can be pulled out of the C/C++ header files in the SDK or WDK:

```
typedef enum _NDIS_PHYSICAL_MEDIUM
{
    NdisPhysicalMediumUnspecified,
    NdisPhysicalMediumWirelessLan,
    NdisPhysicalMediumCableModem,
    NdisPhysicalMediumPhoneLine,
    NdisPhysicalMediumPowerLine,
    NdisPhysicalMediumDSL,      // includes ADSL and UADSL (G.Lite)
    NdisPhysicalMediumFibreChannel,
    NdisPhysicalMedium1394,
    NdisPhysicalMediumWirelessWan,
    NdisPhysicalMediumNative802_11,
    NdisPhysicalMediumBluetooth,
    NdisPhysicalMediumInfiniband,
    NdisPhysicalMediumWiMax,
    NdisPhysicalMediumUWB,
    NdisPhysicalMedium802_3,
    NdisPhysicalMedium802_5,
    NdisPhysicalMediumIrda,
    NdisPhysicalMediumWiredWAN,
    NdisPhysicalMediumWiredCoWan,
    NdisPhysicalMediumOther,
    NdisPhysicalMediumMax       // Not a real physical type, defined as an upper-bound
} NDIS_PHYSICAL_MEDIUM, *PNDIS_PHYSICAL_MEDIUM;
```

A value of 0 translates to NdisPhysicalMediumUnspecified, 1 to NdisPhysicalMediumWirelessLan, 14 translates to NdisPhysicalMedium802_3, and so on.

In Windows 8, this got a lot easier. With the NetAdapter module, we can quickly determine the physical media type of an adapter using the Get-NetAdapter cmdlet.

```
PS> Get-NetAdapter | Where-Object PhysicalMediaType -eq 'Native 802.11'
```

Notice that now we use the value of the media type, not the numeric value. The PhysicalMediaType definition shows the mapping:

```
PS> (Get-NetAdapter | Get-Member PhysicalMediaType).Definition
System.Object PhysicalMediaType {get=$out = switch ($this.NdisPhysicalMedium)
          {
            0 {"Unspecified"}
            1 {"Wireless LAN"}
            2 {"Cable Modem"}
            8 {"Wireless WAN"}
            9 {"Native 802.11"}
            10 {"BlueTooth"}
            11 {"Infiniband"}
            12 {"WiMAX"}
            13 {"UWB"}
            14 {"802.3"}
            16 {"IRDA"}
            17 {"Wired WAN"}
            18 {"Wired Connection Oriented WAN"}
            19 {"Other"}
            default {"Unknown"}
          }
          $out;}
```

Depending on your environment, you could also use this command to cover all Wi-Fi media types:

```
Get-NetAdapter |
Where-Object {$_.PhysicalMediaType -eq 'Native 802.11' -or $_.PhysicalMediaType -eq 'Wireless LAN' -or 'Wireless WAN" }
```

Lastly, here's a valuable piece of information you might want to consider when you query Wireless adapters:

- Native 802.11: Most WiFi drivers
- Wireless LAN : Very old WiFi drivers
- Wireless WAN : Some 3G/4G mobile broadband adapters (not all)

The latest Windows 8 telemetry shows that approximately 0.2% of WiFi adapters are of the very old variety.
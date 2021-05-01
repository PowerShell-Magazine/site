---
title: '#PSTip Preview the members of a dynamic distribution group'
author: Shay Levy
type: post
date: 2013-09-13T18:00:13+00:00
url: /2013/09/13/pstip-preview-the-members-of-a-dynamic-distribution-group/
categories:
  - Tips and Tricks
  - Exchange
tags:
  - Tips and Tricks
  - Exchange

---
**Note**: This tip requires PowerShell 2.0 or above.

Getting the members of a dynamic distribution group in the Exchange Management Console (EMC) is not an issue; you open the group and click the Preview button.

![](/images/preview.png)

However, if you list the properties of the group in the Exchange Management Shell (EMS) you will not find the members as they are determined at the run time by the custom filter used when the group was created. Viewing the members of the group is twofold&#8211;you get the group&#8217;s recipient filter and use it with the _Get-Recipient_ cmdlet!

```
$ddg = Get-DynamicDistributionGroup ITUsers
Get-Recipient –RecipientPreviewFilter $ddg.RecipientFilter
```


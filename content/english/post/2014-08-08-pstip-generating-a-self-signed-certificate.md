---
title: '#PSTip Generating a self-signed certificate'
author: Jan Egil Ring
type: post
date: 2014-08-08T18:00:50+00:00
url: /2014/08/08/pstip-generating-a-self-signed-certificate/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Self-signed certificates might be needed for different purposes, such as the test environments. A more practical example is Azure Recovery Services where self-signed certificates can be used as <a href="http://msdn.microsoft.com/en-us/library/azure/dn169036.aspx">vault certificates</a>.

Many online articles suggests using the MakeCert.exe tool available in the Windows SDK for creating a self-signed certificate, but now there is an easier approach available.

Introduced in Windows 8.1 and Windows Server 2012 R2, the Public Key Infrastructure (PKI) Client module offers the [New-SelfSignedCertificate][1] cmdlet to create a self-signed certificate.

```
New-SelfSignedCertificate -DnsName test.powershellmagazine.com -CertStoreLocation cert:\LocalMachine\My
```


[1]: http://technet.microsoft.com/en-us/library/hh848633.aspx
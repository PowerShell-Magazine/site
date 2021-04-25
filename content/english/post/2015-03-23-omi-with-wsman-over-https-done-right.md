---
title: 'OMI with WSMan over HTTPS: Done right.'
author: Bartek Bielawski
type: post
date: 2015-03-23T16:00:02+00:00
url: /2015/03/23/omi-with-wsman-over-https-done-right/
views:
  - 5690
post_views_count:
  - 1924
categories:
  - Linux
  - OMI
tags:
  - Linux
  - OMI

---
When you install OMI for the first time, a pair of keys is generated for you. When an automated process is used, the only option that will &#8220;just work&#8221; is to generate self-signed certificate. The logical next step is to convince your machine that it should connect to OMI server even though it doesn't really trust this certificate. Therefore most of examples that use CIM connection to OMI server you will find will look like this:

```powershell
$paramCimOption = @{
    UseSsl = $true 
    SkipCACheck = $true 
    SkipCNCheck = $true 
    SkipRevocationCheck = $true
}
$opt = New-CimSessionOption @paramCimOption
$paramCimSession = @{
    Credential = $cred 
    ComputerName = '192.168.3.20' 
    Authentication = 'Basic' 
    SessionOption = $opt 
}
$demo1 = New-CimSession @paramCimSession
```


Good for very basic demos but if you consider using OMI and/or Linux DSC in the production environment you shouldn't depend on automatically generated, self-signed certificate. The whole point of communication over HTTPS is that both devices on each side of the wire know and trust each other. With self-signed certificates it becomes false-protection, a security not applied.

If you have internal PKI, it&#8217;s relatively straight-forward to create proper, domain-wide trusted certificate for any network device. Existence of the internal PKI should be a safe assumption: any organization that uses SSL and doesn't have own PKI is either wasting money on 3rd-party certificates or is &#8220;doing it wrong&#8221; for any SSL-based application.

First thing we have to do is to generate certificate request and private key on Linux machine. To achieve this we will use _openssl_ command. To avoid prompts we will provide all parameters inline. To call commands on Linux we will use Posh-SSH module, described <a href="http://104.131.21.239/2014/07/03/posh-ssh-open-source-ssh-powershell-module/" target="_blank">here</a>:

```powershell
$ssh = New-SSHSession -ComputerName DSCTest.bielawscy.com -AcceptKey $true -Credential $root
$command = -join @(
    'openssl req -new -newkey rsa:4096 -nodes',
    ' -subj "/C=PL/L=Koszalin/O=Bielawscy/CN=DSCTest.bielawscy.com"',
    ' -keyout omikey.pem -out omi.req -passout pass:s3cr3t!'
)
Invoke-SSHCommand -SSHSession $ssh -Command $command | ForEach-Object Error
```


Request file (omi.req) needs to be copied from Linux box to our Windows system. There are many methods to achieve this; we will use another part of Posh-SSH module, _Get-SCPFile_ command. Just to be sure we are using correct file we will first remove any files with base name _omi_ already present in the _c:\temp_ folder:

```powershell
Remove-Item -Path c:\temp\omi.* -ErrorAction SilentlyContinue
$getFile = @{
    ComputerName = 'DSCTest.bielawscy.com' 
    Credential = $root 
    LocalFile = 'c:\temp\omi.req' 
    RemoteFile = 'omi.req'
}
Get-SCPFile @getFile
```


Next step requires communication with an internal CA. We will use traditional _certreq.exe_ tool to perform this operation:

```shell
certreq -submit -attrib "CertificateTemplate:SSL" -config CA.bielawscy.com\bielawscy-CA-CA c:\temp\omi.req c:\temp\omi.crt
```

> The procedure to request/issue certificate may differ in your environment: in my case I had certificate template "SSL" available for me that suits the requirements of OMI certificate and I had necessary rights to get the final certificate immediately.

Next step is to copy the certificate back to our OMI server. For that we will use Posh-SSH again, this time _Set-SCPFile_ cmdlet to copy a local file to the remote system:

```powershell
$setFile = @{
    ComputerName = 'DSCTest.bielawscy.com' 
    Credential = $root 
    LocalFile = 'c:\temp\omi.crt' 
    RemoteFile  = 'omi.crt'
}
Set-SCPFile @setFile
```


We have our certificate ready, but the file format does not match the one used for the private key. We need to convert it to the correct format using _openssl_ command. With the both files ready we can copy existing files (just in case) to separate folder, replace certificate with the one we trust and restart OMI server:

```powershell
$commands = @(
    'openssl x509 -in omi.crt -out omi.pem -outform PEM',
    'mkdir self-signed',
    'cp /opt/omi-1.0.8/etc/ssl/certs/omi* self-signed/',
    'cp ~/omi*.pem /opt/omi-1.0.8/etc/ssl/certs/',
    'systemctl restart omi'
)
foreach ($command in $commands) {
    $exit = Invoke-SSHCommand -SSHSession $ssh -Command $command
    if ($exit.ExitStatus -gt 0) {
        $exit.Error
    }
}
```

> The command used to restart OMI service depends on the tool that is used to control the services on the Linux box. In my case (CentOS 7) it was _systemd/systemctl_.

Our OMI server is using a proper certificate, so we should be able to connect to it without any of the switches that we&#8217;ve used before to accept self-signed certificate. Once connected, we ask for instance of _OMI_Identify_, to make sure we are talking to the correct server:

```powershell
$paramCimSession = @{
    ComputerName = 'DSCTest.bielawscy.com' 
    Credential = $root 
    Authentication = 'Basic' 
    SessionOption = New-CimSessionOption -UseSsl
}

$cimSession = New-CimSession @paramCimSession
Get-CimInstance -CimSession $cimSession -ClassName OMI_Identify -Namespace root\OMI
```


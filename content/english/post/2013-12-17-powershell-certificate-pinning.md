---
title: PowerShell Certificate Pinning
author: Carlos Perez
type: post
date: 2013-12-17T17:00:50+00:00
url: /2013/12/17/powershell-certificate-pinning/
categories:
  - How To
tags:
  - How To

---
Recently, we have seen in the news how governments intercept communications. We’ve seen evidence that they are abusing their powers against those who run valid Certificate Authorities, creating fraudulent certificates to [intercept SSL/TLS encrypted communications][1]. Criminals have also succeeded in such schemes, as with [DigiNotar][2].

Currently, we work with SSL/TLS by using a chain of trust, where we trust CAs and their root certificates in our applications. We use certificate pinning, where we check the thumbprint or public key of the certificate to remove the &#8220;conference of trust&#8221;. When we pin a certificate or public key, we no longer must depend on others to make peer identity security decisions.

In Windows PowerShell, we have several ways of performing certificate pinning. Let’s start with the one that will work on Windows PowerShell 2.0, 3.0 and 4.0. We validate our work by using the .NET Framework _System.Net.ServicePointManager_ class. The client validates the server certificate by using a code block to which we have assigned the class property _ServerCertificateValidationCallback_. If the code block returns a Boolean value of _True_, then the validation passed; it returns _False_ if it did not.

Because we can create several callbacks, we start by limiting the maximum number to one, and limiting the idle time of the service point before it is eligible for garbage collection. A _ServicePoint_ object is idle when the list of connections associated with the object is empty. This way, we can reset it to its default behavior. We can also change the code block, and validate another certificate for use against another site. (Thanks to **James Forshaw** for helping me to figure this out.)

```
[Net.ServicePointManager]::MaxServicePoints = 1
[Net.ServicePointManager]::MaxServicePointIdleTime = 1
```


We now set the code block we will use to validate the certificate:


    [Net.ServicePointManager]::ServerCertificateValidationCallback = {
        $ThumbPrint = "91a6316868bb63d7203f2594da582386210b698"
        $certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]$args[1]
    
        if ($certificate -eq $null)
        {
            $Host.UI.WriteErrorLine("Null certificate.")
            return $false
        }
    
        if ($certificate.Thumbprint -eq $ThumbPrint)
        {
            return $true
        }
        else
        {
            $Host.UI.WriteErrorLine("Thumbprint mismatch. Certificate thumbprint $($certificate.Thumbprint)")
        }
    
        return $false
    }
In the preceding example, we check only the certificate thumbprint saved in a variable. We cast the second argument to the _X509Certificate2_ type, since this type includes the thumbprint information in its properties. If the code block returns _True_, the certificate is accepted; if _False_, it fails validation, and the connection is terminated.

First, we check for a null certificate, which can happen if our connection was redirected (such as with a SSLStrip attack). Because we are working with a callback, to show a message, we must use _$Host.UI_. Next, we compare the thumbprint of the certificate obtained from the connection to the one we are expecting to validate the host.

The callback runs each time we use the .NET Framework _System.Net.WebRequest, System.Net.FtpWebRequest_, or _System.Net.WebClient_ classes. Because the failure to validate results in an exception, we must control the execution of our code.


    Try
    {
        # Create web request
    	$WebRequest = [Net.WebRequest]::Create("https://encrypted.google.com/")
        # Get response stream
        $ResponseStream = $webrequest.GetResponse().GetResponseStream()
    
        # Create a stream reader and read the stream returning the string value.
        $StreamReader = New-Object System.IO.StreamReader -ArgumentList $ResponseStream
        $StreamReader.ReadToEnd()
    }
    catch
    {
        Write-Error "Failed: $($_.exception.innerexception.message)"
    }
If the server is going through a proxy (in this example, I have my system using Burp Proxy with a self-signed certificate), I will get an error, and a description of why:

```
PS> C:\Users\Carlos\Desktop\certpin.ps1
Thumbprint mismatch. Certificate thumbprint 91A6316868BB63D7203F2594DA582386210CB698

C:\Users\Carlos\Desktop\certpin.ps1 : Failed: The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.

+ CategoryInfo : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,certpin.ps1
```

The entire example code follows:

    [Net.ServicePointManager]::MaxServicePoints = 1
    [Net.ServicePointManager]::MaxServicePointIdleTime = 1
    [Net.ServicePointManager]::ServerCertificateValidationCallback = {
        $ThumbPrint = "91a6316868bb63d7203f2594da582386210cb698"
        $certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]$args[1]
    
        if($certificate -eq $null)
        {
            $Host.UI.WriteErrorLine("Null certificate.")
            return $false
        }
    
        if ($certificate.Thumbprint -eq $ThumbPrint)
        {
            return $true
        }
        else
        {
            $Host.UI.WriteErrorLine("Thumbprint mismatch. Certificate thumbprint $($certificate.Thumbprint)")
        }
    
        return $false
    }
    
    Try
    {
    
        # Create web request
    	$WebRequest = [Net.WebRequest]::Create("https://encrypted.google.com/")
    
        # Get response stream
        $ResponseStream = $webrequest.GetResponse().GetResponseStream()
    
        # Create a stream reader and read the stream returning the string value.
        $StreamReader = New-Object System.IO.StreamReader -ArgumentList $ResponseStream
        $StreamReader.ReadToEnd()
    }
    catch
    {
        Write-Error "Failed: $($_.exception.innerexception.message)"
    }
If you are using Windows PowerShell 4.0, you can simplify the process, and use the _ServerCertificateValidationCallback_ property of a _System.Net.WebRequest_ object. You can limit it to that object, eliminating the need to modify the service point manager. Here is the code.


    Try
    {
    
        # Create web request
    	$WebRequest = [System.Net.WebRequest]::Create("https://encrypted.google.com/")
        # Set the callback to check for null certificate and thumbprint matching.
        $WebRequest.ServerCertificateValidationCallback = {
            $ThumbPrint = "91a6316868bb63d7203f2594da582386210cb698"
            $certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]$args[1]
            if ($certificate -eq $null)
            {
                $Host.UI.WriteWarningLine("Null certificate.")
                return $false
            }
    
            if ($certificate.Thumbprint -eq $ThumbPrint)
            {
                return $true
            }
            else
            {
                $Host.UI.WriteWarningLine("Thumbprint mismatch. Certificate thumbprint $($certificate.Thumbprint)")
        }
    
        return $false
    }
    # Get response stream
    $ResponseStream = $webrequest.GetResponse().GetResponseStream()
    
    # Create a stream reader and read the stream returning the string value.
    $StreamReader = New-Object System.IO.StreamReader -ArgumentList $ResponseStream
    $StreamReader.ReadToEnd()
    }
    catch
    {
        Write-Error "Failed: $($_.exception.innerexception.message)"
    }

Here is another example, using FTP over SSL to upload a file:

    [Net.ServicePointManager]::MaxServicePoints = 1
    [Net.ServicePointManager]::MaxServicePointIdleTime = 1
    
    [Net.ServicePointManager]::ServerCertificateValidationCallback = {
        $ThumbPrint = "c3c6d4ca0f778817d322cb61302f77a19c285798"
        $certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]$args[1]
    
        if ($certificate -eq $null)
        {
            $Host.UI.WriteErrorLine("Null certificate.")
            return $false
        }
    
        if ($certificate.Thumbprint -eq $ThumbPrint)
        {
            return $true
        }
        else
        {
            $Host.UI.WriteErrorLine("Thumbprint mismatch. Certificate thumbprint $($certificate.Thumbprint)")
        }
    
        return $false
    }
    
    Try
    {
    	#create the FtpWebRequest and configure it
    	$ftp = [System.Net.FtpWebRequest]::Create("ftp://192.168.6.158/secretdocuments.zip")
        $ftp.Method = [System.Net.WebRequestMethods+Ftp]::UploadFile
        $ftp.Credentials = New-Object System.Net.NetworkCredential("snowden","5up3r53cr37")
        $ftp.UseBinary = $true
        $ftp.UsePassive = $true
        $ftp.EnableSsl = $true
        # read in the file to upload as a byte array
    $content = [System.IO.File]::ReadAllBytes("C:\Windows\Temp\secretdocument.zip")
    $ftp.ContentLength = $content.Length
    
        # get the request stream, and write the bytes into it
        $rs = $ftp.GetRequestStream()
        $rs.Write($content, 0, $content.Length)
    
        # be sure to clean up after ourselves
        $rs.Close()
        $rs.Dispose()
    }
    catch
    {
        Write-Error "Failed: $($_.exception.innerexception.message)"
    }
You can use this method when you download information from an HTTPS protected site to validate the confidentiality of the information and the integrity of the data during transport. It can also be used to upload a file to a WebDav Share, or via FTP using SSL. This method ensures that no “man in the middle” attack can steal or modify the information.

[1]: http://www.zdnet.com/google-catches-french-govt-spoofing-its-domain-certificates-7000024062/
[2]: https://productforums.google.com/forum/#!topic/gmail/3J3r2JqFNTw/discussion
---
title: Working with PowerShell DSC for Linux, part 3
author: Bartek Bielawski
type: post
date: 2015-02-25T17:00:54+00:00
url: /2015/02/25/working-with-powershell-dsc-for-linux-part-3/
views:
  - 8267
post_views_count:
  - 1605
categories:
  - PowerShell DSC
  - Linux
  - OMI
tags:
  - PowerShell DSC
  - Linux
  - OMI

---
Linux is a system that stores configuration in text files. As a result, _nxFile_ should be considered one of two most powerful resources offered in the CTP release. With the option to create and modify files on Linux box, we should be able to change or create configuration for services, modify default environment, configure networking, configure ssh authorized_keys for a given user to name just a few examples. On top of that, this resource can be used to deploy and configure web sites and other, less frequently used applications. In this article, we will take a look at few example definitions that take advantage of _nxFile_ resource. We will also look at one pitfall that may be important while using it and a way to walk around it. Last but not least we will take a look at one bug and a way to fix it.

Using file resource to configure services is the best option when service has a concept of &#8220;included&#8221; configuration. This means that we can create a file from scratch and &#8220;main&#8221; configuration document will load it during startup of the service. If service doesn&#8217;t support it and uses single file then replacing it with the one created with DSC may be tricky (unless there is really good reason to prevent any changes to this configuration file). In our example, we will use three _nxFile_ resource items to create very simple web application with virtual hostname.

First thing we have to do is creating virtual host configuration in /etc/httpd/config.d folder with extension conf. Name is not important for the service, but it makes sense to name the file the same as virtual hostname that we want to describe in it:

```powershell
nxFile vhostConfig {
    DestinationPath = '/etc/httpd/conf.d/bielawscy.conf'
    Contents = @'
<VirtualHost *:80>
    ServerName www.bielawscy.com
    ServerAdmin webmaster@bielawscy.com
    ErrorLog /var/log/httpd/bielawscy.err
    CustomLog /var/log/httpd/bielawscy.log combined
    DocumentRoot /var/www/bielawscy.com
    <Directory "/var/www/bielawscy.com">
        Order allow,deny
        Allow from all
	</Directory>
</VirtualHost>
'@
}
```


Once we have configuration defined, we need to create folder and root document with appropriate masks/content:

```powershell
nxFile documentRoot {
    DestinationPath = '/var/www/bielawscy.com'
    Type = 'Directory'
    Mode = 755
}

nxFile index {
    DestinationPath = '/var/www/bielawscy.com/index.html'
    Mode = 644
    Contents = @'
<html>
    <head>
        <title>Welcome to www.bielawscy.com!</title>
    </head>
	<body>
        <h1>Welcome to BIELAWSCY.COM!</h1>
	</body>
</html>
'@
}
```

To activate this change we would need _nxScript_: Apache configuration would have to be updated and there is no build-in way to achieve this without some plumbing on the user side.

Environment for the users is defined by various profiles. Global profile is stored in _/etc/profile_ but in a way similar to extending Apache configuration, we can &#8220;inject&#8221; our configuration by creating separate file in _/etc/profile.d_ folder. For example, if we want to define _OMI_HOME_ variable and make sure that folder where OMI was installed is part of the _PATH_ variable we would need to add following _nxFile_ item to our configuration:

```powershell
nxFile omiPath {
    DestinationPath = '/etc/profile.d/omipath.sh'
    Contents = @'
PATH=$PATH:/opt/omi-1.0.8/bin
export OMI_HOME=/opt/omi-1.0.8
'@
}
```


One problem that I identified while working with this resource is related to encoding. So far our files were not using any characters that could cause encoding problems. Example configuration that includes Polish letters:

```powershell
nxFile webPolish {
    DestinationPath = '/var/www/html/index.html'
    Contents = @'

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<head>
    <title>Linux DSC page on CentOS</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <style type="text/css">
(...)
    </style>
        </head>

<body>
(...)
    <h2>Zażółć gęślą jaźń - UTF8 is necessary here!
</body>
</html>
'@
}
```

Resulting MOF document (at least in my tests) was created using Little Endian encoding. If you are using DSC on Windows nodes only than it won&#8217;t bother you. But Linux (or at least: Linux DSC) doesn&#8217;t work with MOF encoded like that. When you try to apply this configuration to Linux node you can expect following error message:

Buffer is not supported. Check encoding and length of the buffer


To work around this problem we have to add extra step before applying the configuration. We have to open MOF document and save it again using correct encoding. In my experience both UTF8 and UTF8 without BOM work fine. There is a better solution though. Instead of hardcoding content of files in our configuration document we can use external source and download entire folder directly to target location:

```powershell
nxFile pageSource {
    SourcePath = '/mnt/www/Symisun'
    DestinationPath = '/var/www/bielawscy.com'
    Type = 'Directory'
    Force = $true
    Recurse = $true
}
```


We are using SMB share as a source (mounted under /mnt/www). That gives us a flexibility and easy way to deploy entire application, rather than deploying individual files one by one. Unfortunately, this is the moment when we may trip on a bug I mentioned at the beginning. If you would try to apply this configuration it will fail with the error message that I wouldn&#8217;t consider helpful:

Loading the instance document from the pending location did not succeed.


Logs on Linux side don&#8217;t reveal anything. Based on my previous experience I was suspecting that the problem is related to schema: if MOF created on Windows box is not recognized correctly on Linux box, than that&#8217;s the error message you can expect. Eventually I identified the cause. Schema MOF for _nxFile_ provider is different in nx module that we use on Windows and the one used by DSC on Linux. I knew something was wrong with both _Recurse_ and _Force_ property (removing these properties was enough to prevent error from happening) and sure enough, these properties are defined differently on Windows and Linux:

```
/* Windows */
       [Write, ValueMap{"true", "false"}] string Recurse;
       [Write, ValueMap{"true", "false", "yes", "no"}] string Force;
/* Linux */
       [Write] boolean Recurse;
       [Write] boolean Force;
```


Once the MOF on Windows side is fixed, our configuration should work fine. And with option to recurse/force we can easily deploy whole application, without hardcoding anything in the configuration itself. And that&#8217;s it for this part of the series. In the next part I will take a closer look at the resource that is both very powerful and flexible: _nxScript_.
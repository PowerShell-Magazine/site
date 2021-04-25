---
title: '#PSTip Sending emails using your Gmail account'
author: Shay Levy
type: post
date: 2012-10-25T18:00:34+00:00
url: /2012/10/25/pstip-sending-emails-using-your-gmail-account/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

The Send-MailMessage cmdlet enables you to quickly and easily send e-mail message from within Windows PowerShell. In version 2.0, establishing connections that required alternate port numbers wasn&#8217;t possible simply because there wasn&#8217;t a way to specify them.

In PowerShell 3.0, we now have the _Port_ parameter, together with the _UseSsl_ switch (required to secure the session). You can send an email using your Gmail account.

The following example creates a [splatting][1] hash table and passes it to the Send-MailMessage cmdlet as one object. Execute the code (change the values to match your own and supply your Gmail credentials when prompted), and then check your email account.

```
$param = @{
    SmtpServer = 'smtp.gmail.com'
    Port = 587
    UseSsl = $true
    Credential  = 'you@gmail.com'
    From = 'you@gmail.com'
    To = 'someone@somewhere.com'
    Subject = 'Sending emails through Gmail with Send-MailMessage'
    Body = "Check out the PowerShellMagazine.com website!"
    Attachments = 'D:\articles.csv'
}
Send-MailMessage @param
```

For a PowerShell 2.0 solution, see this [forum thread][2].

[1]: http://technet.microsoft.com/en-us/library/jj672955
[2]: http://stackoverflow.com/a/1254914/9833
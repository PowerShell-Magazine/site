---
title: 'Weekly Module Spotlight: Polaris'
author: Ravikanth C
type: post
date: 2019-09-03T05:41:44+00:00
url: /2019/09/03/weekly-module-spotlight-polaris/
post_views_count:
  - 8188
views:
  - 8031
categories:
  - Module Spotlight
  - Polaris
tags:
  - Modules

---
I create HTTP REST APIs a lot in my proof-of-concept work and I generally use the [.NET HTTPListener](https://docs.microsoft.com/en-us/dotnet/api/system.net.httplistener?view=netframework-4.8) class for this purpose. Using this class we can create simple and programmatically controlled HTTP listeners. For some quick prototype this totally makes sense as it is inbox and there is no need for any external modules or libraries. However, creating complex endpoints won&#8217;t be easy. This is where [Polaris](https://github.com/PowerShell/Polaris) plays a role. 

Polaris is a cross-platform, minimalist web framework for PowerShell. This is an experimental module but stable enough to try it out. Before you can try out the examples, install Polaris module from [PS Gallery](https://www.powershellgallery.com/packages/Polaris/).

Here is a quick example of how we create a HTTP endpoint.

```powershell
New-PolarisGetRoute -Path "/helloworld" -Scriptblock {
    $Response.Send('Hello World!')
}

Start-Polaris
```

Once these commands are executed, if you access http://localhost:8080/helloworld in a browser, you will see &#8216;Hello World!&#8217; returned as response. In the above example, there is just one endpoint or route. This implements HTTP GET method. Whenever this route gets accessed, the PowerShell commands specified as an argument to _-Scriptblock_ parameter gets invoked. In this example, we are just using the _$Response_ automatic variable that Polaris provides and use the _.Send()_ method to send the response back to browser.

Similar to this, you can create other HTTP routes as well for POST, PUT, DELETE, and so on. In the next example, you will see how we can combine what [PSHTML provides with Polaris](https://www.powershellmagazine.com/2019/08/21/weekly-module-spotlight-pshtml/). 

Save the following script as content.ps1 in a folder of your choice.

```powershell
html {
    head {
        title "Example 2"
    }
    
    body {
        h1 {"This is just an example of using PSHTML as view engine for Polaris"}
    
        $Languages = @("PowerShell","Python","CSharp","Bash")
    
        "My Favorite language are:"
        ul{
          foreach($language in $Languages){
               li {
                    $Language
               }
           }
        }
    }
    
    Footer {
        h6 "This is h1 Title in Footer"
    }
}
```

The following script is the route that we need to create for displaying the HTML content from the above script.

```powershell
New-PolarisGetRoute -Path "/languages" -Scriptblock {
    $response.SetContentType('text/html')
    $html = . "C:\scripts\content.ps1"
    $response.Send($html)
}

Start-Polaris -Port 8080
```

Once you start Polaris and load the routes, you can access http://localhost:8080 to see the content generated from a PS1 script. It should be like this!

{{< figure src="/images/polaris1.png" >}} {{< load-photoswipe >}}

Hope you got a hang of what you can achieve with PSHTML and Polaris combined. I have built dashboards with simply this combination and nothing more. In the future posts, I will show one such example from my demo at PowerShell Conference Europe 2019.
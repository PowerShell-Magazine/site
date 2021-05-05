---
title: 'Weekly Module Spotlight: PSHTML'
author: Ravikanth C
type: post
date: 2019-08-21T04:00:12+00:00
url: /2019/08/21/weekly-module-spotlight-pshtml/
post_views_count:
  - 5546
views:
  - 5512
categories:
  - Module Spotlight
  - PSHTML
tags:
  - Modules

---
Ever wanted to generate HTML documents dynamically? Until now, you had to statically code the HTML tags in the PowerShell scripts and ensure you open and close all needed HTML elements. Very cumbersome, tedious, and not very efficient.

Do that no more! [PSHTML](https://github.com/Stephanevg/PSHTML ) is here.

PSHTML is a cross platform PowerShell module to generate HTML markup language within a DSL on Windows and Linux.

With PSHTML, you can write HTML documents the same way you write PowerShell scripts. You can use every possible language artifact and generate HTML code dynamically based on the input and context. Let us see a quick example!

Before you go forward and try out the example below, install the module from the PS Gallery.

```powershell
html {
    head {
        title 'This is a test HTML page'
    }

    body {
        h2 'PSHTML is cool!'    
        p {
            'Using PSHTML, offers code completion and syntax highlighting from the the default powershell language.'
            'As PSHTML respects the W3C standards, any HTML errors, will be spotted immediately.'
        }
    }
    
    footer {
        p {
            'This is footer. All credits reserved to PSHTML'
        }
    }
}
```

This generates the following HTML text.

{{< figure src="/images/pshtml1.png" >}} {{< load-photoswipe >}}

This is very easy. Let us give it some styles. 

```powershell
html {
    head {
        title 'This is a test HTML page'
    }

    body {
        h2 'PSHTML is cool!' -Style 'color:blue;background-color:powderblue'
    
        p -Style 'color:red' {
            'Using PSHTML, offers code completion and syntax highlighting from the the default powershell language.'
            'As PSHTML respects the W3C standards, any HTML errors, will be spotted immediately.'
        }
    }
    
    footer {
        p -Style 'color:green' {
            'This is footer. All credits reserved to PSHTML'
        }
    }
}
```

This results in a HTML page as shown below!

{{< figure src="/images/pshtml2.png" >}}

This is all good but very trivial. Let us try generating some tables and we will use bootstrap for styles.

```powershell
html {
    head {
        title 'Top 5 Processes - HTML report Powered by PSHTML'
        Link -href 'https://maxcdn.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css' -rel 'stylesheet'
        script -src 'https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js' -type 'text/javascript'
        script -src 'https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js' -type 'text/javascript'
        script -src 'https://maxcdn.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js' -type 'text/javascript'
    }

    body {
        $topFiveProcess = Get-Process | Sort CPU -descending | Select -first 5 -Property ID,ProcessName,CPU
        div -Content {
            table -class 'table table-striped' {
                thead {
                    tr {
                        th {
                            ID
                        }
        
                        th {
                            ProcessName
                        }
        
                        th {
                            CPU
                        }
                    }
                }
                                
                Tbody {
                    foreach ($process in $topFiveProcess)
                    {
                        tr {
                            td {
                                $process.id
                            }
            
                            td {
                                $process.Name
                            }
            
                            td {
                                $process.CPU
                            }
                        }
                    }
                }
            }
        }
    }
    
    footer {
        p -Class 'lead' -Content 'All Credits Reserved. PSHTML!'
    }

}
```

This results in a nice table shown below!

{{< figure src="/images/pshtml3.png" >}}

See how easy was that!? I will stop this article here as this is not a PSHTML tutorial. Hope you have got a good idea about how useful the module is. There are several [community members](https://github.com/Stephanevg/PSHTML) who did some great work with PSHTML. Check out their work as well.
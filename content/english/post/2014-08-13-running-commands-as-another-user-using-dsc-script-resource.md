---
title: Running commands as another user using DSC script resource
author: Ravikanth C
type: post
date: 2014-08-13T18:42:42+00:00
url: /2014/08/13/running-commands-as-another-user-using-dsc-script-resource/
categories:
  - PowerShell DSC
tags:
  - PowerShell DSC

---
A while ago, I received an email from a reader who wanted to run a few Exchange PowerShell cmdlets using DSC. DSC runs as SYSTEM and using Exchange cmdlets will result in an access denied error because SYSTEM won&#8217;t be a part of Exchange administration security groups. So, we need to be able to pass credentials to run those cmdlets as an Exchange administrator. Given this context, the first thought would be to create a new custom resource but given the number of configuration settings Exchange has, it would be overwhelming to create such granular resources. The immediate alternative is to use the DSC script resource. Using the Script resource, we can run any ad hoc script block on the target systems. I use the Script resource as a prototyping mechanism when starting custom DSC resource development. There are a few glitches with this though.

### Script resource and parameters

The values provided as arguments to the _GetScript_, _SetScript_, and _TestScript_ properties get converted to a string when the configuration MOF gets generated. There is no a variable expansion done in these script blocks. Here is an example:


    Configuration DemoConfig {
        param (
        	$name
        )
    	Node WSR2-1 {
    		Script DemoScript {
                GetScript = {
                #Do Nothing
                }
                SetScript = {
                Write-Verbose -Message $name
                }
    
                TestScript = {
                $false
                }
    	}
      }
    }
    DemoConfig -Name "PowerShell"
When we enact this configuration, we see an error.

![](/images/11-1024x239.png)

It is evident from the error message that the value of the variable $name did not get expanded in the SetScript property. If you still need a proof, you can open the MOF file and check its contents. You will see that the values provided as the arguments to Script resource properties simply get converted to string with no variable expansion done inside them.

```
/*
@TargetNode='WSR2-1'
@GeneratedBy=Administrator
@GenerationDate=08/13/2014 15:31:46
@GenerationHost=DEMO-AD
*/

instance of MSFT_ScriptResource as $MSFT_ScriptResource1ref
{
ResourceID = "[Script]DemoScript";
GetScript = "\n #Do Nothing\n ";
TestScript = "\n $false\n ";
SourceInfo = "::6::9::Script";
SetScript = " \n Write-Verbose -Message $name\n ";
ModuleName = "PSDesiredStateConfiguration";
ModuleVersion = "1.0";

};

instance of OMI_ConfigurationDocument
{
Version="1.0.0";
Author="Administrator";
GenerationDate="08/13/2014 15:31:46";
GenerationHost="DEMO-AD";
};
```

So, one workaround is to hardcode these variable values inside the Script resource properties or dynamically generate the script blocks used for GetScript, SetScript, and TestScript. Dave Wyatt [shows this second method as an example here][1]. So, I won&#8217;t repeat it. For now and for demonstration purposes, we will hard-code the values. If you enact this configuration, you will see the word PowerShell in the verbose output from the Start-DscConfiguration cmdlet.


    Configuration DemoConfig {
    	Node WSR2-1 {
    		Script DemoScript {
                GetScript = {
                	#Do Nothing
                }
                SetScript = {
                	Write-Verbose -Message "PowerShell"
                }
                TestScript = {
                	$false
                }
    		}
    	}
    }
    
    DemoConfig
### Running commands as another user

Coming to the subject of this article, to run commands as another user, we can use the _Invoke-Command_ cmdlet. However, we need to pass the credentials of the user. We just saw that it is not possible to expand variables inside SetScript. So, instead of passing the credentials as a variable, we need to construct those credentials from plain text. To really prove that we are using alternate credentials, I am using whoami command inside the _Invoke-Command_ script block. This should return the username of the account used to run the command.


    Configuration DemoConfig {
    	Node WSR2-1 {
    		Script DemoScript {
    			GetScript = {
    				#Do Nothing
    			}
    			SetScript = {
    				$secpasswd = ConvertTo-SecureString "Dell1234" -AsPlainText -Force
    				$mycreds = New-Object System.Management.Automation.PSCredential ("Administrator", $secpasswd)
    				$output = Invoke-Command -ScriptBlock { $(whoami) } -ComputerName localhost -Credential $mycreds -Verbose
    				Write-Verbose $output
    			}
                TestScript = {
                	$false
                }
    		}
    	}
    }
    
    DemoConfig
When we enact this, we will see that the alternate credentials are used to run Invoke-Command script block instead of SYSTEM account.

![](/images/dscscr.png)

You can replace the value of -Scriptblock parameter of the Invoke-Command cmdlet to execute whatever you need to run as a different user. This is easy! But, do I recommend using plain text credentials in a configuration script? No! You should always encrypt the credentials when using DSC and that is the subject of our next article. Stay tuned.

[1]: http://social.technet.microsoft.com/Forums/en-US/2eb97d67-f1fb-4857-8840-de9c4cb9cae0/dsc-configuration-data-for-script-resources?forum=winserverpowershell
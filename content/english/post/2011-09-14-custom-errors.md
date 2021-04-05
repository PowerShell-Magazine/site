---
title: Custom errors
author: Robert Robelo
type: post
date: 2011-09-14T07:44:49+00:00
url: /2011/09/14/custom-errors/
post_views_count:
  - 4191
categories:
  - How To
tags:
  - How To

---
The most common ways of reporting errors in PowerShell are through the Write-Error Cmdlet or the Throw statement. Write-Error writes non-terminating errors while the throw statement writes terminating errors which halt execution but are not very descriptive. Both methods write Management.Automation.ErrorRecord objects to the error stream. Write-Error lets you customize the ErrorRecord it reports through its parameters; this allows you to provide tailored specifics about an error, assisting the user to avoid or resolve the reason that caused the problem. In contrast, the Throw statement only lets us provide a custom message, which in some cases could be enough.

Custom ErrorRecord objects are not very common, but if you ever want to provide better details about an error that is not very clear, you can create and report your own. For instance, I wrote a function that appends data to an existing CSV file; before any piped input is processed, the function will report one of three custom ErrorRecord objects as a terminating error if the destination -or target- file:

  1. does not exist
  2. is empty, or
  3. has a different character encoding than the specified -or default- encoding

After the piped input is processed but before the processed data is written to disk, the function compares the original header fields against the processed data header fields, if any inconsistency is found the function reports the fourth custom ErrorRecord and stops execution without appending the processed data.

I could have used a Throw statement to report the problem and stop execution, but decided to make my advanced function behave more professionally with custom ErrorRecord objects.

PowerShell 2.0 introduced advanced functions which are very similar to Cmdlets. Through advanced functions, we haves access to most members of the PSCmdlet Class. The object through which we have access to these members is the $PSCmdlet object, which is only available in advanced functions. The $PSCmdlet object lets us report a terminating error through its ThrowTerminatingError method which takes one argument, an ErrorRecord. To construct an ErrorRecord we need four arguments:

  * [System.Exception]$exception
      * The Exception that will be associated with the ErrorRecord
  * [System.String]$errorId
      * A scripter-defined identifier of the error.
      * This identifier must be a non-localized string for a specific error type.
  * [Management.Automation.ErrorCategory]$errorCategory
      * An ErrorCategory enumeration that defines the category of the error.
  * [System.Object]$targetObject
      * The object that was being processed when the error took place.

Notice that the first argument is a System.Exception, this is the object from which all Exception objects derive from. There are three ways to construct most Exception objects, one that takes no arguments, just the full name of the exception; another that takes one argument:

  * [System.String]$message
      * Describes the Exception to the user.

…and the third one that takes two arguments:

  * <span style="font-family: Arial;"><span style="font-size: small;">[System.String]$message</span></span> 
      * Describes the Exception to the user.
  * [System.Exception]$innerException
      * The Exception instance that caused the Exception association with the ErrorRecord.

Did you notice that the last argument is also a System.Exception? Some Exception objects have either customized constructors or no constructors at all; those Exception objects are the <em>exception</em>, if you know what I mean, so will stick with the regulars.

This code snippet shows how you would report a terminating error in an advanced function:

```powershell
$message = "File '$Path' is empty."
$exception = New-Object InvalidOperationException $message
$errorID = 'FileIsEmpty'
$errorCategory = [Management.Automation.ErrorCategory]::InvalidOperation
$target = $Path
$errorRecord = New-Object Management.Automation.ErrorRecord $exception, $errorID,
    $errorCategory, $target
$PSCmdlet.ThrowTerminatingError($errorRecord)
```


…first the Exception, next the ErrorRecord and finally report the ErrorRecord.

To make the custom ErrorRecord creation process a bit simpler, I wrote the New-ErrorRecord function…

```powershell
<# 
.Synopsis
    Creates an custom ErrorRecord that can be used to report a terminating or non-terminating error. 
    
.Description
    Creates an custom ErrorRecord that can be used to report a terminating or non-terminating error.
    
.Parameter Exception
    The Exception that will be associated with the ErrorRecord.
    
.Parameter ErrorID
    A scripter-defined identifier of the error.      This identifier must be a non-localized string for a specific error type.  
    
.Parameter ErrorCategory
    An ErrorCategory enumeration that defines the category of the error.  .Parameter TargetObject      The object that was being processed when the error took place.  
    
.Parameter Message
    Describes the Exception to the user.  
    
.Parameter InnerException
    The Exception instance that caused the Exception association with the ErrorRecord.  
    
.Example
    # advanced functions for testing
    function Test-1 {
        [CmdletBinding()]
        param(
            [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
            [String]
            $Path
        )
        process {
            foreach ($_path in $Path) {
                $content = Get-Content -LiteralPath $_path -ErrorAction SilentlyContinue
                if (-not $content) {
                    $errorRecord = New-ErrorRecord InvalidOperationException FileIsEmpty InvalidOperation $_path -Message "File '$_path' is empty."
                    $PSCmdlet.ThrowTerminatingError($errorRecord)
                }   
            }
        }
    }
    
    function Test-2 {
        [CmdletBinding()]
        param(
            [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
            [String]
            $Path
        )
        process {
            foreach ($_path in $Path) {
                $content = Get-Content -LiteralPath $_path -ErrorAction SilentlyContinue
                if (-not $content) {
                    $errorRecord = New-ErrorRecord InvalidOperationException FileIsEmptyAgain InvalidOperation $_path -Message "File '$_path' is empty again." -InnerException $Error[0].Exception     
                    $PSCmdlet.ThrowTerminatingError($errorRecord)
                }
            }
        }
    }
    
    # code to test the custom terminating error reports
    Clear-Host
    
    $null = New-Item -Path .\MyEmptyFile.bak -ItemType File -Force -Verbose
    
    Get-ChildItem *.bak | Where-Object {-not $_.PSIsContainer} | Test-1 Write-Host System.Management.Automation.ErrorRecord -ForegroundColor Green $Error[0] | Format-List * -Force Write-Host Exception -ForegroundColor Green $Error[0].Exception | Format-List * -Force
    
    Get-ChildItem *.bak | Where-Object {-not $_.PSIsContainer} | Test-2 Write-Host System.Management.Automation.ErrorRecord -ForegroundColor Green $Error[0] | Format-List * -Force Write-Host Exception -ForegroundColor Green $Error[0].Exception | Format-List * -Force
    
    Remove-Item .\MyEmptyFile.bak -Verbose
    
    Description
    ===========
    Both advanced functions throw a custom terminating error when an empty file is being processed.
    -Function Test-2's custom ErrorRecord includes an inner exception, which is the ErrorRecord reported by function Test-1.
        The test code demonstrates this by creating an empty file in the curent directory -which is deleted at the end- and passing its path to both test functions.
        The custom ErrorRecord is reported and execution stops for function Test-1, then the ErrorRecord and its Exception are displayed for quick analysis.
        Same process with function Test-2; after analyzing the information, compare both ErrorRecord objects and their corresponding Exception objects.
    -In the ErrorRecord note the different Exception, CategoryInfo and FullyQualifiedErrorId data.
    -In the Exception note the different Message and InnerException data.  

.Example
    $errorRecord = New-ErrorRecord System.InvalidOperationException FileIsEmpty InvalidOperation $Path -Message "File '$Path' is empty." $PSCmdlet.ThrowTerminatingError($errorRecord)
    
    Description
    ===========
    A custom terminating ErrorRecord is stored in variable 'errorRecord' and then it is reported through $PSCmdlet's ThrowTerminatingError method.
    The $PSCmdlet object is only available within advanced functions.  

.Example
    $errorRecord = New-ErrorRecord System.InvalidOperationException FileIsEmpty InvalidOperation $Path -Message "File '$Path' is empty." Write-Error -ErrorRecord $errorRecord
    Description
    ===========
    A custom non-terminating ErrorRecord is stored in variable 'errorRecord' and then it is reported through the Write-Error Cmdlet's ErrorRecord parameter.

.Inputs      System.String  

.Outputs      System.Management.Automation.ErrorRecord  

.Link      Write-Error      Get-AvailableExceptionsList
.Notes
    Name:      New-ErrorRecord
    Author:    Robert Robelo
    LastEdit:  08/24/2011 12:35  #>
 
function New-ErrorRecord {
    param(
        [Parameter(Mandatory = $true, Position = 0)]
        [System.String]
        $Exception,
        [Parameter(Mandatory = $true, Position = 1)]
        [Alias('ID')]
        [System.String]
        $ErrorId,
        [Parameter(Mandatory = $true, Position = 2)]
        [Alias('Category')]
        [System.Management.Automation.ErrorCategory]
        [ValidateSet('NotSpecified', 'OpenError', 'CloseError', 'DeviceError',
            'DeadlockDetected', 'InvalidArgument', 'InvalidData', 'InvalidOperation',
                'InvalidResult', 'InvalidType', 'MetadataError', 'NotImplemented',
                    'NotInstalled', 'ObjectNotFound', 'OperationStopped', 'OperationTimeout',
                        'SyntaxError', 'ParserError', 'PermissionDenied', 'ResourceBusy',
                            'ResourceExists', 'ResourceUnavailable', 'ReadError', 'WriteError',
                                'FromStdErr', 'SecurityError')]
        $ErrorCategory,
        [Parameter(Mandatory = $true, Position = 3)]
        [System.Object]
        $TargetObject,
        [Parameter()]
        [System.String]
        $Message,
        [Parameter()]
        [System.Exception]
        $InnerException
    )
    begin {
        # check for required function, if not defined...
        if (-not (Test-Path function:Get-AvailableExceptionsList)) {
            $message1 = "The required function Get-AvailableExceptionsList is not defined. " +
            "Please define it in the same scope as this function's and try again."
            $exception1 = New-Object System.OperationCanceledException $message1
            $errorID1 = 'RequiredFunctionNotDefined'
            $errorCategory1 = 'OperationStopped'
            $targetObject1 = 'Get-AvailableExceptionsList'
            $errorRecord1 = New-Object Management.Automation.ErrorRecord $exception1, $errorID1,
            $errorCategory1, $targetObject1
            # ...report a terminating error to the user
            $PSCmdlet.ThrowTerminatingError($errorRecord1)
        }
        # required function is defined, get "available" exceptions
        $exceptions = Get-AvailableExceptionsList
        $exceptionsList = $exceptions -join "`r`n"
    }
    process {
        # trap for any of the "exceptional" Exception objects that made through the filter
        trap [Microsoft.PowerShell.Commands.NewObjectCommand] {
            $PSCmdlet.ThrowTerminatingError($_)
        }
        # verify input exception is "available". if so...
        if ($exceptions -match "^(System\.)?$Exception$") {
            # ...build and save the new Exception depending on present arguments, if it...
            $_exception = if ($Message -and $InnerException) {
                # ...includes a custom message and an inner exception
                New-Object $Exception $Message, $InnerException
            } elseif ($Message) {
                # ...includes a custom message only
                New-Object $Exception $Message
            } else {
                # ...is just the exception full name
                New-Object $Exception
            }
            # now build and output the new ErrorRecord
            New-Object Management.Automation.ErrorRecord $_exception, $ErrorID,
            $ErrorCategory, $TargetObject
        } else {
            # Exception argument is not "available";
            # warn the user, provide a list of "available" exceptions and...
            Write-Warning "Available exceptions are:`r`n$exceptionsList" 
            $message2 = "Exception '$Exception' is not available."
            $exception2 = New-Object System.InvalidOperationExceptionn $message2
            $errorID2 = 'BadException'
            $errorCategory2 = 'InvalidOperation'
            $targetObject2 = 'Get-AvailableExceptionsList'
            $errorRecord2 = New-Object Management.Automation.ErrorRecord $exception2, $errorID2,
            $errorCategory2, $targetObject2
            # ...report a terminating error to the user
            $PSCmdlet.ThrowTerminatingError($errorRecord2)
        }
    }
}
```

…takes all the arguments used to build both an ErrorRecord and an Exception, six in all. The first four are required and the last two are optional. The function guards against nonexistent Exception objects but allows the absence of the &#8216;<em>System</em>.&#8217; prefix from its <em>Exception</em> argument, adhering to PowerShell&#8217;s tolerance. You can omit or include the &#8216;<em>System</em>.&#8217; prefix from the full name when creating an Exception object.

The function shields against nonexistent Exception objects by relying on a list of <em>available </em>Exception objects that is retrieved through another function, Get-AvailableExceptionsList. The <em>Exception</em> argument -which is of Type String, not Exception- is compared against the list to verify its <em>availability</em>. The other precaution in the New-ErrorRecord function is for those <em>exceptional </em>Exception objects that make the <em>available</em> list but have <em>customized</em> constructors, that is, different constructors from regular Exception objects. In either case, New-ErrorRecord reports a terminating error.

The Get-AvailableExceptionsList function…

```powershell
<#
.Synopsis
    Retrieves all available Exceptions to construct ErrorRecord objects.
    
.Description
    Retrieves all available Exceptions in the current session to construct ErrorRecord objects.
    
.Example
    $availableExceptions = Get-AvailableExceptionsList
    
    Description
    ===========      
        Stores all available Exception objects in the variable 'availableExceptions'.
        
.Example
    Get-AvailableExceptionsList | Set-Content $env:TEMP\AvailableExceptionsList.txt
    
    Description
    ===========      
        Writes all available Exception objects to the 'AvailableExceptionsList.txt' file in the user's Temp directory.
        
.Inputs
    None  
    
.Outputs
    System.String
    
.Link
    New-ErrorRecord  
    
.Notes 
    Name:      Get-AvailableExceptionsList 
    Author:    Robert Robelo
    
LastEdit:  08/24/2011 12:35
#>
function Get-AvailableExceptionsList {
    [CmdletBinding()]
    param()
    end {
        $irregulars = 'Dispose|OperationAborted|Unhandled|ThreadAbort|ThreadStart|TypeInitialization'
        [AppDomain]::CurrentDomain.GetAssemblies() | ForEach-Object {
            $_.GetExportedTypes() -match 'Exception' -notmatch $irregulars |
            Where-Object {
                $_.GetConstructors() -and $(
                $_exception = New-Object $_.FullName
                New-Object Management.Automation.ErrorRecord $_exception, ErrorID, OpenError, Target
                )
            } | Select-Object -ExpandProperty FullName
        } 2> $null
    }
}
```

…retrieves all Type objects -from the assemblies in the current domain- whose names contain the word <em>Exception<strong> </strong></em>but also excludes those whose names match part of <em>some</em> <em>of the</em> <em>exceptional</em> Exception objects&#8217; names. Then, if the Exception has at least one constructor, the function outputs its full name. The function can be called independently to provide a list to select the adequate Exception object name before you create a custom ErrorRecord. This function must be defined before calling the New-ErrorRecord function.

After defining both functions, the code snippet we previously used to report a terminating error in an advanced function becomes:

```powershell
$errorRecord = New-ErrorRecord System.InvalidOperationException FileIsEmpty `
    InvalidOperation $Path -Message "File '$Path' is empty."
$PSCmdlet.ThrowTerminatingError($errorRecord)
```


…a bit simpler.

Remember, ErrorRecord objects are not just for terminating errors. To report a customized non-terminating ErrorRecord, create it with the function New-ErrorRecord and pass it to Write-Error through its ErrorRecord parameter:

```powershell
$errorRecord = New-ErrorRecord System.InvalidOperationException FileIsEmpty `
    InvalidOperation $Path -Message "File '$Path' is empty."
Write-Error -ErrorRecord $errorRecord
```


This is the least you need to know about custom ErrorRecord objects, and how to report them as terminating or non-terminating errors. I know -through personal experience- this topic can be unapproachable, I threw away the ten-foot pole and explored it closer; we are good friends now, but there is still more to discover. At least I feel that this information can help you create custom ErrorRecord objects to better inform the user.

&nbsp;

[1]: http://technet.microsoft.com/en-us/library/dd315265.aspx
[2]: http://technet.microsoft.com/en-us/library/dd819510.aspx
[3]: http://technet.microsoft.com/en-us/library/system.management.automation.errorrecord%28VS.85%29.aspx
[4]: http://technet.microsoft.com/en-us/library/system.management.automation.pscmdlet_members(VS.85).aspx
[5]: http://technet.microsoft.com/en-us/library/system.management.automation.cmdlet.throwterminatingerror(VS.85).aspx
[6]: http://technet.microsoft.com/en-us/library/system.management.automation.errorrecord(VS.85).aspx
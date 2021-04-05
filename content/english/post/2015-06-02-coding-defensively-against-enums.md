---
title: Coding defensively against enums
author: Jakub Jareš
type: post
date: 2015-06-02T16:00:54+00:00
url: /2015/06/02/coding-defensively-against-enums/
views:
  - 7782
ratings_users:
  - 2
ratings_score:
  - 10
ratings_average:
  - 5
post_views_count:
  - 1698
categories:
  - How To
tags:
  - How To

---
Today I will show how to defensively script against enums and how to test invalid enum values.

### What is an enum?

An enum is a list of identifiers that map to values of a primitive type, usually System.Int32 which is the default in C#. An example of a simple enum is the [System.ConsoleColor] enum, listing the colors you can use in PowerShell console:

```powershell
[enum]::GetNames(System.ConsoleColor)

Black
DarkBlue
DarkGreen
DarkCyan
DarkRed
DarkMagenta
DarkYellow
Gray
DarkGray
Blue
Green
Cyan
Red
Magenta
Yellow
White
```

### Can it change?

Even though we usually see the enums as such, the values they contain are not definitive and might change in future version of the type. This is also marketed as one of the upsides of using enum over the boolean type, when you anticipate more than two values in the future, because you can easily add third state to true/false, on/off, up/downand so on.

This flexibility is great for the framework developer, who can ship his API with a newer version of his enum without forcing the consumers of the API to change their code.

But there is one caveat: The consumers have to be prepared for the enum to change. Let&#8217;s see an example.

### Example

Let&#8217;s imagine we wrote a function that converts values &#8220;On&#8221; and &#8220;Off&#8221; of a SystemStatus enum to &#8220;Online&#8221; and &#8220;Offline&#8221; respectively. The resulting value will become part of a report that also reports status of servers and other systems, hence the Online/Offline statuses.

The enum and our function are implemented and tested like this; I‘ll put both the production and test code in one file for brevity:

```powershell
Add-Type -TypeDefinition '
    namespace Nohwnd.Samples.TestingEnums
    {
        public enum SystemStatus
        {
            Off,
            On
        }
    }
'

function Get-UnifiedSystemStatus {
    param(
        [Parameter(Mandatory = $true)]
        [Nohwnd.Samples.TestingEnums.SystemStatus]
        $Status
    )
    if ($Status -eq 'Off')
    {
        'Offline'
    }
    else
    {
        'Online'
    }
}

Describe 'Get-UnifiedSystemStatus' {
    It 'Reports Offline when Off' {
        Get-UnifiedSystemStatus Off | Should Be 'Offline'
    }
    It 'Reports Online when On' {
        Get-UnifiedSystemStatus On | Should Be 'Online'
    }
}
```

The code first defines the SystemStatus with possible values &#8220;Off&#8221; and &#8220;On&#8221;. Then it defines a function that will translate the input value into values appropriate for our report.

The Get-UnifiedSystemStatus function requires its $Status parameter to be specified and to be of type [Nohwnd.Samples.TestingEnums.SystemStatus] which has only two possible values. For that reason we decided to use the if condition to translate the values.

We also have tests in place that both pass, so everything should be working correctly.

### Updating to a newer version

One day a new minor version of the API is released. You are not concerned with this update. After all it&#8217;s just a minor version bump and those should not include any breaking changes.

You update to the new version 1.2 and everything seems to work fine. You run you automated tests and they are still passing. Everything is good.

After few days the users of the system start noticing something strange: Sometimes they can&#8217;t connect to the printer remotely, even though it shows that it is online.

You suspect there was something wrong with the update so you investigate and with some luck you find out that the SystemStatus enum was updated as such:

```powershell
Add-Type -TypeDefinition '
    namespace Nohwnd.Samples.TestingEnums
    {
        public enum SystemStatus
        {
            Off,
            On,
            //Added new values in version 1.2 of the library
            Starting,
            Stopping,
            Busy
        }
    }
'
```


The intention was to let you see not only if the printer is on or off, but also if it&#8217;s starting, stopping or busy. This is great new functionality, but unfortunately it also means that any of the new statuses report Online in our report.

This also explains the user complains. When our hypothetical printers are starting (or stopping) they do not accept remote management requests, but the report shows them as Online. And online printers should not reject remote management requests.

### Preventing the issue

So let&#8217;s go back to our original code and see what is wrong with it, and how to write it better next time.

```powershell
function Get-UnifiedSystemStatus {
    param(
        [Parameter(Mandatory = $true)]
        [Nohwnd.Samples.TestingEnums.SystemStatus]
        $Status
    )
    if ($Status -eq 'Off')
    {
        'Offline'
    }
    else
    {
        'Online'
    }
}
```


The function builds on two assumptions:

  * The enum has only two possible values, so only two states are possible
  * The enum will not change in the future

Both of those assumptions are false and make our code less robust than it might be. Let&#8217;s use a little &#8220;trick&#8221; to break them both:

<pre class="brush: powershell; title: ; notranslate" title="">[Enum]::Parse([Nohwnd.Samples.TestingEnums.SystemStatus], 999)
</pre>

The static Parse method of the [System.Enum] type will happily produce a value of the given enum type even if that value is not defined in that enum type.

In simple words, you can get value 999 that will have type [Nohwnd.Samples.TestingEnums.SystemStatus], and successfully pass it to our function.

In our code this will make the if condition return &#8220;Online&#8221; for the value 999. This is something that our code did not expect, and hence it should fail rather than return incorrect value.

Let&#8217;s bake the expected values in our code and add appropriate test:

```powershell
function Get-UnifiedSystemStatus {
    param(
        [Parameter(Mandatory = $true)]
        [ValidateSet('On','Off')]
        #The version 0.1 of SystemStatus enum is anticipated here, if this throws an incorrect value was 
        #specified. Possible causes of this are that the enum was updated with more values, or parsed value 
        #of the enum was provided.
        [Nohwnd.Samples.TestingEnums.SystemStatus]
        $Status
    )
    if ($Status -eq 'Off')
    {
        'Offline'
    }
    else
    {
        'Online'
    }
}

Describe 'Get-UnifiedSystemStatus' {
    It 'Reports Offline when Off' {
        Get-UnifiedSystemStatus Off | Should Be 'Offline'
    }
    It 'Reports Online when On' {
        Get-UnifiedSystemStatus On | Should Be 'Online'
    }
    It 'Throws when invalid value is provided' {

        # -- Arrange

        $invalidValue = [int]::MaxValue$enum
        $enumType = [Nohwnd.Samples.TestingEnums.SystemStatus]        
        [Enum]::IsDefined($enumType, $invalidValue) | 
            Should Be $false # making sure the invalid value is not defined in this enum
        $invalidEnumValue = [Enum]::Parse($enumType, $invalidValue)

        # -- Act & Assert

        { Get-UnifiedSystemStatus $invalidEnumValue } | 
            Should Throw ('Cannot validate argument on parameter ''Status''. '+
                          'The argument "2147483647" does not belong to the set "On,Off" '+
                          'specified by the ValidateSet attribute. Supply an argument '+
                          'that is in the set and then try the command again.')
    }
}
```

This version adds the ValidateSet parameter constraint that checks for the values that were contained in the original version of the enum and throws an exception if a different value is provided. Also notice that a comment is used to explain why the constraint is there. Without it, it would likely look like somebody just forgot to add all the values to the constraint.

Another way to implement this would be to manually validate if the input value belongs to the set and throw [System.ArgumentOutOfRangeException] if it does not. This has the benefit of showing the possible causes of the exception directly to the developer.

```powershell
function Get-UnifiedSystemStatus {
    param(
        [Parameter(Mandatory = $true)]
        [Nohwnd.Samples.TestingEnums.SystemStatus]
        $Status
    )
    if ('Off','On' -notcontains $Status)
    {
        throw [ArgumentOutOfRangeException]('The provided value is not one of the expected values. ' +
        'Possible causes of this exception are parsing the enum value or using a newer version of ' +
        'the enum which contains more values than expected.')
    }
    if ($Status -eq 'Off')
    {
        'Offline'
    }
    else
    {
        'Online'
    }
}


Describe 'Get-UnifiedSystemStatus' {
    It 'Reports Offline when Off' {
        Get-UnifiedSystemStatus Off | Should Be 'Offline'
    }
    It 'Reports Online when On' {
        Get-UnifiedSystemStatus On | Should Be 'Online'
    }
    It 'Throws when invalid value is provided' {

        # -- Arrange

        $invalidValue = [int]::MaxValue
        $enumType = [Nohwnd.Samples.TestingEnums.SystemStatus]
        [Enum]::IsDefined($enumType, $invalidValue ) | 
            Should Be $false # making sure the invalid value is not defined in this enum
        $invalidEnumValue = [Enum]::Parse($enumType, $invalidValue)

        # -- Act & Assert

        { Get-UnifiedSystemStatus $invalidEnumValue } | 
            Should Throw ('The provided value is not one of the expected values. ' +
                          'Possible causes of this exception are parsing the enum value ' +
                          'or using a newer version of the enum which contains more values than expected.')
    }
}
```

In both cases an assertion that would simply check the type of the thrown exception would be very useful:

```powershell
Should Throw [SomeExceptionType]
```


But unfortunately no such assertion is present in the Pester framework at the moment.

### Reviewing the changes

Let&#8217;s look at the updated code again, and imagine what would happen if we updated the enum now. To our surprise we find out that the tests would still pass. But running the report would fail, if there was at least one device in the new status that is. Failing the application on run time might be good enough, and with a bit of luck the report would fail right after the update.

Could we make it better? We can list all the values in the enum and make the test run for all of them.

```powershell
Add-Type -TypeDefinition '
    namespace Nohwnd.Samples.TestingEnums
    {
        public enum SystemStatus
        {
            Off,
            On,
            //Added new values in version 0.2 of the library
            Starting,
            Stopping,
            Busy
        }
    }
'

function Get-UnifiedSystemStatus {
    param(
        [Parameter(Mandatory = $true)]
        [ValidateSet('On','Off')]
        #The version 0.1 of SystemStatus enum is anticipated here, if this throws an incorrect value was 
        #specified. Possible causes of this are that the enum was updated with more values, or parsed value 
        #of the enum was provided.
        [Nohwnd.Samples.TestingEnums.SystemStatus]
        $Status
    )

    if ($Status -eq 'Off')
    {
        'Offline'
    }
    else
    {
        'Online'
    }
}

Describe 'Get-UnifiedSystemStatus' {
    It 'Reports Offline when Off' {
        Get-UnifiedSystemStatus Off | Should Be 'Offline'
    }
    It 'Reports Online when On' {
        Get-UnifiedSystemStatus On | Should Be 'Online'
    }
    It 'Throws when invalid value is provided' {

        # -- Arrange

        $invalidValue = [int]::MaxValue
        $enumType = [Nohwnd.Samples.TestingEnums.SystemStatus]        
        [Enum]::IsDefined($enumType, $invalidValue) | 
            Should Be $false # making sure the invalid value is not defined in this enum
        $invalidEnumValue = [Enum]::Parse($enumType, $invalidValue)

        # -- Act & Assert

        { Get-UnifiedSystemStatus $invalidEnumValue } | 
            Should Throw ('Cannot validate argument on parameter ''Status''. '+
                          'The argument "2147483647" does not belong to the set "On,Off" '+
                          'specified by the ValidateSet attribute. Supply an argument '+
                          'that is in the set and then try the command again.')
    }

    It 'Can handle all defined enum values' {
        foreach ($name in [enum]::GetNames([Nohwnd.Samples.TestingEnums.SystemStatus]))
        {
            { Get-UnifiedSystemStatus $name } | Should Not Throw
        }
    }
}
```

The new test gets all the values from the enum and runs them from our function. If any of the values does not belong to the set of &#8216;On&#8217;,&#8217;Off&#8217; it will fail the test.

### Summary

Enums do not guarantee that their values will not change. This is something to be aware of.

To future proof your code you should provide incorrect enum values to your code and test it&#8217;s reaction. This is especially important for code which uses a switch statement to take an action based on value of an enum. Such switch statement often defines a default case which often remains untested until a new version of the enum is provided.
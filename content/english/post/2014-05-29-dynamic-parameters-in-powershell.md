---
title: Dynamic Parameters in PowerShell
author: Ben Ten
type: post
date: 2014-05-29T18:00:00+00:00
url: /2014/05/29/dynamic-parameters-in-powershell/
categories:
  - How To
tags:
  - How To
---
Has there ever been a time when you wanted to call a function or cmdlet with specific parameters that were based on conditional criteria that is provided? As an example: if a user requests your cmdlet, with a specific value supplied for a parameter, it should then require a second parameter. This goes beyond utilizing ParameterSets. These are conditional parameters, or more properly referred to as dynamic parameters.

To help illustrate the concept of dynamic parameters, I will be using the example of an electronic lemonade stand. We will call it ELS for short. The primary function of ELS is to sell lemonade, but it also sells water, tea, and coffee. The ELS will assume you want lemonade if you don&#8217;t specify the product. The only thing it requests by default is the quantity. I&#8217;ve created the base script for our ELS system without any dynamic parameters included.


    Function Get-Order {
        [CmdletBinding()]
            Param(
              [Parameter(
                 Mandatory=$true,
                 Position=1,
                 HelpMessage="How many cups would you like to purchase?"
              )]
              [int]$cups,
              
              [Parameter(
                 Mandatory=$false,
                 Position=2,
                 HelpMessage="What would you like to purchase?"
              )]
              [ValidateSet("Lemonade","Water","Tea","Coffee")]
              [string]$product="Lemonade"
    		)
    
           Process {
              $order = @()
              for ($cup = 1; $cup -le $cups; $cup++) {
                  $order += "$($cup): A cup of $($product)"
              }
              $order
         }
    }
The above script works in this way. The customer will specify how many _$cups_ they would like. The customer can also specify the _$product_ if they don&#8217;t want the default product of Lemonade. Then it will return an order with that many _$cups_ of the _$product_.

Here is the output from our new ELS system:

```
PS C:\psf\scripts> Get-Order 3
1: A cup of Lemonade
2: A cup of Lemonade
3: A cup of Lemonade

PS C:\psf\scripts&gt; Get-Order 3 "Water"
1: A cup of Water
2: A cup of Water
3: A cup of Water

PS C:\psf\scripts&gt; Get-Order 3 "Tea"
1: A cup of Tea
2: A cup of Tea
3: A cup of Tea

PS C:\psf\scripts>
```

With the base script ready to go we have a new requirement for our ELS system. The system needs to be able to support a new alcoholic product, **Hard Lemonade**. If the customer asks for **Hard Lemonade** as the product, then the ELS system must verify that the customer is 21 years old or older. This leads us into utilizing dynamic parameters.

### Dynamic Parameters Defined

A dynamic parameter is defined in the about\_Functions\_Advanced_Parameters documentation. This can be found at <http://technet.microsoft.com/en-us/library/hh847743.aspx> or by typing: Get-Help about\_Functions\_Advanced_Parameters. The documentation defines dynamic parameters as “parameters of a cmdlet, function, or script that are available only under certain conditions.” It also specifies that dynamic parameters can also be created so that they appear “only when another parameter is used in the function command or when another parameter has a certain value.” This is exactly what we need to enhance the ELS system to offer **Hard Lemonade**.

### Dynamic Parameters in Get-Help or Get-Command

It is important to note that the documentation does point out one word of caution with the use of dynamic parameters. It states to only use them “when necessary” because “they can be difficult for users to discover.” If the conditions are not met for your dynamic parameter to be used it will not show up in the Get-Help unless the user uses the **_&#8211;_**_Path_ parameter of **Get-Help** or the **_&#8211;_**_ArgumentList_ parameter of **Get-Command**.

### Creating a Dynamic Parameter

To create a dynamic parameter, you will need to use the DynamicParam keyword. Unlike the Param keyword, you enclose the statement list in curly brackets {}. Dynamic parameters are declared after the Param() definition if used in a cmdlet.

### DynamicParam

The syntax to create a dynamic parameter is:

```
DynamicParam {<statement-list>}
```


You specify the condition logic for the parameter in the <statement-list> section. Using our ELS system we can start with checking the value of $product for our dynamic parameter.

```
DynamicParam {
    if ($product -eq "Hard Lemonade") {
        #create $age parameter here.
    }
}
```

### Steps Overview

Before you can use a dynamic parameter you have to do a few things beforehand. Here is a list of the steps that need to be completed before you can use your dynamic parameter.

  1.  Define the parameter attributes with the ParameterAttribute object.
  2. Create an Attribute collection object.
  3. Add your ParameterAttribute to the Attribute collection.
  4. Create the RuntimeDefinedParameter specifying: 
      1. The name of the parameter.
      2. The type of the parameter.
      3. The Attribute collection object you created in step 2.
  5. Create a RuntimeDefinedParameterDictionary object.
  6. Add the RuntimeDefinedParameter to the RuntimeDefinedParameterDictionary.
  7. Return the RuntimeDefinedParameterDictionary object.

### Creating the ParameterAttribute

If we want to specify the attributes of a Parameter, we have to specify it before we create the RuntimeDefinedParameter. This is done with the System.Management.Automation.ParameterAttribute object. The ParameterAttribute object is used to represent the attributes of the parameter. You can set the following properties with the ParameterAttribute object:

```
HelpMessage                     Property   string HelpMessage {get;set;}
HelpMessageBaseName             Property   string HelpMessageBaseName {get;set;}
HelpMessageResourceId           Property   string HelpMessageResourceId {get;set;}
Mandatory                       Property   bool Mandatory {get;set;}
ParameterSetName                Property   string ParameterSetName {get;set;}
Position                        Property   int Position {get;set;}
ValueFromPipeline               Property   bool ValueFromPipeline {get;set;}
ValueFromPipelineByPropertyName Property   bool ValueFromPipelineByPropertyName {get;set;}
ValueFromRemainingArguments     Property   bool ValueFromRemainingArguments {get;set;}
```


Going back to our ELS system, we need to create our parameter attribute for the $age parameter. We want to ensure that Mandatory is set to $true, HelpMessage is set to “Please enter your age:”, and Position is set to 3.

```
$ageAttribute = New-Object System.Management.Automation.ParameterAttribute
$ageAttribute.HelpMessage = "Please enter your age:"
$ageAttribute.Mandatory = $true
$ageAttribute.Position = 3
```


Our dynamic parameter code looks like this now:

```
DynamicParam {
    if ($product -eq "Hard Lemonade") {
        $ageAttribute = New-Object System.Management.Automation.ParameterAttribute
        $ageAttribute.Position = 3
        $ageAttribute.Mandatory = $true
        $ageAttribute.HelpMessage = "Please enter your age:"
    }
}
```

### Adding the Attribute to a Collection

Now the RuntimeDefinedParameter constructor only accepts a Collection of type Attribute. To pass our $ageAttribute to the RuntimeDefinedParameter we need to first add it to a new collection object. We use the System.Collections.ObjectModel.Collection[Type] object for this. We use the .Add() method to add our $ageAttribute to the collection.

```
$attributeCollection = New-Object System.Collections.ObjectModel.Collection[System.Attribute]
$attributeCollection.Add($ageAttribute)
```


Our dynamic parameter code looks like this now:

```
DynamicParam {
    if ($product -eq "Hard Lemonade") {
        $ageAttribute = New-Object System.Management.Automation.ParameterAttribute
        $ageAttribute.Position = 3
        $ageAttribute.Mandatory = $true
        $ageAttribute.HelpMessage = "Please enter your age:"
        $attributeCollection = New-Object System.Collections.ObjectModel.Collection[System.Attribute]
        $attributeCollection.Add($ageAttribute)
    }
}
```

### Creating the RuntimeDefinedParameter

Now that we have our attributes defined for our parameter and we have it in a collection, we can create the RuntimeDefinedParameter object.

The System.Management.Automation.RuntimeDefinedParameter object is used to add a parameter to the parameter list at runtime. The RuntimeDefinedParameter constructor is used to define the parameter name, type, and the attributes of the parameter.

The syntax for System.Management.Automation.RuntimeDefinedParameter has two constructors:

<pre class="brush: powershell; title: ; notranslate" title="">New-Object System.Management.Automation.RuntimeDefinedParameter()
New-Object System.Management.Automation.RuntimeDefinedParameter([String]Name, [Type]ParameterType, [Collection]Attributes)
</pre>

We want to define the name, type, and the attributes so we will use the second constructor for this object. In the command below I specify that I want to use &#8216;age&#8217; as the parameter name, it&#8217;s of type Int16, and to use the attributes in the $attributeCollection.

<pre class="brush: powershell; title: ; notranslate" title="">$ageParam = New-Object System.Management.Automation.RuntimeDefinedParameter('age', [Int16], $attributeCollection)
</pre>

Our dynamic parameter code looks like this now:

```
DynamicParam {
    if ($product -eq "Hard Lemonade") {
        $ageAttribute = New-Object System.Management.Automation.ParameterAttribute
        $ageAttribute.Position = 3
        $ageAttribute.Mandatory = $true
        $ageAttribute.HelpMessage = "Please enter your age:"
        $attributeCollection = New-Object System.Collections.ObjectModel.Collection[System.Attribute]
        $attributeCollection.Add($ageAttribute)
        $ageParam = New-Object System.Management.Automation.RuntimeDefinedParameter('age', [Int16], $attributeCollection)
   }
}
```

### Creating the RuntimeDefinedParameterDictionary

Before we can reference our new parameter, we have to expose it to the runspace. We use the RuntimeDefinedParameterDictionary object for this. After we create the RuntimeDefinedParameter object we add our $ageParam RuntimeDefinedParameter. Finally, we return that dictionary.

<pre class="brush: powershell; title: ; notranslate" title="">$paramDictionary = new-objectSystem.Management.Automation.RuntimeDefinedParameterDictionary
$paramDictionary.Add('age', $ageParam)
return $paramDictionary
</pre>

Our dynamic parameter code looks like this now:

```
DynamicParam {
     if ($product -eq "Hard Lemonade") {
         $ageAttribute = New-Object System.Management.Automation.ParameterAttribute
         $ageAttribute.Position = 3
         $ageAttribute.Mandatory = $true
         $ageAttribute.HelpMessage = "Please enter your age:"
         $attributeCollection = New-Object System.Collections.ObjectModel.Collection[System.Attribute]
         $attributeCollection.Add($ageAttribute)
         $ageParam = New-Object System.Management.Automation.RuntimeDefinedParameter('age', [Int16], $attributeCollection)
         $paramDictionary = New-Object System.Management.Automation.RuntimeDefinedParameterDictionary
         $paramDictionary.Add('age', $ageParam)
         return $paramDictionary
    }
}
```



### Testing our Dynamic Parameter

Now we have a dynamic parameter of &#8216;age&#8217; that will only show up if the $product is set to **Hard Lemonade**. Now we can get this information for our new product under these conditions. Here is our ELS code right now.


    function Get-Order {
        [CmdletBinding()]
        Param(
            [Parameter(
                Mandatory=$true,
                Position=1,
                HelpMessage="How many cups would you like to purchase?"
            )]
            [int]$cups,
            
            [Parameter(
                Mandatory=$false,
                Position=2,
                HelpMessage="What would you like to purchase?"
            )]
            [ValidateSet("Lemonade","Water","Tea","Coffee","Hard Lemonade")]
            [string]$product="Lemonade"
    	)
    
        DynamicParam {
             if ($product -eq "Hard Lemonade") {
                  #create a new ParameterAttribute Object
                  $ageAttribute = New-Object System.Management.Automation.ParameterAttribute
                  $ageAttribute.Position = 3
                  $ageAttribute.Mandatory = $true
                  $ageAttribute.HelpMessage = "This product is only available for customers 21 years of age and older. Please enter your age:"
    
                  #create an attributecollection object for the attribute we just created.
                  $attributeCollection = new-object System.Collections.ObjectModel.Collection[System.Attribute]
    
                  #add our custom attribute
                  $attributeCollection.Add($ageAttribute)
    
                  #add our paramater specifying the attribute collection
                  $ageParam = New-Object System.Management.Automation.RuntimeDefinedParameter('age', [Int16], $attributeCollection)
    
                  #expose the name of our parameter
                  $paramDictionary = New-Object System.Management.Automation.RuntimeDefinedParameterDictionary
                  $paramDictionary.Add('age', $ageParam)
                  return $paramDictionary
            }
        }
    
        Process {
            $order = @()
            for ($cup = 1; $cup -le $cups; $cup++) {
                $order += "$($cup): A cup of $($product)"
            }
            $order
        }
    }
Testing our ELS script we get the following output:

```
PS C:\psf\scripts> Import-Module .\get-order.ps1 -Force
PS C:\psf\scripts> Get-Order 3 "Hard Lemonade"
cmdlet Get-Order at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
age: 21
1: A cup of Hard Lemonade
2: A cup of Hard Lemonade
3: A cup of Hard Lemonade

PS C:\psf\scripts> Get-Order 3
1: A cup of Lemonade
2: A cup of Lemonade
3: A cup of Lemonade

PS C:\psf\scripts> Get-Order 3 "Water"
1: A cup of Water
2: A cup of Water
3: A cup of Water

PS C:\psf\scripts> Get-Order 3 "Tea"
1: A cup of Tea
2: A cup of Tea
3: A cup of Tea
PS C:\psf\scripts>
```


Excellent! If we specify “Hard Lemonade” as the product parameter, we get prompted for the age parameter. It doesn&#8217;t ask our age for the default of “Lemonade” or for “Water”, or “Tea”. However, we are not done yet. We still need to verify that their age is 21 or older! We haven&#8217;t done anything with the new age parameter.

### Adding Criteria for the Age ParameterAttribute

To verify the age is set to 21 or older we can access our dynamic parameter through the $PSBoundParameters variable. For our dynamic variable we would reference $PSBoundParameters.age. We want to ensure that age is set to 21 or higher so we need to do this logic test in our Begin {} block. If the age is set and less than 21 it should write an error and stop the script; otherwise, it should continue as normal.

<pre class="brush: powershell; title: ; notranslate" title="">Begin {
    if ($PSBoundParameters.age -and $PSBoundParameters.age -lt 21) {
        Write-Error "You are not old enough for Hard Lemonade. How about a nice glass of regular Lemonade instead?" -ErrorAction Stop
    }
}
</pre>
### Final Code and Testing

Once we add our age check in the Begin block of our code we now have our finished Electronic Lemonade Stand script!


    Function Get-Order {
        [CmdletBinding()]
        param(
            [Parameter(
                Mandatory=$true,
                Position=1,
                HelpMessage="How many cups would you like to purchase?"
            )]
            [int]$cups,
            [Parameter(
                Mandatory=$false,
                Position=2,
                HelpMessage="What would you like to purchase?"
            )]
            [ValidateSet("Lemonade","Water","Tea","Coffee","Hard Lemonade")]
            [string]$product="Lemonade"
        )
    
        DynamicParam {
            if ($product -eq "Hard Lemonade") {
                #create a new ParameterAttribute Object
                $ageAttribute = New-Object System.Management.Automation.ParameterAttribute
                $ageAttribute.Position = 3
                $ageAttribute.Mandatory = $true
                $ageAttribute.HelpMessage = "This product is only available for customers 21 years of age and older. Please enter your age:"
    
                #create an attributecollection object for the attribute we just created.
                $attributeCollection = new-object System.Collections.ObjectModel.Collection[System.Attribute]
    
                #add our custom attribute
                $attributeCollection.Add($ageAttribute)
    
                #add our paramater specifying the attribute collection
                $ageParam = New-Object System.Management.Automation.RuntimeDefinedParameter('age', [Int16], $attributeCollection)
    
                #expose the name of our parameter
                $paramDictionary = New-Object System.Management.Automation.RuntimeDefinedParameterDictionary
                $paramDictionary.Add('age', $ageParam)
                return $paramDictionary
           }
      } 
    
       Begin {
           if ($PSBoundParameters.age -and $PSBoundParameters.age -lt 21) {
               Write-Error "You are not old enough for Hard Lemonade. How about a nice glass of regular Lemonade instead?" -ErrorAction Stop
           }
       }
    
       Process {
           $order = @()
           for ($cup = 1; $cup -le $cups; $cup++) {
               $order += "$($cup): A cup of $($product)"
           }
           $order
       }
    }
Here is the output from our ESL system.

**Test to ensure previous items still work as originally scripted.**

```
PS C:\psf\scripts> Get-Order 3
1: A cup of Lemonade
2: A cup of Lemonade
3: A cup of Lemonade

PS C:\psf\scripts> Get-Order 3 Water
1: A cup of Water
2: A cup of Water
3: A cup of Water

PS C:\psf\scripts> Get-Order 3 Tea
1: A cup of Tea
2: A cup of Tea
3: A cup of Tea

PS C:\psf\scripts> Get-Order 3 Coffee
1: A cup of Coffee
2: A cup of Coffee
3: A cup of Coffee
```

### Now to test the “Hard Lemonade” product prompt and age logic for a value less than 21.

```
PS C:\psf\scripts> Get-Order 3 "Hard Lemonade"
cmdlet Get-Order at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
age: 18
Get-Order : You are not old enough for Hard Lemonade. How about a nice glass of regular Lemonade instead?
At line:1 char:1
+ Get-Order 3 "Hard Lemonade"
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,Get-Order
```

### Now to test the “Hard Lemonade” product prompt and age logic for a value 21 or greater.

```
PS C:\psf\scripts> Get-Order 3 "Hard Lemonade"
cmdlet Get-Order at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
age: 21
1: A cup of Hard Lemonade
2: A cup of Hard Lemonade
3: A cup of Hard Lemonade
```

### Testing our age dynamic parameter as a 3rd positional parameter.

```
PS C:\psf\scripts> Get-Order 3 "Hard Lemonade" 34
1: A cup of Hard Lemonade
2: A cup of Hard Lemonade
3: A cup of Hard Lemonade
PS C:\psf\scripts>
```

### Conclusion

Success! We have created a dynamic parameter that only appears when our product is “Hard Lemonade” and will only return that product if the age is 21 or older. Our previous functionality is still there and our enhancement only shows up under a specific condition.

Dynamic parameters can be very useful and powerful. They may not be the best approach for every situation, but if they are used properly they can save on development time and error checking.

I encourage you to mess around with the ESL script. Try to add more dynamic parameters based on the product condition. Hopefully this article has increased your understanding of dynamic parameters and how to implement them properly.
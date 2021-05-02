---
title: 'Dynamic Parameters in C# Cmdlets'
author: Ben Ten
type: post
date: 2014-06-23T16:00:39+00:00
url: /2014/06/23/dynamic-parameters-in-c-cmdlets/
categories:
  - PSCmdlet
  - How To
tags:
  - PSCmdlet
  - How To

---
In my [last article][1] I wrote, I covered how to create dynamic parameters in a PowerShell cmdlet. If you haven&#8217;t read that article, please read that one before continuing with this one as I reference it several times in this article. I also encourage you to read [Carlos Perez&#8217;s articles on how to create cmdlets with C#][2].

In this article, I will be using the example of an electronic lemonade stand (ELS) again. As a reminder, the primary function of ELS was to sell lemonade, but it could also sell water, tea, and coffee. The ELS assumes you want lemonade if you don&#8217;t specify the product. The only thing it requests by default is the quantity.

### C# Base Code

The base code for the C# cmdlet is similar to the base script for our PowerShell cmdlet. I will point out some of the differences that you should know. Following Carlos&#8217; article, I created a Class Library project in Visual Studio and named the project _GetOrder_. I renamed the _Class1.cs_ to _GetOrder_ and changed the namespace to _GetOrder_. Here is our base code for our _Get-Order_ cmdlet. You can download the starter code for this cmdlet by visiting <https://github.com/Ben0xA/Get-Order>

    using System;
    using System.Collections.ObjectModel;
    using System.Management.Automation;
    
    namespace GetOrder
    {
        [Cmdlet(VerbsCommon.Get, "Order")]
        public class GetOrder : PSCmdlet
        {
            [Parameter(
                Mandatory = true,
                Position = 1,
                HelpMessage = "How many cups would you like to purchase?"
            )]
            public int Cups;
            [Parameter(
                Mandatory = false, 
                Position = 2, 
                HelpMessage = "What would you like to purchase?"
            )]
            [ValidateSet("Lemonade","Water","Tea","Coffee")]
            public string Product = "Lemonade";
    
            protected override void ProcessRecord()
            {
                Collection&lt;String&gt; order = new Collection&lt;string&gt;();
                for (int cup = 1; cup &lt;= Cups; cup++) {
                    order.Add(cup.ToString() + ": A cup of " + Product);
                }
                WriteObject(order, true);
            }
    	}
    }
The above code works the same way as our previous PowerShell cmdlet. The customer will specify how many _Cups_ they would like. The customer can also specify the _Product_ if they don&#8217;t want the default product of Lemonade. Then it will return an order with that many _Cups_ of the _Product_.

The best thing about using C# or PowerShell is that the code is &#8216;almost&#8217; good enough to copy and paste from one to the other. You do have to make changes to the declarations and variable names but otherwise it follows the same layout and logic flow. If you compare the above code to our previous base script, you will see that they just about mirror each other.

> You must compile the Class Library before you can use it. You do this by pressing F7 or use the Build Solution from the Build menu.
>

Here is the output from our new C# ELS system which matches our previous PowerShell ELS system:

```
PS C:\psf\scripts> Import-Module C:\Development\Get-Order\Get-Order\bin\Release\Get-Order.dll
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


Now we can begin to add support for the alcoholic product, **Hard Lemonade**. Again, as a reminder, if the customer asks for **Hard Lemonade** as the product, then the ELS system must verify that the customer is 21 years old or older.

### Dynamic Parameters Definition and Resources

A dynamic parameter is defined in the about\_Functions\_Advanced_Parameters documentation. This can be found at <http://technet.microsoft.com/en-us/library/hh847743.aspx> or by typing: _Get-Help about\_Functions\_Advanced_Parameters_.

For C# we can utilize two other resources when implementing dynamic parameters into our C# cmdlets. The first resource available to us is &#8216;Cmdlet Dynamic Parameters&#8217; which can be found at [http://msdn.microsoft.com/en-us/library/dd878299][3]. This resource defines dynamic parameters as “parameters that are available to the user under special conditions, such as when the argument of another parameter is a specific value. These parameters are added at runtime and are referred to as dynamic parameters because they are added only when they are needed.”

The second resource available to us is &#8216;How to Declare Dynamic Parameters&#8217; which can be found at [http://msdn.microsoft.com/library/dd878334][4]. Our ELS system will use this resource as the guide for creating our dynamic parameter.

### Creating a Dynamic Parameter

Creating a dynamic parameter in C# is similar to our PowerShell script with some obvious differences. The biggest difference is that our dynamic parameters are in a separate class. You use the GetDynamicParameters function to return the parameter object from our dynamic parameter class. Here is a list of steps to create the dynamic parameter in C#.

### Steps Overview

Here is a list of the steps that need to be completed before you can use your dynamic parameter.

  1. Add the IdynamicParameters interface to the cmdlet class declaration.
  2. Define the GetDynamicParameters function.
  3. Create the age class that returns the dynamic parameter.
  4. Define the age parameter.
  5. In the _GetDynamicParameters_ function, create a new instance of the dynamic parameter class.
  6. Return the parameter from the dynamic parameter class.

### Add the IdynamicParameters Interface

We need to add the IdynamicParameters to our _Get-Order_ class declaration. This is done by adding _IdynamicParameters_ to the end of the line that reads:

<pre class="brush: csharp; title: ; notranslate" title="">public class GetOrder : PSCmdlet
</pre>

So our declaration line now looks like this:

<pre class="brush: csharp; title: ; notranslate" title="">public class GetOrder : PSCmdlet, IdynamicParameters
</pre>
### Define the GetDynamicParameters Function

The _GetDynamicParameters_ function in C# is very similar to the _DynamicParam_ block in our previous PowerShell script. This function determines if a dynamic parameter should be used based on our logic test and then creates a new instance of the dynamic parameter class and returns that object. Using our ELS system we can start with checking the value of Product for our dynamic parameter.

<pre class="brush: csharp; title: ; notranslate" title="">public object GetDynamicParameters()
{
    if (Product == "Hard Lemonade")
    {
        //create age parameter here    
    }
}
</pre>

You will notice in Visual Studio that will tell you that the _GetDynamicParameters_ function has an error of _&#8216;GetOrder.GetOrder.GetDynamicParameters()_&#8216;: not all code paths return a value. That&#8217;s okay for now. Our function needs to return a value. We haven&#8217;t returned anything yet. We will come back to this function in a minute.

### Create the Age Dynamic Parameter Class

Going back to our ELS system, we need to create our parameter attribute for the age parameter. We want to ensure that Mandatory is set to true, _HelpMessage_ is set to _“Please enter your age:”_, and _Position_ is set to 3. The nice thing about using C# for creating the dynamic parameter is that you don&#8217;t need to create the _RuntimeDefinedParameter_ or the Dictionary. You create the parameter the same way you did for the previous two parameters.

You can either create a new .cs class file for your _AgeDynamicParameter_ or you can put this code after the closing bracket } of your _GetOrder_ class. For this example, we will put our code after the closing bracket and not use a separate .cs file.


	public class AgeDynamicParameter
	 {
		  private int _age;
	      [Parameter(
	            Mandatory = true,
	            Position = 3,
	            HelpMessage = "Please enter your age:"
	      )]
	      public int Age
	      {
	            get { return _age; }
	            set { _age = value; }
	      }
	 }
After we add “Hard Lemonade” to our ValidateSet for Product, our code looks like this now:

    using System;
    using System.Collections.ObjectModel;
    using System.Management.Automation;
    
    namespace GetOrder
    {
        [Cmdlet(VerbsCommon.Get, "Order")]
        public class GetOrder : PSCmdlet, IDynamicParameters
        {
            [Parameter(
                Mandatory = true,
                Position = 1,
                HelpMessage = "How many cups would you like to purchase?"
            )]
            public int Cups;
            [Parameter(
                Mandatory = false, 
                Position = 2, 
                HelpMessage = "What would you like to purchase?"
            )]
            [ValidateSet("Lemonade","Water","Tea","Coffee","Hard Lemonade")]
            public string Product = "Lemonade";
    
            public object GetDynamicParameters()
            {
                if (Product == "Hard Lemonade")
                {
                    //create age parameter here    
                }
            }
    
            protected override void ProcessRecord()
            {
                Collection&lt;String&gt; order = new Collection&lt;string&gt;();
                for (int cup = 1; cup &lt;= Cups; cup++) {
                    order.Add(cup.ToString() + ": A cup of " + Product);
                }
                WriteObject(order, true);
            }
        }
    
        public class AgeDynamicParameter
        {
            private int _age;
    
            [Parameter(
                Mandatory = true,
                Position = 3,
                HelpMessage = "Please enter your age:"
            )]
            public int Age
            {
                get { return _age; }
                set { _age = value; }
            }        
        }
    }
### Creating a New Instance of AgeDynamicParameter

Now that we have our _AgeDynamicParameter_ class and _Age_ parameter defined, we need to create a new instance of that class in our _GetDynamicParameters_ function. Before we can create that instance, we need to create a privately scoped variable for our _AgeDynamicParameter_ class. We will call this variable agedynparm. We can put this variable at the top of our _GetOrder_ class just after its declaration.


	public class GetOrder : PSCmdlet, IDynamicParameters
	{
		private AgeDynamicParameter agedynparm = null;
	    [Parameter(
	         Mandatory = true,
	         Position = 1,
	         HelpMessage = "How many cups would you like to purchase?"
	    )]
	    public int Cups;
Going back to our _GetDynamicParameters_ we can now create a new instance of the _AgeDynamicParameter_ class with our _agedynparm_ variable and return that object. Unlike our PowerShell script where we only have to “return” an object if our logic criteria is met, we have to return on all code paths in C#. This is why you see the else {} returning null. This is what the _GetDynamicParameters_ function looks like now:

<pre class="brush: csharp; title: ; notranslate" title="">public object GetDynamicParameters()
{
	if (Product == "Hard Lemonade")
	{
		 agedynparm = new AgeDynamicParameter();
		 return agedynparm;
	}
	else
	{
		 return null;
	}
}
</pre>
### Testing our Dynamic Parameter

Now we have a dynamic parameter of Age that will only show up if the Product is set to **Hard Lemonade**. Now we can get this information for our new product under these conditions. Here is our ELS code right now.

    using System;
    using System.Collections.ObjectModel;
    using System.Management.Automation;
    
    namespace GetOrder
    {
        [Cmdlet(VerbsCommon.Get, "Order")]
        public class GetOrder : PSCmdlet, IDynamicParameters
        {
            private AgeDynamicParameter agedynparm = null;
            
            [Parameter(
                Mandatory = true,
                Position = 1,
                HelpMessage = "How many cups would you like to purchase?"
            )]
            public int Cups;
    
            [Parameter(
                Mandatory = false, 
                Position = 2, 
                HelpMessage = "What would you like to purchase?"
            )]
            [ValidateSet("Lemonade","Water","Tea","Coffee","Hard Lemonade")]
            public string Product = "Lemonade";
    
            public object GetDynamicParameters()
            {
                if (Product == "Hard Lemonade")
                {
                    agedynparm = new AgeDynamicParameter();
                    return agedynparm;
                }
                else
                {
                    return null;
                }
            }
    
            protected override void ProcessRecord()
            {
                Collection&lt;String&gt; order = new Collection&lt;string&gt;();
                for (int cup = 1; cup &lt;= Cups; cup++) {
                    order.Add(cup.ToString() + ": A cup of " + Product);
                }
                WriteObject(order, true);
            }
        }
    
        public class AgeDynamicParameter
        {
            private int _age;
    
            [Parameter(
                Mandatory = true,
                Position = 3,
                HelpMessage = "Please enter your age:"
            )]
            public int Age
            {
                get { return _age; }
                set { _age = value; }
            }        
        }
    }
Don't forget to build the Class Library before testing. Testing our ELS script we get the following output:

```
PS C:\psf\scripts> Import-Module C:\Development\Get-Order\Get-Order\bin\Release\Get-Order.dll
PS C:\psf\scripts> Get-Order 3 "Hard Lemonade"
cmdlet Get-Order at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
Age: 21
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

Excellent! If we specify “Hard Lemonade” as the product parameter, we get prompted for the age parameter. It doesn&#8217;t ask our age for the default of “Lemonade” or for “Water”, or “Tea”. However, just like our PowerShell script, we are not done yet. We still need to verify that their age is 21 or older! We haven&#8217;t done anything with the new age parameter.

Also, note that you can&#8217;t _Import-Module -Force_ with the dll file. You need to close PowerShell completely to unload the DLL from memory before you import it again.

### Adding Criteria for the Age Parameter

We want to ensure that age is set to 21 or higher so we need to do this logic test in the _BeginProcessing()_ method. We don&#8217;t have the _BeginProcessing()_ method in our code so we need to add it above our _ProcessRecord()_ method.

<pre class="brush: csharp; title: ; notranslate" title="">protected override void BeginProcessing()
{
	base.BeginProcessing();
}
</pre>

The logic for this should be that if the age is set and less than 21 it should write an error and stop the script; otherwise, it should continue as normal. So let&#8217;s replace the “_base.BeginProcessing();_” with the following code:

<pre class="brush: csharp; title: ; notranslate" title="">protected override void BeginProcessing()
{
	if (agedynparm != null && agedynparm.Age &lt; 21)
	{
		 // write error here.               
	}
}
</pre>

The error that is thrown is not going to be a simple _WriteError()_ method call. The _WriteError()_ method does not have an _ErrorAction_ enum defined in the declaration. What we will use is the _ThrowTerminatingError(ErrorRecord)_ method. To use this method we need to do the following:

  1. Create an Exception. For this we will use the _ParameterBindingException_. We will also define the error message here.
  2. Create an Error Record using the _ParameterBindingException_, a null value for the _errorId_, the _ErrorCategory_ of _PermissionDenied_, and point it to our _agedynparam_.Age parameter.
  3. Call the _ThrowTerminatingError_ method with our custom _ErrorRecord_.

You can get more information on the ErrorRecord constructor [here][5] . You can also get more information on the _ErrorCategory_ enumeration and the explanation of each category [here][6] .

Our _BeginProcessing()_ method looks like this now:

<pre class="brush: csharp; title: ; notranslate" title="">protected override void BeginProcessing()
{
	if (agedynparm != null && agedynparm.Age &lt; 21)
	{
		 ParameterBindingException pbe = new ParameterBindingException("You are not old enough for Hard Lemonade. How about a nice glass of regular Lemonade instead?");
		 ErrorRecord erec = new ErrorRecord(pbe, null, ErrorCategory.PermissionDenied, agedynparm.Age);
		 ThrowTerminatingError(erec);                
	}
}
</pre>
### Final Code, Build, and Testing

Once we add our age check in the _BeginProcessing()_ method of our code we now have our finished Electronic Lemonade Stand cmdlet!

```
using System;
using System.Collections.ObjectModel;
using System.Management.Automation;
namespace GetOrder
{
    [Cmdlet(VerbsCommon.Get, "Order")]
    public class GetOrder : PSCmdlet, IDynamicParameters
    {
        private AgeDynamicParameter agedynparm = null;
        [Parameter(
            Mandatory = true,
            Position = 1,
            HelpMessage = "How many cups would you like to purchase?"
        )]
        public int Cups;

        [Parameter(
            Mandatory = false, 
            Position = 2, 
            HelpMessage = "What would you like to purchase?"
        )]
        [ValidateSet("Lemonade","Water","Tea","Coffee","Hard Lemonade")]
        public string Product = "Lemonade";

        public object GetDynamicParameters()
        {
            if (Product == "Hard Lemonade")
            {
                agedynparm = new AgeDynamicParameter();
                return agedynparm;
            }
            else
            {
                return null;
            }
        }

        protected override void BeginProcessing()
        {
            if (agedynparm != null && agedynparm.Age &lt; 21)
            {
                ParameterBindingException pbe = new ParameterBindingException("You are not old enough for Hard Lemonade. How about a nice glass of regular Lemonade instead?");
                ErrorRecord erec = new ErrorRecord(pbe, null, ErrorCategory.PermissionDenied, agedynparm.Age);
                ThrowTerminatingError(erec);                
            }
        }

        protected override void ProcessRecord()
        {
            Collection&lt;String&gt; order = new Collection&lt;string&gt;();
            for (int cup = 1; cup &lt;= Cups; cup++) {
                order.Add(cup.ToString() + ": A cup of " + Product);
            }
            WriteObject(order, true);
        }
    }

    public class AgeDynamicParameter
    {
        private int _age;

        [Parameter(
            Mandatory = true,
            Position = 3,
            HelpMessage = "Please enter your age:"
        )]
        public int Age
        {
            get { return _age; }
            set { _age = value; }
        }        
    }
}
```

Do your final build of the Class Library and test it in PowerShell. Here is the output from our ESL system.

### Test to ensure previous items still work as originally scripted

```
PS C:\psf\scripts> Import-Module C:\Development\Get-Order\Get-Order\bin\Release\Get-Order.dll
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
PS C:\psf\scripts>
```

```
### Now to test the “Hard Lemonade” product prompt and age logic for a value less than 21
PS C:\psf\scripts> Get-Order 3 "Hard Lemonade"
cmdlet Get-Order at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
Age: 18
Get-Order : You are not old enough for Hard Lemonade. How about a nice glass of regular Lemonade instead?
At line:1 char:1

+ Get-Order 3 "Hard Lemonade"

  ~~~~~~~~~~~~~~~~~~~~~~~~~~~
  + CategoryInfo          : PermissionDenied: (18:Int32) [Get-Order], ParameterBindingException
  + FullyQualifiedErrorId : GetOrder.GetOrder
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~

PS C:\psf\scripts>
```

### Now to test the “Hard Lemonade” product prompt and age logic for a value 21 or greater.

```
PS C:\psf\scripts> Get-Order 3 "Hard Lemonade"

cmdlet Get-Order at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
Age: 21
1: A cup of Hard Lemonade
2: A cup of Hard Lemonade
3: A cup of Hard Lemonade
PS C:\psf\scripts>
```

Testing our age dynamic parameter as a 3<sup>rd</sup> positional parameter.

<pre class="brush: powershell; title: ; notranslate" title="">Testing our age dynamic parameter as a 3rd positional parameter.
PS C:\psf\scripts&gt; Get-Order 3 "Hard Lemonade" 34
1: A cup of Hard Lemonade
2: A cup of Hard Lemonade
3: A cup of Hard Lemonade
PS C:\psf\scripts&gt;
</pre>
### Conclusion

Success! Just like our PowerShell script, we have created a dynamic parameter that only appears when our product is “Hard Lemonade” and will only return that product if the age is 21 or older. Our previous functionality is still there and our enhancement only shows up under a specific condition. Except this time we did it in C#.

I hope you have enjoyed this follow up article on dynamic parameters. They are very powerful and very useful in helping you streamline your cmdlets. I wrote this article side by side with my previous one so that you can follow both from start to finish. Please feel free to download the code from my GitHub repository and build your own C# cmdlet with a dynamic parameter.

As I mentioned at the beginning of this article, the starter code for this cmdlet is available at <https://github.com/Ben0xA/Get-Order>. There are two branches available in that repository. The master branch is the starter code. The Final-Code branch contains the final Electronic Lemonade Stand C# cmdlet.

[1]: /2014/05/29/dynamic-parameters-in-powershell/
[2]: /tags/pscmdlet/
[3]: http://msdn.microsoft.com/en-us/library/dd878299(v=vs.85).aspx
[4]: http://msdn.microsoft.com/library/dd878334(v=vs.85).aspx
[5]: http://msdn.microsoft.com/en-us/library/system.management.automation.errorrecord%28v=vs.85%29.aspx
[6]: http://msdn.microsoft.com/en-us/library/system.management.automation.errorcategory%28v=vs.85%29.aspx
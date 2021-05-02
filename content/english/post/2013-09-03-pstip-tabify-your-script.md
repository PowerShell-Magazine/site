---
title: '#PSTip Tabify your Script'
author: Josh Miller
type: post
date: 2013-09-03T18:00:32+00:00
url: /2013/09/03/pstip-tabify-your-script/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

There are several things you can do to make more readable scripts.  You can do good documentation and use variables that are meaningful.  If you have your curly brackets in random places and functions that are longer than 5 or 6 lines things can still be unreadable.  This is a little easier if you are using the PowerShell 3.0 or 4.0 ISE and can collapse blocks of code.

Fortunately there are a few things that we can do to make the braces show up in a more uniform location.

PowerShell 2.0 provides a Tokenize function in the PSParser class that will parse your script and give you a collection containing all text parts of the script.  A quick utility that can be made from the tokens is a tabification of the script.


	function Tabify-PS2Script {
	    Param($ScriptText) 
	
	    $CurrentLevel = 0
	    $ParseError = $null
	    $Tokens = [System.Management.Automation.PSParser]::Tokenize($ScriptText,[ref]$ParseError)
	
	    if($ParseError) { 
	        $ParseError | Write-Error
	        throw "The parser will not work properly with errors in the script, please modify based on the above errors and retry."
	    }
	
	    for($t = $Tokens.Count -1 ; $t -ge 1; $t--) {
	
	        $Token = $Tokens[$t]
	        $NextToken = $Tokens[$t-1]
	
	        if ($Token.Type -eq 'GroupStart') { 
	            $CurrentLevel-- 
	        }  
	
	        if ($NextToken.Type -eq 'NewLine' ) {
	            # Grab Placeholders for the Space Between the New Line and the next token.
	            $RemoveStart = $NextToken.Start + 2  
	            $RemoveEnd = $Token.Start - $RemoveStart
	            $tabText = "`t" * $CurrentLevel  
	            $ScriptText = $ScriptText.Remove($RemoveStart,$RemoveEnd).Insert($RemoveStart,  $tabText)
	        }
	
	        if ($token.Type -eq 'GroupEnd') { 
	            $CurrentLevel++ 
	        }		
	    }
	
	    $ScriptText
	}
PowerShell 3.0 provides an additional parse routine that generates the AST that is used to run the script. The 2.0 Tokenize will work in 3.0 but it is good to see how when accomplishing the same task there are a few nuanced differences.  These small differences are helpful to understand if you want to try to do other things with the AST to do analysis on a script or do more advanced code cleanup.


	function Tabify-PS3Script {	
	    Param($ScriptText) 
	
	    $CurrentLevel = 0
	    $ParseError = $null
	    $Tokens = $null
	    $AST = [System.Management.Automation.Language.Parser]::ParseInput($ScriptText, [ref]$Tokens, [ref]$ParseError)	
	
	    if($ParseError) { 
	        $ParseError | Write-Error
	        throw "The parser will not work properly with errors in the script, please modify based on the above errors and retry."
	    }
	
	    for($t = $Tokens.Count -2; $t -ge 1; $t--) {
	
	        $Token = $Tokens[$t]
	        $NextToken = $Tokens[$t-1]
	
	        if ($token.Kind -match '(L|At)Curly') { 
	            $CurrentLevel-- 
	        }  
	
	        if ($NextToken.Kind -eq 'NewLine' ) {
	            # Grab Placeholders for the Space Between the New Line and the next token.
	            $RemoveStart = $NextToken.Extent.EndOffset  
	            $RemoveEnd = $Token.Extent.StartOffset - $RemoveStart
	            $tabText = "`t" * $CurrentLevel  
	            $ScriptText = $ScriptText.Remove($RemoveStart,$RemoveEnd).Insert($RemoveStart,$tabText)
	        }
	
	        if ($token.Kind -eq 'RCurly') { 
	            $CurrentLevel++ 
	        }		
	    }
	
	    $ScriptText
	}
	
	$psISE.CurrentFile.Editor.Text = Tabify-PS2Script $psISE.CurrentFile.Editor.Text 
	#or
	$psISE.CurrentFile.Editor.Text = Tabify-PS3Script $psISE.CurrentFile.Editor.Text 
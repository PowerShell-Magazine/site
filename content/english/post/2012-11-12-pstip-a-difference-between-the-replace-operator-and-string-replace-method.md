---
title: '#PSTip A difference between the –replace operator and String.Replace method'
author: Jakub Jareš
type: post
date: 2012-11-12T19:00:25+00:00
url: /2012/11/12/pstip-a-difference-between-the-replace-operator-and-string-replace-method/
categories:
  - Tips and Tricks
  - Brainteaser
tags:
  - Brainteaser
  - Tips and Tricks

---
The -replace operator  takes a regular expression (regex) replacement rule as input and replaces every match with the replacement string. The operator itself is used as shown in the following schema: `<input string> -replace <replacement rule>,<replacement string>`

```
PS> "Hello. Yes, this is a cat." -replace 'cat','dog'
Hello. Yes, this is a dog.
```

Although it is not obvious from the example, the ‘cat’ string is in fact a regular expression rule, and this rule can include special characters like “.” that don’t behave as you might expect.

```
PS> "Hello. Yes, this is a cat." -replace '.','dog'
dogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdogdog
```

Problem is you often need to replace “.” , “|” or other characters that has special meaning in regex language. One way to achieve this is to escape every special character by “\”.

```
PS> "Hello. Yes, this is a dog." -replace '\.','!'
Hello! Yes, this is a dog!
```

_Now you have two problems._ You need to learn what characters are considered special in regex and check every replacement rule for these special characters, or take it a step forward and explicitly escape the whole string using a static method of the Regex type accelerator.

```
PS> "Hello. Yes, this is a dog." -replace [regex]::Escape('.'),'!'
Hello! Yes, this is a dog!
```

This enables you not only to escape the whole string at once, but also to ask the user for the replacement rule without worrying about any special character it may contain.

All that said there is String.Replace() method that does exactly the same as the example above. The main difference between the two is the replacement rule which is just plain text, not a regex rule. The method is used as such:

```
PS> ("Hello. Yes, this is a dog.").Replace('.','!')
Hello! Yes, this is a dog!
```

But since the –split, -replace, and -match operators are used pretty heavily in scripts there is a chance you are going to mix the –replace operator and Replace() method in one script, producing a script that is hard to understand and difficult to maintain because you require the maintainers to know the little differences in the usage.
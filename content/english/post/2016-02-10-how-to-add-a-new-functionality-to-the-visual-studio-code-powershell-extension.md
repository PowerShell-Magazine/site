---
title: How to add a new functionality to the Visual Studio Code PowerShell extension
author: Jan Egil Ring
type: post
date: 2016-02-10T19:59:38+00:00
url: /2016/02/10/how-to-add-a-new-functionality-to-the-visual-studio-code-powershell-extension/
post_views_count:
  - 2560
categories:
  - How To
  - VS Code
tags:
  - How To
  - VS Code

---
[Visual Studio Code][1] is a free code editor redefined and optimized for building and debugging modern web and cloud applications.

It supports [extensions][2] for adding additional languages, themes, debuggers, commands, and more.

![](/images/codeise1.png)

As you might notice in the above image, one of the extensions is a PowerShell extension. The extension provides PowerShell language support including syntax highlighting, code snippets, IntelliSense, code navigation, real-time script analysis, and local script debugging. More information and installation instructions can be found in [this article][3] on the PowerShell Team blog.

Earlier this month I reached out to [David Wilson][4], a software engineer in the PowerShell team at Microsoft, to ask the following question:

![](/images/codeise2.png)

Microsoft MVP [Doug Finke][5] also reached out to me to ask whether I was going forward with adding that feature to the extension. Thanks to him and David I got the jumpstart I needed to begin creating

It did require some knowledge of [TypeScript][6], which is a typed superset of JavaScript that compiles to plain [JavaScript][7]. As I’ve never work with any other languages than those related to “admin-scripting”, this was new ground for me. Although I found it very interesting to look into something new, and a lot of concepts I’m used to from working with PowerShell could be leveraged.

After spending a few evenings working on it, as well as getting very fast and great support from David and Doug along the road, I got the extension working as I had hoped.

The PowerShell extension for Visual Studio Code is hosted on [GitHub][8], so I started by [forking][9] my own [copy of the repository][10]. A good practice when working with Git (or any other source control system for that matter), is to create a new [branch][11] when developing new features. The dedicated branch can then be merged into the master branch and a pull request can be sent to the original repository in order to get the new feature added to the product.

After forking the original repository I created a new branch in my fork called [OpenInISE][12], where I started working on the implementation.

The implementation is added to multiple files in the Visual Studio Code PowerShell Extension repository:

Package.json on line [60-62][13]:

![](/images/codeise3.png)

This is creating a so called key binding, so that the command PowerShell.OpenInIse is triggered when the editor is focusing on text and the current language ID is PowerShell (which is automatic when opening ps1/psd1/psm1-files).

Next in the same file (Package.json), the following is defined on line [82-84][14]:

![](/images/codeise4.png)

In main.ts on [line 15][15], the command is imported from a sub-directory:

![](/images/codeise5.png)

On [line 105][16] in main.ts the OpenInIseCommand is registered:

![](/images/codeise6.png)

The feature itself is defined in [OpenInISE.ts][17]:

![](/images/codeise7.png)

In the same way you test code when developing for example a PowerShell script, you need to test it when doing changes. In Visual Studio Code, which I used as an editor when writing this, the recommended workflow I got from David was the following:

  1. From your PowerShell console in the vscode-powershell directory, type code &#8211;extensionDevelopmentPath=&#8221;c:\your\path\to\vscode-powershell&#8221; . A development version of VS Code will appear with your current code loaded.
  2. Type npm install. (Author’s note: Npm is the default package manager for the JavaScript runtime environment Node.js)
  3. Type npm run compile. This will build the code and wait for any further file saves then it will recompile again.
  4. When you make changes to the code you can press the Ctrl+R  hotkey to have VS Code reload itself and pick up any changes you made to the code after they are recompiled
  5. In your code, you can temporarily call showInformationMessage  to make some text appear at the top of the VS Code screen. This might be helpful for seeing the state of your variables while you&#8217;re working out the details of the function.

This worked well for me, but since I didn’t have any experience in this area I missed a basic pre-requisite which was to install [node.js][18]:

_Node.js is an open-source, cross-platform runtime environment for developing server-side web applications. Node.js applications are written in JavaScript and can be run within the Node.js runtime on a wide variety of platforms_

I downloaded and installed version 5.4.0 from <https://nodejs.org/en>, and after that the suggested workflow worked as expected.

When the extension was finished, tested, and reviewed by David and Doug I submitted a pull request which was accepted:

![](/images/codeise8.png)

The new command is available in [version 0.4.0][19] of the PowerShell extension for Visual Studio Code, which was released February 8, 2016. After you have upgraded to that version, you can enter Ctrl+Shift+I to open PowerShell files in PowerShell ISE.

Personally I find myself using Visual Studio Code more and more when working on larger PowerShell artifacts, such as repositories for PowerShell Desired State Configuration resources. It’s very fast and easy to use when you need a quick overview of a repository, as well as to make changes. When testing scripts still prefer to use PowerShell ISE, hence the request for an _Open in ISE extension_.

I hope this introduction to adding new capabilities to the Visual Studio Code PowerShell extension was useful, and that you may have gotten the curiosity to try it out for yourself.

[1]: https://code.visualstudio.com/
[2]: https://marketplace.visualstudio.com/#VSCode
[3]: http://blogs.msdn.com/b/powershell/archive/2015/11/17/announcing-windows-powershell-for-visual-studio-code-and-more.aspx
[4]: https://twitter.com/daviwil
[5]: https://twitter.com/dfinke
[6]: http://www.typescriptlang.org/
[7]: https://en.wikipedia.org/wiki/JavaScript
[8]: https://github.com/PowerShell/vscode-powershell
[9]: https://help.github.com/articles/fork-a-repo/
[10]: https://github.com/janegilring/vscode-powershell
[11]: https://guides.github.com/introduction/flow/
[12]: https://github.com/janegilring/vscode-powershell/tree/OpenInISE
[13]: https://github.com/janegilring/vscode-powershell/blob/OpenInISE/package.json#L60-L62
[14]: https://github.com/janegilring/vscode-powershell/blob/OpenInISE/package.json#L82-L84
[15]: https://github.com/janegilring/vscode-powershell/blob/OpenInISE/src/main.ts#L15
[16]: https://github.com/janegilring/vscode-powershell/blob/OpenInISE/src/main.ts#L105
[17]: https://github.com/janegilring/vscode-powershell/blob/OpenInISE/src/features/OpenInISE.ts
[18]: https://en.wikipedia.org/wiki/Node.js
[19]: https://github.com/PowerShell/vscode-powershell/blob/master/CHANGELOG.md#040
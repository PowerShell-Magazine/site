---
title: 'Git for IT Professionals: Working with Repositories'
author: Ravikanth C
type: post
date: 2015-07-15T16:00:31+00:00
url: /2015/07/15/git-for-it-professionals-working-with-repositories/
views:
  - 16612
post_views_count:
  - 2725
categories:
  - DevOps
  - Git
tags:
  - DevOps
  - Git

---
In this series so far:

[Part 1 &#8211; Git for IT Professionals: Getting Started](/2015/07/13/git-for-it-professionals-getting-started-2/)

Part 2 &#8211; Git for IT Professionals: Working with Repositories (this article)

[Part 3 &#8211; Git for IT Professionals: Life cycle of repository files](/2015/07/27/git-for-it-professionals-life-cycle-of-repository-files-2/)

In the previous article, I touched upon creating a Git repository but did not quite explain what those commands meant and what different ways of creating a repository are. In this part of the article, we will explore different ways of creating repositories and working with them.

Before we get started, let’s understand what a Git repository is. A Git repository is where your project lives. It is just another folder with .git subfolder inside it that contains all the version controlled objects. The _git init_ command is what creates this .git subfolder and the required objects inside that. There are multiple ways of creating a new repository.

The first method is to simply run the _git init <projectname>_ command at the GitBash console. This command create a directory with the specified project name and initializes the .git subfolder inside the project directory. By default, GitBash console starts in the user&#8217;s home directory. So, folders created by _git init_ command will be inside the user&#8217;s home directory.

![](/images/git21.png)

As you see in the above picture, running _git init MyFirstRepo_ command creates a directory with the same name as the project and initializes the .git subfolder within that. This subfolder contains all the required files and folders to start tracking version control information for the project.

In the second method of creating a repository, we can go to an existing folder (even when it contains files) and initialize that directory as a repository.

![](/images/git22.png)

In the above example, you will see that I have an existing folder with a PowerShell script. Within that folder, I had initialized the repository and a .git subfolder gets created with similar contents as our first example. We will discuss the need and use of these special files and folders within the .git subfolder as we progress in this series.

There is a third method to create a Git repository and that is called cloning. We will discuss that at a later point in time. To see the status of the projects we just created, we can run _git status_ command. This needs to be run in the project directory.

![](/images/git23.png)

The output shown above has two important things we need to know. First is the branching concepts and how to use it effectively. For now, understand that like every other VCS, Git supports branching and that master branch is the main branch in the repository. The second thing we need to know is the commit concepts. The above command output shows that there is nothing to commit (for _MyFirstRepo_) and we can use _git add_ command to create or copy files to track. For the second repository (_MySecondRepo_) which was initialized in an existing directory, we see that there are untracked files and output suggesting that we use _git add <file>_ to start tracking those files.

Unlike other VCS, Git does not track or commit any files into the repository unless they are explicitly added using the _git add_ command. This eliminates accidental commits that developers generally face with other VCS. Let’s run _git add TestScript.ps1_ inside the _MySecondRepo_ repository and see what happens.

![](/images/git24.png)

As shown in the above output, running _git add_ command inside the repository adds the file specified to **tracking** and **stages** it for any future commits. Internally, Git takes a snapshot of the tracked files in the working tree and stores that in the Git index. The below output from the project’s .git directory shows that the tracked files are now added index (file named index).

![](/images/git25.png)

We can now commit this file by using the _git commit_ command.

![](/images/git26.png)

When we use _git commit_ command, we are expected to specify the commit message. If that was not provided as a part of the command line arguments, the default VIM editor gets opened for prompting the message.

**Note:** If you are not a big fan of VIM and instead want to use PowerShell ISE as the editor for these messages, you can change it in the Git configuration settings using _git config core.editor powershell_ise.exe_ command.

If you want to avoid this prompt, you can specify the _–m_ switch. For example,

```powershell
git commit -m 'This is my first commit'
```

Once the commit is complete, you will see output similar to the below.

![](/images/git27.png)

If you have followed the steps so far and completed a _git commit_, congratulations! You just followed the end to end flow for creating your first version controlled repository.

Since Git takes a snapshot of the working tree, any changes made after the _git add_ command won’t be staged for commit. So, if we modify a file that is tracked and committed, running _git status_ command will tell us that there are changes to tracked or committed files that are not staged yet.

![](/images/git28.png)

At this point, if you want to commit the updates to a tracked file or start tracking new files, you know what exactly you need to do. Yes, you need to use the _git add_ command. Running _git add_ command again will take a snapshot of all the tracked and/or modified files to the index and stages it for next commit.

If you all you want to do is update already tracked files, we can use _git add_ command with _–update_ switch. This will modify the index for already tracked and modified files but does not add any new files that are not tracked.

What we have learned so far is the basics of working with repositories. There is certainly more. However, let’s stop here for now and practice what we learned so far.

I have mentioned different states of a Git controlled file. For example, Git file goes from untracked to tracked state and unstaged to staged state and finally, it goes to a committed state. The _git status_ command provides this information. However, it is important to understand the life cycle of a Git repository to make better use of VCS. And, that is our next article. Stay tuned.

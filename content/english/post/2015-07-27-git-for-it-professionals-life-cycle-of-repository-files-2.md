---
title: 'Git for IT Professionals: Life Cycle of Repository Files'
author: Ravikanth C
type: post
date: 2015-07-27T17:52:57+00:00
url: /2015/07/27/git-for-it-professionals-life-cycle-of-repository-files-2/
views:
  - 13382
post_views_count:
  - 2073
categories:
  - DevOps
  - Git
tags:
  - DevOps
  - Git

---
In this series so far,

[Part 1 - Git for IT Professionals: Getting Started](/2015/07/13/git-for-it-professionals-getting-started-2/)

[Part 2 – Git for IT Professionals: Working with Repositories](/2015/07/15/git-for-it-professionals-working-with-repositories/)

Part 3 – Git for IT Professionals: Life Cycle of Repository Files (this article)

In the previous part, we walked-through creating our first repository and adding files to it and finally committing the files to the repository. This was a simple and basic workflow to get started with Git. In the process, I had mentioned about tracked and untracked files and staged and unstaged files and so on. For a beginner, it is important to understand the life cycle of version-controlled files in a Git repository. This is what we will discuss in this part of the series. Similar to the other part of the series, we will use an example walk-thorough to understand the concepts. I recommend trying out the commands as you read the article.

For this article, we will start by creating a new repository with no files in it.

```powershell
git init FirstRepo
```

This initializes an empty Git repository under the home directory of the logged in user. Running _git status_ inside the repository reveals that there are no files that are tracked.

![](/images/git31.png)

Let us add a few files here. PowerShell scripts, of course. I chose to copy a few readymade scripts I have into this repository directory directly.

![](/images/git32.png)

Running _git status_ now shows that I have some files that are in the repository directory or the working tree but are **untracked**.

![](/images/git33.png)

So, that first state of files in a newly created Git repository is **Untracked**. Git is helpful enough and tells us what needs to be done to start tracking these files. We have to run _git add_.

**Tip:** You can use _git add ._ if you **intend to add all files** in the working tree to the index for next commit. I have highlighted part of the earlier sentence to ensure you understand the importance of this command. If you don’t intend to add all files in the working tree to next commit, specify them explicitly.

In my example, I am going to add all the files in the repository folder to the next commit.

![](/images/git34.png)

As you see in this output the files moved from **Untracked** to **Staged**! Now, if we use the _git commit_ command, all these files will get added to the master branch and get **tracked**.

![](/images/git35.png)

Let us now modify one of the scripts and see what happens. In my example, I modified the wim.ps1 script to add a comment at the beginning of the file.

![](/images/git36.png)

As expected, we see the file that we modified as modified! Hurray! This happens because the file is already present in the Git tracked files and the index. In technical terms, the last commit includes this file. So, Git finds that the file content is different from what is there in the last commit and tells us the same. If you read the message closely, it also tells us that we can either use _git add_ to stage this file for next commit or _git commit_ to commit the changes directly. For now, let us stage the changes for next commit.

![](/images/git37.png)

Instead of doing a commit now, let us modify this file (wim.ps1) again and check _git status_.

![](/images/git38.png)

Unexpected? Not really. I said Git is different from other VCS. Git does not track changes to files per se. If we go revisit Git basics, we discussed that every time we run _git add_, Git actually takes a point-in-time snapshot of the file and tracks that for the next commit. So, when we modified the file again after _git add_, Git finds that the content of the file on disk is different from what is being tracked for next commit. This is the reason why we see the same file in the staged section as well as unstaged section of _git status_. If we use _git commit_ now, the version of the file which is in the snapshot becomes tracked.

**Caution:** Git tells us that we can use _git checkout_ to discard changes. Be very careful when you do this. This command takes the last staged/commit version from the repository and overwrites that on the disk. This may have unintended consequences or result in loss of data.

So, instead of discarding changes on the disk, if you want to update the staged version of the file, we can use _git add –u_ or simply _git add_. The _git add –u_ commands ensure that it updates only the modified files in staged snapshot and does not add any new files in the working tree that are not tracked yet.

While you were busy reading through and practicing what you just learned, I went ahead and made a few random changes to already tracked files, added a few more files and modified a few staged files. Here is how my _git status_ looks like now.

![](/images/git39.png)

This may be confusing when more than one file is listed in different states. Git provides a short status too that can be very helpful understanding in an easy manner than a more verbose output.

![](/images/git310.png)

You will see that there are two columns in the output shown by _git status –s_. The left column indicates that the files are staged and the right-hand column indicates the files that are modified.

  * The _??_ at the end indicates that the file is not tracked.
  * The status _A_ shown next to wim4.ps1 and wim5.ps1 indicates that these files are new and staged for next commit.
  * The status _M_ next to newly added wim5.ps1 indicates that the file was modified after it was staged.
  * The status _M_ next to Support-Scripts.ps1 indicates that the file is modified and the same has been staged for commit.
  * The status _M_ next to wim2.ps1 indicates that the file was modified but the contents are not staged for next commit yet.
  * The status _MM_ next to wim.ps1 indicates that the already tracked file was modified and staged for next commit but was modified again.

Just remember that the left column indicates files that are staged for next commit and the right column indicates files that are modified. As you start using Git more and more, looking at this short status and making sense of what’s going on in the repository becomes very easy. Finally, what if we want to stop tracking a file in the repository? Or in other words, how do we mark a file untracked and remove it from the commit snapshot? The _git rm_ command is the answer. Note that using _git rm_, **removes the file from disk too**. If you don’t intend to remove the file from disk, use the _&#8211;Cached_ switch with the _git rm_ command. However, doing so will show the removed file as untracked in the _git status_ output.

![](/images/git311.png)

Overall, we have seen files moving from one state to another as we made changes or added new files to repository. In a nutshell, we had **untracked** file which moved to **staged** and then **tracked** state after a _git commit_. We now have the same file in **staged** as well as **unstaged** state. We finally, removed files from the repository and made them **untracked** again. And, that is the complete life cycle of files in a Git repository. A picture is worth a thousand words. Let us conclude today’s article by looking at this diagram that shows different states of files in a Git repository and what actions bring files to those states.

![](/images/git312.png)

This is it for today! Your feedback is more than welcome and will help me improve the overall series. Stay tuned for more!

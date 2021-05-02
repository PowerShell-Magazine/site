---
title: Write For Us
author: Ravikanth C
type: page
date: 2011-09-03T12:51:08+00:00
url: /write-for-us

---
PowerShell Magazine serves exclusive PowerShell content mostly authored by the community members and Microsoft MVPs. A few selected guest authors will be rewarded for the content either in cash or other means. You are welcome to work with us as a contributing author. As a part of our authoring and editorial process, we have put together guidelines for the aspiring authors on writing articles for the PowerShell Magazine. These guidelines are provided to ensure we have uniformity and quality of content. These are, by no means, to limit your creativity and writing style.

### Submitting an article pitch

We need to know what your article is about and see an outline of the article before we can decide whether we can include your content in the magazine or not.

  * Articles should be at least 600 words long (one page). If an article is longer, then the number of words should be multiples of 600.
  * When submitting the article pitch, make sure you specify the target audience for your article.
  * We do not accept commercial messages as articles unless you are sponsor of PowerShell Magazine. This includes product reviews and promotional messages.
  * PowerShell Magazine serves IT administrators using PowerShell as well as developers working on PowerShell. However, we always try to strike a balance when publishing the content. Your article pitch can fall into any of the two categories as long as the content is PowerShell specific.
  * To submit an article pitch, reach out to contribute@powershellmagazine.com.
  * It is recommended to have an outline when you submit the pitch. This helps us decide faster.
  * Please give us at least a week to get back to you.

### Once your pitch gets approved

You need to adhere to the following guidelines to ensure the authoring and editorial process works smoothly and the content you are writing fits into the overall magazine framework.

  * PowerShell Magazine is a statically built site that uses markdown format for all articles. We recommend that you use  PSMagAuthorToolkit module for creating (`New-PSMagArticle`) a markdown draft for your article. If you are a first time PowerShell Magazine author, you need to use `New-PSMagAuthorPage`  for generating a markdown page for author profile.
  * Use informal and personal tone.
  * For interactive examples, always use PS> prompt.
  * A line of code can have no more than 60 characters. If a line is longer than this limit, you need to break it properly.
  * Indents and spaces in code should be made using spaces, not tabs. (`"3 spaces" -eq "1 tab"`)
  * Use full type (except for type accelerators), cmdlet, and parameter names with proper capitalization (`New-Object`, `-ComputerName`)
  * Use lowercase for keywords and operators.
  * Use Pascal case for attributes and member names (`ToString`, `ToCharArray`, and so on). If identifier consists of just two letters, all letters in the identifier are capitalized (`System.IO`, `CurrentUICulture`, and so on)
  * Attach longer code listings in a file/gist. Use Unicode/UTF-8 character encoding when the code contains non en-US characters.
  * Avoid too many screenshots and images in your content. You can always copy/paste console output and format it using code highlights in markdown.
  * Once you have a draft ready for review, deliver it as a [markdown document](https://en.wikipedia.org/wiki/Markdown) to editors@powershellmagazine.com.

Based on the content, the editorial process may take a few days to complete. We will keep you updated, ask questions, and may ask you to re-write a few sections, as required.

### Once your content gets approved

Once the review process is complete and gets editorial nod to publish, your article gets into the publishing queue. We will communicate publishing timeline and anything else that is needed for us to finalize/publish the content. We use [GitOps](https://about.gitlab.com/topics/gitops/) for publishing articles, your reviewed draft will be sent as a pull request to the live branch of PowerShell Magazine site. Based on when the article needs to be published, our GitHub workflows will take care of the rest. 


From a legal standpoint,

- PowerShell Magazine assumes all rights on the content you submit. You cannot cross post this content and/or publish the content elsewhere without prior written permission from the editors.
- At the same time, we don't want to publish content that is already popular on your blog or elsewhere. There is no point in publishing the same content in the magazine. To some extent, you are allowed to reuse the content from your own blog or your own articles but not someone else's content.

If you have any further questions about submitting an article pitch or developing the content for your pitch, feel free to reach out to us @ editors@powershellmagazine.com.


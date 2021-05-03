---
title: The all new PowerShell Magazine is here and more to come!
author: Ravikanth C
type: epic
date: 2021-05-08
images: 
  - "/images/welcomehome.png"
url: /2021/05/03/The-all-new-PowerShell-Magazine-is-here-and-more-to-come/
categories:
- News
tags:
- News

---
This one took a while but we finally moved to a new home! No more painful management of plugins and updates to PHP.

When we [launched in 2011](/2011/09/12/announcing-the-launch-of-powershell-magazine-website/), there was not much choice but use WordPress as the platform for content management. It worked well. We brought up a site in no time and started authoring and inviting new authors. In no time, we grew from 6 editors writing everything to [85+ contributing authors](/author). We published the largest collection of [PowerShell Tips and Tricks](/tags/tips-and-tricks), ran in-depth series of articles on [InfoSec](/tags/infosec) and [PowerShell DSC](/tags/powershell-dsc), conducted many author contests, and published a ton of other interesting content. More than 70% of the content on this site was published between 2014 and 2018. We built a community. The most powerful community. We had over 1 million views per year.

We slowed down since 2018 and did not really invest much time on reviving this. Also, the overhead of content management meant someone had to continuously look into issues that were popping up because of a failed plugin to making sure that the content is secure. We had instances of content hijacking because of vulnerabilities in WordPress. To be honest, with full time jobs, none of the editors had that time. Early 2020, I started looking for options to move [my personal blog](https://ravichaganti.com/) to a static site for more or less similar reasons and [finally moved it](https://ravichaganti.com/blog/moved-to-static-site-using-hugo-and-github-pages/) to use [Hugo platform](https://gohugo.io/about/what-is-hugo/) for static page generation. With Hugo, I can write all my content in markdown and it gets compiled to HTML. With simple automation and a GitHub repository, I could automate the overall publishing process. 

Looking at the simplicity in authoring and publishing workflow for my own blog, I made a decision that PowerShell Magazine should go the same way. There are tools to export WordPress content to markdown. It, of course, comes with its own issues. We had 833 articles to move and each article gets exported to a markdown file. However, this markdown may not be compatible with the theme that we choose to implement. This meant, I had to go through each and every article, convert code snippets to match the theme style, convert absolute URLs to relative URLs, download images to a new folder structure, and make sure each article loaded as expected. Not much of this could be automated. This  was a painfully long process. I am glad I finally finished. You see the result here.

The content for this site is hosted in a GitHub repository. Every time we commit a new article or change to an existing one, a GitHub Action gets triggered that builds the static site and pushes it to the public facing repository which then hosts it using GitHub pages. This process is very simple. And, if you are familiar with how GitHub workflows can be automated, we have endless possibilities. We are yet to complete building a fully-automated publishing workflow but once we complete that you -- as an author -- can simply submit an article and the rest will happen automatically. 

Whether you are an existing author or a new author who wants to contribute to PowerShell Magazine, do read our updated [contribution guidelines](/write-for-us). If you are a featured author on PowerShell magazine, check out your [author](/author) page to see if there is anything that you want to update. We will publish a toolkit to generate article drafts and author pages very soon. This will help you create a good and clean starting point to write your content as markdown.

We are working on generating some exciting content for you and cannot wait to start sharing it with you. While we are busy bringing in more content, feel free to share any [feedback and/or questions with us](/feedback).

Mahalo!




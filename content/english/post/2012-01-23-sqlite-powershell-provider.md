---
title: SQLite PowerShell Provider
author: Shay Levy
type: post
date: 2012-01-23T16:20:27+00:00
url: /2012/01/23/sqlite-powershell-provider/
views:
  - 9416
post_views_count:
  - 1714
categories:
  - News
  - SQL
tags:
  - SQL
  - News

---
PowerShell MVP [Jim Christopher][1] has published a new project on codeplex &#8211; [SQLite PowerShell Provider][2]. The new provider enables you to use SQLite databases from your PowerShell session by mounting the database as a drive. You can then use the standard provider cmdlets to perform CRUD operations on the database tables and records.

The provider supports both persistent (on-disk) and transient (memory-only) SQLite databases. In addition, the provider is transaction-aware.

[1]: https://twitter.com/#!/beefarino/
[2]: http://psqlite.codeplex.com/
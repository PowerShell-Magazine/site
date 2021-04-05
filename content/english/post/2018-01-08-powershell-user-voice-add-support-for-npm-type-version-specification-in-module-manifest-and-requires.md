---
title: 'PowerShell UserVoice: Add Support for NPM Type Version Specification in Module Manifest and #Requires'
author: Ravikanth C
type: post
date: 2018-01-08T17:00:53+00:00
url: /2018/01/08/powershell-user-voice-add-support-for-npm-type-version-specification-in-module-manifest-and-requires/
views:
  - 27583
post_views_count:
  - 4860
categories:
  - Feedback
tags:
  - Modules

---
If you have ever used Node.js, the packages.json file is used to specify the module dependencies. Here is an example:

```json
{
  "name": "MyNodeJSApp",
  "version": "1.0.0",
  "description": "First Node JS Application",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Ravikanth Chaganti",
  "license": "MIT",
  "dependencies": {
    "express": "^4.16.2"
  }
}
```


In the above snippet, line 12 specifies express module with a version string _^4.16.2_. The version string here is prefixed with a caret (^) symbol. NPM supports different specification strings. We can prefix the version string with a tilde (~) as well or simply use an asterisk (*) to mean the most recent version or latest version of the module. Through the use of version range comparators, version can be specified in multiple ways. The [node-semver repository][1] provides in-depth view into this.

From the node-semver page,

A `comparator` is composed of an `operator` and a `version`. The set of primitive `operators` is:

  * `<` Less than
  * `<=` Less than or equal to
  * `>` Greater than
  * `>=` Greater than or equal to
  * `=` Equal. If no operator is specified, then equality is assumed, so this operator is optional, but MAY be included.

For example, the comparator `>=1.2.7` would match the versions `1.2.7`, `1.2.8`, `2.5.3`, and `1.3.9`, but not the versions `1.2.6` or `1.1.0`.

The tilde (~) and caret (^) ranges can be used as well.

#### X-Ranges `1.2.x` `1.X` `1.2.*` `*`

Any of `X`, `x`, or `*` may be used to &#8220;stand in&#8221; for one of the numeric values in the `[major, minor, patch]` tuple.

  * `*` := `>=0.0.0` (Any version satisfies)
  * `1.x` := `>=1.0.0 <2.0.0` (Matching major version)
  * `1.2.x` := `>=1.2.0 <1.3.0` (Matching major and minor versions)

#### Tilde Ranges `~1.2.3` `~1.2` `~1`

Allows patch-level changes if a minor version is specified on the comparator. Allows minor-level changes if not.

#### Caret Ranges `^1.2.3` `^0.2.5` `^0.0.4`

Allows changes that do not modify the left-most non-zero digit in the `[major, minor, patch]` tuple. In other words, this allows patch and minor updates for versions `1.0.0` and above, patch updates for versions `0.X >=0.1.0`, and _no_ updates for versions `0.0.X`.

#### PowerShell UserVoice

Coming to the subject of this article, having similar support in PowerShell module manifests and with #Requires statement, we can specify the module dependencies in a more flexible way. To this extent, I have created a UserVoice item: <https://windowsserver.uservoice.com/forums/301869-powershell/suggestions/32845762-support-for-npm-type-version-strings-in-powershell>

If you think this is useful feature in PowerShell, go ahead and vote it up!

[1]: https://github.com/npm/node-semver
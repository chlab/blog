---
layout: post
title: Connecting BitBucket with ActiveCollab
categories:
- Tools
tags:
- git
---
It's a common practice to reference issues in the commit messages of version control systems. It's great to see which issue was the reason a certain change was made to the codebase.

At my former workplace, we were using [BitBucket (BB)](http://bitbucket.org) for hosting our git repos and [ActiveCollab (AC)](https://www.activecollab.com/) for our issue tracking. BB allows you to reference issues in your commit messages - but only if you're using their issue tracker. AC allows you to import git repositories and scan them, but configuring and maintaining them is a pain. Also, we had in place the workflow that when an issue was fixed, you had to reassign it to the user who opened it in AC. This seemed like an unnecessary manual step to me that begged to be automated, so I automated it: meet [acbb_connector](https://github.com/MediaparX/acbb_connector) (great name, I know).

acbb_connector is a little tool that you deploy on your AC server that receives a webhook callback from BB every time a commit is made and uses the AC API to manage the issue. So you push a commit like:

{% highlight text %}
fixed syntax error
fixes #188
{% endhighlight %}

and it gets parsed, fixes issue 188 of the project that corresponds to the BB repo the commit is coming from and reassigns it to the user that opened the issue.

Documentation on how to install and configure is on the [github page](https://github.com/MediaparX/acbb_connector).

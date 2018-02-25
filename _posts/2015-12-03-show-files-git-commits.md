---
layout: post
title: Show changed files between two git commits
---
Sometimes I want to get a list of files I changed within the history of a branch (or between two commits). To find out which commits I want to use I just refer to the graph, you can use any number of tools for this such as gitx, sourcetree or even *git log --graph --oneline --all*.

Run this command to add a git alias:
{% highlight bash %}
git config --global alias.changed-between '!f() { git log --pretty="%H" --author="$(git config user.email)" $1..$2 | while read commit_hash; do git show --oneline --name-only $commit_hash | tail -n+2; done | xargs ls 2>/dev/null | sort | uniq; }; f'
{% endhighlight %}

Then use it like this
{% highlight bash %}
git changed-between 07d2882 4881cdd
{% endhighlight %}

Which will give you an output like:

{% highlight text %}
src/AppBundle/Controller/BaseController.php
src/AppBundle/Controller/UserController.php
src/AppBundle/Entity/Address.php
src/AppBundle/Form/AddressType.php
{% endhighlight %}

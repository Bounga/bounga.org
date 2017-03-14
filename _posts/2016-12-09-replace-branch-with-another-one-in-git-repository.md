---
title: Replace a branch with another one in a Git repository
category: Tips
tags: ['git']
header:
  overlay_image: /assets/images/headers/tips.jpg
---

Last week a [workmate](https://twitter.com/akarzim) did me a pull
request where he wanted to merge a fully new "develop" branch into
"master". The thing was that he realized that he has to change
everything 'cause he wanted to work with the latest version
of [Middleman](https://middlemanapp.com). He has already started some
work with an older version but his latest work needed the latest one.

## The wrong path

The first thing I tried was to dumbly merge "develop" branch into
"master" but as you may have guess if you read this, it didn't
worked. Branches were so much different that [Git](https://git-scm.com)
told me:

```
fatal: refusing to merge unrelated histories
```

That makes sense since we try to merge two branches with totally
unrelated changes. Our "develop" branch was not even based on
"master", it was just a fully new branch with no ancestor.

So after a few digging on the web I tried to tell to Git that I wanted
to keep changes from develop without bothering about master but merge
into it:

{% highlight console %}
$ git merge -s ours --no-commit master
{% endhighlight %}

No luck with that, it was a misunderstanding of what this command does
so Git tells me:

```
fatal: refusing to merge unrelated histories
```

Ok so what? The first thing you can be tempted to do is to create a
new branch based on the one you want to merge, delete "master" then
rename this branch to "master". That's kind of hacky to me. Let's
find a real good solution to this problem.

## The good path

Git is so powerful that there must be a clean way to do this. And yes
there's one.

Go to the branch you want to become the new "master", then merge
"master" by telling you don't care about unrelated histories and what
you want to keep is the work in the current branch:

{% highlight console %}
$ git checkout develop
$ git merge -s ours --allow-unrelated-histories master
{% endhighlight %}

Then you're good to go, you can checkout "master", merge the new
version and push it:

{% highlight console %}
$ git checkout master
$ git merge develop
$ git push
{% endhighlight %}

That was pretty tough to figure it out so I thought writing about it
would help someone and that it would help the future me when I'll
have forget about it.

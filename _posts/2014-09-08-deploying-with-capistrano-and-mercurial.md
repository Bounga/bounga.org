---
title: Capistrano using Mercurial (a.k.a Hg)
category: Unix
tags: ['sysadmin']
---

As you may notice I'm a big fan of [Mercurial](http://mercurial.selenic.com) which is a great [DSCM](http://en.wikipedia.org/wiki/Distributed_revision_control) a lot more easier to learn and use than [Git](http://git-scm.com) in my opinion.

As a [Rails](http://rubyonrails.org) developer, I like to use [Capistrano](http://capistranorb.com) to deploy my websites.

Most of Rails developers use [Git](http://git-scm.com) because it is the de-facto DSCM to use with Rails since all projects / gems / plugins / whatever use it. The thing is I always used [Mercurial](http://mercurial.selenic.com) and [BitBucket](http://bitbucket.org) because I found Mercurial very easy to use and BitBucket a really nice DSCM hosting solution with unlimited private repos for small teams.

This long introduction leads to a small post that will teach you how to deploy
Rails apps using Capistrano if you want to version your code using Mercurial.

Capistrano is an awesome tool if you know how to use it at its best. All you have to do if you're hosting your code in a Mercurial repository is to set your Capistrano this way:

{% highlight ruby %}
set :scm, :mercurial
{% endhighlight %}

Yes, that all… For sure your deployment server needs to have its key set on your Mercurial server (let's say bitbucket.org) or you may use SSH forwarding.

That said you should really have a look to Mercurial if you never tried it. It's an awesome DSCM and it's really easy to learn. Of course it's not as fast as Git (for Biggggs projects) but it's ways simpler to learn.

---
title: Downgrading RubyMotion
category: tips
tags: ['rubymotion']
---

Sometimes after upgrading [RubyMotion][] I face some bugs introduced in the new release. It can be bugs which can easily be workaround or really annoying bugs that will be pain in the ass in a daily work.

In this case most of the times I choose to downgrade RubyMotion rather than waiting for a new release or patching it locally.

Downgrading [RubyMotion][] consists in fact of creating a new copy of the toolchain in a new directory then use it in projects. Let's say you want to downgrade to [RubyMotion][] 3.7:

{% highlight bash %}
$ sudo motion update --cache-version=3.7
{% endhighlight %}

It will install the toolchain in `/Library/RubyMotion3.7`. It's now possible to use it in a given project by editing the `Rakefile` and changing this line:

{% highlight ruby %}
$:.unshift("/Library/RubyMotion/lib")
{% endhighlight %}

by

{% highlight ruby %}
$:.unshift("/Library/RubyMotion3.7/lib")
{% endhighlight %}

Ok that's nice. We're now able to use the old release and the new one but we have to specify it per project.

But what if your teammates have not upgraded but you commited this change to the `Rakefile`? They won't be able to use the project anymore since they don't have the new directory.

When I really want to downgrade -- not having two versions of [RubyMotion][] available locally -- I install the older version using the command we've seen before then I move it into the usual path:

{% highlight bash %}
$ sudo mv /Library/RubyMotionX.Y /Library/RubyMotion
{% endhighlight %}

There's one gotcha, you'll have to "activate" your license again since your `/Library/RubyMotion/license.key` file has been deleted in the operation:

{% highlight bash %}
$ motion activate your_rubymotion_license_key
{% endhighlight %}

If you really need the new stuff but you encounter a bug you can easily clone a
copy of RubyMotion and patch it locally to fit your needs. Steps to do so are
documented in the [README][].

[RubyMotion]: http://www.rubymotion.com
[README]: https://github.com/HipByte/RubyMotion/blob/master/README.rdoc

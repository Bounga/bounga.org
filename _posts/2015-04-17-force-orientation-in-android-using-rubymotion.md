---
title: Force screen orientation in Android using RubyMotion
category: ruby
tags: ['rubymotion', 'android']
---

As you may know since a couple of month you can write native [Android](https://www.android.com) apps using [RubyMotion](http://www.rubymotion.com).

Last weeks I used it to build an Android app and had the need to lock the screen is portrait mode.

Using the classical Java / XML toolchain to build an app, you have
to add an attribute to your activity in the `AndroidManifest.xml` file:

{% highlight xml %}
<activity android:name="MainActivity" android:label="Awesome App" android:screenOrientation="portrait" >
{% endhighlight %}

When using RubyMotion you don't edit this manifest file by hand, all
configuration stuff are done through the `Rakefile`. Translating this in
RubyMotion is simple:

{% highlight ruby %}
Motion::Project::App.setup do |app|
  # …
  app.manifest.child('application').child('activity').update('android:screenOrientation' => 'portrait')
end
{% endhighlight %}

This is the only thing needed to lock the application in portrait mode. Of course you can lock in `landscape` mode.

Note it's not always a good thing to lock screen orientation in your application. Don't do this to workaround bugs or difficulties. Use it only if
your application isn't meant to be used in both orientations.

---
title: Loading shared library in Android using RubyMotion
category: ruby
tags: ['rubymotion', 'android']
---

# Problem

Using a vendor library may require to load a shared library (a.k.a `.so`) along with it to make it work.

I faced this problem when I was trying to use [Android PDFView](https://github.com/JoanZapata/android-pdfview) which relies on [VuDroid](https://code.google.com/p/vudroid/) to display PDFs in an Android app.

At the beginning I just added the `android-pdfview-1.0.2.jar` file in the `vendor` directory then used it in my `Rakefile`:

{% highlight ruby %}
app.vendor_project jar: 'vendor/android-pdfview-1.0.2.jar'
{% endhighlight %}

It was compiling and all seems to be ok until I got to the screen where I used an Android PDFView. Trying to display a PDF was throwing an exception:

{% highlight text %}
Caused by: java.lang.UnsatisfiedLinkError: Couldn't load vudroid from loader dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.company.awesome_app.apk"],nativeLibraryDirectories=[/data/app-lib/com.company.awesome_app-1, /vendor/lib, /system/lib]]]: findLibrary returned null
{% endhighlight %}

Clearly `vudroid` shared library can't be found.

Android PDFView doesn't bundle vudroid shared library. It's up to you to get it and load it in your app. That's why it's available in [Android PDFView repo](https://github.com/JoanZapata/android-pdfview/tree/master/android-pdfview/libs).

This is the `armeabi` version which is of interest to us. For now this is the only architecture supported by RubyMotion.

# Solution

If you were using the classical Java toolbelt you would just put this file in your lib directory and you would be good to go.

In RubyMotion you can't do that. You have to specify the dependency on the shared library in your `Rakefile`.

Sadly this option isn't mentioned anywhere in the RubyMotion documentation for Android, that's why I'm writing this post and you'll see it's really easy to load a shared library.

Back to your `Rakefile`, you'll have to do this:

{% highlight ruby %}
app.vendor_project jar: 'vendor/android-pdfview-1.0.2.jar', native: ["vendor/armeabi/libvudroid.so"]
{% endhighlight %}

As you can see, I dropped the `.so` file in `vendor/armeabi` directory. The `armeabi` bit is required by RubyMotion. It considers the shared library useable only if it's the armeabi version and relies on its name to check it.

You're now good to go, you can `rake device` and the shared library will be loaded along with the jar. The runtime exception will be gone.

Good PDF viewing ;-)

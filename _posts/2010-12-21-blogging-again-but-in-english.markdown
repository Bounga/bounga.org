---
layout: post
title: Blogging again… in English
category: misc
tags: ['blog', 'jekyll', 'english', 'synbioz']
---

I'm back again to blog about things I like. You know what I'll be talking about? I'm pretty sure you know. Main topics will be about my favorite stuff when it comes to programming: **Mac OS X**, **Ruby**, **Rails**, **MooTools** and **Objective C**. 

Goodbye French. Hello English!
==============================

Most of you, English readers, don't know me because I was blogging in French before. For those of you who were reading me before, I really hope you'll be able and still wish to read me even if I'm writing in English now.

Maybe you're wondering why I stopped blogging in French? Writing in French is cool since it's my mother tongue and because there's not much resources in French. But writing in English has many pros — much more people can read my posts and as a consequence I'll probably have more feedbacks on my thoughts, choices and my open-source projects. IMHO it's a really good reason to switch my blog language, I mean I really love what I do and I like when I have feedbacks (good or bad) on it, ideas to improve my style or my softwares. There's many many more developers speaking English than developers speaking French so that's the deal if I want to have some visibility in communities.

Jekyll powered
==============

As you can see, layout of my blog hasn't changed that much but behind the scene there are many changes.

First of all, this blog don't use [DotClear](http://dotclear.org/) anymore. I'm now generating fast and simple static pages using a site generator called [Jekyll](https://github.com/mojombo/jekyll). This tool uses [Ruby](http://www.ruby-lang.org) so I'm able to extend it easily if needed. Jekyll uses a templating language called [Liquid](http://www.liquidmarkup.org/) that I already know so it's easy for me to build my templates and customize it as much as I need. As I was rewriting my templates I decided to use [HTML 5](http://en.wikipedia.org/wiki/HTML5) to learn it a little bit and to start my new blog with an extensible and promising markup language. I'm also using some CSS3 properties.

All my blog posts will be written using [Markdown](http://en.wikipedia.org/wiki/Markdown) syntax. What I like in it is that it's a very simple and straightforward syntax so you can really concentrate on what you want to write rather than having to think about how your post will look like, what buttons you need to click in your WYSIWYG interface. I just write my blog post in my [text editor of choice](http://macromates.com/) as my thoughts come. I can save it, duplicate it, send it over email and the best thing is that I can control all my blog sources (templates, images, preferences, posts and drafts) using my favorite DSCM [Mercurial](http://mercurial.selenic.com/). This way I can start a blog post on a computer, save it as a draft, push it to my repo, work another day on it on another computer. I can do all the things a DSCM allows. Talking about versioning of my code, this [blog sources](https://bitbucket.org/Bounga/blog/src) (including drafts and posts) can be found on my central repo which is hosted on [Bitbucket](https://bitbucket.org/), the hosting service of choice when you use Mercurial.

I've made some tweaks on Jekyll to do things that are not available by default. The main changes are :

- generation of post category pages  
- generation of post tag pages
- archive page
- posts ATOM feed
- sitemap
  
All the complicated processes are handled by a [Rakefile](http://rake.rubyforge.org/) I wrote. This Rakefile is used to:

- build category pages
- build tag pages
- build the full website
- deploy the website on the hosting server

Comments are handled by [Disqus](http://disqus.com/) which allows to easily include comments on my static website. It's uses javascript to include what is needed in my post pages.

So I've got a full stack to blog like a hacker.

Synbioz hosted
==============

I'm working at [Synbioz](http://www.synbioz.com) since May 2010. They offered me to host my websites for free. I really want to thank [Martin](http://twitter.com/#!/_fuse) for allowing me to put back my websites online using a reliable server.

At Synbioz we're building Web Apps for [various kind of customers](http://www.synbioz.com/realisations). We're using Ruby and all the cool things available in the eco-system (Rails, Sinatra, MacRuby, …) every day.

As you can figure out we're big fans of Ruby and open source in general. Working on our projects is not really a job but it's much more about having fun, discovering new top notch technologies and building evolutive, consistent and robust softwares. If you need some resources for your project we'll be very pleased to hear from you and help you!

Hope you'll like reading me.
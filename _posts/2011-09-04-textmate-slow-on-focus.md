---
title: How to solve Textmate slowdown on focus
category: Tips
tags: ['editor']
---

On Friday I was having slowdowns on [Textmate](http://macromates.com/) focus. I never had this problem before and this "bug" was only happening on this given project. I had to wait for about 5 to 10 seconds to be able to use my text editor. Really annoying when you need to switch beetween your editor and a web browser to test your code hundred times a day.

First thing I did was to Google for "Textmate slow focus" and found a fix to this issue. The solution was to install a Textmate plugin called [ReMate](http://ciaranwal.sh/remate/). ReMate is straightforward way of speeding up Textmate focus by disabling file changes tracking. It works well and Textmate became usable again but it was not a good solution to me.

I'm using terminal to create, remove or change files all day long. External changes wasn't tracked anymore so my Textmate project wasn't in sync with filsystem. I investigate a bit more and found that my textmate was slow on focus 'cause I was having a folder **full of** sub-folders and files. This directory was useless for development.

I'm using [DragonFly](https://github.com/markevans/dragonfly/) which is an engine that ease file uploading in [Rails](http://www.rubyonrails.org). DragonFly creates a lot of sub-directories and files for a single upload, so when you have a lot of uploads you also have thousand of files and directory created on the fly.

I decided to tells Textmate to ignore this upload root directory to see if there's any speed improvement and that was the case. Just by ignoring this directory full of sub-directories and files Textmate becomes fast as light again.

To ignore a given directory in your projects, go to "Preferences" -> "Advanced" -> "Folder References". In folder pattern you can add pattern for directories to be ignore in your projects. In my case  default:

  !.*/(\.[^/]*|CVS|_darcs|_MTN|\{arch\}|blib|.*~\.nib|.*\.(framework|app|pbproj|pbxproj|xcode(proj)?|bundle))$

becomes:

  !.*/(\.[^/]*|CVS|_darcs|_MTN|\{arch\}|blib|.*~\.nib|.*\.(framework|app|pbproj|pbxproj|xcode(proj)?|bundle)|public/system)$

I only had "|public/system" to the end of the regular expression.

So if you have slow focus with textmate be sure to check for large directories that you don't rely on to develop and ignore them!

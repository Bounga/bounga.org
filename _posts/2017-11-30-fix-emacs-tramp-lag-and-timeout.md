---
title: Emacs Tramp lag and timeout
category: Tips
tags: ['emacs', 'ssh']
header:
  overlay_image: /assets/images/headers/tips.jpg
  teaser: /assets/images/teasers/emacs.png
excerpt: "You're experiencing excessive lag and timeouts when using Tramp
on remote servers through SSH. Here is the solution to one of the most
common cause."
---


## Context ##

I use [Emacs][emacs] for pretty much everything . When I need to
navigate the file-system, search for a string in a file or edit it, I
do it within [Emacs][emacs].

Sometimes I need to do this kind of
manipulation on a remote host using SSH. Could I do this using
[Emacs][emacs]? Sure I can!

[Emacs][emacs] is able to handle remote files through SSH. It is a
feature shipped with it called [Tramp][tramp] that allows to access
remote hosts transparently from within [Emacs][emacs] using various
access methods such as SSH, FTP, …

It can do a lot of things, it literally allows to work with remote
hosts as if you were working locally. Pure awesomeness, the power of
my customized Emacs everywhere.

## What happened to me ##

I decided to give it a try as soon as I got the opportunity. What a
disappointment when I saw that opening a remote file was lagging as hell,
lagging so much that it always ended up with a timeout.

I googled for my issue and found out that it was working flawlessly
for a lot of people.

My immediate thought was "F***ing OS X, that must be your fault!". I
googled more focusing on "OS X" keyword and found some people having
the same issue. They were using OS X too.

I did try some of the advices given by people that weren't experiencing
this issue. No luck…

## RTFM ##

After several hours of research I finally landed on
the [Tramp FAQ][tramp_faq] and I saw this:

> tramp needs a clean recognizable prompt on the remote host for
> accurate parsing

I knew it was the source of my issue and felt so lame of not having
read the official documentation sooner. My prompts are always fancy
and full of info even on my remote servers.

I did try to simplify my prompt and [Tramp][tramp] started to work
seamlessly!

## So what's the fix? ##

[Tramp][tramp] expects a really simple prompt on the remote host to
parse it and detect when a command has finished. Basically it wants a
prompt ending with `#`, `$`, `%` or `>`. It's also having hard time to
parse prompts containing escape sequences for coloring.

My remote prompt was using a feature of [ZSH][zsh] which allow to have
a prompt on the right side of the terminal. This was confusing
[Tramp][tramp] because it couldn't find the "magical" character at the
end of the line.

There are two solutions to fix this. 

The first solution is nice if you can't change the remote prompt. You
can change a variable in your [Emacs][emacs] config that describes the
regexp used by [Tramp][tramp] to recognize the prompt. This variable
is named `tramp-shell-prompt-pattern`.

The second solution — the one I chose — is even simpler in my
opinion. Just change the prompt on the remote host. You can add a
condition in your `.zshrc` (or whatever the init file of your shell
is) and do something like this:

{% highlight sh %}
[ $TERM = "dumb" ] && unsetopt zle && PS1='$ '
{% endhighlight %}

When [Tramp][tramp] connects to a remote host it sets the `TERM`
environment variable as `dumb` so you know you're in the situation
were you want a really simple prompt. What to do then is to set the
prompt to a simple "$ " string.

This way, if I connect on my remote using a real terminal I still have
my fancy prompt with all my info, colors, right prompt and everything.

## Moral of this story ##

Adding a single line in my `.zshrc` file on my remote server was
enough to solve the problem I was experiencing for months and that
[some other users are experiencing for many
years][emacs_stackexchange] without a single answer.

I can't tell you enough to always check the official documentation and
issues in the tracker of a software / lib / whatever when you're
experiencing an issue. It often gives an answer to your problem and it
will avoid some hours lost in reading half of the web.

Hope it helps if you're digging for the same issue and you're left
without answers until now.

[emacs]: https://www.gnu.org/software/emacs/
[tramp]: https://www.gnu.org/software/tramp/#Overview
[tramp_faq]: https://www.gnu.org/software/emacs/manual/html_node/tramp/Frequently-Asked-Questions.html
[emacs_stackexchange]: https://emacs.stackexchange.com/questions/16489/tramp-is-unbearably-slow-osx-ssh/37177#37177
[zsh]: http://zsh.sourceforge.net

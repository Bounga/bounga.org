---
title: Set system-wide PATH environment variable for Mac OS GUI apps
category: Tips
tags: ['tips', 'Mac OS']
header:
    overlay_image: /assets/images/headers/tips.jpg
excerpt: "A lot of people struggles to find how to instruct their Mac
    OS GUI apps to be aware of their PATH environment variable.
    Sometimes when you install GUI apps they try to use a binary which
    is installed on your system but for some reason it can't be found.
    Let's fix that."
---

This particular problem got me struggled for a while. The first app I
installed and wasn't aware of all my binaries was
[Emacs](https://www.gnu.org/software/emacs/).

For some reason it was aware of everything in `/usr/bin` but not
`/usr/local/bin`. For quite a while I thought it was something related
to Emacs.

Then I installed [Qute Browser](https://qutebrowser.org) and once
again, some binaries available on my system were not in the path when
searched from within Qute.

I did some research but didn't find anything useful. At some point I
figured out that if I was starting the GUI app (whether it was Emacs,
Qute Browser, MacVim, …) from the terminal, everything was ok. All the
binaries on my system could be found.

It got me curious. Why such a different behavior when I launched the
app by clicking its icon in the <kbd>Applications</kbd> folder or by
using Spotlight compared to when I launched it using my term?

Oh boy, this one was a very long road to the true knowledge, the real
mastering of Mac OS internals!

The first step to this full understanding was asking myself why my GUI
apps would know about the `PATH` environment variable I set in my
shell? It could be set in `~.profile`, `~/.bashrc`, `~/.bash_profile`,
`~/.zsh_profile`, `~/.zshrc`, `~/.config/fish/config.fish`, you named
it.

It doesn't makes any sense unless if you start the app from the shell
knowing the full blown `PATH`.

Being confident thanks to this discover I investigated to understand
where the GUI apps got their paths from and after some deep diving I
got the answer.

Every single app that is started by clicking its icon or through
spotlight gets its `PATH` from whatever is set by `launchd` daemon.

Now we know this crucial info there's one simple step left to
customize the `PATH` GUI apps inherit from. It's as simple as editing
`/etc/launchd.conf` like so:

```sh
setenv PATH /usr/local/bin:/usr/local/sbin:/usr/bin:/bin:/usr/sbin:/sbin
```

Now rather than scanning for binaries only in `/usr/bin` and
`/usr/sbin`, GUI apps are going to search for binaries in all these
paths: `/usr/local/bin` `/usr/local/sbin` `/usr/bin` `/bin`
`/usr/sbin` `/sbin`.

If you want your changes to be effective without having to reboot your
computer -- I hate rebooting my computer -- you'll have to follow some
more steps.

This steps will help `launchd` and Spotlight to be aware of the new
settings:

```sh
$ egrep "^setenv\ " /etc/launchd.conf | xargs -t -L 1 launchctl
```

This one greps all environment related settings and forward them to
`launchd`.

Then you have to restart your `Dock` and `Spotlight` apps:

```sh
$ killall Dock
$ killall Spotlight
```
Now every GUI app you launch inherits of this PATH env variable, either
you launch it by clicking the icon in Applications folder or using
Spotlight!

Enjoy!

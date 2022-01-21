---
title: Setting up multiple IMAP and SMTP accounts in Gnus
category: Tips
tags: ["Emacs", "gnus"]
header:
  overlay_image: /assets/images/headers/tips.jpg
  teaser: /assets/images/teasers/multiple_accounts_in_gnus.jpg
excerpt: “When it comes to Emacs and emails reading, one of the most
  popular options is Gnus. Let's see how to configure it for
  multiple IMAP / SMTP servers without using any external tool.”
---

Emacs' users tend to like to do everything in it as I do. A frequent
desire is to be able to read emails in it. A popular option is
[Gnus](https://gnus.org).

Gnus is a beast it can do a lot of things from newsgroups, to RSS to
emails. So configuring and using it can be a bit scary at first.

There's a really widespread misinformation about Gnus. I thought for long
it was true but it comes out it wasn't.

This is about using multiple SMTP servers seamlessly for sending
emails. I always read you have to use external tools such as
[MSMTP](https://marlam.de/msmtp/) with specific configuration to make
it acts as a proxy between your software and your multiple remote
SMTP. Some other advice you to use a homemade function to play with
message fields and hook it via `message-send-hook`.

Some will even tell you to hack your `/etc/hosts`.

But hey, Gnus is there since the nineties. Someone must have think of
a solution and it must be built-in!

I stopped searching blog posts and Stack Overflow and read the [manual
entry](https://www.gnu.org/software/emacs/manual/html_node/message/Mail-Variables.html)
which clearly states that you **can** set up a complex workflow using
multiple SMTP servers.

This works by using [posting
styles](https://www.gnu.org/software/emacs/manual/html_node/gnus/Posting-Styles.html)
which is a way to instruct Gnus of how you want to prepare your email
(headers, body, signature) according to the context.

Before digging into it I'd like to explain my IMAP settings because
it's tightly related to how we're going to setup SMTP.

So let's take a look at the “select-method” definitions. This is where
we tell Gnus about our IMAP servers, their local name and how they
must behave:

```elisp
(setq gnus-select-method '(nnnil nil))
(setq gnus-secondary-select-methods
      '((nnimap "home"
                (nnimap-address "imap.gmail.com")
                (nnimap-server-port "imaps")
                (nnimap-stream ssl)
                (nnir-search-engine imap)
                (nnmail-expiry-target "nnimap+home:[Gmail]/Trash")
                (nnmail-expiry-wait 'immediate))
        (nnimap "work"
                (nnimap-address "imap.gmail.com")
                (nnimap-server-port "imaps")
                (nnimap-stream ssl)
                (nnir-search-engine imap)
                (nnmail-expiry-target "nnimap+work:[Gmail]/Trash")
                (nnmail-expiry-wait 'immediate))))
```

We set `gnus-select-method` to `nnnil` which is a NOOP back-end. I
prefer to set all the accounts in the same place
`gnus-secondary-select-methods`.

In this variable I have declared two IMAP servers. The first one will
be known locally as `home` and the second one as `work`.

Both are using Gmail, so we have to find a way to distinguish these to
account to provide credentials.

The standard Unix way to share credentials across software is to
store them in `~/.authinfo` file. In my case I use `~/.authinfo.gpg`
so my credential are encrypted with GPG and no one but me can read it.

Here is the content of `~/.authinfo.gpg`:

```config
machine home login home@gmail.com password my_pasword port imaps
machine work login work@gmail.com password my_other_password port imaps
```

Now Gnus can read this file to get the credentials and log on the IMAP
servers. Gnus know how to bind a given credential to a specific
account because their share the same name `home` and `work`.

Ok from now on, we can get emails from Gnus through these two email
accounts.

Now it's time to configure these accounts to send emails using their
respective SMTP server / credential.

Just for the sake of clarity, I do use the same IMAP / SMTP addresses
for both accounts in my situation but this technique would work the
same way with two accounts on two different email providers.

All the magic happens by taking advantage of [Gnus posting
styles](https://www.gnu.org/software/emacs/manual/html_node/gnus/Posting-Styles.html):

```elisp
;; Reply to mails with matching email address
(setq gnus-posting-styles
      '((".*" ; Matches all groups of messages
         (address "Nicolas Cavigneaux <home@gmail.com>"))
        ("work" ; Matches Gnus group called "work"
         (address "Nicolas Cavigneaux <work@gmail.com>")
         (organization "Corp")
         (signature-file "~/.signature-work")
         ("X-Message-SMTP-Method" "smtp smtp.gmail.com 587 work@gmail.com"))))
```

The first line with the `.*` is kind of catch-all rule which tells
Gnus that no matter what is the group I'm in, my sender email address
is going to be `home@gmail.com`.

For those who are not familiars with Gnus, a group is just an IMAP
folder.

Then the second rule tells Gnus, if the current group matches anything
with `work` in it then I want to handle my outgoing emails
differently. My sender address is going to be `work@gmail.com`, my
`organization` header is going to be `Corp`, my automatically inserted
signature at the bottom of the email is going to be read from
`~/.signature-work` file **and here happens the magic** we use a
special header that Gnus and `message-mode` understand called
`X-Message-SMTP-Method` that was designed for this exact purpose,
being able to specify an alternative SMTP server to use. So we specify
that we want to use `smtp` protocol using the address `smtp.gmail.com`
on port `587` and that the user account to use is `work@gmail.com`.

There's one last thing to setup and you'll be good to go. You need to
provide your SMTP credentials. Once again it takes place in the
`~/.authinfo` file:

```config
machine smtp.gmail.com login home@gmail.com password my_password port 587
machine smtp.gmail.com login work@gmail.com password my_other_password port 587
```

By searching the server name / username Gnus will be able to know
the right credential to use.

It basically enables multi SMTP accounts in Gnus without bothering with
[any](https://www.emacswiki.org/emacs/MultipleSMTPAccounts) of [these](https://www.emacswiki.org/emacs/MultipleSMTPAccounts) [techniques](https://www.emacswiki.org/emacs/GnusMSMTP).

Moral of the story, when it comes to Emacs you should always read the
official doc first since most of the time you'll find the info you
need.

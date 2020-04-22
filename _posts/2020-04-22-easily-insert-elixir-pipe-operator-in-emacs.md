---
title: Easily insert Elixir pipe operator in Emacs
category: Emacs
tags: ['elixir', 'emacs']
header:
    overlay_image: /assets/images/headers/emacs.jpg
    teaser: /assets/images/teasers/emacs.jpg
excerpt: "When writing [Elixir](https://elixir-lang.org/) code you'll
often find yourself typing the pipe operator which is not very
convinient. Let's see how to ease this in
[Emacs](https://www.gnu.org/software/emacs/)."
---

I'm using [Elixir](https://elixir-lang.org/) more and more lately and
I love it!

Elixir is a functional language and as such it's very common to feed a
function with the return of another one like so:

```elixir
length(String.split(line))
```

It can quickly become are to read so Elixir provides a syntactic sugar
to pipe a function return to another one:

```elixir
line
|> String.split()
|> length()
```

Easier on the eyes isn't it?

I like this syntactic sugar a lot but I **hate** to type it. It's not
an easy one on [azerty](https://en.wikipedia.org/wiki/AZERTY) or
[bépo](https://en.wikipedia.org/wiki/BÉPO) layouts.

As an [Emacs](https://www.gnu.org/software/emacs/) user I can finely
customize everything so I decided to pimp the
[elixir-mode](https://github.com/elixir-editors/emacs-elixir) to make
the pipe operator easy to use.

# Main function #

First of all we need a function that will describe what we want to do:

```elisp
(defun bounga/insert-elixir-pipe-operator ()
  "Insert a newline and the |> operator"
  (interactive)
  (end-of-line)
  (newline-and-indent)
  (insert "|> "))
```

We defined the function `bounga/insert-elixir-pipe-operator` that:

- acts interactively
- goes to the end of the current line
- adds a newline and indents the cursor
- inserts the pipe operator followed by a white space

If you try it by calling `M-x bounga/insert-elixir-pipe-operator` in a
buffer, you'll see it does what we want.

# Bind the function to a key chord #

To use this new function effectively you should bind it to a key chord
(a keyboard shortcut).

We'll bind the function to `M-RET` (Alt + Enter on most keyboards).

Depending on the way you handle your packages there's two ways of
setting it up.

## Vanilla Emacs ##

```elisp
(define-key elixir-mode-map (kbd "M-RET") 'bounga/insert-elixir-pipe-operator)
```

## use-package ##

```elisp
(use-package elixir-mode
  :bind (:map elixir-mode-map
              ("M-RET" . bounga/insert-elixir-pipe-operator)))
```

Now you're good to go!

If you're in an Elixir buffer then `M-RET` will insert a new line and
add the pipe operator whether you're at the end of the line, at the
beginning or in the middle of it.

# Demo #

Let's see it in action:

![emacs_elixir_pipe_operator_demo.gif](/assets/images/emacs_elixir_pipe_operator_demo.gif)

Have fun with Emacs and Elixir!

---
title: Using I18n with Hanami
category: Ruby
tags: ['ruby', 'hanami', 'i18n']
---

Last week I wrote about settings up [Sikekiq](http://sidekiq.org/)
in [Hanami](http://hanamirb.org/). Another gem I often use
is [I18n](https://github.com/svenfuchs/i18n) since it allows to localize
strings and keep them separated from my code by using a dedicated file. This way my
code don't have any hard-coded string but rather uses something that looks like
a keys / values store.

## Setup ##

You first should add `I18n` dependency explicitly to your `Gemfile` even if it's
already a dependency of `hanami-validation` because of `dry-validation` which is
used underneath. So add this to your `Gemfile`:

{% highlight ruby %}
gem 'i18n'
{% endhighlight %}

`I18n` needs some setup to know where to find translations and the default
locale to use. You can set this in an initializer file so it will be loaded on
server start, let's create `config/initializers/locale.rb`:

{% highlight ruby %}
require 'i18n'

I18n.load_path = Dir[Hanami.root.join("config/locales/**/*.{yml,rb}")]
I18n.default_locale = 'en'
I18n.backend.load_translations
{% endhighlight %}

Initializers in `config/initializers` are not auto-loaded so you need to
`require` it explicitly in `config/environment.rb`:

{% highlight ruby %}
require_relative './initializers/locale.rb'
{% endhighlight %}

With this simple setup you're ready to go with Hanami-wide translations.

You can create translations files in `config/locales` sub-directories, for
example English translations would go in `config/locales/en/app.yml`.

You could go further by adding the ability to create application specific
translations. To do this, you need to instruct each application of it by adding
some code in the `configure` block of its `application.rb` file:

{% highlight ruby %}
configure do
  I18n.load_path << Dir[root.join("config/locales/**/*.{yml,rb}")]
  I18n.backend.load_translations
end
{% endhighlight %}

Now your applications can host their own translations so you won't pollute global
translation files with application specific stuff.

## Using translations ##

Let's see how you can use this translations in your code. It's as simple as using
`I18n.t` method.

For example in a view you can do:

{% highlight erb %}
<%= I18n.t('title') %>
{% endhighlight %}

and it will search for a key like `en.title` where `en` is the current locale.

You could also scope your call to a given namespace:

{% highlight erb %}
<%= I18n.t('title', scope: 'users.show') %>
{% endhighlight %}

## Easier soon ##

According to this [Github issue](https://github.com/hanami/hanami/issues/610),
I18n support should be shipped along with Hanami starting from v0.9.0. We can
hope that it will be nicely integrated and that it'll provide useful helpers out
of the box.

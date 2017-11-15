---
title: Translating ActiveRecord data using JSON
category: Ruby
tags: ['rails', 'i18n']
header:
  overlay_image: /assets/images/headers/ruby.png
  overlay_color: "#4D4D4D"
excerpt: "Following the mass and using de-facto libraries isn't always
the best choice. Here are some thoughts about handling translation of
data stored in database when using Rails."
---

In [Rails][rails] world, when it comes to translating
data stored in database, most people use
[Globalize][globalize] which defines
itself as the "Rails I18n de-facto standard library for ActiveRecord
model/data translation".

I used it many times and let be honest, I don't like it, at all. It
isn't a bad or buggy gem but its inner design doesn't fit my needs and
expectations.

## How Globalize works ##

[Globalize][globalize] relies on having a table dedicated to store
translations for a model attributes. So you will have to add one table
per model that needs translations.

When you retrieve an object with translated attributes through
ActiveRecord, [Globalize][globalize] creates a SQL query which
retrieves the actual data of the object and does a join on the related
translation table to get according translations.

It can become quickly pretty heavy for the database and results in
slow requests. I often found myself having big performance issues just
because of this automatic joins when I was retrieving an ActiveRecord
object and its relationships.

The main advantage of this solution — and I think that's why
[Globalize][globalize] team did it this way — is that it works whatever the
database you use. It uses standard SQL only.

## Toward a better solution ##

In my day to day work with Rails, I mostly use [PostgreSQL][pg] and
sometimes [MySQL][mysql]. I don't have the need to be compatible with
all the databases available out there. With that in mind I searched
for another solution to store the translations, one that would be more
efficient for my use cases.

I wanted to leverage some specific features of [PostgreSQL][pg] and
[MySQL][mysql] to avoid the need of other tables.

The first thing that came to my mind was to use
[hstore](https://www.postgresql.org/docs/current/static/hstore.html)
to store the translations. That is a nice solution and there's already
some gems available to do that but it works with PostgreSQL only.

I searched for another solution and found out that I could use JSON
datatype which is available for both MySQL and PostgreSQL.

This is a huge improvement over [Globalize][globalize] since
translations can be stored in the same table. [Rails][rails]
introduced JSON datatype support in 4.2. It's as easy as manipulating
a plain old hash. [Rails][rails] automatically does the conversion job
for you.

No more *join*s needed, no more separated translation tables!

That sounds pretty good, it feels like a more natural and powerful
solution to me. We still have the same features and flexibility that
provides Globalize.

## Example of using JSON to store translations ##

### Adding translation columns to the table ###

First, for each translatable attribute of our model we need to create
a JSON column:

{% highlight ruby %}
class CreateNews < ActiveRecord::Migration[5.1]
  def change
    create_table :news do |t|
      t.column :title, 'jsonb'
      t.column :body,  'jsonb'
      t.timestamps
    end
  end
end
{% endhighlight %}

Nothing fancy here. Note that [PostgreSQL][pg] uses `jsonb` columns
type where [MySQL][mysql] uses `json`.

### Using the newly added JSON attributes ###

Now that we've created our `News` model and the according migration,
we can use it to store our translations:

{% highlight ruby %}
news = News.create(
  title: { en: "Latest news", fr: "Dernière nouvelle" },
  body: { en: "something", fr: "quelque chose" }
)

# => #<News id: 3, title: {"en"=>"Latest news", "fr"=>"Dernière nouvelle"}, body: {"en"=>"something", "fr"=>"quelque chose"}, created_at: "2017-11-15 19:15:01", updated_at: "2017-11-15 19:15:01">

news.title[I18n.locale] = "Edited title" # set new title for current locale
news.body[I18n.locale] # body content for current locale
{% endhighlight %}

Easy!

### Improve translatable attributes interface ###

You must think "Dude that's a pretty roots solution…". Yes but this is
a proof-of-concept. We now know that it works and we can build a
better interface based on this.

Sure you'll want something fancier, for instance you would want to
have a method that returns the data for the current locale without
having to pass it explicitly.

You'll also want to return the data for the default locale if there's
no translation for the given locale.

A minimal implementation could look like this:

{% highlight ruby %}
module Translate
  def translates(*attrs)
    attrs.each do |attr_name|
      define_method(attr_name) do |locale = I18n.locale|
        hash = read_attribute(attr_name)
        hash[locale.to_s] || hash[I18n.default_locale.to_s]
      end
      
      define_method("#{attr_name}=") do |value|
        hash = read_attribute(attr_name) || {}
        hash[I18n.default_locale.to_s] = value
        write_attribute(attr_name, hash)
        
        value
      end
    end
  end
end

ActiveRecord::Base.extend(Translate)
{% endhighlight %}

Then we use it in our models:

{% highlight ruby %}
class News < ActiveRecord::Base
  translates :title, :body
end

news = News.last

news.title     # value of title for the current locale
news.body(:fr) # value of body for the fr locale

# Both fallback on default locale if requested locale has no translation

news.body = "Edited body" # set body for the current locale
{% endhighlight %}

With very little amount of code we already have a much better
interface to use our translation system!

You could also want to be able to find a record based on its
translated content without having to build the JSON by hand.

## There's already a Gem to keep your hands clean ##

I'm not the first one to think of this solution and someone has
already wrote a gem that package all this with a nice interface.

It's a french developer called [Cédric Fabianski][cfabianski] who
wrote [JSON Translate][json_translate] and it does everything I
explained above and more.

You really should give it a try.

## Don't follow, think by yourself ##

The main message behind this blog post is that you should not follow
the mass blindly and always think twice when you're choosing a library
you're going to depend on in your application.

Adding a dependency in your app can lead to huge impact on performance
and on how you'll design it. Always try to think about available
alternatives and the real advantages of using a potentially heavy
library over your own light implementation.

[rails]: http://rubyonrails.org
[globalize]: https://github.com/globalize/globalize
[pg]: https://www.postgresql.org
[mysql]: https://www.mysql.com/
[cfabianski]: https://twitter.com/cfabianski
[json_translate]: https://github.com/cfabianski/json_translate

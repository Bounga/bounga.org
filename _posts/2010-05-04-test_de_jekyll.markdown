---
layout: post
title: Testing Jekyll
category: rails
tags: ['test', 'blah']
---
youhou !

Some Ruby code :

{% highlight ruby %}
class Category < ActiveRecord::Base
  acts_as_list :scope => 'parent_id'
  acts_as_tree

  default_scope :order => 'parent_id, position'

  validates_presence_of :name

  has_many :associations, :order => 'position'
  has_many :products, :through => :associations, :dependent => :destroy


  translates :name

  def to_param
    if self.name.blank?
      self.id.to_s
    else
      "#{self.id}-#{self.name.parameterize}"
    end
  end
end
{% endhighlight %}
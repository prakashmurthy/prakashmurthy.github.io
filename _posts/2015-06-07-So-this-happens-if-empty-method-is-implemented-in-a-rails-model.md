---
layout: post
title:  "So this happens if `empty?` method is implemented in a rails model"
date:   2015-03-22 21:18:41
categories: 
---

Came up with an interesting issue related to Active Record `empty?` method while implementing shopping cart functionality today morning. 

I have the following models to start with:

```
class Cart
  has_many :line_items
  ...
end

class LineItem
  belongs_to :cart
  validates :cart, presence: true
  ...
end

```

With these classes in place, I have some working tests that validate the basic shopping cart functionality. 

To conditionally display the `cart summary view` in a page, I wanted a method that tells me if there are any items in the cart. So I added the following method:

```
class Cart
  ...
  
  def empty?
    !line_items.exists?
  end
end
```

Now the fun started as the code for creating `line_items` failed with validation errors, saying `cart cannot be empty`. I was stumped for a while wondering why this code breaks most of the tests with this validation error. Adding to the confusion was the fact that when I did not have this method defined, the unit test I was using for test driving this functionality was failing with a `There is no 'empty?' method defined on 'Cart' class` message. 

Googling showed the rails api has a [`ActiveRecord::Relation.empty?` method](http://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-empty-3F) that seemed likely to be related to this behavior change. Turns out I had over-written this method. Previously, `validates :cart, presence: true` was checking if there was an associated `cart` for every `line_item`. Now with the `empty?` method implemented in `Cart`, the validation logic had changed to check for a `cart` having atleast one associated `line_item`. 

Seems pretty obvious in hind-sight, and made me feel a bit stupid at the end of it :-). Instead of renaming the method, I decided to use `cart.line_items.empty?` or `cart.line_items.exists?` for my conditional logic in the view. And also decided to take a break from coding today.

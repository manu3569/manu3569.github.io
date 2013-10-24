---
layout: post
title: "Enumerable's Best Kept Secrets"
date: 2013-10-07 18:41
comments: true
categories: Ruby
---


> An `Enumerator` is defined as a "class which allows both internal and external iteration." It represent a series of objects, it can be lazily generated and it can be infinite. In contrast, the an `Enumerable` is a module and it "provides collection classes with several traversal and searching methods, and with the ability to sort."

<br/>

In Ruby, enumerators are used quite frequently for their extensive abstraction of iteration functionality. When we call the `Array#each` method, we can focus on dealing with the individual `Array` items rather than having to worry about how to retrieve each item one by one. When we need to find a subset of items from a `Hash`, we can use `#select` with a code block to decide which item we want in the result, without necessarily knowing how the `Hash` is traversed.

<br/>

All this power comes from the `Enumerable` module, and it is inlucded by many of the common Object types such as `Array`, `Hash`, `Range`, and `Set`. When first learning Ruby, `#each` and `#collect` quickly become an often-used tool we use for iterating over data. Most functionality can be accomplished by their use, however, there are many more advanced iterators available with very specific funtionality, but unless we constantly skim over the Ruby documentation, we may not know about them. I'd like to introduce (or re-introduce for some of you) some of my favorites.

<br/>


## #count

Alright, alright. You may use `#count` on a daily basis, but did you know that you can specify a parameter to the method call? It will return the number of occurrances of that particular value.

``` ruby
fruit = %w(apple orange banana apple kiwi grapes apple)

fruit.count("apple") #=> 3
```

If you though that was cool, then hold on to your pants. When you add a block that evaluates a condition for each item in the collection, you can add logic regarding which items you want to count. If the block return `true` the item gets counted.

``` ruby
fruit = %w(apple orange banana apple kiwi grapes apple)
fruit.count { |item| %w(banana kiwi).include? item } #=> 2
```

<br/>

## #lazy

We all like to be lazy sometimes. Starting in Ruby 2.0, we can now let `Enumerable` objects be lazy too. All this means is that the result of the enumerable function will not get evaluated until you tell it to.

``` ruby
car_names = %w(audi chevrolet dodge)

capitalized_car_names = car_names.lazy.map &:capitalize
 => #<Enumerator::Lazy: #<Enumerator::Lazy: ["audi", "chevrolet", "dodge"]>:map>

capitalized_car_names.to_a
 => ["Audi", "Chevrolet", "Dodge"] 
```

The behavior in this example doesn't really show the full potential of `#lazy`. Imagine that you have an operation that you want to apply to a collection, however, the contents of the given collection are dynamically changing. Using `#lazy` keeps the enumerator updated with the latest state of the collection.


``` ruby

car_names << 'volkswagen'

capitalized_car_names
 => #<Enumerator::Lazy: #<Enumerator::Lazy: ["audi", "chevrolet", "dodge", "volkswagen"]>:map> 

capitalized_car_names.to_a
 => ["Audi", "Chevrolet", "Dodge", "Volkswagen"] 
 
car_names.shift
 => "audi" 

capitalized_car_names.to_a
 => ["Chevrolet", "Dodge", "Volkswagen"] 
```
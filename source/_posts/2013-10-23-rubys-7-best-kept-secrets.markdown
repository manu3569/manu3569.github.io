---
layout: post
title: "Ruby's 7 Best Kept Secrets"
date: 2013-10-23 22:50
comments: true
categories: 
---


### 1. Syntactical Shortcuts for Iterations

When you need to apply a method to all items within an `Enumerable`, instead of creating a block syntax with a local variable, you can prepend an `&` before the `Symbol` that represents the method name and pass it to the iteration method as an argument. This works for all functions that do not take an argument.

``` ruby
%w(alice bob carl).map {|name| name.capitalize}
 => ["Alice", "Bob", "Carl"] 

%w(alice bob carl).map &:capitalize
 => ["Alice", "Bob", "Carl"] 
```

These shortcuts work because calling `Symbol#to_proc` returns a proc that responds to the symbol's method. So the `:capitalize` symbol is first converted to a proc, and then to a block.

Similarly, you can pass the `Symbol` for a function into `#inject` and it will sequentially apply the method to items. This method needs to take one argument, which in this case will be `num` and it gets sent to the collector `sum`.

``` ruby
(1..10).inject {|sum,num| sum += num }
 => 55 

(1..10).inject :+
 => 55
```

<br/>
<br/>
### 2. Set Intersection for an Array

There's an easy way to get the intersection, or set of common elements, of two `Array`s. By applying the `&` operator to one `Array` and passing the other as an argument.

``` ruby
[1,2,3,4] & [1,3]
 => [1, 3] 

%w(Hi, my name is Alice) & %w(My name is Bob)
 => ["name", "is"]   # The comparison is case sensitive

class Dog
  def ==(other_dog)
    self.name == other_dog.name
  end
end

shared_dog = Dog.new('Fido')

[Dog.new('Scooby'), Dog.new('Snoopy'), shared_dog] & [Dog.new('Scooby'), shared_dog]
 => [#<Dog:0x007fd96c0f4658 @name="Fido">]
  # shared_dog is considered a match because the object IDs match. 
  # The Dog#== method is not used
```

<br/>
<br/>
### 3. Ignoring Arguments and Return Values

In cases where argument variables need to be set in order to receive values, you can end up with verbose code becaues unused arguments are specified. By using the `_` (underscore) as an argument, it visually hides some of the received values. Also, when defining `_` multiple times it will not raise a `SyntaxError` unlike any other variable name.

``` ruby 
[["Alice", 24, "female"], ["Bob", 28, "male"], ["Trudy", 100, "alien"]].map {|ignore,ignore,gender| gender }

SyntaxError: (irb):81: duplicated argument name
... "alien"]].map {|ignore,ignore,gender| gender }

[["Alice", 24, "female"], ["Bob", 28, "male"], ["Trudy", 100, "alien"]].map {|_,_,gender| gender }
 => ["female", "male", "alien"]
```

``` ruby
name, _, _, hometown = ["Alice", 24, "female", "New York City"]
name
 => "Alice" 
hometown
 => "New York City" 
```

<br/>
<br/>
### 4. Autovivification

`Hash.new` can take a block that is run when a non-existing key is accessed. The code snipped below ensures that a `Hash` with infinite sub-Hash nesting gets created when accessing an arbitrary depth of non-existing keys.

``` ruby
endless = Hash.new{|hash, key| hash[key] = Hash.new(&hash.default_proc) }

endless[:one][:two][:three][:or][:more][:levels][:deep] = "This is insane!!!"
 => "This is insane!!!" 
```

<br/>
<br/>
### 5. Advanced Text Interpolation

When interpolating variable values into a string, the variables are commonly placed within the delimited text. The `%` allows us to replace the character string `%s` within the text with items from an array.

``` ruby
"Hi, my name is %s, and I live in %s." % ["Alice","New York"]
 => "Hi, my name is Alice, and I live in New York."
 
phone_template = "I'm %s and my phone number is %s."
phone_template % ["Jenny","867-5309"]
```

<br/>
<br/>
### 6. Lazy Loading

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

<br/>
<br/>
### 7. Counting A Subset

You may use `#count` on a daily basis, but did you know that you can specify a parameter to the method call? It will return the number of occurrances of that particular value.

``` ruby
fruit = %w(apple orange banana apple kiwi grapes apple)

fruit.count("apple") #=> 3
```

If you though that was cool, then hold on to your pants. When you add a block that evaluates a condition for each item in the collection, you can add logic regarding which items you want to count. If the block return `true` the item gets counted.

``` ruby
fruit = %w(apple orange banana apple kiwi grapes apple)
fruit.count { |item| %w(banana kiwi).include? item } #=> 2
```

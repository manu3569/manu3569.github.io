---
layout: post
title: "What the Splat?"
date: 2013-10-08 22:12
comments: true
categories: Ruby
---

The asterisk operator commonly known as splat can be a powerful ally in writing clean and highly functional code.  The splat operator has a variety of behaviors depending on where it is used, and in this post I'll focus on the use of splat in conjunction with methods in Ruby 1.9 and later.

<br/>

### Arbitrary Argument Length in Method Signature

Occasionally, we would like to define a method that will accept an undefined number of argument.  Typically, we use an optional `Hash` as the last argument, which holds a variety of arguments.  However, we can also take advantage of the splat operator by placing it in front of an argument variable name in the method definition.  It will cause the argument variable to capture an `Array` of all arguments passed to the function that have not been matched to other arguments in the method signature.  The splat operator can be used at the beginning or at the end of the argument list, though for the latter, the last argument that is passed into the function must be a `Hash`.

``` ruby
def add_to(playlist, *songs)
  songs.each do |song|
    playlist << song
  end
end

rock = []
add_to(rock, "Beatles - Yellow Submarine", "Counting Crows - Big Yellow Taxi", "Coldplay - Yellow")

rock.inspect
 #=> "["Beatles - Yellow Submarine", "Counting Crows - Big Yellow Taxi", "Coldplay - Yellow"]" 
```

``` ruby
def display_songs(*songs, formatting)
  songs.each do |song|
    puts "#{formatting[:start]} #{song} #{formatting[:end]}"
  end
end

display_songs("Stairway to Heaven", "Hotel California", "Hold the Line", {:start => ">>>", :end => "<<<"})
 # >>> Stairway to Heaven <<<
 # >>> Hotel California <<<
 # >>> Hold the Line <<<
```

There is one more &mdash;for the lack of a better word&mdash; *interesting* use of the splat operator in the method signature at the end when it's by itself.  It allows for additional arguments to be accepted, but ignored by the method.

``` ruby
def my_favorite_food(first,second,*)
  puts "My favorite food items are #{first} and #{second}."
end

my_favorite_food("pizza","sushi","salad") 
 #=> My favorite food items are pizza and sushi.
```

<br/>
<br/>

### Calling Methods

The reason for this blog entry is actually this section.  I recently worked on a project in which I wanted to use `Object#send` to dynamically call an instance method. The method `#play` takes an argument, while `#list` doesn't. I quickly ran into a problem with this particular code:

``` ruby
def play(song)
  # code to play given song
end

def list
  # code to display all songs
end

def process(input)
  command, args = input.split(" ", 2)
  puts args.inspect
  self.send(command, args)
end

process("play Michael Jackson - Beat It")
 #=> nil 

process("list")
 #ArgumentError: wrong number of arguments (1 for 0)
 # from (irb):32:in `list'
 # from (irb):38:in `process'
```
When processing the input "list", `args` is set to `nil`, which is still considered an argument and it triggers an `ArgumentError` because `#list` does not take any arguments.

Our little friend splat can actually help us resolve this issue.  When calling a method and prepending the splat to an array variable, it will cause it to be distributed across the arguments of the method.  In my case, the empty `args` variable was distributed across exactly no arguments.  By adding the splat, we can now dynamically send either one or two arguments to the `#send` function.

```ruby
def process(input)
  command, args = input.split(" ", 2)
  puts args.class
  self.send(command, *args)
end

process("play Michael Jackson - Beat It")
 #=> nil 

process("list")
 #=> nil
```

When using splat to distribute the values to the arguments, the count must match exactly.  Also, only a splat in front of an empty `Array`, `Hash` or `nil` will yield no value to argument assignment.

``` ruby

def zero
  puts "Nothing"
end

def one(a)
  puts "#{a}"
end

def two(a,b)
  puts "#{a}, #{b}"
end

def three(a,b,c)
  puts "#{a}, #{b}, #{c}"
end

values = [1, 2] 

zero(*values)  # Error
one(*values)   # Error
two(*values)   # Success
three(*values) # Error

zero(*[])      # Success
zero(*{})      # Success
zero(*nil)     # Success
```




---
title: Ruby crash course
featured: /assets/images/education.jpg
layout: post
---

I recently had the pleasure of doing a workshop for my non-ruby programmer colleagues.
For unknown reasons I decided to write the crash-course from scratch in slide form, and now that the workshop is over I felt like persisting that work somewhere. So here goes: A ruby syntax crash course for programmers!

<!--more-->

{% highlight ruby %}
# This is a comment

# Variable assignment
foo = 1

# Strings can be expressed with double-quotes
foo = "I have 10 apples"

# Or single-quotes
bar = 'I eat 2 apples'

# Double-quoted strings can be interpolated
message = "I now have #{10-2} apples" # "I now have 8 apples"

# We use single-quoted strings when we don't need interpolation

# Basic arithmetics follow basic algebraic priority
2 + 8 / 2 + 2 # 8
(2 + 8) / 2 + 2 # 7

# Integers gets converted to floats only when we mix them
7 / 3 # 2
7 / 3.0 # 2.333..

# We can call methods on objects (parentheses optional)
'apple'.length # 5

# We can pass methods arguments (parantheses optional)
'apple'.sub('app', 'fidd') # "fiddle"
'apple'.sub 'app', 'fidd' # "fiddle"

# We have booleans expressions
(true || false) && true # true

# We have nil (similar to null and void)
'apple'.index('o') # nil (because the letter o is not in the string)

# Everything is truthy except `false` and `nil`
false || nil # false

# And I mean everything
!!('' && 0 && -1) # true

# We got basic control flow
if 'apple'.length == 5
  # Code here executes
else
  # Code here doesn't
end

# We can even do inline conditionals
x = 'I get assigned!' if x.nil?

# There's loops
5.times do
  x = x + 1
end

while(true) do
  # I run forever!
end

until(false) do
  # I also run forever!
end

# We can define methods
def yell_at_terminal(message)
  puts message.upcase + "!!!" # `puts` prints the message to the terminal
end

# and call them in the same scope
yell_at_terminal('I have a different opinion than you')
{% endhighlight %}

## Objects objects objects
In ruby, everything is an object. Really. We can check for ourselves:
{% highlight ruby %}
1.is_a?(Object) # true

false.is_a?(Object) # true

nil.is_a?(Object) # true

Object.is_a?(Object) # true

Class.is_a?(Object) # true
{% endhighlight %}

That means they share some methods
{% highlight ruby %}
Object.new.methods
=> [
:instance_of?,
:public_send,
:instance_variable_get,
:instance_variable_set,
:instance_variable_defined?,
:remove_instance_variable,
:private_methods,
:kind_of?,
:instance_variables,
:tap,
:is_a?,
:extend,
:define_singleton_method,
:to_enum,
:enum_for,
:<=>,
:===,
:=~,
:!~,
:eql?,
:respond_to?,
:freeze,
:inspect,
:display,
:send,
:object_id,
:to_s,
:method,
(...)
{% endhighlight %}
---
## Classes
---

{% highlight ruby %}
class Animal # class names are PascalCase
  ORIGIN = 'Earth' # constants are upper case

  def breed(other_animal)
    return unless other_animal.is_a?(Animal)
    Animal.new # if nothing is explicitly returned, last expression is returned
  end

  def introduce_yourself
    "I'm a #{self.class.to_s.downcase}, I come from #{ORIGIN}"
  end
end

class Fish < Animal # Fish is a subclass of Animal
  def initialize(kind) # Initialize is a special method (constructor)
    @kind = kind # variables starting with @ are instance variables
  end

  def breed(other_fish) # fish has its own rules for breeding
    return unless other_fish.is_a?(Fish)
    Fish.new([self.kind, other_fish.kind].sample) # randomly pick a parent type
  end

  def kind
    @kind
  end
end

{% endhighlight %}
---
## Instances
---

{% highlight ruby %}
first_animal, second_animal = Animal.new, Animal.new # multi-assignment!
first_fish, second_fish = Fish.new('Goldfish'), Fish.new('Shark')

baby_animal = first_animal.breed(second_animal)
baby_animal.class # Animal

baby_fish = first_first.breed(second_fish)
baby_fish.class # Fish
baby_fish.kind # 'Goldfish' or 'Shark'

baby_animal.breed(baby_fish) # nil

baby_fish.introduce_yourself # "I'm a fish, I come from Earth"
{% endhighlight %}

---
## Ruby method dispatch
---

{% highlight ruby %}
baby_fish.introduce_yourself
{% endhighlight %}

* will first check if the Fish class has this method, but it does not
* will then check if the parent class has it, and it does.

{% highlight ruby %}
baby_fish.this_method_is_not_there
{% endhighlight %}

* will check fish, and all it's superclasses with no luck
* will then check fish for a special method called `method_missing`
* will check all superclasses for `method_missing`
* will find it on `BasicObject`, which tells ruby to raise a `NoMethodError`

`method_missing` is one of the most common (and most dangerous!) ways to meta-program ruby.

---
## Basic data-structures
---

{% highlight ruby %}
# We have arrays!
things = ['fizz','buzz']
things.length # 2
things.first # "fizz"
things.last # "buzz"

more_things = things + things # ["fizz", "buzz", "fizz", "buzz"]

# We can iterate over arrays
more_things.each do |thing|
  puts thing
end

# We can access elements with square brackets
more_things[1] # "buzz" (0-indexed)

# We have hashes (other languages calls them dicts, hashmaps, etc.)
pets = {'dog' => 'Fluffy', 'cat' => 'Princess Sprinkles'}
pets.keys # ['dog', 'cat']
pets.values # ["Fluffy", "Princess Sprinkles"]

# We can get values by keys
pets['dog'] # "Fluffy"
{% endhighlight %}

---
## Easier hashes
---

Since we use hashes a LOT in ruby, there's a few things to make them easier to work with

{% highlight ruby %}
# Ruby has symbols
x = :herp # symbols start with :

# Any two identical symbols are actually the same instance
:herp.object_id == :herp.object_id # true

# If we use symbols as hash keys, we can avoid the hash-rockets (=>)
{:herp => 'derp'} == {herp: 'derp'} # true

herpderp = {herp: 'derp'}

herpderp[:herp] # "derp"

# WARNING: the {key: 'value'} syntax only works in ruby 1.9+

{% endhighlight %}

---
## Blocks
---

{% highlight ruby %}
# Blocks are chunks of code we give as method arguments
# We already saw some
5.times do
  # this is a block with do/end
end

5.times {
  # we can also use brackets
}

(1..5).each do |number|
  # this block takes an argument, and makes it available as `number`
end

[1, 2, 3, 4, 5].map {|element| element.even?} # [false, true, false, true, false]

# We can define a method that accepts a block like this
def powerful_method
  block_returned = yield # yield executes a given block
  "The block returned #{block_returned}"
end

powerful_method do
  "a meaningful answer!"
end

# "The block returned a meaningful answer!"

{% endhighlight %}

That was all!

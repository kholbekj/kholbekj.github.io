---
title: Creating convenience queries on Ecto models
layout: post
---
These days most of my free time coding is taking place in elixir, and as anyone
who knows anything knows, that means [Phoenix][phoenix]. Phoenix is trying to
win grounds by being Rails, but with less loading everything all the time, less
magic, and much better performance. In many ways I'm loving it, but one of the
first things I got really tired of was working working with my database in a
console. In Rails, my console sessions will often look something like this:

{% highlight ruby %}
irb(main)> Post.last.methods.grep /publish/
=> :publish!, :publish_all_due!
irb(main)> Post.unpublished.count
=> 300
irb(main)> Post.publish_all_due!
=> nil
irb(main)> Post.unpublished.count
=> 250
{% endhighlight %}

Ecto, the persistence layer in Phoenix take a very different approach, than
ActiveRecord, and will instead insist on having persistence and models separate.
This is a good thing in itself, but it does cost us some really neat ways to
quickly fetch data. For instance to get the last post, one might do something
like:

{% highlight elixir %}
iex> import Ecto.Query
nil
iex> from(p in Post, order[desc: :id], limit: 1) |> Repo.one
%Post{data}
{% endhighlight %}

Or to count:

{% highlight elixir %}
iex> from(p in Post, select: count(p.id)) |> Repo.one
{% endhighlight %}

It is quite obvious that there is some distance between the ease of which you
can explore your data in this way. So I figured I'd set out try to make
something to make a few of the common things a bit easier.

 With the goal in mind to make at least count, first and last functions, first
decision is where to put them.

While it somehow might make more sense to put the methods on the Repo module, I
figured I'd be a good opportunity to dig a bit into elixir's metaprogramming,
and put them on the models themselves.

First step is to make a module that can define functions on other modules. This can be achieved with the use macro. The use macro is usually for setting up some kind of state, and should not be used for fun, but this is educational, so we'll do it!

In an Ecto model we add:
{% highlight elixir %}
use CommonQueries
{% endhighlight %}

And then we go ahead and define that module:

{% highlight elixir %}
defmodule CommonQueries do
end
{% endhighlight %}

If we try to compile this, we'll hit an error:

{% highlight elixir %}
** (UndefinedFunctionError) undefined function: CommonQueries.__using__/1
    (elixir) CommonQueries.__using__([])
{% endhighlight %}

That's because the use macro expects us to define a macro on the module called \_\_using\_\_/1
The \_\_using\_\_ macro should return the expression that should be evaluated in the calling place, in this case our function definitions.

We can add the macro definition:
{% highlight elixir %}
defmacro __using__(_) do
  quoted do
  end
end
{% endhighlight %}

but before we can implement any of the query methods, there's two things we need to solve:

 * We need to import Ecto.Query
 * We need to reference the module that is using this one, so as to query the model.

The first part is easy, but there is one important thing: We can scope the import to the include function, but not within the quoted expression. If we do that, we import the module into the calling class, which in this case seems safe, but in general risks colliding with existing methods.

{% highlight elixir %}
defmacro __using__(_) do
  import Ecto.Query
  quoted do
  end
end
{% endhighlight %}

The second part can be achieved with another macro: \_\_info\_\_/1


\_\_info\_\_ takes an atom out of a few select ones, and returns some info about the environment.
In this case, we'll pass it :module which will return the surrounding module as an atom
To make things easier, we wrap that in a function (inside the quoted expression):

{% highlight elixir %}
defp module do
  __info__(:module)
end
{% endhighlight %}

Alright. All there is left now is to implement the queries:

{% highlight elixir %}
defmodule CommonQueries do
  defmacro __using__(_) do
    import Ecto.Query

    quote do
      def count do
        from x in module,
        select: count(x.id)
      end

      def last do
        from x in module,
        order_by: [desc: :id],
        limit: 1
      end

      def first do
        from x in module,
        limit: 1
      end

      defp module do
        __info__(:module)
      end
    end
  end
end
{% endhighlight %}

And we're done! Now for each model we can add

{% highlight elixir %}
use CommonQueries
{% endhighlight %}

And in our console we'll be able to do this:

{% highlight elixir %}
iex> Post.count |> Repo.one
6
iex> Post.last |> Repo.one
%Post{data}
{% endhighlight %}

Now isn't that a delight!

I've left the phoenix app I made writing post on [Github][repo], with the common_queries module sitting in the lib directory.

Next up will probably be a post about expanding these methods to become composable and better integrate with model queries.

Stay tuned!

[phoenix]: http://www.phoenixframework.org
[repo]: https://github.com/kholbekj/ectofun

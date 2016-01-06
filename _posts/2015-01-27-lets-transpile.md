---
layout: 'default'
---
Or compile? I'm not sure anymore. A lot of effort is going into writing ruby and ending up with javascript, these days. [Opal][opal], providing the baseline, have inspired a variety of
projects, but people I've talked to were a bit split the issue of what kind of
code would end up being produced this way. I think it's brilliant.

### Why?
Recall the question:

"Which programming language should I learn?"

Any experienced programmer knows the answer to this. It depends on what it is you
want to achieve. Wanna write a super fast algorithm for machine learning?
Then you're probably not looking at learning ActionScript. Want to make a website?
C++ might not be your best option.

It holds true within a project. Being a Rails developer, I love ruby.
Reading ruby code is like reading Cyberspace Poetry on Steroids™. It's
gorgeous. It's vitamin D directly from your monitor. But if I want to make a tab
on my website
hide when I click it's title, I got to javascript, because it's easier than making
a new request to the web server, serving the same HTML, with a hidden attribute on
an element. One of javascript's advantages is DOM manipulation.

However, getting really good at multiple languages is even harder than a single
one, and a popular desire is being able to deal with everything through only one
language. In the web world, that's a problem, because DOM manipulation can only be
handled by javascript. That's just how it is. The browser has to understand
something, and javascript seemed a good choice at one point. So, a whole bunch of
people decided the only solution to the 'one-language'-problem was write everything
in javascript. Back- as well as frontend. There's only one problem with that.

[Javascript sucks.][javascript]

It's horrible to write, you get ten minutes into writing a class, and before you
know it you're sliding the curly-bracket roller coaster like there's no tomorrow.

It's hard to get an overview, the language is full of quirks and pitfalls, and
while there are certainly worse languages, I'm always left with the feeling
that it just can't be the solution.

### Que transpilation
When I then first learned coffeescript, I was content. This seemed like an
acceptable compromise, a few of the quirks were taken care of, the thing read much
better, and as long as I avoided relying too heavily on the list comprehension
methods the language provides, it's still quite performant.

A lot of people would keep warning me about how hard it was to debug because I
would 'still have to look at the javascript'. I never had huge problems with that,
but when the subject of ruby compiling to javascript came up, it remained the main
argument. And it bothered me more and more. Why should this be so hard? I mean,
we're not still writing assembler because it'd be too hard to debug the ASM if
it was compiled from abstract C code. And while the two situations are different,
it seems obvious that it depends on how you want to treat javascript.

And I want to treat it like assembler. To me, javascript is what the browser
understands. And to me, that is no reason why I should. All the benefits of
javascript are benefits for compatibility, executiontime, fitting common
environments. The benefits of ruby, is the writing and reading experience. The
stuff humans deal with. The stuff that makes a developer happy.

You know what makes me happy?
{% highlight ruby %}
form = Element.find('.register-form')
form.children.each do |child|
  Timeout.new 500 do
    child.effect 'fadeIn'
  end
end
{% endhighlight %}

### Exaggerations may have occurred
I'm not crazy. Of course I wanna know how javascript works. For a lot of parts, I
do. And it's gonna be a
while before anything else is gonna be an option. But I think being scared of
improving the ability to abstract code, because of loss of access to the
underlying engine is too shortsighted. Of course debugging is a solvable problem.
Of course performance can be analyzed and optimized. And it's for the better,
because while there will certainly remain a lot of javascript enthusiasts doing
crazy awesome things with the raw language, there is no reason to force people
who just want access to DOM manipulation and functional websites (and sooner or
later every type of UI) out of their ecosystem,
merely because browsers doesn't speak anything else.



[javascript]: http://whydoesitsuck.com/why-does-javascript-suck/
[opal]: http://opalrb.org

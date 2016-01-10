---
title: I made a site!
layout: post
---
And to be quite frank, that was so much harder than it could have been.
I spend way way too much time redeploying jekyll to github pages, only to
be met by an email claiming that page build failed, but not offering any
insight into why this might be. They did link to a troubleshooting guide,
which had the courtesy to list a few common problems, but my setup was
super vanilla-y, so i was honestly confused.

As it turns out, the syntax highlighter I had chosen, rouge, (because why
go all the way to python, when you can have the world in ruby?), was not
supported by github. The first highlighter the Jekyll website tells you how
to get up and running. Oh well. So much for my sanity for 2 hours.

## But you make so many stupid sites, and then abandom them?

Who told you that? This time is gonna be.. well, probably similar. I never
did get into the habit of documenting my findings, thoughts, or lack
thereof, but I guess there's a time for anything. Maybe this one is it.
Maybe not.

To offer some sort of technical payoff for getting this far in the text,
and for testing out that the syntaxhighlighter github likes uh and oh so
much actually works, here's some perfectly normal ruby:

{% highlight ruby %}
eval(File.read(`ls`.split("\n").select{|f|f.include?('.rb')}.sample.lines.sample))
{% endhighlight %}

(I think this is the point where I make a disclaimer about me not accepting
any responsibility if you chose to apply my code to your
production-environment as a cronjob)

## Next up
Not sure. Think I'll go on and try to style this thing a bit, get some
reasonable defaults up and running. Research if Jekyll somehow allows me
automatically publish this draft, or if I get to be a script-inventor.

Maybe I'll just ramble more.

Adios.

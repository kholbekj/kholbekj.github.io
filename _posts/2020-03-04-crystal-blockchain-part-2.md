---
title: Crystal blockchain - beyond the basics part 2
featured: /assets/images/crystal-hallway.jpg
layout: post
---

In this post we follow up from [my previous one]({% post_url 2020-02-23-crystal-blockchain-part-1 %}). Check that out first.

Until now we have a node which mines blocks, and allows us to send data to be
included in said blocks. It's all nice and well, but something is missing to
make it useful. Since blocks are guaranteed by the difficulty of producing a
valid chain of the same length as the one we already have, our chain is as
secure as the total computation power we've used to produce it. That is not very
secure if the entire chain is generated on a single computer. Many computers
could generate a longer chain in a short time. We need to start mining on
multiple computers.

For this we need to introduce an important concept in crypto-economics:
consensus mechanisms. A consensus mechanism is a way in which multiple miners
can come to an agreement about what the current chain looks like.

For instance, if there are two miners in the network, and they both mine for 20
minutes without talking to eachother, upon syncronizing, they need to agree on
which chain is the right one. In this sense, only one chain is ever really
valid, but some miners may not know about it yet.

Bitcoin uses a fairly simple consensus mechanism - the longest (most difficult)
chain is always right.

If two miners talk to each other, and one sees that the other has a block more
than itself, it will immediately copy the state of the other miner, and start
mining from the new last block.

To do this in our program, we need a few things. First, we need a way for miners
to find eachother and communicate.

## Node discovery

In bitcoin, there are several layers of service discovery fallbacks and
mechanisms, and a lot of concerns in which notes to pick in order to avoid
certain types of attacks and censorship. 

We'll go a bit simpler, and use basically the last fallback that bitcoin has:
some locally hardcoded addresses. To do so, we need a way for two nodes to act
differently. We can start out by simple running them on two different ports -
we can use a commandline argument through `ARGV` - let's default to 3000 if we
don't supply one.


{% highlight crystal %}
  my_port = ARGV.any? ? ARGV.first.to_i : 3000
  Kemal.run(my_port)
{% endhighlight %}

now we can spin up our server with a port:

{% highlight bash %}
  crystal src/crystal-blockchain.cr 1111
{% endhighlight %}

And in a different session spin up another:

{% highlight bash %}
  crystal src/crystal-blockchain.cr 1112
{% endhighlight %}

Alas! We have two selfish miners! Let's make them curious.

Let's hardcode a list of discoverable node ports, and select one that's not our
own:

{% highlight bash %}
  PORTS = [1111, 1112]
  my_port = ARGV.any? ? ARGV.first.to_i : 3000
  other_port = PORTS.find { |p| p != my_port }
{% endhighlight %}

We want to occasionally check other nodes to see if their chain is longer than
ours. We can start with a new fiber which checks every 10 seconds:

{% highlight bash %}
  spawn do
    loop do
      puts
      puts "Checking for new blocks"
      begin
        response = HTTP::Client.get "localhost:#{other_port}"
        body = JSON.parse(response.body)
        puts "Other chain is #{body.as_a.size} long"
        puts "My chain is #{blockchain.size} long"
      rescue 
        puts "Can't connect to node with port #{other_port}"
      end
      sleep 10
    end
  end
{% endhighlight %}

Let the two nodes run for a little while, and one is sure to get ahead.
The next step is to import the chain from the other block when it gets longer
than ours. Of course, in the real world only the blockers newer than the ones we
already have would be needed, but it's a bit simpler to just import the chain.

{% highlight bash %}
  puts "My chain is #{blockchain.size} long"
  if body.as_a.size > blockchain.size
    puts "Importing..."
    new_blockchain = [] of Block
    body.as_a.each do |json_block|
      data = json_block["data"].as_a.map(&.as_s)
      new_blockchain << Block.new(
        json_block["index"].as_i,
        json_block["timestamp"].as_s,
        data,
        json_block["prev_hash"].as_s
      )
    end
    blockchain = new_blockchain
{% endhighlight %}

As soon as the other node has a longer node, we create a fresh array of blocks,
and parse each element of the json endpoint into a fresh block. In the end we
have a copy of the remote chain, which we overwrite our own with.

This is great, but it puts us at risk - if a node would start to cheat, we
wouldn't catch it. It could generate a block that is actually not valid, and
we'd happily copy the chain anyway. Let's fix that. First, let's move the json
stuff into a method on the block to clear up our code a bit:

Adding in `block.cr`:
{% highlight bash %}
  def self.from_json(json_block : JSON::Any)
    data = json_block["data"].as_a.map(&.as_s)
    new_blockchain << Block.new(
      json_block["index"].as_i,
      json_block["timestamp"].as_s,
      data,
      json_block["prev_hash"].as_s
    )
  end
{% endhighlight %}

And in `crystal-blockchain.cr`:
{% highlight bash %}
  puts "Importing..."
  new_blockchain = [] of Block
  body.as_a.each do |json_block|
    new_blockchain << Block.from_json(json_block)
  end
  blockchain = new_blockchain
  puts "Imported!"
{% endhighlight %}

This gives us space to validate as we go:

{% highlight bash %}
new_blockchain << Block.from_json(json_block)
  unless new_blockchain.size == 1 || new_blockchain.last.valid?(new_blockchain.last(2).first)
    puts "cheats!"
    puts new_blockchain.last.render
    puts new_blockchain.last(2).first.render
    raise "Other node is cheating!"
  end
{% endhighlight %}

However, as it turns out we're not importing nonce's, meaning we're getting
unsolved blocks. Whops. Let's add it as an initialization argument:

{% highlight bash %}
  def initialize(@index : Int32, @timestamp : String, @data : Array(String), @prev_hash : String, @nonce : Int32 = 0)
     @hash = calculate_hash
   end
{% endhighlight %}

Let's add it to our `.from_json` method:
{% highlight bash %}
  def self.from_json(json_block : JSON::Any)
    data = json_block["data"].as_a.map(&.as_s)
    Block.new(
      json_block["index"].as_i,
      json_block["timestamp"].as_s,
      data,
      json_block["prev_hash"].as_s,
      json_block["nonce"].as_i
    )
  end
{% endhighlight %}

In order to better track the output, I deleted the line that prints every mining iteration:

{% highlight bash %}
  puts "Mining! Not solution: #{block.render}"
{% endhighlight %}

Now when you spin two nodes up, they will start synchronizing whenever one gets
ahead. We know that only blockchains that are valid are getting imported, and
herein lies the core of a blockchain: It's quite fast to validate that it took
as much computation power to generate it as expected. What we have now is a
system that is very hard to overwrite. If two people runs these two notes for a
while, one could not easily change the state. The bad actor would have to use
more computing power than the other node, and the longer the system runs, the
harder to manipulate. The more nodes we add, the more difficult.

We could extend the algorithm to work with a few more nodes if we liked. One
more might be good, to illustrate how:

First, let's add the new port, and select all the foreign ports:


{% highlight bash %}
  PORTS = [1111, 1112, 1113]
  my_port = ARGV.any? ? ARGV.first.to_i : 3000
  other_ports = PORTS.select { |p| p != my_port }
{% endhighlight %}

Then we loop around the ports:
{% highlight bash %}
  other_ports.each do |other_port|
    begin
      response = HTTP::Client.get "localhost:#{other_port}"
      body = JSON.parse(response.body)
      puts "Other chain is #{body.as_a.size} long"
      puts "My chain is #{blockchain.size} long"
      if body.as_a.size > blockchain.size
        puts "Importing..."
        new_blockchain = [] of Block
        body.as_a.each do |json_block|
          new_blockchain << Block.from_json(json_block)
          unless new_blockchain.size == 1 || new_blockchain.last.valid?(new_blockchain.last(2).first)
            puts "cheats!"
            puts new_blockchain.last.render
            puts new_blockchain.last(2).first.render
            raise "Other node is cheating!"
          end
        end
        blockchain = new_blockchain
        puts "Imported!"
      end
    rescue 
      puts "Can't connect to node with port #{other_port}"
    end
  end
{% endhighlight %}

That's all it takes. We can now spin up `1113` and they all start working
together.

This concludes the basics of mining. Next step is to bring a real monetary
system into those blocks of data. To be continued...

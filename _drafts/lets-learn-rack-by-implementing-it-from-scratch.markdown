---
title: Let's learn rack by implementing it from scratch
---

Rack is a framework for ruby web frameworks. If you developed apps in ruby frameworks like rails , hanami , sinatra , you already used rack in your app . Almost all ruby webframeworks use rack under the hood , If you are already familar with rack then you can skip next step and goto rewriting part

Rack provides minimalistic api to interact , First let's have a look about rack

A rack app is an object which takes request environment hash and provides array of 3 elements the output, rack object should respond to the method ```call```

* The HTTP response code
* Headers hash
* response body object which responds to each

Obvious question here is why does rack says about responding to particular method . This one of the powerfull paradigm available in ruby called [duck typing](https://en.wikipedia.org/wiki/Duck_typing) . ie rack doesn't care about object or it's implementation as long as it responds to the particular method

Let's take a look at a simple example

Install rack 

```bash
  gem install bash
```

create a file with name ```config.ru```

```ruby
# config.ru
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['Hello World\'d']] }
```

and run the command ```rackup```

Now open browser and vist ```localhost:9292```

Congratulations !, You just made a rack app with just a single line

Now let's have look at the above example

We have a proc object which responds to [call](https://ruby-doc.org/core-2.2.0/Proc.html#method-i-call) method . also an array consist of

```200``` as Status code

```{'Content-Type' => 'text/html'}``` as Response header
and 
```['Hello World \'d']``` as body

But this doesn't do anything interesting , this will just display ```Hello World``` for all requests . 



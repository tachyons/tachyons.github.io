---
title: Let's learn rack by implementing it from scratch
date: 2017-02-16T01:28:20+05:30
comments: true 
tags: ['ruby','rack']
---

Rack is a framework for ruby web frameworks. If you developed apps in ruby frameworks like rails, hanami, Sinatra, you already used  Rack. Almost all ruby web frameworks use rack under the hood, If you are already familiar with rack then you can skip next step and go to re building  part

## Introduction To Rack

Rack provides minimalistic API to interact, First let's have a look at rack

A rack app is an object which takes request environment hash and provides array of 3 elements the output, rack object should respond to the method ```call```

* The HTTP response code
* Headers hash
* response body object which responds to each

The obvious question here is why does rack says about responding to the particular method. This one of the powerful paradigm available in ruby called [duck typing](https://en.wikipedia.org/wiki/Duck_typing). ie rack doesn't care about the object or it's implementation as long as it responds to the particular method

Let's take a look at a simple example.

Install rack 

```bash
  gem install rack
```

create a file with name ```config.ru```

```ruby
# config.ru
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['Hello World\'d']] }
```

and run the command ```rackup```

Now open the browser and vist ```localhost:9292```

Congratulations !, You just made a rack app with just a single line

Now let's have look at the above example

We have a proc object which responds to [call](https://ruby-doc.org/core-2.2.0/Proc.html#method-i-call) method. also an array consist of

```200``` as Status code

```{'Content-Type' => 'text/html'}``` as Response header
and 
```['Hello World \'d']``` as body

Since rack do not care about the kind of rack object, we can do the same using class or an object

```ruby

class SuperCoolApp
  def call(env)
    ['200', {'Content-Type' => 'text/html'}, ['Hello World\'d']]
  end
end
run SuperCoolApp

```

```ruby
class CoolApp
  def self.call(env)
    ['200', {'Content-Type' => 'text/html'}, ['Hello World\'d']]
  end
end

run SuperCoolApp.new
```
But this doesn't do anything interesting, this will just display ```Hello World``` for all requests . Because we were returning same output without even considering the parameter ```env``` . Let's have a look into by returning env hash as output 

```ruby
class CoolApp
  def self.call(env)
    ['200', {'Content-Type' => 'text/html'}, [env.inspect]]
  end
end

```

Now run ```rackup``` and goto ```localhost:9292/hello/world```
Output will be something like this
```ruby
{"rack.version"=>[1, 3], "rack.errors"=>#>>, "rack.multithread"=>true, "rack.multiprocess"=>false, "rack.run_once"=>false, "SCRIPT_NAME"=>"", "QUERY_STRING"=>"", "SERVER_PROTOCOL"=>"HTTP/1.1", "SERVER_SOFTWARE"=>"puma 3.6.0 Sleepy Sunday Serenity", "GATEWAY_INTERFACE"=>"CGI/1.2", "REQUEST_METHOD"=>"GET", "REQUEST_PATH"=>"/hello/world", "REQUEST_URI"=>"/hello/world", "HTTP_VERSION"=>"HTTP/1.1", "HTTP_HOST"=>"localhost:9292", "HTTP_CONNECTION"=>"keep-alive", "HTTP_UPGRADE_INSECURE_REQUESTS"=>"1", "HTTP_USER_AGENT"=>"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36", "HTTP_ACCEPT"=>"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", "HTTP_DNT"=>"1", "HTTP_ACCEPT_ENCODING"=>"gzip, deflate, sdch, br", "HTTP_ACCEPT_LANGUAGE"=>"en-US,en;q=0.8,ml;q=0.6", "SERVER_NAME"=>"localhost", "SERVER_PORT"=>"9292", "PATH_INFO"=>"/hello/world", "REMOTE_ADDR"=>"127.0.0.1", "puma.socket"=>#, "rack.hijack?"=>true, "rack.hijack"=>#, "rack.input"=>#>, "rack.url_scheme"=>"http", "rack.after_reply"=>[], "puma.config"=>#"development", :pid=>nil, :Port=>9292, :Host=>"localhost", :AccessLog=>[], :config=>"/home/tachyons/code/rack/config.ru"}, {:log_requests=>false, :environment=>"development", :binds=>["tcp://localhost:9292"], :app=>#, @content_length=nil>>, @logger=#>>>>}, {:environment=>"development"}, {}], @defaults={:min_threads=>0, :max_threads=>16, :log_requests=>false, :debug=>false, :binds=>["tcp://0.0.0.0:9292"], :workers=>0, :daemon=>false, :mode=>:http, :worker_timeout=>60, :worker_boot_timeout=>60, :worker_shutdown_timeout=>30, :remote_address=>:socket, :tag=>"rack", :environment=>"development", :rackup=>"config.ru", :logger=>#>, :persistent_timeout=>20}>, @plugins=#>, "rack.tempfiles"=>[]}

```
Now change url and see the changes output

To make it clear Let's build a simple app to hello
```ruby
class CoolApp
  def self.call(env)
    ['200', {'Content-Type' => 'text/html'}, [ "Hi " + env['REQUEST_PATH'].split('/').join(" ")]]
  end
end

run CoolApp
```

Run rackup again, and go to ```localhost:9292/aboobacker/mk```

App will respond "Hi aboobacker mk"

You can implement your own logic using the env variable provided by rack

Rack also provides [Rack Request](http://www.rubydoc.info/gems/rack/Rack/Request) Abstraction which provides a convenient interface to a Rack environment.

But that is not the end, rack also provides feature called middleware, which let you use multiple rack apps as pipeline . ie output of one rack app will feed as input to next rack app . Let's check that by one example

```ruby
class ReverseOutput
  def initialize(app)
    @app = app 
  end

  def call(env) 
    status, headers, body = @app.call(env) 
    body = body.map { |msg| msg.reverse } 
    [status, headers, body] 
  end 
end 

class CoolApp
  def self.call(env)
    ['200', {'Content-Type' => 'text/html'}, [ "Hi " + env['REQUEST_PATH'].split('/').join(" ")]]
  end
end

use ReverseOutput
run CoolApp
```

Here we made a simple middleware ```ReverseOutput``` which will reverse the response body, You can add any number of middlewares like this, also you can use pre defined middlewares provided by the rack and open source general purpose middlewares. [List of middlewares](https://github.com/rack/rack/wiki/List-of-Middleware) 

## Building from scratch
Now let's have a look at how the rack works by making a rack like library from scratch, Let's name it Srack. But one obvious question here is why rackup file is ```config.ru```, not ```config.rb``` ? . Also from where the methods like use, run etc are coming

Let's look at our first code sample in a different way

```ruby
# app.rb
  Rack::Builder.app do
    run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['Hello World\'d']] }
  end
```
Here we can see that ```config.ru``` is a block that is to be passed to Rack::Builder.app method
```bash
bundle gem srack
```
Now remove all TODOs from ```srack.gemspec``` So that we can run the test cases . Now if we run test cases it will show one failure message
```
Failed examples:

rspec ./spec/srack_spec.rb:8 # Srack does something useful

```

And it is true, we haven't done anything useful yet

First thing we have to do is to build an executable equiallant to rackup, Let's call it srackup

```
touch exe/srackup
```

```ruby
#!/usr/bin/env ruby

require "srack"
Srack::Server.start
```

I copied above file from [rack repo](https://github.com/rack/rack/blob/master/bin/rackup) to make sure that we are following the same way . Since we haven't implemenetd Srack::Server this won't work yet . So let's make that first

Since we have to make instance of ```Rack::Server``` we can make it as a `class` and define ```start``` as the class method

```ruby
module Srack
  class Server
    def self.start
    end
  end
end
```

Now we have Srack::Server.start method. But it is doing nothing. Since we want Server object, we can delegate our start method to it's instance method.

```ruby
module Srack
  class Server
    def self.start
      new.start
    end

    def start
    end
  end
end
```

Now let's set some default options for our app

```ruby
module Srack
  class Server
    def initialize
     @options = default_options
    end

    def self.start
      new.start
    end

    def start
    end

    private
    
    def default_options
      {
        environment: "localhost",
        Port: "9393",
        Host: "localhost"
      }
    end
  end
end
```
Now we have to build the app from config.ru or the file specified as argument, we can store it in ```@options``` hash with the key ```config```
```ruby
module Srack
  class Server
    def initialize
      @options = default_options 
      @options[:config] = ARGV[0] if ARGV[0]
      @app = build_app
    end

    def self.start
      new.start
    end

    def start
    end

    private
    
    def default_options
      {
        environment: "localhost",
        Port: "9393",
        Host: "localhost",
        config: 'config.ru'
      }
    end

    def build_app
      Builder.parse_file(@options[:config])
    end
  end
end
```
Here we are we are using the ```Srack::Builder``` to parse the config file and load app from it . Let's implement that logic in ```Builder``` factory

```ruby
module Srack
  class Builder
    def self.parse_file(config)
      config_file = ::File.read(config)
      new_from_string(config_file)
    end

    def self.new_from_string(builder_script)
      eval "Rack::Builder.new {\n" + builder_script + "\n}.to_app"
    end
  end
end

```
The first method is self-explanatory, it just read the file and passes the file body to ```new_from_string``` . The method ```new_from_string``` takes the file contents, convert it into a proc and pass to ```Rack::Builder.new``` . So that we can execute the contents of config.ru in the context of the builder

Remember our first rack app ? 
```ruby
# config.ru
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['Hello World\'d']] }
```
In order to execute this

1. Builder class should accept block for initialize method
2. And execute it within the context of ```Builder object```
3. Builder class also should have methods run and to_app as setter and getter
Let's see it in code
```ruby
module Srack
  class Builder
    def initialize(&block)
      instance_eval(&block) if block_given?
    end

    def run(app)
      @app = app
    end

    def to_app
      @app
    end

    def self.parse_file(config)
      config_file = ::File.read(config)
      new_from_string(config_file)
    end

    def self.new_from_string(builder_script)
      eval "Srack::Builder.new {\n" + builder_script + "\n}.to_app"
    end
  end
end
```

Now we have Builder class, But start method in ```Srack::Server``` class is still empty, In order to do that we have to connect to some real server. Remember when we mentioned rack is an interface to web servers? 

```ruby
module Srack
  class Server
    ...
    def start
      server.run @app, @options
    end

    private

    def server
      @server ||= Srack::Handler.default
    end
    ...
  end
end
```

Srack will have handlers for each type of servers, So that we can global api for handlers, ie all handlers should respond to ```run``` method with 2 arguments ```@app``` and ```@options```

```ruby
# lib/srack/handler.rb`
module Srack
  module Handler
    autoload :Thin, 'srack/handler/thin'
    def self.default
      Handler::Thin
    end
  end
end
```

```ruby
# lib/srack/handler/thin.rb
require 'thin'
module Srack
  module Handler
    class Thin
      def self.run(app, options = {})
        host = options[:Host]
        port = options[:Port]
        args = [host, port, app, options]
        server = ::Thin::Server.new(*args)
        server.start
      end
    end
  end
end
```

Here made handler module which can accomodate multple handlers, In this example we used thin as the default server . To use thin inside our app, we have to include it in our ```srack.gemspec``` 
```ruby
spec.add_dependency "thin"
```
Now you can build the gem to test

```
gem build srack.gemspec
gem install srack-0.1.0.gem
```

Now our srack is capable for running our first rack app
Just goto the directory with the file ```config.ru``` and run ```srackup```


## Implementing middleware

As discussed earlier one of the widely used feature in the rack is middleware. Let's see how it works

A middleware will take the output(triplet) of rack app and modify it and give to next middleware or the app

We can make some tweaks in ```Srackup::Builder``` to accommodate this

```ruby
...
class Builder
  ...
  def initialize(&block)
      @use = []
      instance_eval(&block) if block_given?
    end

    def run(app)
      @run = app
    end

    def use(middleware, *args, &block)
      @use << proc {|app| middleware.new(app, *args, &block)}
    end

    def to_app
      app = @run
      app = @use.reverse.inject(app) { |a,e| e[a] }
      app
    end
    ...
end
...
```

Here we defined an extra method ```use``` which will accept middleware as input. Also we have a new instance variable array ```@use``` which will store procs which accept app as input and returns new middleware object in return

Also, we changed ```to_app``` in such way that middlewares will be executed in the reverse order of calling 

Now our app can also handle middlewares

If something is missing, or getting some errors, you can cross check with my [repo here](https://github.com/tachyons/srack)

### References
* https://github.com/rack
* http://www.kavinder.com/blog/2014-10-10-rebuild-a-gem-rack/
* http://www.kavinder.com/blog/2014-10-10-rebuild-a-gem-rack/

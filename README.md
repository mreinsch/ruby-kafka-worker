# kafka-worker

Generalizes kafka intialization and event consumption. Now you just need to register Handler classes for each consumed event.

## How to use kafka-worker

Add the following line to your Gemfile

```
gem 'kafka-worker', github: 'creatubbles/ruby-kafka-worker'
```

For each kafka topic you want to respond to, create a class that includes `KafkaWorker::Handler`
, `consumes('name-of-topic')` and a custom definition of the `handler(message)` method.

```ruby
class HelloWorldTopicHandler
  includes KafkaWorker::Handler
  consumes 'hello-world'
  
  def handler(message)
    print message
  end
end
```

Now `KafkaWorker::Worker` will call `HelloWorldTopicHandler::handler` every time 
it receives a message for the topic `hello-world`. You can define multiple 
handlers across separate files and they will all be automatically registered 
in `KafkaWorker::Handler`. Next you need to intialize the `KafkaWorker::Worker`
and run it to consume kafka topics.

```ruby
opts = {
   kafka_ips: "127.0.0.1:9092",
   client_id: 'test',
   group_id: 'test'
}

kw = KafkaWorker::Worker.new(opts)
kw.run
trap("QUIT") { kw.stop_consumer }
```

If you want to initialize and share values between handlers, you need to declare 
a class that the handler will subclass, and then initialize the variables before 
running the kafka worker.

```ruby
class BaseKafkaHandler
  cattr_accessor :x
end

class ChildTopicHandler < BaseKafkaHandler
  includes KafkaWorker::Handler
  consumes 'hello-world'
  
  def handler(message)
    print x
  end
end

class SecondChildTopicHandler < BaseKafkaHandler
  includes KafkaWorker::Handler
  consumes 'goodbye'
  
  def handler(message)
    print x
  end
end

BaseKafkaHandler.x = "this is shared"

opts = {
   kafka_ips: "127.0.0.1:9092",
   client_id: 'test',
   group_id: 'test'
}

kw = KafkaWorker::Worker.new(opts)
kw.run
trap("QUIT") { kw.stop_consumer }
```

## Rollbar support

This gem will automatically log errors to Rollbar if these conditions are met. 

```ruby
ENV['ROLLBAR_ACCESS_TOKEN'] && ['staging', 'production'].include?(ENV['ENV_DOMAIN_NAME'] || Rails.env)
```

You can also override `on_error(message, err)` in each handler.

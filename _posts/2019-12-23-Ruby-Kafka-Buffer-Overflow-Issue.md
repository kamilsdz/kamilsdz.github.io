---
layout: post
title: Ruby-Kafka, async_producer and large messages - buffer overflow issue
---

Waiting for a new project (I am so excited about a new one!) is a good time to go back to recent problems. (Un)fortunately some time ago me and my colleague noticed problems on the one of our production servers. After a few hours we discovered the reason. There were two things - our lack of knowledge and a little bug in `ruby-kafka` gem. Debugging was really fun, so I decided to describe it.
![_config.yml]({{ site.baseurl }}/images/posts/rubykafkabufferoverflow/header.png)

### Symptoms:
Let's imagine a simple Ruby application that produces messages to single Kafka topic. It uses the latest `ruby-kafka` gem (v. 0.7.10) and its async producer. The structure of the message is always the same:
```ruby
{
  "data": "260KbBase64String"
}
```

Under some load our app stops sending new messages to Kafka topic (yay!). On logs we can find errors such as:
`Kafka::BufferOverflow (Cannot produce to our_topic_name, max queue size (1000 messages) reached)`

### Issue reproduction:
[I've created a repo](https://github.com/kamilsdz/rubykafka-reproduced-issue) with a simple application that shows exactly what happened.

You can run it by `docker-compose up`.

What we can see is that application produces messages that aren't processed (and the queue is still growing):
```
...
Starting again
Creating new messages..
All enqueued!
Ruby-Kafka queue size: 939
Ruby-Kafka queue size: 939
Ruby-Kafka queue size: 939
Ruby-Kafka queue size: 938
Ruby-Kafka queue size: 938
Ruby-Kafka queue size: 938
Ruby-Kafka queue size: 938
Ruby-Kafka queue size: 938
Ruby-Kafka queue size: 938
Ruby-Kafka queue size: 938
Starting again
Creating new messages..
All enqueued!
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Ruby-Kafka queue size: 988
Starting again
Creating new messages..
Traceback (most recent call last):
	7: from ./run:8:in `<main>'
	6: from ./run:8:in `loop'
	5: from ./run:10:in `block in <main>'
	4: from ./run:10:in `times'
	3: from ./run:11:in `block (2 levels) in <main>'
	2: from /app/workers/sender.rb:5:in `perform'
	1: from /usr/local/bundle/gems/ruby-kafka-0.7.10/lib/kafka/async_producer.rb:109:in `produce'
/usr/local/bundle/gems/ruby-kafka-0.7.10/lib/kafka/async_producer.rb:168:in `buffer_overflow': Cannot produce to cats-data, max queue size (1000 messages) reached (Kafka::BufferOverflow)
```

We can check how many messages Kafka consumer received as well:
```
docker-compose exec kafka bash
cd opt/kafka/bin
kafka-console-consumer.sh --zookeeper zookeeper:2181 --topic cats-data --from-beginning
```
And there is no message!

To sum up - our `app` service should produce a lot of messages to queue `cats-data`, but as a result none were produced. Ruby-Kafka's asynchronous producer was running with `delivery_threshold: 10` option, so it should send all messages from the queue when the buffer size reaches 10 messages. Worth mentioning is the fact that during program execution we don't see any exceptions until buffer overflow.


### What happened?
I had started debugging from running the same piece of code again, but this time using IRB and sync producer and this is what I got: `Kafka::MessageSizeTooLarge`.

Kafka works best when the messages are not too big. The default message size is 1MB (we are trying to send ~260KB). Sending them in a batch of 10 exceeds the limit. It is a bit tricky, because as I've noticed one batch == one message. Exception is throwed by Kafka with [code 10](https://github.com/zendesk/ruby-kafka/blob/master/lib/kafka/protocol.rb#L81).

Our app uses async producer, so there is a problem with handling those exception. Therefore, ruby-kafka cannot send any message to the topic after exception. Take a look at snippet below:
```ruby
kafka_producer = kafka.async_producer
kafka_producer.produce("a", topic: "test-topic") # correct msg 
kafka_producer.deliver_message # 1 msg delivered

kafka_producer.produce("a" * 100_000_000, topic: "test-topic") # too large msg
kafka_producer.deliver_message # 0 msgs delivered

kafka_producer.produce("a", topic: "test-topic") # correct msg again
kafka_producer.deliver_message # 0 msg delivered
```

So, as I understand correctly, we have two problems here:
* handling of too large messages (batches)
* queue blocking

The first problem is IMO quite complex, becouse we would have to validate batch size each time we produce message to make sure that not only our queue is smaller than `max_queue_size`, but also batch size (bytes) is smaller than Kafka message size limit for this topic. Also, we shoud raise exception if single message is too large.

Second problem is related to our blocked queue. Now, we can't produce any new message after producing too large message.

### Solution
Ruby-Kafka allows us delve into Kafka internals. Check this out:
```ruby
kafka.describe_topic("cats-data", ["max.message.bytes"])
# => {"max.message.bytes"=>"1000012"}
```
It confirms that out message max size is 1000012 bytes (1MB). Using our library we can manipulate this value:

```ruby
kafka.alter_topic("cats-data", "max.message.bytes" => 10000000)
kafka.describe_topic("cats-data", ["max.message.bytes"])
# => {"max.message.bytes"=>"10000000"} (10MB)
```

The easiest workaround is based on simple calculation. We know that our message has around 260KB. We can increase kafka message limit for our queue to 5MB and set some `delivery_threshold` value. To keep it safe, let it be 10 (260KB * 10 == 2,6MB). Will it work? I've made a test and it worked perfectly!

Can we do it better?
Probably yes, but there are many problems to solve. We should remember about multithreading principles, which makes it difficult. The workaround with calculations is fine until one of our instances begins to send larger messages. Also, each topic has own limits. If you are more interested in this topic, [here](https://github.com/zendesk/ruby-kafka/blob/031ab6fb884a8cc8ee2e2c6d87aae5d953271ea8/lib/kafka/async_producer.rb#L211) you can find the piece of code with delivering messages and threshold. Further debugging can be a great challenge!

Have a nice day!

---
title: Messaging At Ease - Powered by Kafka!
published: true
---

I have exposure to traditional enterprise wide deployed 
messaging systems (typically JMS based and some AMQP based at later 
stages). Kafka was only experimentational to see if it is a 
fit-for-purpose tool for the use-cases at hand. This blog is a 
refresher for me - thought will implement a sample use-case and see 
how to leverage the features that seems out of box and specific for 
a particular transport/flow based on code/configuration.

### Kafka 
Kafka is a high through-put distributed messaging platform 
platform typically used as a transport mechanism. Note: It does not do 
any application processing unless you fiddle with kafka-streams.


### Typical use-cases for using Kafka
* Messaging and decoupling systems  
* Activity gathering
* Metrics gathering from different locations, systems
* Application logs gathering
* Stream processing 
* Typically used in micro-services architecture, big-data integration
 along side with spark(some change this newly introduced streams)

Intention is not to go thru Kafka features and architecture - 
books, blogs are plenty, the idea is just list bullet out the points
that I thought we could have leveraged at ease when using a JMS based 
middleware and the learning from the sample impl.

### Application implementation 

* Installed and configured Kafka with topic created 
* Source data from twitter 
    - Java based 
    - Created a twitter app
    - Generated keys for access 
    - Sourced the tweets
* Published tweets to Kafka on the topic of choice (java based and 
not using Kafka connect)
* Setup consumer/consumer-groups
* Sinked data to elastic search
    - Used cloud based [bonsai.io](https://bonsai.io/) -- this so cool and free!
* Setup Kibana again used the cool free bonsai.io for some 
crude visualization

<img src="/assets/images/Messaging-Kafka-Application_Arch.PNG" 
width="700" height="150" />

### Features that caught me

Few features that caught my eye that is so nice to leverage and 
out of box  - without a single line of code! And this is contextual 
to the old messaging systems (say JMS). Note: I
 am *not* trying to highlight the features that Kafka from the 
 platform point of view brings to the table cluster 
 configuration, partitions, replication etc!
 
* Idempotent Producer
    - This is out of box! How cool if messaging system would know 
    that it has already seen the message when the producer is sending
     the message but the ack did not reach and when the producer 
     sends it again - it just discards the dup!
    - When I compare with the old messaging systems (say JMS) - this is 
    difficult to do. Producer will always send the message and it is 
    left actually on the consumer application to check if they have 
    seen the message and ignore. No good.

<br/>

* Safe Producer 
   - How cool if we just have to do some configuration based on the 
   particular producer say - number of retries, acks, ordering. 
   This is ease at producer level rather than configuration on the 
   broker/topic level. Below is all we need to set for that producer!
   - While old messaging systems (say JMS) has some features like 
   retry and then move to DLQ; the nice thing here it is the producer
    which decides so the control is on the app

```console
Producer{
...
        //safe producer
        properties.setProperty(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
        properties.setProperty(ProducerConfig.ACKS_CONFIG, "all");
        properties.setProperty(ProducerConfig.RETRIES_CONFIG, "100");
        properties.setProperty(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, "5");//use 1 for ordering
...        
}
```

* Optimize for ThroughPut - producer level
    - Another nice one is we just add a few properties and it 
    does the compression (and the choice of compression - snappy!) 
    and batching at a producer level. I produce the message - I know 
    how best to compress, how critical is the message and how many 
    times it should retry (is ever the default!)
   
<br/>
  
```console
Producer{
...
        //High thruput settings
        properties.setProperty(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");//use 1 for ordering
        properties.setProperty(ProducerConfig.LINGER_MS_CONFIG, "20");//use 1 for ordering
        properties.setProperty(ProducerConfig.BATCH_SIZE_CONFIG, Integer.toString(32 * 1024));//use 1 for ordering
...
}
```

* Idempotent Consumer
    - Relieve apps of the burden to check if it has already processed
     the message and if the messaging system will take care and not 
     send.
    - Old messaging systems (say JMS) leave this to the application 
    to check rather than the messaging system de-dup it!   

<br/>
    
* Poll Behaviour
    - Kafka is poll based against typical messaging systems consumers
     are push based! This provides how to batch systems, size of 
     batches it want to retrieve, the interval to wait and the size. 
     For e.g. if my use-case does and need all the data immediately 
     rather my application is a bulk processor why get the message as
      a stream immediate and rather if the messaging system would 
      give me out of the box with properties set at the 
      consumer/group level - I just would use it, I know my app and I
       know how best to consume!

<br/>       

* Commits and the bulk requests
    - Choice of committing the data at choice rather than just 
    auto-commit - while this feature exists in messaging systems it 
    does not have out of the box features on bulk request, batch etc. 
    Well you might still want to store the message in a secondary 
    storage like DB for replay or audit purposes (or if it is just 
    audit I would log, send to Splunk/ELK - just a design/impl choice)
    
Besides monitoring, security and management features and the cluster 
configuration that Kafka brings to be table -- I think the real power
is the choice and flexibility it brings to the engineers/developers 
when they produce/consume the message that is right and 
fit-for-purpose for that particular flow.
    
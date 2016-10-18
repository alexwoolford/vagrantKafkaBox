Kafka in a box
==============

When building a prototype, it often isn't necessary to deploy a cluster of machines. Perhaps a single VM would suffice. This Vagrant recipe creates a VM, on the localhost, that's running Kafka in a single-node fashion with a local Zookeeper.

Steve Connelly has a more thoughtful blogpost on this topic: http://www.objectpartners.com/2014/05/06/setting-up-your-own-apache-kafka-cluster-with-vagrant-step-by-step/

My goal was to get a local Kafka instance running with the minimim of effort. To provision:

    git clone https://github.com/alexwoolford/vagrant_kafkabox
    cd vagrant_kafkabox
    vagrant up

You can test using kafka-python:

    pip install kafka-python

Then send some messages to the queue:

    >>> from kafka import KafkaClient, SimpleProducer
    >>> kafka = KafkaClient("localhost:9092", timeout=120)
    >>> kafka.ensure_topic_exists('turkey')
    No handlers could be found for logger "kafka"
    >>> producer = SimpleProducer(kafka)
    >>> for i in range(3):
    ...     producer.send_messages("turkey", "gobble gobble")
    ... 
    [ProduceResponse(topic='turkey', partition=0, error=0, offset=0)]
    [ProduceResponse(topic='turkey', partition=1, error=0, offset=0)]
    [ProduceResponse(topic='turkey', partition=0, error=0, offset=1)]

... and retrieve them from another session:

    >>> from kafka import KafkaClient, SimpleConsumer
    >>> kafka = KafkaClient("localhost:9092", timeout=120)
    >>> consumer = SimpleConsumer(kafka, "my-group", "turkey")
    >>> for message in consumer:
    ...     print(message)
    ... 
    OffsetAndMessage(offset=0, message=Message(magic=0, attributes=0, key=None, value='gobble gobble'))
    OffsetAndMessage(offset=1, message=Message(magic=0, attributes=0, key=None, value='gobble gobble'))
    OffsetAndMessage(offset=0, message=Message(magic=0, attributes=0, key=None, value='gobble gobble'))

### Background on Kafka

Kafka is a distributed streaming platform that is used to publish and subscribe to streams of records.

- Step 1: Start Kafka broker on Host machine
Make sure you have docker and docker-compose installed on your host machine.
create a file namely docker-compose.yml and put below content.

 **_replace the `<ADD-YOUR-HOST-IP-HERE>` with your condition, url domain name or Ip are all acceptable._** 

```
kafka:
    image: confluentinc/cp-kafka:5.4.3
    depends_on:
      - zookeeper
    ports:
     - 9092:9092
    environment:
      KAFKA_BROKER_ID: 0
      KAFKA_LOG_DIRS: /kafka/logs
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://<ADD-YOUR-HOST-IP-HERE>:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 100
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS: kafka:29092
      CONFLUENT_METRICS_REPORTER_TOPIC_REPLICAS: 1
      CONFLUENT_METRICS_ENABLE: 'true'
      CONFLUENT_SUPPORT_CUSTOMER_ID: 'anonymous'
```

Yes you need to have zookeper container running in order for kafka to work, please update <ADD-YOUR-HOST-IP-HERE> in the above file otherwise deepstream will not be able to connect to your host pc. Start the containers using below command

`sudo docker-compose up`

or run with service

`sudo docker-compose up -d`

- Step 2: Test if kafka is working

Very basic thing you can do to make sure Kafka is running is by checking the running containers, open another terminal on Host machine and run

`sudo docker ps`

Output:


```
CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS                          NAMES
0e9c253875b4        confluentinc/cp-kafka:5.4.3       "/etc/confluent/dock…"   7 hours ago         Up 52 minutes       0.0.0.0:9092->9092/tcp         kafka-exp_kafka_1
58b24793db15        confluentinc/cp-zookeeper:5.4.3   "/etc/confluent/dock…"   7 hours ago         Up 52 minutes       2181/tcp, 2888/tcp, 3888/tcp   kafka-exp_zookeeper_1
```

 **make sure your cloud server firewall allowed port 9092** 

Now test if you can publish / subscribe to kafka, I am using python for this.
Install python client using:

`pip3 install kafka-python==2.0.2`

Create a file called producer.py and put in below content.

**_replace the `<ADD-YOUR-HOST-IP-HERE>` with your condition, url domain name or Ip are all acceptable._** 

```
import time
from json import dumps
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers='<ADD-YOUR-HOST-IP-HERE>:9092',
    value_serializer=lambda x: dumps(x).encode('utf-8')
)

idx = 0
while True:
    data = {'idx': idx,'type':'producer-consumer_test_msg'}
    producer.send('test', value=data)
    print(data)
    time.sleep(1.5)
    idx += 1
```
Run producer.py
`python3 producer.py`
Output:

```
{'idx': 0,'type':'producer-consumer_test_msg'}
{'idx': 1,'type':'producer-consumer_test_msg'}
.
.
.
```

This means we are able to publish data to kafka. Now let’s try to consume.
Create another file consumer.py and put in below content.

**_replace the `<ADD-YOUR-HOST-IP-HERE>` with your condition, url domain name or Ip are all acceptable._** 

```


import time
from kafka import KafkaConsumer
import json
import uuid
import sys
#
#sys.argv[2] - topic
print("sys.argv[1] - '-raw' or '-pretty', control append raw or just show pretty msg.")
print("sys.argv[2] - 'yourSpecificTopic' or '\".*\"' for all topics")
print(sys.argv)
consumer = KafkaConsumer(
    bootstrap_servers='localhost:9092',
    auto_offset_reset='latest',
    enable_auto_commit=True,
    group_id=str(uuid.uuid1()),
    value_deserializer=lambda x: json.loads(x.decode('utf-8'))
)

if(sys.argv[2]==".*"):
    print("will use pattern to match topics...")
    consumer.subscribe(pattern=".*")
else:
    consumer.subscribe(topics=[sys.argv[2]])
# do a dummy poll to retrieve some message
consumer.poll()

# go to end of the stream
consumer.seek_to_end()

for event in consumer:
    event_data = event.value
    #js_obj = json.loads(event_data)
    objs_pretty_log_str = ""
    raw_objs_log_str = ""
    for obj in event_data["objects"]:
        conf = obj.split('|')[-1][0:4]
        raw_objs_log_str += obj+"; "
        if "Vehicle|#|DoorWarningSign" in obj:
            objs_pretty_log_str += "DS("+conf+");"
        elif "Vehicle|#|TwoWheeler" in obj:
            objs_pretty_log_str += "EleBic("+conf+");"
        elif "Person|#" in obj:
            objs_pretty_log_str += "Per("+conf+");"
        elif "Vehicle|#|Bicycle" in obj:
            objs_pretty_log_str += "Bic("+conf+");"
    if sys.argv[1] =="-raw":
        pass
    elif sys.argv[1] =="-pretty":
        raw_objs_log_str = ""

    print(event_data['@timestamp']+": "+ event_data['sensorId'].ljust(32,' ')+", objs: "+objs_pretty_log_str+"        "+raw_objs_log_str)


```
Note I am using random group_id so that I can have Independent consumers receiving the same data.
Run consumer.py

`python3 consumer.py`

Output:
```
{'idx': 31,'type':'producer-consumer_test_msg'}
{'idx': 32,'type':'producer-consumer_test_msg'}
.
.
.
```

If you are able to receive the messages, that means your kafka broker is working. just to make sure that deepstream will be able to connect, try to run the producer from Jetson and consumer from Host machine. This will confirm that the kafka is reachable from other machines too.
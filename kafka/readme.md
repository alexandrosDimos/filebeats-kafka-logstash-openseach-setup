# Kafka Setup and Usage Documentation
## Prerequisites
Ensure you have Java installed on your system. You can verify the installation by running:
```bash
java -version
```

Kafka should be properly installed in `/usr/local/kafka`.

## Steps to Run Kafka
### 1. Start the Kafka Server
Navigate to the Kafka installation directory and start the Kafka server using the following commands:
```bash
cd /usr/local/kafka
```
Before starting the Kafka server it is advisable if not necessary to go to the `server.properties` file inside the `config` folder. There you will find two lines as shown below:

```bash
listeners=INTERNAL://localhost:9093,EXTERNAL://:9092
```
```bash
advertised.listeners=INTERNAL://localhost:9093,EXTERNAL://:9092
```

These settings define the protocols and addresses that the Kafka broker will advertise to clients. Clients use these addresses to connect to the broker. So essentially clients (like producers and consumers) outside the Kafka broker's host machine will use this listener to connect to the broker, meaning Filebeat will communicate with the `EXTERNAL` listener. Through trial and error it has been determined that the IP should be explicitely written for the `EXTERNAL` listener, like below:
```bash
listeners=INTERNAL://localhost:9093,EXTERNAL://<ip>:9092
```
```bash
advertised.listeners=INTERNAL://localhost:9093,EXTERNAL://<ip>:9092
```


```bash
sudo bin/kafka-server-start.sh config/server.properties
```

This command initiates the Kafka broker using the configuration specified in `server.properties`.

### 2. Create a Topic
After the Kafka server is up and running, create a topic named `apache` with a replication factor of 1 and 1 partition by
executing:

```bash
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic apache
```

This command sets up a new topic in the Kafka broker.

### 3. Start a Consumer
To start consuming messages from the "apache" topic, run the following command:
```bash
bin/kafka-console-consumer.sh --bootstrap-server <ip>:9092 --topic apache --from-beginning
```

This command initializes a consumer that will read messages from the beginning of the specified topic.

### 4. Start a Producer
Finally, to produce messages to the "apache" topic, use the command below:
```bash
bin/kafka-console-producer.sh --broker-list <ip>:9092 --topic apache
```

This command allows you to send messages to the "apache" topic.

## Notes
Ensure that the `sl100pipeline` hostname resolves correctly within your network for the producer and consumer
commands.
Monitor the Kafka server logs located in the `logs` directory for any issues or debugging information.
You can stop the Kafka server by terminating the process running the `kafka-server-start.sh` script.
## Conclusion
Following these steps will allow you to successfully set up and use Kafka for message production and consumption.
For further configurations and advanced usage, refer to the [Kafka documentation]
(https://kafka.apache.org/documentation/).
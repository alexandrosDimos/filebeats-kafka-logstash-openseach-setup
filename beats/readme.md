# Filebeat Setup and Usage Documentation

## Prerequisites
- Ubuntu 22.04




## 1. Filebeat Installation
For the installation follow the commands in this [Elastic search Documemtation](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html). In a nutshell go to your root directory and execute the following commands:

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.16.0-amd64.deb
```

```bash
sudo dpkg -i filebeat-8.16.0-amd64.deb
```

### Apache2 Installation
As a test scenario we want to check if filebeat can forward apache server logs to a kafka topic. For this reason we have installed Apache2 in the test machine. It is really simple, you can fine the instructions [here](https://ubuntu.com/server/docs/how-to-install-apache2).

What we transmit to kafka are the access logs to the apache server. They are usually located in a file called `access.log` inside `/var/log/apache2/access.log`. In order to see the file expanding you can simply perform http requests to the apache server via the command:

```bash
curl http://localhost
```

## 2. Steps to run Filebeat and Transmit data to kafka

### Configure Filebeat
Before starting Filebeat we need to configure Filebeat's input and Filebeat's output, i.e. from where Filebeat reads the data and where it sends them. To do that:

- Navigate to the correct folder:
```bash
  cd etc/filebeat
```

- There should be a file named `filebeat.yml` there:
```bash
sudo nano filebeat.yml
```

- Replace the content of the file with the following, remember this file contains multiple commented configurations, if you do not want to replace the whole file comment the part you do not need and play with the input and output. 

```plaintext
filebeat.inputs:
- type: filestream
  enabled: true
  paths:
    - /var/log/apache2/access.log

output.kafka:
    hosts: ["<ip>:9092"]
    topic: "apache-logs"

    partition.round_robin:
        reachable_only: false

    required_acks: 1
    compression: gzip
    max_message_bytes: 1000000

```

- Bellow we explain what each line of the file does:
    - `filebeat.inputs`: This section defines the input sources from which Filebeat will read log data.
    - `type: filestream`: This specifies that the input type is `filestream`. The `filestream` input is a newer input type in Filebeat that is designed to efficiently read log files, handle file rotations, and process log entries with enhancements over the older log input type.
    - `enabled`: true: This indicates that this input is enabled and Filebeat should actively monitor it.
    - `paths`: This section specifies the paths to the log files that Filebeat should monitor.
    - `/var/log/apache2/access.log`: This specifies the exact log file that Filebeat will read. In this case, it is the Apache HTTP server access log.
    - `output.kafka`: This section defines the output configuration for sending log data to Apache Kafka.
    - `hosts: ["<ip>:9092"]`: This specifies the Kafka broker's address and port to which Filebeat will send data. In this case, it is pointing to a broker at IP address `ip` on port 9092.
    - `topic: "apache-logs"`: This specifies the Kafka topic where the logs will be sent. Filebeat will publish the log entries collected from the specified log file to this topic.
    - `partition.round_robin`: This configuration controls how messages are distributed across partitions in the Kafka topic.
    - `reachable_only: false`: When set to false, it allows Filebeat to send messages to any partition, regardless of whether the partition leader is reachable at the time of the send. This can help in scenarios where some brokers may be temporarily unavailable.
    - `required_acks: 1`: This setting specifies the acknowledgment level for the messages sent to Kafka. Setting it to 1 means that the broker will acknowledge the receipt of the message as soon as it has been written to the leader of the partition. This allows for a balance between performance and reliability; however, it may mean that messages can be lost if the leader crashes after acknowledging but before replicating to followers.
    - `compression: gzip`: This setting specifies that messages sent to Kafka should be compressed using the Gzip algorithm. Compression can help reduce the bandwidth used when sending messages and improve performance.
    - `max_message_bytes: 1000000`: This setting specifies the maximum size (in bytes) for a single message sent to Kafka. Setting this to 1,000,000 bytes (or approximately 1 MB) means that messages larger than this size will not be sent, and an error will occur if an attempt is made to send such messages.


### Test Filebeat 
Before staring filebeat it is advised to test your configuration:
- To test the configuration file we just wrote is ok execute the following:
    ```bash
    sudo filebeat test config
    ```
    The correct response should be:
    ```bash
    Config OK
    ```

- To test the connection with the kafka broker execute:
    ```bash
    sudo filebeat test output
    ```
    The correct response should be:
    ```bash
    Kafka: <ip>:9092... (your ip)
    parse host... OK
    dns lookup... OK
    addresses: <ip> (your ip)
    dial up... OK
    ```


### Start Filebeat
In order to start Filebeat and watch it execute in the foreground run the following command:
```bash
sudo filebeat -e
```

If you want to run Filebeat in the background use:
```bash
sudo systemctl start filebeat
```

In order to check if it executed correctly run:
```bash
sudo systemctl status filebeat
```


## 3. Check smooth integration with Kafka
In order to test if Kafka receives the logs correctly keep sending http requests to the apache server and open a consumer for the specific Kafka topic you are supposed to be forwarding. If the consumer displays the logs then your pipeline works corrctly.


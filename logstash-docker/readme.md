# Logstash Setup and Usage Documentation
## Prerequisites
- Docker must be installed and running on your system. Use this [guide](https://docs.docker.com/engine/install/ubuntu/).
- Ubuntu 22.04
- Ensure that Kafka is set up and running, with a topic named "apache" created as described in the Kafka
documentation.

## Reference
This setup is based on a guide from [Medium](https://medium.com/@bsrini/dockerizing-logstash-a-step-by-step-guideed7f4e594cb4) and [Opensearch Docs](https://opensearch.org/docs/latest/install-and-configure/install-opensearch/docker/), with some modifications.

## Steps to Run Logstash via Docker

### 1. Create the Job Configuration File
Create a job configuration file, inside the logstash/config folder, (e.g., `kafka-pipeline.conf`) with the following content:

```plaintext
input {
    kafka {
        bootstrap_servers => "192.168.2.223:9092"
        topics => "apache"
    }
}

#Change the ip and port in hosts
output {
    opensearch {
        hosts => ["https://192.168.2.225:9200"]
        index => "opensearch-logstash-docker-%{+YYYY.MM.dd}"
        user => "admin"
        password => "admin"
        ssl => true
        ssl_certificate_verification => false

    }
}

#For Debugging
output {
    stdout { codec => rubydebug }
}
```

#### Explanation of Configuration:
- Input Section: Configures Logstash to read messages from the Kafka topic "apache" using the specified Kafka
bootstrap server.

- Output Section: The commented-out section is intended for sending data to OpenSearch. The debugging output section prints the incoming messages to the standard output in a human-readable format
(`rubydebug` codec).

### 2. Logstash Configuration

Let’s add global logstash configuration `logstash.yml` file under the directory `./logstash/config/` with content below.

```bash
node.name: logstash_project_name
api.environment: dev
```

You can add additional configurations based on your requirement. Let’s add pipeline configuration `pipeline.yml` in the same location as below.

```bash
- pipeline.id: your-pipeline-name
  path.config: "/usr/share/logstash/pipeline/your-pipeline-name.conf"
```
Here you need to define a unique id for the pipeline and the location of the pipeline job configuration that we created before. You may notice that the directory mentioned here is misleading. You’re right, but we are going to move the configuration to the location as such in the docker image.

### 3. Run Logstash in Docker
To run Logstash with the specified configuration, execute the following commands (be sure to be inside the logstash-docker folder):

#### Create an image from the Dockerfile
```bash
sudo docker build -t logstash-job .
```

#### Then run a container with the image you just created. We recommend this option for testing:
```bash
sudo docker run --add-host=sl100pipeline:192.168.2.223 -t  logstash-job
```

#### If you want to run logstash as a service using the following command. We recommend this oprion for production:
```bash
sudo docker compose up -d 
```



#### Explanation of Command:
`--add-host=sl100pipeline:192.168.2.223`: Adds an entry to the `/etc/hosts` file within the Docker container,
allowing it to resolve the `sl100pipeline` hostname to the specified IP address.
`-t`: Allocates a pseudo-TTY, which is useful for running interactive processes.
`logstash-job`: This is the name of the Docker image you are using.


---
Feel free to add any additional sections or modify existing ones to better suit your documentation needs
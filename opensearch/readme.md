# Opensearch Setup and Usage Documentation

## Prerequisites
- Docker must be installed and running on your system. Use this [guide](https://docs.docker.com/engine/install/ubuntu/).
- Ubuntu 22.04

## Reference
This setup is based on a guide from [Opensearch Docs](https://opensearch.org/docs/latest/install-and-configure/install-opensearch/docker/) on how to install an Opensearch cluster using Docker.

### 1. Overview
This Docker Compose configuration file sets up two main services:

- `OpenSearch`: A distributed search and analytics engine.
- `OpenSearch Dashboards`: A web-based interface for visualizing data stored in OpenSearch. It defines the services, networks, and volumes required for the deployment.

### 2. Version Compatibility
Ensure that the version of opensearch-dashboards matches the version of the opensearch service (not explicitly shown here, but implied).

### 3. Services
#### opensearch
- `Image`: This service uses the opensearchproject/opensearch Docker image to run OpenSearch.
- `Container Name`: The container is named opensearch-node1, and you can scale this out by adding more nodes if needed.
- `Environment Variables`: Configures settings for OpenSearch, such as cluster name, node roles, and network host.
- `Ports`: Exposes OpenSearch’s port 9200 (for HTTP API) and 9300 (for inter-node communication).
- `System Limits`: Defines memlock and nofile settings, which are useful for optimizing performance and ensuring OpenSearch runs correctly.
    - `memlock`: Locks memory for OpenSearch to prevent it from being swapped out, which is critical for performance in production environments.
    - `nofile`: Increases the file descriptor limit for OpenSearch, allowing it to handle more simultaneous connections and file operations.
- `Volumes`: Data is persisted in a Docker volume (opensearch-data2) to ensure that data remains intact across container restarts.
- `Networks`: The service is connected to the opensearch-net network to allow communication with other containers, like OpenSearch Dashboards.

#### opensearch-dashboards
- `Image`: Uses the opensearchproject/opensearch-dashboards image. It's important to match this version with the version of OpenSearch you are running.
- `Container Name`: The container is named opensearch-dashboards.
- `Ports`: Maps host port 5601 to the container’s port 5601, allowing you to access the Dashboards interface through your browser.
- `Expose`: Exposes port 5601 to the internal Docker network for other services to access (optional, but useful for inter-container communication).
- `Environment`: The environment variable OPENSEARCH_HOSTS defines the list of OpenSearch nodes that the Dashboards will connect to, making it aware of the OpenSearch cluster's endpoints. You should update this to match your cluster's actual nodes.
- `Networks`: Also connected to the opensearch-net network for internal communication with OpenSearch nodes.

### 4. Volumes
- `opensearch-data1 and opensearch-data2`: These are named volumes used to persist data.
    - `opensearch-data2` is explicitly mounted in the opensearch service to persist OpenSearch data across container restarts.
    - Volumes ensure that even if the containers are stopped or deleted, data such as logs and indices are not lost.

### 5. Networks
- `opensearch-net`: A custom Docker network allowing containers (OpenSearch and Dashboards) to communicate with each other. Using a dedicated network for the services ensures they can resolve each other by container names (e.g., opensearch-node1 or opensearch-dashboards).


### 6. Deployment
To start opensearch execute the following command:

```bash
sudo docker compose up -d
```
# app-log-management-elasticsearch-beats-kibana
Application Log Management with Elasticsearch, Filebeat, and Kibana by using Docker Compose.

**Filebeat:**
Filebeat is a part of Elastic Stack. It is an open source tool that serves to forward logs with the agent role, its purpose is to move the logs to the targeted location. In this scenario, Filebeat transmits Hello World Application logs to ElasticSearch.

**Elasticsearch:**
Elasticsearch is an open-source, distributed search and analytics engine designed for real-time search, log analytics, etc. In this scenario, Elasticsearch stores the Hello World Rest API Application logs sent by Filebeat.

**Kibana:**
Kibana is a tool used to visualize data stored in Elasticsearch. It allows us to create a variety of visualizations such as charts, tables, and maps. In this scenario, Kibana visualizes Hello World Rest API Application logs stored in Elasticsearch.

**Explanation:**
![image](https://github.com/eliifmeric/app-log-management-elasticsearch-beats-kibana/assets/66569972/d182b3a7-48c1-4189-b1b8-197f51fbf7c3)

In this case, I used Docker Compose because it provides a better way to manage and deploy this kind of multi-container applications. It allows us to define all of these containers and their relationships in a single YAML file.

**Port Explanations in the docker-compose.yaml file:**
9200: Elasticsearch HTTP
5601: Kibana
8080: Hello World Rest API App

docker-compose.yaml file explanation:

![image](https://github.com/eliifmeric/app-log-management-elasticsearch-beats-kibana/assets/66569972/c1b71e93-279f-4670-a2e3-48d941dcfc3e)

This service will run the thomaspoignant/hello-world-rest-json Docker image on the container named hello-world-app with the restart policy ‘’always’’. It means, it will always be restarted if it crashes. Also, the port 8080 will be exposed for the Hello World Rest API App. And it will connect to “es” network. It means that the container will be able to communicate with other containers on the same network (es).

![image](https://github.com/eliifmeric/app-log-management-elasticsearch-beats-kibana/assets/66569972/ac8cb770-614b-484c-bde0-9b701d3f9497)

This part defines a service named “filebeat”. It runs the Docker image “docker.elastic.co/beats/filebeat:8.1.0”. And the container name is assigned as “filebeat” and will always be restarted if it crashes. In the other hand, the container will run as the “root” user (I set root user to run Filebeat as a root user because it should be able to read all the logs generated by Hello World Rest API App.) and will have access to the volumes below: 

**./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro** This volume mounts the file “filebeat.yml” in the current directory  to the /usr/share/filebeat/filebeat.yml path (The “ro” at the end means read-only). “filebeat.yml” file contains the Filebeat configuration settings.
**/var/lib/docker/containers:/var/lib/docker/containers:ro **It mounts the “/var/lib/docker/containers” to the /var/lib/docker/containers path. (The “ro” at the end means read-only)
**/var/run/docker.sock:/var/run/docker.sock:ro **It mounts /var/run/docker.sock file on the host machine to the /var/run/docker.sock file inside the Filebeat container. This file is used by Filebeat to communicate with Docker daemon. 

Additionally, the container will connect to the network named “es” and it depends on the hello-world-app service being started before it starts. Also, the environment variable ELASTICSEARCH_HOSTS value is defined in .env file. This variable is used to specify the hostnames and ports of Elasticsearch. Also, I defined co.elastic.logs/enabled: "false" . It indicates that the logs should not be collected for this container.

![image](https://github.com/eliifmeric/app-log-management-elasticsearch-beats-kibana/assets/66569972/d05ed843-a800-4290-a2d8-29c753051bea)

This part defines a service named “elasticsearch” and will run the Docker image docker.elastic.co/elasticsearch/elasticsearch:8.1.0 in the container named “elasticsearch”. 
Restart:always means that the container should always be restarted automatıcally if the container stops/crashes.
In the ports section, the container will expose ports 9200 on the host. It means that, we can Access Elasticsearch on port 9200 (I did not expose port 9300 which is used to communicate with other Elasticsearch nodes on port 9300 because I have created this cluster as a single-node).
NOTE: I set the addresses as 0.0.0.0 only for test because I deployed this project on a cloud server and I needed to access it from my local computer.
In the environments section, ELASTICSEARCH_PASSWORD reads its value from .env file. xpack.security.enabled=false disables the SSL settings for Elasticsearch. I set this as false to do not need to set SSL certificate. Additionally, the next variable is discovery.type=single-node. I set it as single-node to only use a single node for Elasticsearch. The last one is ES_JAVA_OPTS=-Xmx1g -Xms1g to limit the JVM heap size, allocating 1 GB of memory for both the maximum heap size (-Xmx) and the initial heap size (-Xms).
The next one is volumes part. It mounts the elasticsearch-data directory on the host machine to the /usr/share/elasticsearch/data path inside the container. It stores Elasticsearch data.
The networks section indicates that the container will connect to the network “es”
The part “depends_on” means that the “elasticsearch” service depends on “hello-world-app” service, so it will ot start until the hello-world-app service is started.

![image](https://github.com/eliifmeric/app-log-management-elasticsearch-beats-kibana/assets/66569972/dbf4c067-a962-4c54-8ee4-f2f6360c67b0)

This part defines a service named “kibana” and will run the Docker image docker.elastic.co/kibana/kibana:8.1.0 in the container named “kibana”. 
Restart Policy: 
restart: always ensures that the container will automatically restart if it crashes, etc.
Environment Variables:
ELASTICSEARCH_HOSTS and ELASTICSEARCH_USERNAME variables reads the required values from .env file that I created. Also, the other variable xpack.security.enabled=false disables the SSL settings for Kibana. I set this as false to do not need to set SSL certificate.
Ports:
I expose the port 5601 for Kibana.
NOTE: I set the addresses as 0.0.0.0 only for test because I deployed this project on a cloud server and I needed to access it from my local computer.
Networks:
This part indicates the networks that the Kibana container will join, so the container joins “es” network.
Dependencies: This part defines the dependency relationship between the Kibana container and other containers, so the “kibana” container depends on the “elasticsearch” container. Kibana starts only after Elasticsearch is ready because Kibana needs data from Elasticsearch to visualize.

![image](https://github.com/eliifmeric/app-log-management-elasticsearch-beats-kibana/assets/66569972/3456f9e3-a799-422c-970d-b8d9264983d5)

The volumes section defines a named volüme called “elasticsearch-data” It is used to store persistent data for the “elasticsearch” container. Also, the “driver: local” means that the volume should be stored in the local filesystem of the host machine.
The networks section indicates a network named “es”. It is used to connect the “elasticsearch” and “kibana” containers. Also, “driver: bridge” lets containers connected to the same bridge network communicate. It apply to containers running on the same Docker daemon host.


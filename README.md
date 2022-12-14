# database-for-logs

Execute following command in cmd.exe to increase max size of VM:

```
wsl -d docker-desktop
sysctl -w vm.max_map_count=262144
```

Pulling and building an existing ELK Stack docker image:

```
git clone https://github.com/spujadas/elk-docker.git
```

<h2 align=center>Installing Plugins:</h2>
Open the Dockerfile

```
sudo nano Dockerfile
```

<h3>ElasticSearch</h3>

1. Add the following at the end of the Dockerfile:

```
FROM sebp/elk
ENV ES_HOME /opt/elasticsearch
WORKDIR ${ES_HOME}
RUN yes | CONF_DIR=/etc/elasticsearch gosu elasticsearch bin/elasticsearch-plugin \
    install -b <plugin name or link>
```

2. Save the Dockerfile and close the editor.
3. Build the image using either docker build or docker-compose. 

<h3>Logstash</h3>
1. Add the following code to the Dockerfile:

```
FROM sebp/elk
WORKDIR ${LOGSTASH_HOME}
RUN gosu logstash bin/logstash-plugin install <plugin name>
```

2. Save the contents and close the Dockerfile.
3. Run the build to install the plugin.

<h3>Kibana</h3>

1. Insert the following code at the end of the Dockerfile:

```
FROM sebp/elk
WORKDIR ${KIBANA_HOME}
RUN gosu kibana bin/kibana-plugin install <plugin name or link>;
```

2. Save the file and close.
3. Build the Docker image and check the output for the installation results.

<h2 align=center>Running the ELK Container</h2>

1. Run following commands in <b>/elk-docker</b>:

```
docker build -t elk-docker .
```

2. Then you need to run following commands in <b>/elk-docker/nginx-filebeat/</b>:

```
docker stop elk
docker stop elk_filebeat
docker stop elk-docker
docker stop elk_filebeat-docker

docker rm elk
docker rm elk_filebeat
docker rm elk-docker
docker rm elk_filebeat-docker

docker network create -d bridge elknet
docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk --network=elknet elk-docker
docker run -p 80:80 -it --name elk_filebeat --network=elknet elk_filebeat-docker
```

Install Elastic Agent on elk_filebeats:

```
docker exec -it elk_filebeat /bin/bash
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.5.3-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.5.3-linux-x86_64.tar.gz
cd elastic-agent-8.5.3-linux-x86_64
sudo ./elastic-agent install --url=https://eb844be5acaa4359971c6893b1273c64.fleet.us-central1.gcp.cloud.es.io:443 --enrollment-token=cDhNbUc0VUJnNHBmUHkxQU5KMkc6QldyTF9vdm9UalNfRGV3OGIyaHdxdw==
```
The command publishes the following ports:
* 5601: Kibana web interface.
* 9200: Elasticsearch JSON interface.
* 5044: Logstash Beats interface

<b>Access Kibana web interface with </b>http://<host>:5601

<h3>Documentation</h3>

* <a href="https://www.elastic.co/guide/index.html?blade=cloud.elastic.co">Elastic documentation</a>
* <a href="https://www.elastic.co/guide/en/cloud/current/ec-cloud-ingest-data.html?blade=cloud.elastic.co">Indexing data into Elasticsearch</a>
* <a href="https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html?blade=cloud.elastic.co" >Elasticsearch REST API</a>

<h3>Acknowledgments</h3>

* <a href=https://phoenixnap.com/kb/elk-stack-docker>ELK Stack with Docker</a>

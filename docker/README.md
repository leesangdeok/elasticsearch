# Install Elasticsearch with Dockeredit


### Pulling the image
```
$ docker pull docker.elastic.co/elasticsearch/elasticsearch:7.15.2
```


### Starting a multi-node cluster with Docker Composeedit
```
$ docker run -p 127.0.0.1:9200:9200 -p 127.0.0.1:9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.15.2
```


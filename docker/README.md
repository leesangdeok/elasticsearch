# Install Elasticsearch with Docker


### Pulling the image
```
$ docker pull docker.elastic.co/elasticsearch/elasticsearch:7.15.2
```


### Starting a single node cluster with Docker
```
$ docker run -p 127.0.0.1:9200:9200 -p 127.0.0.1:9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.15.2
```
* 16gb 사용 시 -e ES_JAVA_OPTS="-Xms16g -Xmx16g" 추가 가능
* production에서 ES_JAVA_OPTS는 사용하지 않는 것을 권장(노드의 역할과 가용한 메모리에 따라 기본으로 할당)

### Starting a multi-node cluster with Docker Compose
* docker-compose.yml 파일 생성

* Run docker-compose
```
$ docker-compose up
```

* node 확인
```
$ curl -X GET "localhost:9200/_cat/nodes?v=true&pretty"
```

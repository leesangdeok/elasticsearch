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

### Using the Docker images in production
* max_map_count 설정
  * vm.max_map_count 커널 최소한 262144 세팅
  * linux 
  ```
  # 영구적으로 /etc/sysctl.conf에 세팅되어야 한다.
  $ grep vm.max_map_count /etc/sysctl.conf
  vm.max_map_count=262144
  
  # 라이브 시스템에 적용
  $ sysctl -w vm.max_map_count=262144
  ```

* config 파일 권한 설정
  * user elasticsearch는 config 파일을 읽을 수 있어야 하고 keystore 생성을 위해 config, data and log dirs에 작성권한이 있어야 한다.
```
$ mkdir esdatadir
$ chmod g+rwx esdatadir
$ chgrp 0 esdatadir
```

* Increase ulimits for nofile and nproc

* Disable swapping
  * 성능과 안정성을 위해 사용하지 않는다.

* Randomize published ports
  * 기본적으로 TCP ports 9200와 9300를 사용하지만 production clusters에서는 --publish-all 권장

* Pin deployments to a specific image version
  * 특정 버전을 명시하여 사용

* Always bind data volumes
  * /usr/share/elasticsearch/data를 볼륨으로 지정한다.

* Avoid using loop-lvm mode
* Centralize your logs
* Configuring Elasticsearch with Docker
  * docker container 내 config 파일 위치 /usr/share/elasticsearch/config/.
  * config를 개별 파일로 생성한 뒤 변수로 사용가능
  ```
  -e ELASTIC_PASSWORD_FILE=/run/secrets/bootstrapPassword.txt
  ```
  * command line 옵션으로 사용
  ```
  docker run <various parameters> bin/elasticsearch -Ecluster.name=mynewclustername
  ```
* Mounting Elasticsearch configuration file
```
-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```

* Create an encrypted Elasticsearch keystore
  * 기본적으로 자동생성지만 암호화되진 않는다.
  * keystore 암호화하기 
    * bind-mount the config directory (elasticsearch.keystore 직접 bind-mount 금지)
    * elasticsearch-keystore 툴 사용 (옵션 create -p)
    ```shell script
    docker run -it --rm \
    -v full_path_to/config:/usr/share/elasticsearch/config \
    docker.elastic.co/elasticsearch/elasticsearch:7.15.2 \
    bin/elasticsearch-keystore create -p
    ```
    * docker run 사용경 (keystore의 secure settings 추가 및 변경)
    ```shell script
    docker run -it --rm \
    -v full_path_to/config:/usr/share/elasticsearch/config \
    docker.elastic.co/elasticsearch/elasticsearch:7.15.2 \
    bin/elasticsearch-keystore \
    add my.secure.setting \
    my.other.secure.setting
    ```
    * elasticsearch.keystore 파일이 생성되어있는 경우 업데이트 필요없고 컨테이너 실행 시 환경변수 KEYSTORE_PASSWORD를 설

* Using custom Docker image
  * Dockerfile
  ```shell script
  FROM docker.elastic.co/elasticsearch/elasticsearch:7.15.2
  COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/
  ```
  * build & run
  ```shell script
  docker build --tag=elasticsearch-custom .
  docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
  ```

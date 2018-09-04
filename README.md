# About this repository
Docker Swarm을 통해 ELK Cluster를 설치하는 설정을 저장한 저장소

# 필수 사항

## Set Memory lock
- https://www.elastic.co/guide/en/elasticsearch/reference/current/_memory_lock_check.html
- https://www.elastic.co/guide/en/elasticsearch/reference/6.3/setting-system-settings.html#systemd
- https://github.com/elastic/elasticsearch-docker/issues/152

Elasticsearch에 메모리 설정하지 제대로하지 않을 경우 Swap을 사용하게되는데, 위 레퍼런스 문서들을 잘 확인하도록 하자.
문제는 호스트 시스템에 설정된 값을 Dockerd이 그대로 가져가기 때문에, 반드시 먼저 호스트에 다음과 같이 설정을 해주도록 한다.

```
[ -d /etc/systemd/system/docker.service.d ] || mkdir -p /etc/systemd/system/docker.service.d/
echo -e "[Service]\nLimitMEMLOCK=infinity" > /etc/systemd/system/docker.service.d/override.conf
systemctl daemon-reload
systemctl restart docker
```

### 확인
정상적으로 등록되었을 경우 다음과 같이 unlimited로 출력되어야한다.
```
# grep locked /proc/$(ps --no-headers -o pid -C dockerd | tr -d ' ')/limits
Max locked memory         unlimited            unlimited            bytes
```

# Global Environment
다음은 아래 서비스 생성 시 사용할 값들을 환경변수로 저장한다.
```
NETWORK_ELASTIC=elasticsearch
CLUSTER_NAME=graylog_cluster
```

# Overlay network 생성

```bash
docker network create -d overlay --attachable $NETWORK_ELASTIC
```

# Master / Coordinator node 생성

```
docker stack deploy -c docker-compose-stack.yml elastic
```

# Data 노드 생성
Data 노드는 직접적인 Data를 저장하는 노드이기 때문에 비교적 I/O 성능을 필요로 한다.
본 예에서는 다음과 같이 각 I/O가 높은 Docker 노드에 es label로 설정되어졌으며, 각 노드에는 단 하나의 Data 노드가 생성되어지게 함을 예로 보인다.

**Label 설정 예**
```
docker node update infa-swarm-t1101 --label-add es=1
docker node update infa-swarm-t1102 --label-add es=2
docker node update infa-swarm-t1103 --label-add es=3
```

## 첫번째 Data 노드 생성
```
NUM=1
docker service create --name elastic-data$NUM \
  --network $NETWORK_ELASTIC \
  --env "ES_JAVA_OPTS=-Xms2048m -Xmx2048m" \
  --env "cluster.name=$CLUSTER_NAME" \
  --env "network.host=_eth0:ipv4_" \
  --env "node.name=DATA-{{.Node.Hostname}}" \
  --env "node.master=false" \
  --env "node.data=true" \
  --env "node.ingest=false" \
  --env "discovery.zen.minimum_master_nodes=2" \
  --env "discovery.zen.ping.unicast.hosts=tasks.elasticsearch-master" \
  --env "MAX_LOCKED_MEMORY=unlimited" \
  --env "xpack.security.enabled=false" \
  --env "xpack.monitoring.enabled=false" \
  --env "xpack.watcher.enabled=false" \
  --env "xpack.ml.enabled=false" \
  --env "bootstrap.memory_lock=true" \
  --reserve-memory 3G --limit-memory 4G \
  --constraint "node.labels.es==$NUM" \
  docker.elastic.co/elasticsearch/elasticsearch:5.6.9
```

## 두번째, 세번째 Data 노드 생성
> node label에 es=X 가 있는 만큼 증설한다. 예제에서는 3개(es=3까지 있음)의 Data 노드가 있다

```
NUM=2
docker service create --name elastic-data$NUM \
  --network $NETWORK_ELASTIC \
  --env "ES_JAVA_OPTS=-Xms2048m -Xmx2048m" \
  --env "cluster.name=$CLUSTER_NAME" \
  --env "network.host=_eth0:ipv4_" \
  --env "node.name=DATA-{{.Node.Hostname}}" \
  --env "node.master=false" \
  --env "node.data=true" \
  --env "node.ingest=false" \
  --env "discovery.zen.minimum_master_nodes=2" \
  --env "discovery.zen.ping.unicast.hosts=tasks.elasticsearch-master" \
  --env "MAX_LOCKED_MEMORY=unlimited" \
  --env "xpack.security.enabled=false" \
  --env "xpack.monitoring.enabled=false" \
  --env "xpack.watcher.enabled=false" \
  --env "xpack.ml.enabled=false" \
  --env "bootstrap.memory_lock=true" \
  --reserve-memory 3G --limit-memory 4G \
  --constraint "node.labels.es==$NUM" \
  docker.elastic.co/elasticsearch/elasticsearch:5.6.9

NUM=3
docker service create --name elastic-data$NUM \
  --network $NETWORK_ELASTIC \
  --env "ES_JAVA_OPTS=-Xms2048m -Xmx2048m" \
  --env "cluster.name=$CLUSTER_NAME" \
  --env "network.host=_eth0:ipv4_" \
  --env "node.name=DATA-{{.Node.Hostname}}" \
  --env "node.master=false" \
  --env "node.data=true" \
  --env "node.ingest=false" \
  --env "discovery.zen.minimum_master_nodes=2" \
  --env "discovery.zen.ping.unicast.hosts=tasks.elasticsearch-master" \
  --env "MAX_LOCKED_MEMORY=unlimited" \
  --env "xpack.security.enabled=false" \
  --env "xpack.monitoring.enabled=false" \
  --env "xpack.watcher.enabled=false" \
  --env "xpack.ml.enabled=false" \
  --env "bootstrap.memory_lock=true" \
  --reserve-memory 3G --limit-memory 4G \
  --constraint "node.labels.es==$NUM" \
  docker.elastic.co/elasticsearch/elasticsearch:5.6.9
```

# Cerebro 생성
> 본 예제에서는 Manager 노드에만 Cerebro가 생성되도록 하였다.

```
docker service create --name elastic-cerebro \
  --network $NETWORK_ELASTIC \
  --publish "9000:9000" \
  --reserve-cpu 0.5 \
  --reserve-memory 2G --limit-memory 3G \
  --hostname cerebro \
  --constraint "node.role==manager" \
  yannart/cerebro:latest
```

**접속 방법**

```http://<Your one of Swarm manager IP>:9000```

> 접속 시 ES URL을 물어볼 경우 다음과 같이 입력한다.
```http://elasticsearch:9200```


# Kibana 생성
```
docker service create --name elastic-kibana \
  --network $NETWORK_ELASTIC \
  --publish "5601:5601" \
  --reserve-cpu 0.5 \
  --reserve-memory 2G --limit-memory 3G \
  --hostname kibana \
  kibana:5.6.9
```
**접속 방법**
- http://<Your one of Swarm manager IP>:5601



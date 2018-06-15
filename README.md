# About this repository
Docker Swarm을 통해 ELK Cluster를 설치하는 설정을 저장한 저장소

# How to

## Set Memory lock
- https://www.elastic.co/guide/en/elasticsearch/reference/current/_memory_lock_check.html
- https://www.elastic.co/guide/en/elasticsearch/reference/6.3/setting-system-settings.html#systemd
- https://github.com/elastic/elasticsearch-docker/issues/152

Elasticsearch에 메모리 설정하지 제대로하지 않을 경우 Swap을 사용하게되는데, 위 레퍼런스 문서들을 잘 확인하도록 하자.
문제는 호스트 시스템에 설정된 값을 Dockerd이 그대로 가져가기 때문에, 반드시 먼저 호스트에 다음과 같이 설정을 해주도록 한다.

```
echo -e "[Service]\nLimitMEMLOCK=infinity" | SYSTEMD_EDITOR=tee systemctl edit docker.service
systemctl daemon-reload
systemctl restart docker
```

### 확인
정상적으로 등록되었을 경우 다음과 같이 unlimited로 출력되어야한다.
```
# grep locked /proc/$(ps --no-headers -o pid -C dockerd | tr -d ' ')/limits
Max locked memory         unlimited            unlimited            bytes
```

```bash
docker service scale elasticsearch-data=3
```

### ElasticSearch coordinating node
다음 명령은 ElasticSearch coordinating 노드를 5개까지 증가시킨다.

```bash
docker service scale elasticsearch=3
```


## Kibana 접속 정보
- http://<Your one of Swarm manager IP>:5601

## Cerebro 접속 정보
- http://<Your one of Swarm manager IP>:9000
접속 시 ES URL을 물어볼 경우 다음과 같이 입력한다.
- http://elasticsearch:9200

## Clean up containers
다음 명령을 사용하면 사용중인 모든 컨테이너 및 볼륨들을 삭제한다.
```
docker-compose down -v
```


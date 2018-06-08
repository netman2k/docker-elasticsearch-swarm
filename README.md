# About this repository
Docker Swarm을 통해 ELK Cluster를 설치하는 설정을 저장한 저장소

# How to

## Start containers
다음 커맨드는 ElasticSearch 3 master nodes, 3 data nodes, 3 Coordinating nodes, Kibana, Cerebro를 기동시킨다.
```bash
docker stack deploy -c docker-stack.yml elasticsearch
```

## Scaling ElasticSearch
### ElasticSearch Master node
다음 명령은 ElasticSearch master노드를 3개까지 증가시킨다.
```bash
docker service scale elasticsearch-master=3
```

### ElasticSearch data node
다음 명령은 ElasticSearch data 노드를 5개까지 증가시킨다.

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


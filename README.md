# About this repository
Docker Compose 를 통하여 ElasticSearch Cluster를 구성하는 방법을 기록하는 저장소

# How to

## Start containers
다음 커맨드는 ElasticSearch 2 master nodes, 1 data node, Kibana, Cerebro를 기동시킨다.
```
docker-compose up -d
```

## Scaling ElasticSearch
### ElasticSearch Master node scaling
다음 명령은 ElasticSearch master노드를 3개까지 증가시킨다.
```
docker-compose up --scale elasticsearch=3
```
### ElasticSearch data node scaling
다음 명령은 ElasticSearch data 노드를 3개까지 증가시킨다.
```
docker-compose up --scale elasticsearch-data=3
```

## Clean up containers
다음 명령을 사용하면 사용중인 모든 컨테이너 및 볼륨들을 삭제한다.
```
docker-compose down -v
```

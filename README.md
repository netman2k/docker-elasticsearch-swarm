# About this repository
Docker Compose 를 통하여 ElasticSearch Cluster를 구성하는 방법을 기록하는 저장소

# How to

## System setting
### vm.max_map_count 증가
다음 설정을 적용하지 않을 경우, 컨테이너가 실행되지 않을 수 있으니 설정을 반드시 한다.
자세한 사항은 Reference Document를 참조한다.
```
cat <<EOF > /etc/sysctl.d/20-vm-max-map-count.conf 
# https://www.elastic.co/guide/en/elasticsearch/reference/5.6/docker.html
vm.max_map_count=262144
sysctl -p
```

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
## Kibana 접속 정보
- http://localhost:5601

## Cerebro 접속 정보
- http://localhost:9000
접속 시 ES URL을 물어볼 경우 다음과 같이 입력한다.
- http://elasticsearch:9200

## Clean up containers
다음 명령을 사용하면 사용중인 모든 컨테이너 및 볼륨들을 삭제한다.
```
docker-compose down -v
```


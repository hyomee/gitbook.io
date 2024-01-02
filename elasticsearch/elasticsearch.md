# Elasticsearch 환경 수정

## **1.  Elasticsearch 환경 수정**

Elasticsearch를 최초 설치 하면 localhost만 접근이 되어지므로 "/bin/elasticsearch.yml"파일에 있는 Network영역의 network.host를 수정 합니다. ( 모든 ip를 받기 위해서 0.0.0.0 으로 수정 )

```
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
#
#network.host: xxx.xxx.xxx
network.host: 0.0.0.0
```

**2. Virtual Memory 오류 수정**&#x20;

Elasticsearch를 재 실행 하면 다음과 같은 오류가 발생 합니다. 해당 오류는 가상 메모리 부족 오류

```
RROR: [2] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

\
sudo vi /etc/sysctl.conf 으로 확인을 하면 "vm.max\_map\_count = 65530" 으로 확인이 됩니다. 이 값을 변경 하면 해결을 할 수 있습니다.

```
sudo sysctl -w vm.max_map_count=262144
```

&#x20;

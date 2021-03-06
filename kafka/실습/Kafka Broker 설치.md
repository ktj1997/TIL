# 카프카 브로커 설치
- AWS 기준 t2.micro가 적당
  - KafkaBroker, ZooKeeper 설치
  - KafkaBroker: Port 9092
  - ZooKeeper: Port 2181
- 실행을 위해서는 JDK가 필요하다.
  - 둘다 HeapMemory 할당이 필요하다.


## Kafka Broker
- Kafka 서버 이다.
- 여러개가 모여서 Kafka Cluster가 된다.
- Scala 2.12, 2.13의 성능차이는 없다.

### 설치

```shell
$ wget https://dlcdn.apache.org/kafka/3.1.0/kafka_2.12-3.1.0.tgz
$ tar xvf kafka_2.12-3.1.0.tgz
```

### Heap Memory 설정
```shell
## kafka-server-start.sh
if [ $# -lt 1 ];
then
        echo "USAGE: $0 [-daemon] server.properties [--override property=value]*"
        exit 1
fi
base_dir=$(dirname $0)

if [ "x$KAFKA_LOG4J_OPTS" = "x" ]; then
    export KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:$base_dir/../config/log4j.properties"
fi

if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
fi

EXTRA_ARGS=${EXTRA_ARGS-'-name kafkaServer -loggc'}

COMMAND=$1
case $COMMAND in
  -daemon)
    EXTRA_ARGS="-daemon "$EXTRA_ARGS
    shift
    ;;
  *)
    ;;
esac

exec $base_dir/kafka-run-class.sh $EXTRA_ARGS kafka.Kafka "$@"
```

```shell
## Heap Memory 환경변수 설정
export KAFKA_HEAP_OPTS="-Xmx400m -Xms400m"
```

- default 1GB
- 레코드의 내용은 PageCache로 시스템 메모리를 사용한다.
  - PageCache로 Memory를 사용한다.
  - 나머지 객체는 Heap  Memory에 저장한다.
- daemon 옵션으로 백그라운드로 실행해야한다.

### Kafka Broker 실행 옵션 설정
- config 폴더의 *8server.propertis**파일에 운영에 필요한 옵션들을 지정할 수 있다.
  - 설정을 변경하면 재시작 해주어야 한다.
- advertised.listerners 설정을 해주어야 한다.
  - Kafka Client 혹은 CLI를 Broker와 연결 할 때 사용한다.
```properties
## 원래는 주석처리가 되어 있다.
## Kafa Broker가 설치된 IP를 입력해주면 된다.
#advertised.listeners=PLAINTEXT://your.host.name:9092

```

### Kafka Broker 실행
```shell
## 위치: Kafka 설치 폴더
$ bash bin/kafka-server-start.sh -daemon config/server.properties
```
- ZooKeeper를 먼저 실행해야 한다.
- config옵션은 메인 메소드에 전달된다.

***

## ZooKeeper
- 분산 코디네이션을 수행하는 오픈소스 프로젝트
  - 설정 관리: Cluster 설정정보를 최신으로 유지
  - 클러스터 관리: Cluster안의 서버 추가 및 삭제 시의 서버들간의 정보공유
  - 리더 채택: 여러개의 Node중 Leader 채택
  - 락, 동기화 서비스: RaceCondition 방지
- 상용환경에서는 안전한 실행을 위해서 최소 3개의 ZooKeeper로 Cluster를 구성한다.
  - ZooKeeper 한대만을 사용하는 것을 "Qucik-and-Dirty Single Node" 라고한다.

## ZooKeeper 설치
- 설치한 Kafka Binary 폴더에 Zookeeper가 함께 들어있다.

### ZooKeeper 실행
```shell
## 위치: Kafka 설치 폴더
$ bash bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```
### Heap Memory 설정
- default 512MB

***

## Kafka와 Zookeeper 동작확인

## 1. jps
- JVM 프로세스 상태를 보는 도구이다.
- -v 옵션은 JVM에 전달된 인자를 보는 도구이다.
- -m 옵션은 Main 메소드에 전달된 인자를 보는 도구이다.

```shell
$ sudo jps -m

###
# 42389 QuorumPeerMain config/zookeeper.properties
# 42920 Jps -m
# 42762 Kafka config/server.properties
###
```

### 2. API를 통한 확인
- Kafka설치시 Binary 폴더에 kafka-broker-api-versions.sh라는 쉘 스크립트가 있다.

```shell
# 위치: Kafka 설치 폴더
$ bin/kafka-broker-api-versions --bootstrap-server [Kafa 설치 IP:PORT]
```
***
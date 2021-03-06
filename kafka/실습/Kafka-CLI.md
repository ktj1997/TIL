# Kafka CLI
- Kafka를 운영할 때 제일 많이 사용하는 툴이다.
- 실제 Kafka Broker가 설치된 서버에서도 가능하고, 접근이 열려있다면 원격에서도 가능하다.
- topic 관련 설정에는 필수옵션과 선택옵션이 있다. (선택 옵션 일시에는 default가 정해진다.)
  - 옵션에 대해서 세부적으로 살펴볼 필요가 있다.
  - 기본옵션이 환경에 맞지 않을경우 옵션을 설정해주어야 하기 때문이다.  

## server.properties
- kafka에 대한 기본설정을 담는 파일이다.
- /config 폴더 하위에 존재한다.

```shell
$ bin/kafka-server-start.sh config/server.properties
```

### Kafka 정상 실행 확인
```shell
$ bin/kafka-broker-api-version.sh --bootstrap-server [KafkaBroker 실행 IP:PORT]
```

## kafka-topics.sh
- topic 관련된 명령을 실행 할 수 있다.
- topic 생성이 가능하다.

### topic이 생성되는 경우
1. KafkaBroker에 존재하지 않는 Topic을 Producer가 Produce하거나 Consumer가 Consume하는 경우
2. **CLI로 명시적으로 Topic을 생성하는 경우**

### Topic 생성하기
```shell
$ bin/kafka-toipics.sh\
--create \
--bootstrap-server [KafkaBroker 실행 IP:PORT] \
--topic [토픽 이름] 
```

#### 필수옵션
- --create: 토픽 생성 명령어
- --bootstrap-server: Kafka Broker 위치
  - 이전에는 ZooKeeper에 요청을 보냈지만, 2.2버전 이후로는 Kafka와 직접 통신한다.
- --topic: 토픽 이름
  - topic 생성시 유효한 문자는 (영어,숫자,'.',',','-','_')가 있다.
  - '.'와'_'는 충돌 할 수 있기 때문에 하나만 사용해야 한다.

#### 선택옵션
- --partitions: 파티션의 개수
- --replication-factor: 파티션을 복제할 복제 개수 (default:1 --> 사용하지 않겠다.)
  - 실무에서는 2개 혹은 3개의 KafkaBroker를 사용하기 때문에 2혹은 3이 적당하다.
- --config: 추가적인 명령 실행
  - retention.ms: 토픽의 데이터를 유지하는 기간 (default: 1일)

***
### Topic 조회하기

#### Topic 리스트 조회하기
```shell
$ bin/kafka-topics.sh \
--bootstrap-server [KafkaBroker 실행 IP:PORT] \
--list
```

#### Topic 상세정보 조회하기
```shell
$ bin/kafka-topics.sh \
--bootstrap-server [KafkaBroker 실행 IP:PORT] \
--describe --topic [Topic 이름]
```
```text
Topic: hello.kafka      TopicId: XncBEseKQ_OJrP3UMxwQkQ PartitionCount: 1       ReplicationFactor: 1    Configs: segment.bytes=1073741824
        Topic: hello.kafka      Partition: 0    Leader: 0       Replicas: 0     Isr: 0
```

***
### Topic 옵션 수정
- kafka-topics.sh로 파티션 개수 옵션을 수정한다.
  - ```shell
      $ bin/kafka-toprics.sh --bootstrap-server [KafkaBroker 실행 IP:PORT] \
        --topic [topic 이름] \
        --alter \
        --partitions [파티션 개수]
    ```
- kafka-configs.sh로 토픽 retention 기간을 수정한다.
  - ```shell
     $ bin/kafka-configs.sh --bootstrap-server [KafkaBroker 실행 IP:PORT] \
       --entity-type topics \
       --entity-name [topic 이름] \
       --alter --add-config retention.ms=[Retention 시간(ms)]
    ```
***
### Producer
- Topic에 넣는 데이터는 **Record** 라고 부른다.
- Record는 Key:Value로 이루어져 있다.
  - key가 생략되면 기본적으로 **null**로 전송된다.
    - null로 전송될 경우 Batch에 포함된 데이터를 RoundRobin으로 나누어서 파티션에 저장한다.
  - key가 존재할 경우 Key의 Hash에 따라서 파티션이 결정된다.
    - 그렇기 때문에 같은 Key는 항상 같은 파티션에 저장되는 것이 보장된다.
- CLI에서 보낼 수 있는 타입은 String만 가능하다.
  - UTF-8 기반의 ByteArray로 변환된다.
  - 그 이외의 타입은 따로 KafkaProducerApplication을 구성해야 한다.
- kafka-console-producer.sh 를 사용한다.
  - 키를 생략한버전
    - ```shell
        $ bin/kafka-console-producer.sh --bootstrap-server [KafkaBroker 실행 IP:PORT] \
          --topic [topic 이름]
      ```
  - 키를 추가한 버전
    - ```shell
        $ bin/kafka-console-producer.sh --bootstrap-server [KafkaBroker 실행 IP:PORT] \
          --topic [topic 이름] \
          --property parse.key=true \
          --property key.separator=":"
      ```
    - 구분자 default는 \t (tab) 이다.
    - 구분자 지정한 것을 넣지않고 Enter를 입력하면 KafkaException과 함께 종료된다.

***

### Consumer
- Kafka Broker에 저장된 Topic을 확인할 수 있다.
- 기본 보기
  - ```shell
    $ bin/kafka-console-consumer.sh --bootstrap-server [KafkaBroker 실행 IP:PORT] \
      --topic [topic 이름] \
      --from-beginning # 처음부터
- Key와 Value 함께 보기
  - ```shell
      $ bin/kafka-console-consumer.sh --bootstrap-server [KafkaBroker 실행 IP:PORT] \
        --topic [topic 이름] \
        --property print.key = true \
        --property key.separator = ":" \
        --group [consumer group 이름] #Consumer Group이 없다면 새로 생성된다. \ 
        --from-beginning
    ```
    - group 옵션같은 경우에는 ConsumerGroup을 지정한다.
      - ConsumerGroup이 없다면 새롭게 생성한다.
      - ConsumerGroup이 지정되면 가져간 레코드는 **Commit** 처리된다.
      - Commit 정보는 __Consumer_offsets이라는 이름의 내부토픽에 저장된다.
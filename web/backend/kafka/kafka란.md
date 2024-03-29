# 1. Kafka란?
## Event Broker
- Event Broker는 Message Broker의 역할을 할 수 있다. (역은 불가능하다.)
  - MessageBroker에는 RabbitMQ, Redis Queue 등이 있다.
- EventBroker는 Message의 영속성을 제공한다.
  - MessageBroker는 Consume 당시 혹은 짧은 시간 내에 Message 소비 후 삭제한다.
  - 설정에 따라 Message의 영속성이 달라진다.

### 시각화
https://softwaremill.com/kafka-visualisation/

***

***
## 종류

### 1. Apache Kafka
- Apache 2.0을 사용하는 오픈소스
- 수정과 배포가 자유롭다.

### 2. Confluent Kafka
- Kafka를 최초로 개발한 개발자들이 창업 하여 (Confluent) 만든 Kafka 
- 지원하는 기능에 따라 Community 버전과, Enterprise 버전으로 나뉜다.

## Kafka의 탄생
### 탄생배경
```text
2011년 LinkedIn에서 파편화된 데이터 수집 및 분배 아키텍쳐를 운영하는데 어려움을 겪었다.
초기 운영시에 단방향 통신을 통해 Soruce Applcation과 Target Application을 연동하였다.
하지만 시간이 지날수록 아키텍쳐는 거대해졌고, 데이터 전송라인이 급격하게 많아졌고 관리가 힘들어졌다.
이렇게 파편화된 데이터 전송라인은 서비스의 안정된 운영에는 치명적이었고 Kafa가 탄생하는 배경이 되었다.
```
### Kafka의 3가지 특징
1. Event Stream을 안전하게 전송
2. EventStream을 Disk에 저장한다.
3. EventStream을 분석 및 처리

### Kafka 주요 사용 사례
1. Messaging System
2. IOT 디바이스로부터 데이터 수집
3. Application에서 발생하는 Log 수집
4. RealTimeEventStreaming Prodcessing (이상감지 등)
5. DB동기화 (MSA 기반의 분리된 DB 동기화)

### 특징 
1. 중앙집중화 되어있다.
    - Source Application과 Target Application의 사이에 Kafka가 위치한다.
    - 여러곳에서 수집되고 소비되는 데이터를 Kafka 한곳에서 관리 할 수 있게 되었다.
2. 어플리케이션간 의존성을 최소화 한다.
    - 기존의 직접적인 연결은 한쪽의 문제가 다른쪽에 영향을 끼쳤다.
    - Kafka를 통하기 때문에, Publisher는 단순하게 Publish 하고, Consumer는 단순히 Consume만 하면 되게되었다.
3. 데이터 포멧에 제한이 없다.
    - 기본적으로 ByteArray, ByteBuffer, Double, Long, String 타입의 직렬화, 역직렬화를 제공한다.
    - Serializer<T>와 Deserializer<T>를 상속하여 커스텀 하게 직렬화, 역직렬화를 할 수 있다.
4. 빠르고 영속성을 보장한다.
   - 다른 MessageQueue와 다르게 Disk 기반으로 동작한다.
     - Disk에 있기 떄문에 비정상적으로 종료되도 데이터가 남아있다.
   - 파티션 파일은 PageCache를 사용하기 때문에 Disk 기반이지만 빠르다.
     - 파일 I/O를 메모리에서 처리한다.
     - 페이지 캐시를 Kafka만 사용해야 성능상 유리하다. 
   - Broker가 하는 일이 단순하다.
     - 메세지 필터링, 재전송과 같은일은 Broker가 하지 않는다.
     - 단순히 Consumer와 Partition과 매핑처리만 해준다.
5. 확장성이 뛰어나다.
   - 클러스터로 구성한다.
   - 클러스터에서의 Scale-Out, Scale-In이 매우 자유롭다.
   - Scale-Out, Scale-In이 무중단으로 가능하다.
6. 고 가용성을 보장한다.
    - 하나의 Broker에만 저장하는 것이 아닌, 여러개의 Broker에 저장한다.
    - 하나의 Broker에 장애가 있더라도, 다른 Broker들에 데이터가 안전하게 저장되어 있다.
***
## 빅 데이터 에서의 Kafka의 역할

### 빅 데이터란 무엇인가
```text
현대의 IT 서비스는 모든 것을 데이터로 저장한다.
쇼핑몰의 결제내역, 방문한 위치정보, 댓글 등 사용자의 모든 것이 기록된다.
기업에 실시간으로 쌓이는 데이터는 최소 TB에서 EB에 까지 이른다.

빅 데이터의 데이터 형식은 정형데이터부터 비정형데이터 까지 다양하다.
빅 데이터는 기존의 DB에 저장하는 것은 불가능하다.

일단 생성되는 데이터를 모으는 것이 중요한데, 이 것을 [DataLake] 라고 한다
[DataLake]에는 데이터가 모여있을 뿐, 필터링 되거나 패키지화가 되어있지 않다.
[DataLake]는 어떻게 구성할 수 있을까?
End To End 방식은 데이터 전송라인의 파편화를 야기한다.

[DataPipeLine]은 End To End방식의 문제를 해결하고 유연하면서도 확장가능하게 자동화한 것이다.
[DataPipeLine]은 추출(Extracting), 변경(Transforming), 적재(loading)의 과정을 유연하고 확장성있게 자동화한다.

Apache Kafka는 이러한 [DataPipeLine] 구축에 효과적이다.
```

### 높은 처리량
- 많은양의 데이터를 묶음 단위로 배치 처리한다.
  - Producer가 Broker에 데이터를 보낼 떄와 Consumer가 Broker에게서 데이터를 받을 때, 모두 묶어서 전송한다.
  - 네트워크 송수신 비용을 줄일 수 있기 때문에 동일 시간내에 더많은 데이터를 처리할 수 있다.
- 파티션을 통해서 병렬적인 처리가 가능하다.

### 확장성
- 데이터의 양을 예측할 수 없는 빅 데이터의 특성에 맞게, 많은 양이 들어오면 Broker를 늘리는 Scale-Out이 가능하다.
- 데이터의 양이 줄어든다면 Broker의 개수를 줄이는 Scale-In이 가능하다.
- Kafka의 Scale-Out, Scale-In은 무중단 운영을 지원한다.
- Kafka EcoSystem을 갖고있기 떄문에, 다양한 곳에 사용이 가능하다.
  - Connect, Streams ...

### 영속성
- 데이터를 메모리에 저장하지 않고, 파일 시스템(Disk)에 저장한다.
- OS의 페이지 캐시의 존재 덕분에, I/O의 성능이 향상된다.
  - 한번 읽은 파일내용은 메모리에 저장시켰다가, 다시 사용하는 방식을 사용한다.
- 파일 시스템에 저장하기 떄문에, 급작스럽게 종료되더라도 프로세스 재시작 후 안전하게 데이터 처리가 가능하다.

### 고 가용성
- 3개이상의 Kafka Broker(서버)로 이루어진 Kafka Cluster로 운영한다.
  - Cluster 구성의 대수는 제한이 없다.
  - 최소 옵션이 3이다.
- Cluster에서 Replication을 통해서 고 가용성을 가지게 된다.
- On-Premise, Cloud환경 모두 지원한다.

### 신뢰성
- acks, idempotence 등의 개념을 도입함으로 하여, Message의 전달을 보증 할 수 있게 되었다.
***

## Kafka 생태계
  
<img width="1666" alt="스크린샷 2022-09-07 오후 1 18 18" src="https://user-images.githubusercontent.com/57896918/191292427-9f90427b-e8b3-475d-9435-e1bbda611535.png">

### 1. Kafka 
Connect
- DataPipeLine을 운영하는 핵심
- DataSouce와 Kafka를 연결해주는 매개체 이다.
- 다른 시스템에서 Kafka Topic에 Message를 적재하기 위해서 사용된다.
  - Source Connector (Producer): MySQL,S3등의 데이터를 Kafka Topic에 적재하기 위해서 사용한다.
  - Sink Connector (Consumer): Target Application이 Message를 Consume 할 때, Sink 해주는 역할을 한다.

### 2. Kafka Streams
- 내부 Topic을 이용한 파이프 라인 구성에 사용된다.
  - 일반적인 Producer -> Source Connector -> Kafka Broker -> Sink Connector -> Consumer 구조에서는 사용이 불 필요하다.
  - Kafka 내부에서 Topic -> Topic 으로 Message를 넘길 때 사용한다.
- 프레임워크에 종속적이지 않은 라이브러리 이다.
- 주로 아래와 같은 목적으로 사용한다.
  1. 민감한 데이터의 마스킹
  2. 다양한 Source에서 들어오는 데이터의 규격화
  3. Topic 내부의 이벤트 감지

***
## Data Lake 아키텍처와 Kafka의 미래

### Data의 구분

### 1. Batch Data (배치 데이터)
- 초, 분, 시간, 일 등으로 한정된 기간 데이터를 의미한다.
- BatchProssesing(일괄처리) 하는 것이 특징이다.

### 2. Stream Data (스트림 데이터)
- 한정되지 않는 데이터 이다. (UnBounded)

### Data Lake Architecture

#### 0. Legacy Architecture
- End To End로 구성되었다.
- 대용량의 실시간 데이터를 실시간으로 분석하기 어려웠다.
   - 배치를 통해서 데이터를 분석하고 처리했다.
- 데이터가 가공됨에 따라, 히스토리를 추적하기 어려웠다.

#### 1. Lambda Architecture (람다 아키텍처)
- 레거시 데이터 수집 플랫폼을 개선하기 위해 구성한 아키텍처이다.
- 대용량 데이터를 실시간으로 분석하기는 어려우니, 배치로 미리 만든 데이터와 실시간 데이터를 혼합하여 사용하는 방식이다.
- 3가지 Layer로 나뉜다.
  - **Batch Layer**: 배치 데이터를 모아서 특정시간, 타이밍 마다 일괄 계산,처리한다.
  - **Serving Layer**: 가공된 데이터의 저장 장소이다. (User, Application 등이 사용 가능)
  - **Speed Layer**: 분석이 필요할 경우에는 원천 데이터를 실시간으로 분석하는 역할을 한다.
    - **Kafka가 위치하는 영역이다.**
    - User 및 Application이 직접 사용 가능하다.
    - SpeedLayer에서 Serving Layer로 데이터를 저장 시킬 수 있다.
- Layer가 2개로 나뉘는 단점이 있다.
  - 로직의 파편화가 일어난다.
  - 배치 데이터와, 실시간 데이터의 융합할 때 유연하지 못하다.

#### 2. Kappa Architecture (카파 아키텍처)
- 람다 아키텍처를 개선하기위해 구성된 아키텍처이다.
- 모든 데이터를 **SpeedLayer** 에서 해결한다.
  - Batch Processing을 하지 않으므로, 모든 데이터를 Stream처리 해야한다.
  - 모든 데이터의 숫자 같은 것을 붙여서 Batch처럼 처리 가능하게 한다.
  - SPOF가 될 수 있기 때문에, 고가용성 (HA)를 보장해야 한다.
- SpeedLayer에서 처리한 데이터는  ServingLayer로 넘겨진다.

#### 3. Streaming DataLake Architecture (스트리밍 데이터 레이크 아키텍쳐)
- SpeedLayer에서 데이터를 모두 처리하고, Consumer가 직접 접근해서 가져간다.
    - 직접 데이터를 처리하고 저장하므로 단일 진실 공급원 (SSOT)이 될 수 있다.
- 많이 사용되지 않는 데이터도 비싼 자원(Broker의 메모리, 디스크)에 저장한다는 점에서 보완할 부분이 있다.\

# Redis (REmote DIctionary Server)
- Dictionary라는 말 그대로, 대부분의 작업을 **상수의 시간복잡도** 로 자료에 접근할 수 있다.
- ANSI C로 작성되어 있다. (ANSI C Compiler가 있는 곳에서는 동작가능)
  - 백업을 제외한 대부분의 작업을 메모리에서 수행한다.
- 대부분의 언어로 작성된 Client가 존재한다.
- Replication, Clustering, Sentinel 등의 기능을 통해서 고가용성을 제공한다.

## 1. InMemory DataBase
- 서버 재시작시 모든 데이터는 유실된다.
  - 캐시 이외에 사용한다면 적절한 데이터 백업이 필요하다.
- 영속성을 지원하는 In-Memory 저장소이다.
  - Disk에 데이터를 저장 할 수 있다. (영속성)
  - 2가지의 옵션을 제공한다.
    1. 특정 간격으로 Disk에 Snpatshot을 저장하는데 시간이 오래걸린다. **(RDB)**
      - 어느정도 데이터 유실을 감수 할 수 있을 때 사용하자.
    2. 로그 (AOF File)에만 추가한다. **(AOF)**
      - 장애상황 직전까지의 데이터가 보장되어야 할 때 사용하자.
      - 기본설정 (everySec)의 경우 최대 1초의 데이터 유실이 발생할 수 있다.
    3. 두가지 모두사용
      - 강력한 내구성이 필요할 때 사용하자.
- InMemory이기 떄문에 DataSet의 크기가 Memory크기를 넘을 수 없다.
  - 비싸다.
- 모든 데이터는 내부적으로 Hash-Table에 저장된다.
  - HashCollision에는 Chaining을 사용한다.
  - 나중에 들어온 것이 최근 Chain에 존재한다. (Stack?)
  - 각 자료구조 별로 Key에 동일하게 들어가는 것이아니라, 여러가지의 자료구조가 동일 Key에 들어갈 수 있다.
    - 모든 Value가 다 들어가 있는 것이 아니고, 시작하는 값이 들어가 있다.

```text
메모리 기반의 작업이기 떄문에, CPU가 병목현상이 아니다.
Redis의 병목현상은 메모리 또는 네트워크 대역폭 크기일 가능성이 크다.
```


## 2. Single-Thread
- 4개의 쓰레드로 동작한다.
  - 1개의 Main Thread, 3개의 Sub Thread
  - 사용자의 명령어를 처리하는건 Main Thread의 역할이다.
  - 3개의 Sub Thread는 시스템 명령어를 수행한다.
  - UserLevel = SingleThread, Kernel Level = Multi Thread
- 동시성 프로그램이에서 이점을 얻을 수 있다.
- Memcached는 Multi-Thread로 동작한다.
- SingleThread이기 떄문에 많은 부하를 주는 명령어가 있다.
  - 하나의 명령어가 밀리게되면, 그 뒤의 명령어들도 영향을 받는다.
    - O(N) 명령어 사용에 주의하자
      - keys * (모든 Key값 출력)
      - flushall (모든 데이터 삭제)
      - ...
    - 비동기 수행 명령어를 지원해준다.


### 3. I/O Multiplexing
- Asynchronous-Blocking I/O라고 볼수도 있다고한다. (논란의 여지는 있다.)
- 하나의 통신채널로 여러개의 통신을 해결
- 하나의 쓰레드가 여러개의 Socket (파일)을 제어한다.
- File이 사용가능하게 되기 전까지의 과정이 Blocking되는 것을 막는 것이다.

## 4. 주요 사용처
- 캐시
- 데이터베이스
  - Key-Value 형식의 No-SQL 이다.
  - Data에 Expiration을 지정할 수 있다.
- MessageBroker
  - channel 에 publish, subscribe
  - 이벤트를 저장하지 않는다. 
  - 바로 subscriber에게 메세지를 전송한다.
    - 실시간으로 subscribe하지 않으면 메세지를 받을 수 없다.
- 세션관리
  - WAS 외부에서 세션을 관리하는 방식이다.
  - Redis에 세션정보가 존재한다면, 어느 서버로 요청이 가던간에 세션을 통해서 Stateful한 작업이 가능하다.

## 5. MemCached와의 차이

### 공통점
- 캐시 기능을 제공한다.
- In-Memory 이다.
- 세션, 캐시 등 기한이 있지만 빠른성능을 요구하는 역할로 사용된다.

### 차이점
- Redis는 고가용성을 위한 Replication, Clustering을  제공하지만, Memcached는 제공하지 않는다.
- MemCached는 LRU알고리즘을 사용, Redis는 6가지의 알고리즘이 존재한다.
- MemCached는 String 만 사용가능, Redis는 5가지의 자료구조 사용이가능하다.
- MemCached는 고성능 분산 메모리 객체 캐싱 시스템으로, 영속성을 보장하지 않는다.
- Redis는 SingleThread, Memcacheds는 MultiThread로 동작한다.

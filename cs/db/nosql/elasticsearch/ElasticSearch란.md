# 0. ElasticSearch란
- Lucene 기반의 오픈소스 검색엔진
- Json기반의 문서를 저장하고 검색하며, 분석또한 가능하다.
- https://esbook.kimjmin.net/

## [1] 준 실시간 검색 시스템
- 색인된 데이터가 빠르게 검색된다.
  - 섹인된 결과가 Memory에 올라 갔을 때, 검색이 가능하다.
  - refresh-interval 옵션 값으로 지정 가능하다. (default: 1s)
    - 색인 후, 1초가지나면 검색이 가능하다.

## [2] 고 가용성

### Cluster
- 고가용성을 위한 1개 이상의 Node들로 Cluster 구성이 가능하다.
-Cluster 성능을 높이기 위해서, Node의 갯수를 늘릴 수 있다.
  - Node의 갯수와,Cluster의 성능은 정비례하지 않는다.
- 어느 Node에 요청해도 같은 결과를 내려준다.
  - Clsuter기 떄문에 한몸처럼 동작하기 때문이다.
  - 하지만, Node끼리 통신하는데에도 네트워크 비용이 들기때문에, 불필요한 노드에 직접 접근을 차단하는 방법을 고려하는 것이 좋다.

### Node
- Cluster를 구성하는 요소
- Cluster이기 떄문에, 어떠한 Node에 요청을 해도 동일한 응답을 보장한다.
  - 각각의 Node가 본연의 역할에 충실 할 수 있도록 구성하는 것이 중요하다.
- 여러가지의 역할이 있다.

| 종류           | 역할                        |
|:-------------|:--------------------------|
| 마스터 노드       | Cluster 상태관리 && 메타 데이터 관리 |
| 데이터 노드       | 문서 색인 && 검색 요청 처리         |
| 코디네이팅 노드     | 검색 요청 처리                  |
| 인제스트 노드      | 색인되는 문서의 데이터 전처리          |

#### Master-EligibleNode
- Master-EligibleNode는 MasterNode가 다운되었을 때를 대비하여 새로운 Master로 선출 될 수 있다.

### Index
- 색인 과정을 거친 문서가 저장되는 논리적인 공간
- 문서를 저장하기 위해서 필수적으로 필요하다.
- Index 설계가 Elastic Search사용에서 가장 선행되어야 한다.
  - 설계에 따라서, 문서의 구조와 검색 쿼리가 달라진다.
  - 사용 패턴과 문서의 특성에 따라서 설계가 정해져야 한다.

#### 하나의 Index에 여러 문서 종류 vs 여러Index에 하나의 문서 종류
| Case       | 장점                            | 단점                  |
|:-----------|:------------------------------|:--------------------|
| 하나의 Index  | 관리할 Index의 수가 적어진다.           | 쿼리와 문서의 구조가 복잡해진다.  |
| 여러개의 Index | 각 경우에 최적화된 쿼리와 문서구조가 사용 가능하다. | 관리할 Index의 수가 많아진다. |


### Shard
- Index에 색인된 문서가 저장되는 공간이다.
  - index에 색인된 문서들이 저장되는 물리적인 공간
**하나의 Index는 반드시 하나이상의 Shard를 가진다.**
- Index를 생성 할 때 Primary Shard의 갯수 설정이 가능하다.
  - **생성시점 이후에는 PrimaryShard의 개수를 변경할 수 없다.**
    - 재색인을 수행해도 PrimaryShard의 개수를 변경 할 수 없다.
    - 하나의 데이터에 대한 원본이 저장되는 것으로, 하나의 데이터의 Primary shard는 1개이다.
  - Replica Shard의 갯수는 동적으로 변경 가능하다.
- RoutingRule => 문서 Id의 % 연산을 통해서, 각 Shard에 골고루 저장된다.
  - Global Relocation 때문에, Primary Shard의 개수는 고정된다. (default: 1 => 성능에 영향을 많이 미침)
  - 색인을 다시하지 않는이상 바꿀 수 없다. 

| 종류 | 역할                                                                                      |
|:---------------|:----------------------------------------------------------------------------------------|
| Primary 샤드         | - 문서가 저장되는 원본 </br>- 색인과 검색 성능 모두에 영향을 준다.                                              |
| Replica 샤드        | - Primary 샤드의 복제본 </br> - 검색 성능에 영향을 준다. </br> - Primary Share에 문제가 있을시, Priamry로 승격된다. |

```shell
PUT /{INDEX}/_settings
{
  "index": {
    "number_of_shards": 3,
    "number_of_replicas": 2
    }
}
```
- Primary Shard: 3개
- Replica Shard: 6개 (number_of_shrads * number_of_replicas)


### Mapping
- 문서의 구조를 나타내는 정의
  - **ElasticSearch는 Schemaless가 아니라, 미리정의하지 않아도 되는 것 뿐이다.**
- Mapping정보가 생성된 후에, 타입 정보가 맞지않으면 Parsing Error가 발생한다.
- DymaicMapping과 StaticMapping 2가지 모두 사용 가능하다.
- 

#### Dynamic Mapping
- 최초 색인 시점에 ElasticSearch가 동적으로 Schema를 생성해낸다.
- 문자열의 경우 Text, Keyword 모두 정의된다.
  - 어떤 Type인지 모르기 때문이다.
  - 정적 Mapping의 경우, 하나만 명시적으로 정의가 가능하기 때문에 성능에서 유리하다.

#### Static Mapping
- Mapping정보를 미리 수동으로 정의한다.
- 중요한 필드에 타입을 지정해야 할 때 사용한다.
  - ex) int필드의 범위를 넘어서는 숫자값을 지정해야 할 때 
- 불필요한 색인을 방지하기 위해서 사용한다.
  - ex) 문자열 필드가 keyword로 매핑 (default)
```shell
PUT /{INDEX}
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      }
    }
  }
}
```

## [3] 동적 스키마 생성
- Schema를 미리 정의하지 않아도 된다.
  - Schemaless한 것은 아니다.
- Mapping이 Schema구조를 나타낸다.

## [4] RestAPI 기반의 인터페이스 제공
- 특별한 Client를 필요로 하지 않는다.
- REST 기반의 인터페이스를 사용하기 때문에, 진입장벽이 매우 낮다.

## [5] FullText-Search (전문검색)
- 오픈소스 Lucene을 기반으로 만들어졌다.
  - InvertedIndex를 기반으로 한다.
- 내부적으로는 InvertedIndex 기반으로 데이터를 정의하나, Json형식으로 결과를 제공한다.
- Json을 유일하게 제공하기 때문에, 파싱이 필요하다.
  - LogStash를 통해서 Json을 가공하는 편이다.

## [6] 구성
| Elastic Search | RDBMS            |
|:---------------|:-----------------|
| index          | database         |
| mapping        | schema           |
| document       | row              |
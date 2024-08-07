# Index
**RDBMS에서 검색속도를 높이기위한 기술**    
**검색 범위를 최소화 하는 것이 목적**

- Index파일을 만들어서 따로 저장
    - 추가공간이 필요하다. (DB 용량의 약 10%)
    - 데이터와 데이터의 위치를 저장한다.
- Table FullScan이아니라, 인덱스파일을 보고 빠르게 검색
- CUD시의 성능저하가 발생한다.
    - CREATE - 데이터에 맞는 Index 생성
    - DELETE - Index를 사용하지 않는다는 작업을 수행한다.
        - Masking: InnoDB,Postgre,sqlserver
        - Delete: Oracle
    - UPDATE - Index를 사용하지 않는다는 작업을 수행 후, 새로운 Index를 매핑한다.
      - 불필요한 정렬, 재구성을 최소화 하기 위한 방법이다.
  - 분포도(Selectivity), Cardinality(Unique한 데이터의 갯수)
      - 분포도는 데이터가 테이블에 평균적으로 분포되어 있는 정보를 의미한다.
      - 분포도가 좋다는 것은 데이터가 골고루 퍼져있어, 검색 시 불필요한 데이터를 걸러내기 쉽다는 것이다.
    - ```sql
          # Cardinality (Unique 한 정도) 
          SELECT COUNT(DISTINCT(d.SELLER_ID)) FROM delivery d;
          
          # Selectivity (전체에서의 Unique한 비율)
          SELECT cardinality.SELLER_ID, (CONVERT(float,cardinality.total) /  (SELECT count(*) FROM delivery)) * 100 as selectivity
          FROM(
               SELECT d.SELLER_ID, COUNT(*) as total
               FROM delivery d
               GROUP BY d.SELLER_ID
          ) cardinality
          ORDER BY selectivity DESC;
      ```
      
## Index의 구조 (Oracle 기준)
- Key: Index가 걸려있는 값으로 정렬되어 있다.
- RowId: DBA(DataBlockAddress) + Row 주소

### DBA(Data-Block-Address)
- 실제 Record를 찾아가기위한 논리적인 주소이다.
- 물리파일 번호 -> Block번호 -> Block내에서의 순번(Row)
```text
DBA = 데이터 파일(OS에 저장되는 물리파일) 번호 + Block 번호

Block번호 = Data파일 내에서 부여한 상대적 순번

Row번호 =  Block내 순번
```



## 복합 인덱스

- 순서가 중요하다.
- 자주 사용하는 것을 선행 칼럼으로 정의한다.
- 등치조건(=)으로 사용하는 것을 선행 칼럼으로 정의한다.
- 분포도 (Selectivity)가 좋은 칼럼을 선행 칼럼으로 정의한다.

## 인덱스 알고리즘

- HashTable은 O(1)인데 왜 B+Tree, B-Tree를 사용하는가?
    - 단일 접근은 HashTable이 빠르다.
    - 부등호 쿼리 (범위)에서는 HashTable은 정렬이되어있지 않기 떄문에 더 오래걸린다.

### 1. B+ Tree

- B-Tree의 확장모델
    - 당연히 정렬되어있다.
- IndexNode와 LeafNode로 구성
    - BranchNode에 데이터를 저장하지 않기 때문에, 더 많은 포인터 저장 가능 (전체적인 depth가 낮아짐)
- LeafNode에 데이터가 저장됨 (다른 노드들에 데이터가 저장안되기 떄문에 메모리 효율성)
- LeaftNode끼리는 Double-LinkedList로 연결
  - 연속적인 데이터 접근에 유리하다.
- 트리의 높이가 낮아진다.
- FullScan시 선형검색
  - 모든 Leaf들이 연결되어있고, Leaf에만 데이터가 있기 떄문

#### cf) B-tree

- BinaryTree의 확장
    - 당연히 정렬되어있다.
- RootNode, BranchNode, LeafNode로 구성된다.
    - BranchNode와 LeaftNode에 데이터 저장 가능
    - BranchNode에 데이터를 저장하기 때문에 B+Tree에 비해 Depth가 깊어질 수 있으나,
      자주 사용하는 데이터라면 rootNode에 가까이 둘 수 있어 성능상의 이점이 있음
- 한 노드당 2개 이상의 자식이 가능
- 어떠한 값에 대해서도 동일한 접근시간 == 리프노드를 같은 높이에 (균일성)
    - 검색, 삽입, 삭제 모두 O(logN)에 수행 가능
    - 삽입,삭제의 경우에 동적으로 균일성 유지
    - 불균형 트리 최악 시간복잡도 O(n)
    - 균형 트리 최악 시간복잡도 O(logn)
- 항상 정렬된 상태
- 데이터가 많아지면 depth가 늘어남
- FullScan시 모든 노드 탐색

### 2. Hash

- 인덱스의 크기가 작음
- 검색속도가 매우 빠르다.
- **충돌되는 경우가 많다.**

## Index의 종류

### 1. Clustered Index
- 한 테이블에 오직 하나만 존재 가능
- leaf에 실제 데이터가 저장된다.
- 일반적으로 테이블의 물리적인 순서를 결정한다.
    - email을 ClusteredIndex로 만든다면? --> 성능저하
    - ALTER를 통해 많은 데이터가 존재하는 곳에 ClusteredIndex를 설정한다면? --> boom
- default로 PrimaryKey에 지정된다.
  - 1순위는 PrimaryKey, 2순위는 UNIQUE, NOT NULL  
  - PrimaryKey에 강제적으로 NonClusteredIndex 지정도 가능하다.
  - 테이블에 이미 ClusteredIndex가 있으면 PK가 NonClusteredIndex가된다.
- PK가 변경되면 실제로 저장되는 물리적 위치가 변경된다.
  - PK를 신중하게 결정해야 한다.

### 2. NonClustered Index
- Table데이터와 별도로 생성되어 있다.
  - 추가적인 물리공간을 요구한다.
- 테이블 당 여러개가 존재한다.
    - 최대 249개 생성이 가능하다.
- **정렬된 Key 값과, 대응하는 위치정보를 갖는다.**
  - leaf에 실제 데이터의 주소가 저장된다.

### 3. Covering Index
- Index만으로 필요한 데이터를 모두 가져오는 최적화 기법
  - 필요한 데이터를 모두 Index로 선언한다. 
- 테이블엦 접근하지 않기 떄문에, DISK I/O를 줄이게 되고 성능을 향상시킨다.
- Index 크기증가에 따른 오버헤드도 고려되어야 한다.

## 인덱스를 사용해야 할 지점

1. where절에 주로 사용돠는 Column
2. between A and B에 주로 사용되는 Column (ClusteredIndex가 유리)
3. order by에 주로 사용되는 Column
4. Join On 조건에 주로 사용되는 Column
5. ForeignKey (1 : N)에 주로 사용 되는 Column
6. **Cardinality**가 높은곳 (겹치는게 적어야한다. )


## Index 사용시 주의 할 점

### 1. Index의 변형
```sql
SELECT 
    *
FROM 
    Member m 
WHERE
    m.age * 10 = 1;
```
- Index가 걸린 필드의 변형은 Index를 타지 못하게 한다.
- Index가 걸린 필드의 **타입의 오지정** 또한 Index를 타지 못하게 하는 요인이다.

## 2. 복합인덱스의 순서
- 선행하는 Index의 기준으로 정렬 된 후, 그 다음 Index기준으로 정렬한다.
- 중복을 최소화 할 수 있는 순서로 Index를 작성하는 것이 유리하다.

## 3. 인덱스 컬럼의 갯수
- Index는 추가적인 공간을 요구하기 때문에, Column수를 적게 잡는 것이 좋다.

## 4. 하나의 쿼리는 하나의 Index만 타게된다.
- 강제적인 Index 지정 (index merge hint) 사용 시에는 여러개의 Index테이블을 거치게 할 수 있다.
- 기본적으로는 여러개의 Index테이블 탐색이 아닌, 하나의 Index만 타게 된다.
- WHERE, ORDER BY, GROUP BY를 모두 아우를 수 있는 인덱스를 지정하는 것이 중요하다.

## 5. Trade-Off 고려하기
- explain을 통해서 실행계획을 적절하게 확인해야 한다.
- Write의 성능을 희생하여 Read성능을 높이는 것이다.
- Index 외의 다른 방법으로 해결 할 수 있는지 또한 고려해봄직하다.
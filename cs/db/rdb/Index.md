# Index
**RDBMS에서 검색속도를 높이기위한 기술**
- Index파일을 만들어서 따로 저장 (추가 공간 필요)
- Table FullScan이아니라, 인덱스파일을 보고 빠르게 검색
- CUD시의 성능저하가 발생한다.
- 제약조건이 없으면 생성되지 않는다. (PrimaryKey or Unique)
- 선택성(Selectivity)가 중요하다
  - 중복이 많아서는 안된다.                                                  

## 인덱스 알고리즘

### 1. B+ Tree
- B-Tree의 확장모델
- IndexNode와 LeafNode로 구성
  - BranchNode에 데이터를 저장하지 않기 때문에, 더 많은 포인터 저장 가능 (전체적인 depth가 낮아짐) 
- LeafNode에 데이터가 저장됨 (다른 노드들에 데이터가 저장안되기 떄문에 메모리 효율성)
- LeaftNode끼리는 LinkedList로 연결
- 트리의 높이가 낮아진다.
- FullScan시 선형검색

#### cf) B-tree
- BinaryTree의 확장
- RootNode, BranchNode, LeafNode로 구성된다.
  - BranchNode와 LeaftNode에 데이터 저장 가능 
  - BranchNode에 데이터를 저장하기 때문에 B+Tree에 비해 Depth가 깊어질 수 있으나,
    자주 사용하는 데이터라면 rootNode에 가까이 둘 수 있어 성능상의 이점이 있음
- 한 노드당 2개 이상의 자식이 가능
- 어떠한 값에 대해서도 동일한 접근시간  == 리프노드를 같은 높이에 (균일성)
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
- 테이블에 데이터가 삽입되는 순서에 상관없이 Index Key를 기준으로 **정렬**된다.
  - email을 ClusteredIndex로 만든다면? --> 성능저하 
  - ALTER를 통해 많은 데이터가 존재하는 곳에 ClusteredIndex를 설정한다면? --> boom
- default로 PrimaryKey에 지정된다. 
  - PrimaryKey에 강제적으로 NonClusteredIndex 지정도 가능하다.
  - 테이블에 이미 ClusteredIndex가 있으면 PK가 NonClusteredIndex가된다.

### 2. NonClustered Index
- 물리적으로 데이터를 정렬하지 않은 상태이다. (key값만 정렬되어 있다.)
- 테이블 당 여러개가 존재한다.
- ClusteredIndex보다 검색은 느리지만, 삽입,수정,삭제는 빠르다.
- NonClusterdKey는 정렬되지않는다. (포인터 형식)
- leaf에 실제 데이터의 주소가 저장된다. 



## 인덱스를 사용해야 할 지점
1. where절에 주로 사용돠는 Column
2. between A and B에 주로 사용되는 Column (ClusteredIndex가 유리)
3. order by에 주로 사용되는 Column
4. Join On 조건에 주로 사용되는 Column
5. ForeignKey (1 : N)에 주로 사용 되는 Column
6. **Cardinality**가 높은곳 (겹치는게 적어야한다. )
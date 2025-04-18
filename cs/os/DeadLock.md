# DeadLock
- 프로세스가 자원을 얻지 못해 다음 처리를 하지 못하는 상태로, ‘교착 상태’라고 한다.
- 무한정 대기에걸려서 빠져 나오지 못하는 상태이다.

## DeadLock의 필요조건
- 4가지 조건을 만족해야 한다.
- 하나라도 만족하지 못하면 DeadLock이 발생하지 않는다.

### 1. Mutal Exclusion 
- 한 자원을 동시에 점유 할 수 없다.
### 2. Hold & Wait
- 하나를 점유한 상태에서 다른 자원을 대기해야 한다.

### 3. No Preemption
- 한 프로세스가 다른 프로세스의 자원을 뺏을 수 없다.

### 4. Circular Wait
- 각 프로세스는 순환적으로 다음 프로세스가 요구하는 자원을 가지고 있다.


## DeadLock 해결법

### 1. Prevention (예방)
- 위의 DeadLock 필요조건 3개중에 한가지(MutualExclusion 제외)를 부정함으로써 DeadLock을 예방한다.

#### MutualExclusion 부정
- 존재하지 않는다.
- 동시에 접근을 부정하는 것은 동시성문제를 야기시키고, 데이터정합성을 깨지게 만든다.

#### Hold & Wait 부정
- 프로세스가 자원을 요청할 때는 어떠한 것도 Hold하지 않아야한다.
- 실행되기 이전에 실행에 필요한 모든 자원을 요청하고 한번에 할당받아야 한다.

#### No-Preemption 부정
- Hold 하고있는 상태에서 할당받을 수 없는 다른 자원을 요청한다면 Hold하고 있는 자원을 반납하게 하는 것이다.

#### Circular Wait 부정
- 각 Resource에 번호를 부여하고 (Ordering), Hold한 자원보다 높거나 낮은순서의 자원만 요청할 수 있게 한다.

### 2. Avoidance (회피)
- 요청이 들어왔을 떄, 자원을 할당해 주었다고 가정하고 DeadLock 발생 유무를 체크한다.
  - 안전상태(Safe State)와 비안전상태(Unsafe State)를 구분하여, DeadLock이 발생할 수 있는 경우 자원할당(X)
- DeadLock 발생 가능성이 있다면 자원이 있어도 할당 해주지 않는다.
- 은행원 알고리즘 (Banker's Algorithm)
- 자원 할당 그래프
    - 요청선과 할당선으로 이루어진다.
    - Cycle이 존재하면 DeadLock 가능성이 있다.

### 3. Detection (탐지)
- DeadLock 발생을 감지하고 회복시킨다.
  - 자원할당 Graph (Resource Allocation Graph) 검사
- DeadLock Detection 알고리즘을 사용한다.

### 4. Recovery (회복)
- Detection기법을 사용하여, DeadLock이 발생하였음을 확인하면 복구를 진행
- Process를 종료하거나 자원을 회수한다.
# CAP 이론
- 분산 시스템의 특성에 대해서 이야기 하는 이론
- 3가지의 특성 모두를 만족하는 분산시스템은 존재하지 않는다.
  - CA 또한 존재 할 수 없다 : 분산시스템에서 Partition Tolerance는 필수적이기 때문이다.
## 구성요소

### 1. Consistency (일관성)
- 모든 Node들의 데이터가 일관성을 보장해야 한다.
- 모든 Node들은 같은 시간에 동일한 항목에 대하여 같은 데이터를 사용자에게 제공할 수 있어야 한다.

### 2. Availability(가용성)
- 어떠한 상황에서도 사용자는 시스템을 사용 가능해야 한다.
- 한 Node의 장애가 다른 Node에 영향을 끼쳐서는 안된다.
- Node간의 네트워크 문제로 인해 응답이 지연되더라도, 정상적인 처리가 보장되어야 한다.

### 3. Partition-Tolerance(파티션 내성)
- 네트워크 분할과 같은 장애상황 (메세지 전달이 실패) 이거나, 시스템 일부가 망가져도 시스템이 계속 동작할 수 있어야 한다.
- 클러스터간 통신의 실패에도 시스템이 독립적으로 (정상적으로) 동작해야 한다.
- 통신 장애가 해결되면 다시 클러스터링을 수행한다.


## CP 데이터베이스
- Parition이 해결 할 때 까지 분산시스템의 기능이 중지된다.
- Partition이 해결되지 않은 Node를 접근 불가능한 상태로 지정한다.
  - System 일부 Node의 가용성을 포기하는 방식이다.

## AP 데이터베이스
- Partition이 해결되었어도, 중지되어 있던 Node가 최신의 데이터를 리턴하지 않을 수 있다.
- 뒤쳐진 Node는 Sync를 수행하여, 최신성을 유지하려 노력한다.

# Consistency와 Availablity중 양자택일을 해야하는가?
```text
은행 ATM 서비스를 예로 들어보자
ATM은 3가지의 기능을 제공한다.

1. 입금
2. 출금
3. 잔액조회

고객이 ATM1에 입금을하였다.
그 때에 Partition이 발생했다고 가정하자.

이 때에, ATM이 3가지의 동작을 모두 수행한다고 가정하자. (Availability 100 %)
ATM2에서의 잔액 조회시에 데이터 정합성이 깨진 것 처럼 보일 것이다.

그렇다면 어느정도의 Availability를 포기한다면 어떨까?
Partition이 일어난 동안 입금만 가능하다고 가정해보자.

잔액조회를 하지 않으니, 데이터 정합성이 깨졌다고 할 수 없다.
Partition이 해결 되었을 때, 다시 통신을 유지하면서 최종적인 데이터 일관성을 유지할 수 있는 것이다.

실제 서비스에서는 양자택일이 아닌, Consistency와 Availablity의 어디에 중점을 맞추는가에 신경을 쓰게 될 것이다.
```
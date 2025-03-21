# Java에서의 동시성보장
- JVM에서 Thread들은 Static영역과 Heap영역을 공유하게된다.
  - 이 두가지영역에 동시접근 하게 됨으로 해서 동시성문제가 발생하게 된다.

## 0. Singleton
- Singleton 패턴을 통해 생성된 객체는 상태를 갖지 않아야 한다.
- 상태를 갖지않기 때문에 Read만이 일어나며, Write를 하지 않으므로 동시성 문제는 발생하지 않는다.

## 1. Synchronized
- 객체단위의 Lock (MonitorLock) 을 건다.
  - ```java
      class Example{
    
          private int count = 0;
          private final Object lock = new Object(); // 동기화를 위한 별도의 객체 생성
          
          /**
            * Method단위의 synchronzied
            * Method 전체를 Lock을 건다. (this 객체가 Lock에 잡히는 것)
            */
          public synchronized void increment(){
              count++;
          }
    
          /**
            * 객체 단위의 synchronized
            * synchronized 대상 객체(lock)에 Lock을 건다.
            * 이 방식을 통해서, 다른 Method의 수행은 Blocking이 걸리지 않는다.
            */
          public void increment(){
                 synchronized(lock){
                   count++;
                 }
             }
    }
    ```
- blocking을 사용하여 Thread-Safe를 보장한다.
- 성능상의 저하를 유발한다.
  - Thread가 block 에 최초 접근시에 lock을 걸게된다.
  - lock이 걸린 block에 다른 Thread들이 접근하면 Blocking이 된다.
    - Blocking 되는 동안 아무런 작업을 수행하지 않는다.
  - Blokikng에서 Thread의 상태를 바꿔주는 것에도 System Resource가 소모된다.




### wait & notifyAll
```java
class Example {
    private final Object lock = new Object();
    private boolean condition = false;

    public void waitingMethod() {
        synchronized (lock) {
            while (!condition) {
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
            // 조건이 만족되었을 때 수행할 작업
        }
    }

    public void signalingMethod() {
        synchronized (lock) {
            condition = true;
            lock.notifyAll();
        }
    }
}
```
- wait
  - 특정 접근 대상에대한 Monitor(Lock)을 획득 할 때 까지, 현재쓰레드를 wait 상태로 만든다.
  - 다른 쓰레드가 notify() 또는 notifyAll()을 호출할 때까지 기다린다.
- notifyAll
  - wait 상태에 있는 모든 쓰레드를 깨운다.
  - notify()는 wait 상태에 있는 쓰레드 중 하나만 무작위로 깨운다.
  - 여러 스레드가 동일한 조건을 기다리는 경우, notifyAll을 사용하는 것이 안전하다.

## 2. Concurrent
- Java5에 포함된 패키지, 다양한 유틸리티 클래스를 제공한다.
  - Synchronized 보다 훨씬 속도,성능 상에서 유리하다.
    - 기존 Proxy패턴을 사용한 Synchronized Collection은 성능이 떨어진다.
    - ```java
         /**
           * synchronized block을 사용하여 원본 Collection을 호출
           * synchronized가 걸려있어서, 성능 개선이 어려움
           */    
         Collections.synchronizedCollection(new ArrayList<>());
      ```
- 다양한 Collection들이 존재한다. 
  - 멀티 쓰레드 환경에서 순서를 보장하는 Collection은 지원하지 않는다. (이건 synchronized collection 사용)
      - list: CopyOnWriteArrayList (ArrayList 대용)
      - set
        - CopyOnWriteArraySet (HashSet 대용)
        - ConcurrentSkipListSet (TreeSet 대용)
      - map
        - ConcurrentHashMap (HashMap 대용)
        - ConcurrentSkipListMap (TreeMap 대용)
      - queue
        - ConcurrentLinkedQueue (non-blocking-queue)
      - deque
        - ConcurrentLinkedDeque (non-blocking-deque)

## 3. Atomic
- NonBlocking 방식을 사용한다.
- 가시성을 위해서 내부적으로 volatile 키워드를 사용한다.
- CAS(Compare And Swap) 알고리즘을 기반으로 한다.
  - **원자적이지 않은 2개의 연산을 H/W단에서 묶어서 제공**
    - x86: cmpxchg 
    - ARM: ldxr/stxr
  - (기존 값, 변경할 값)을 인자로 변경 요청을 보낸다.
  - 기존 값이 메모리에 있는 값과 일치한다면 true와 함께 변경한다.
  - 기존 값이 메모리에 있는 값과 일치하지 않는다면 false와 함께 변경하지 않는다.
- Synchronized보다는 성능이 좋다.
  - 무한 루프로 Checking을 한다면 CPU사용률은 증가하겠지만, Lock으로 Thread의 동작을 Blocking 하는 것이나,     
    Thread의 상태를 바꿔주는 것이 훨씬 비싼 연산이다.

## 4. Volatile
- Java Multi-Thread 환경에서 가시성에 관련된 문제이다.
- Multi-Thread 환경에서, 한 Thread가 값을 변경했을 때, 다른 Thread들이 모두 변경된 값을 바라보고 있지 않다.
  - 각 Core가 Cache를 가지고 있고, Cache에서 읽고 쓰기 때문이다. (일정 시점 마다 동기화)
- **Volatile은 읽기/쓰기 모두 Main Memory 에서 수행하게 한다.**
- 동시성문제를 완벽하게 해결하지 못한다.
  - 두 작업이 동시에 수행 됐을 때, 선행된 작업이 무시될 수 있다. (덮어쓰기)
  - 동시성문제를 해결하려면 Atomic한 연산이 보장 되어야 한다.
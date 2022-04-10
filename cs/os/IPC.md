# IPC (InterProcessCommunication)
- 프로세스는 독립적인 메모리영역을 가지고 있다.
  - 서로 독립적이기 때문에, 다른 프로세스의 메모리영역에 접근 할수 없다.
- IPC를 통해서 프로세스간 커뮤니케이션을 진행할 수 있다.
- 프로세스간 통신하는 방법이다.

## IPC 구현의 주요 개념
1. SharedMemory
   - 공유할 수 있는 메모리 공간에 데이터를 저장한다.
   - 서로 공유된 메모리에 접근한다.
2. MessagePassing
   - OS에게 위임한다.
   - 다양한 방법으로 구현된다.
     - Direct / Indirect
     - Synchronous / Asynchronous

### 기법
1. MessagingQueue
    - FIFO 전략을 따른다.
    - 양방향 통신이 가능하다.
    - Queue에 저장되는 데이터는 커널에 저장된다.
2. SharedMemory
   - Buffer가 SharedMemory 이다.
   - Producer가 Buffer를 채우고, Consumer가 Buffer를 소비한다.
     - Producer는 Buffer가 꽉 찰 때까지 채운다.
     - Consumer는 Buffer가 빌 때 까지 소비한다.
   - 공유 메모리 Key를 가지고 여러 프로세스가 접근한다.
   - 데이터 자체를 공유한다.
4. Pipe
    - 단방향 통신이다.
    - 여러개의 프로세스가 사용하는 임시공간 (파일)
    - 입구와 출구 개념이 없다.
      - 자신이 발행한 데이터를 자신이 구독 할 수도 있다.
      - Read-Only Pipe, Write-Only Pipe 두개로 나누어 사용한다.
5. Signal
    - 미리 정의된 이벤트이다.
    - 정의된 이벤트에 해당하는 동작으로 이벤트를 핸들링 한다.
6. Semaphore
    - 프로세스간 데이터를 동기화하고 보호하는데 목적을 둔다.
7. Socket
    - 네트워크 통신기술이다.
    - OS 내부의 소켓 기술로 IPC를 구현한다.
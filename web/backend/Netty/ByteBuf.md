# ByteBuf
- Netty의 ByteWrapper
    - Java NIO - ByteBuffer에 비해 풍부한 기능을 제공한다.
    - Netty에서 사용하는 데이터 컨테이너이다.
    - **내부적으로 Unsafe를 이용하여 NativeMemory를 직접 할당하던가, ByteBuffer를 Wrapping 한다.**
- 사용자 정의로 확장 가능하다.
- 최대용량을 지정 가능하다.
    - Integer.MAX_VALUE보다 크게 잡으려고 하면 Exception이 발생한다.
- Pooling(메모리 재사용) 지원
- ByteBufAllocator를 통해 힙 또는 다이렉트 버퍼를 손쉽게 생성할 수 있다.
- duplicate(), slice() 등을 사용해 파생(derived) 버퍼를 생성할 수 있으며, 원본과 내부 데이터를 공유한다.
- copy()는 원본 데이터 내용을 완전히 복사하여 독립적인 버퍼를 만든다.
- PooledByteBufAllocator를 사용하면 내부적으로 메모리 풀을 이용해 성능을 높일 수 있다.
- ZeroCopy 사용가능 (내장 복합 버퍼형식 사용)
- Read / Write 마다 각각 고유한 Index 소유
- Method Chain 지원
- **Reference Counting을 통한 메모리 관리 (내부구현)**
    - allocator에 의해서 할당 될 때, Reference Counting이 증가한다.
    - retain()을 통해서 Reference Counting을 증가시킨다. // 비동기 사용시, Memory 해제를 막기위함
    - release()를 통해서 Reference Counting을 감소시킨다.

## Read / Write Index
- 각각 고유한 Index를 가지고 있다.
  - ReadIndex와 WriteIndex가 같아지면 데이터의 끝이다.
  - 그 이상 읽으려고 하면, IndexOutOfBoundsException이 발생한다.
- **read(), write()는 index를 증가시킨다.**
- **get(), set()은 index를 증가시키지 않는다.**
- index는 capacity를 넘겨서는 안도며, readableBytes(), writableBytes()를 통해서 남은 공간을 확인할 수 있다.


## 주요메소드
- get(index): 특정 Index의 Byte를 가져온다. (rIndex를 증가시키지 않는다.)
- set(index): 특정 Index에 Byte를 쓴다. (wIndex를 증가시키지 않는다.)
- read():     rIndex 위치의 값을 가져온다. (자료형 별로 파생 메소드가 존재한다. / rIndex 증가)
- write():    wIndex 위치에 값을 쓴다. (자료형 별로 파생 메소드가 존재한다. / wIndex 증가)
- isReadable(): 읽을 수 있는 데이터가 있는지 확인한다. (읽을 수 있는 Byte가 하나 이상이면 true를 반환한다.)
- isWritable(): 쓸 수 있는 공간이 있는지 확인한다. (쓸 수 있는 Byte가 하나 이상이면 true를 반환한다.)
- readableBytes(): 읽을 수 있는 Byte의 수를 반환한다.
- writableBytes(): 쓸 수 있는 Byte의 수를 반환한다.
- capacity(): 버퍼의 최대 크기를 반환한다.
- hasArray(): backing array가 있는지 확인한다.
- array(): backing array를 반환한다. (없다면 UnsupportedOperationException이 발생한다.)


## ReferenceCount
- Netty4 버전부터 도입 (ReferenceCounted 인터페이스 도입)
  - ReferenceCounted 인터페이스를 상속
  - 기본적으로 ReferenceCount 1을 가지고 시작
  - ReferenceCount가 0으로 감소하면 instance 해제
- **ByteBuf와 ByteBufHolder에 도입되었다.**
- retain()을 통해서 ReferenceCount를 1 증가시킨다.
- release()를 통해서 ReferenceCount를 1 감소시키며, Memory해제가 되었을 시 true를 리턴한다.
  - release() 후 referenceCount가 0이되면 nativeMemory를 해제시킨다.
- 해제된 객체는 다시 사용할 수 없다.
  - ReferenceCount가 0인 객체에 접근을 시도하면 `IllegalReferenceCountException` 가 발생한다.

### 왜 도입했는가?
- GC의 타이밍에 의존하지않고 직접 개발자가 원하는 시점에 release() 하기 위함이다.

## 사용 Pattern

### [1] HeapBuff
- 보조배열 (backing-array)라고 불리는 패턴이며, 가장 많이 사용된다.
- JVM Heap공간에 할당하며, Polling을 사용하지 않는경우 빠른 할당과 해제 속도를 보여준다.
- Legacy데이터를 처리하는데 적합하다.
  - 대규모 Network I/O작업에는 불리하다.
  - Natvie Memory 자원을 접근하는 경우에 불리하다. (JVM의 도움을 받기 때문)
- hasArray가 false인데 접근하려 하면 UnsupportedOperationException이 발생한다.
- 내부적으로 Socket에 전송할 때 DirectBuffer를 사용하고 있다.


```kotlin
class HeapBufferExample {
    fun main(args: Array<String>){
        // 1. Heap Buffer 생성: Unpooled 힙 버퍼 (backing array 사용)
        val heapBuffer = Unpooled.buffer(10); // 기본적으로 heap memory에 할당

        // 2. 버퍼에 데이터 쓰기
        heapBuffer.writeByte('N'.code.toByte())
        heapBuffer.writeByte('E'.code.toByte())
        heapBuffer.writeByte('T'.code.toByte())
        heapBuffer.writeByte('T'.code.toByte())
        heapBuffer.writeByte('Y'.code.toByte())

        // 3. 현재 버퍼의 상태 출력
        println("Writer index: " + heapBuffer.writerIndex())
        println("Reader index: " + heapBuffer.readerIndex())

        // 4. 버퍼에 데이터가 저장된 Array가 있는지 확인
        if (heapBuffer.hasArray()) {
            val array = heapBuffer.array() // backing array 접근

            // 5. 읽기 시작 인덱스와 길이 계산 (arrayOfset()은 BackingArray내에서 ByteBuff가 사용하는 시작 인덱스를 반환)
            val offset = heapBuffer.arrayOffset() + heapBuffer.readerIndex()
            val length = heapBuffer.readableBytes()

            println("BackingArray contains: ")
            for (i in offset until offset + length) {
                print(array[i].toChar()) // 배열에서 직접 읽기
            }
            println()
        }

        // 6. 버퍼에서 데이터 읽기
        println("Heap Buffer contains: ")
        while (heapBuffer.isReadable) {
            print(heapBuffer.readByte().toChar())
        }

        // 7. 메모리 해제 (GC가 관리해주기 떄문에 필수는 아니다)
        heapBuffer.release() // 명시적으로 메모리를 해제
    }
}
```

### [2] DirectBuff
- BackingArray가 존재하지 않는다. (DirectBuffer -> NativeMemory이기 때문)
- 대규모 파일 I/O, 고성능 Network Server/Client 구현에 적합하다.

```kotlin
class DirectBufferExample {
    fun main(args: Array<String>) {
      // 1. Direct Buffer 생성
      val directBuffer = Unpooled.directBuffer(10); // Direct 메모리에 할당된 ByteBuf 생성

      // 2. 버퍼에 데이터 쓰기 (byte 값을 int로 자동 형변환)
      directBuffer.writeByte( 'N'.code); // 'N' 기록, writerIndex 증가
      directBuffer.writeByte( 'E'.code); // 'E' 기록, writerIndex 증가
      directBuffer.writeByte( 'T'.code); // 'T' 기록, writerIndex 증가
      directBuffer.writeByte( 'T'.code); // 'T' 기록, writerIndex 증가
      directBuffer.writeByte( 'Y'.code); // 'Y' 기록, writerIndex 증가

      // 3. 현재 버퍼의 상태 출력
      println("Writer index: " + directBuffer.writerIndex());
      println("Reader index: " + directBuffer.readerIndex());
      println("HasArray: " + directBuffer.hasArray());

      // 4. 버퍼에서 데이터 읽기
      val bytes = ByteArray(directBuffer.readableBytes()); // 10바이트 크기의 배열 생성
      println("From Copied Array: "); // 복사된 배열 출력
      directBuffer.getBytes(directBuffer.readerIndex(),bytes)
      for(byte in bytes) {
        print(byte.toChar());
      }
      println()

      println("From Direct Buffer: "); // NETTY 출력
      directBuffer.readerIndex(0); // readerIndex를 처음부터 읽도록 설정
      while (directBuffer.isReadable) { // 버퍼에 읽을 수 있는 데이터가 있는 동안
        print(directBuffer.readByte().toChar()); // 데이터 읽기
      }
      // 5. 메모리 해제
      directBuffer.release(); // 명시적으로 메모리를 해제
    }
}
```
### [3] CompositeBuff (복합 버퍼)
- 여러 Buffer를 논리적인 단일 Buffer로 관리한다.
  - Instance가 하나만 존재한다면 hasArray는 해당 Instance의 것을 따르며, 여러개일 경우는 false를 리턴한다.
  - DirectBuff 패턴을 이용하는것이 유리하다.
- Memory복사를 줄이는 효율성을 제공한다.
- JDK의 기능이 아닌 Netty의 기능이다.
- DataFrame 조합, Message 조립등을 할 때 유용하다.
- CompositeBuff 단의 rIndex, wIndex와 각각의 rIndex,wIndex 모두 존재한다.

```kotlin
/**
    HEADRBODY-DATA1
    All component: CompositeByteBuf(ridx: 0, widx: 15, cap: 15, components=2)
    First component: UnpooledDuplicatedByteBuf(ridx: 0, widx: 5, cap: 5, unwrapped: UnpooledByteBufAllocator$InstrumentedUnpooledUnsafeHeapByteBuf(ridx: 0, widx: 5, cap: 5))
    Second component: UnpooledDuplicatedByteBuf(ridx: 0, widx: 10, cap: 10, unwrapped: UnpooledByteBufAllocator$InstrumentedUnpooledUnsafeHeapByteBuf(ridx: 0, widx: 10, cap: 10))
 */
class CompositeBufferExample {
    fun main(args: Array<String>) {
        // 1. CompositeByteBuf 생성
        val compositeBuffer = Unpooled.compositeBuffer();

        // 2. 개별 ByteBuf 생성
        val headerBuf = Unpooled.buffer(5);
        headerBuf.writeBytes(byteArrayOf('H'.code.toByte(), 'E'.code.toByte(), 'A'.code.toByte(), 'D'.code.toByte(), 'R'.code.toByte()));

        val bodyBuf = Unpooled.buffer(10);
        bodyBuf.writeBytes(byteArrayOf('B'.code.toByte(), 'O'.code.toByte(), 'D'.code.toByte(), 'Y'.code.toByte(), '-'.code.toByte(), 'D'.code.toByte(), 'A'.code.toByte(), 'T'.code.toByte(), 'A'.code.toByte(), '1'.code.toByte()))

        // 3. CompositeByteBuf에 ByteBuf 추가
        compositeBuffer.addComponents(true, headerBuf, bodyBuf); // true는 읽기 포인터를 자동으로 설정함
        println("All component" + compositeBuffer.toString())
        // 4. CompositeByteBuf에서 전체 데이터 읽기

        for (i in 0 until compositeBuffer.readableBytes()) {
            print(compositeBuffer.getByte(i).toChar()) // 전체 데이터를 하나의 버퍼처럼 읽음
        }
        println()

        // 5. 개별 ByteBuf에 대한 접근
        println("First component: " + compositeBuffer.component(0).toString())
        println("Second component: " + compositeBuffer.component(1).toString())

        // 6. 메모리 해제
        compositeBuffer.release(); // 모든 구성 요소의 참조 카운트를 감소시키고 필요 시 메모리 해제
    }
}
```

## ByteBufAllocator 인터페이스
- ByteBuf를 생성하는 팩토리
- Heap, Direct, Composite 버퍼를 생성할 수 있다.
- Channel에서 얻거나, ChannelHandlerContext에서 얻을 수 있다.

### PooledByteBufAllocator
- **Netty ChannelPipeLine, ChannelHandler에서 사용된다.**
- ByteBuf의 재사용이 가능해진다.
- 메모리 재사용을 통해 GC부담이 감소한다.
  - jemalloc이라는 고효율 메모리 할당 방식을 사용한다.
- Buffer의 release()에 대한 관리가 필요하다.
  - **MemoryLeak이 발생하는 주요 원인이다.**
  - ReferenceCount가 0이되면 다시 Pool로 들어간다.

```kotlin
val allocator = PooledByteBufAllocator.DEFAULT

val buffer: ByteBuf = allocator.buffer(256) // default: DirectByteBuf
buffer.writeBytes(byteArrayOf(1, 2, 3, 4, 5))

println("Buffer capacity: ${buffer.capacity()}") // 256 출력

buffer.release() // Reference Count 감소 (메모리 반환)
```

### UnpooledByteBuf
- Unpooled라는 유틸리티 클래스로도 접근 가능하다.
- Pooling을 사용하지 않는다.
- 매번 새로운 ByteBuf 객체를 생성
- 작은 크기의 단기적인 버퍼를 사용할 때 유리
- 명시적인 release() 호출이 아니어도 GC의 대상이 된다. (Pool에 들어가지 않기 때문)
```kotlin
val allocator = UnpooledByteBufAllocator.DEFAULT

val buffer: ByteBuf = allocator.directBuffer(128) // Direct 메모리에 128바이트 크기의 ByteBuf 할당
buffer.writeBytes(byteArrayOf(10, 20, 30, 40))

println("Buffer capacity: ${buffer.capacity()}") // 128 출력

buffer.release() 
```

### 구현체
1. PooledByteBufAllocator (Default)
   - Pooling을 사용하여 ByteBuf를 생성한다.
     - 같은 ByteBuf를 여러군데서 재사용 가능하게한다.
     - ReferenceCount를 사용하며, ReferenceCount가 0이 되면 Pool로 반환한다.
   - jemalloic이라는 기법을 사용한다.
2. UnpooledByteBufAllocator
   - Pooling을 사용하지 않고, 새로운 ByteBuf를 생성한다.
   - Unpooled 유틸리티 클래스를 통해서 생성가능하다.
   - 다른 NettyComponent가 필요없는, Network와 무관한 프로젝트에서 ByteBuf를 쉽게재공해주는 API이다.

## ByteBufHolder 인터페이스
- 주로 Network 프로그래밍에서 사용된다.
- 내부적으로 ByteBuf를 가지는 컨테이너이다.
- MetaData를 캡슐화하고 관리할 때 유용하다.
  - ex) 실제 컨텐츠와 함께 상태코드, 쿠키도 같이 저장
- 주요 메소드
  - content(): ByteBufHolder에 저장된 ByteBuf를 반환
  - copy(): 포함된 ByteBuf 데이터와 **공유되지 않는 복사본**이 들어있는 ByteBufHolder 복사본을 반환
  - duplicate(): 포함된 ByteBuf 데이터의 **공유된 복사본**이 들어있는 ByteBufHolder의 복사본을 반환

## ByteBufUtil 클래스
- ByteBuf 조작과 관련된 유틸리티 클래스
- Pooling과는 상관없으며, ByteBufAllocator 인터페이스와 별개로 범용적으로 사용된다.
- hexdump(): 16진수로 표현
- equals(): ByteBuf 비교
- ...


## Byte Compact
```text
+---+---+---+---+---+---+---+---+---+---+
| X | X | X |   |   |   |   |   |   |   |
+---+---+---+---+---+---+---+---+---+---+
  0       ^           ^               ^
          |           |               |
       rIndex       wIndex         capacity
```
- discardBytes()를 통해서 이미 읽은 Byte를 폐기하여 공간을 확보 할 수 있다.
- **자주 호출하여 충분한 공간을 확보하는 것이 좋을 것 같지만, 위 동작은 Memory Copy를 유발하므로 성능상 좋지 않다.** 

## 주의 할점

### [1] ByteBuff간의 Read/Write
```kotlin
    val sourceBuffer = Unpooled.buffer(10)
    val targetBuffer = Unpooled.buffer(10)


    targetBuffer.writeBytes(sourceBuffer) // readerIndex에서부터 데이터를 복사
```
- Index를 지정하지 않으면, 복사 크기만큼 sourceBuffer의 rIndex가 증가한다.
- Index를 지정해주면 해당 Index부터 복사가 시작된다. (rIndex는 증가하지 않는다.)

### [2] 파생 Buff
- 원본에 영향을 미치기 때문에 위험하지만 성능때문에 사용되는 측면이 크다.
  - **데이터는 공유되지만 rIndex와 wIndex는 별도로 관리된다.**
- 아래와 같은 명령어로 파생 ByteBuff 생성이 가능하다.
  - duplicate()
  - slice()
  - Unpooled.unmodifiableBuffer()
  - order(ByteOrder)
  - readSlice()
- 파생 Buffer는 rIndex, wIndex, markIndex를 포함하는 새로운 ByteBuf 인스턴스를 반환한다.
- **생성비용이 저렴하지만 내부의 모든 정보는 공유되기 떄문에, 파생Buffer의 변경은 원본에 영향을 미친다.**
- **기존 Buff의 복사본이 필요하다면, copy()를 사용해야 한다.**

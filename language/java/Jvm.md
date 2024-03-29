# JVM

## JVM(Java Virtual Machine)
- JVM은 Class 파일을 읽어서 실행하는 역할을 한다.
- **1 Applcation : 1 JVM Instance이며, 각각은 독립적이다.**
  - 필수적인 것은 아니며, 공유를 가능하게 할 수도 있다 (Multi-Tenancy).
- JVM의 존재 덕분에, 다른 OS에서 작성된 Java파일(+class파일) 동일하게 실행이 가능하다.
- OS에 종속적이며, Java파일을 읽는 것이아닌, Class파일을 읽는 것이기 때문에, Java라는 특정언어에 종속되는 것은 아니다.

## JRE(Java Runtime Environment)
- 자바 실행에 필요한 환경을 정의한 것으로 JVM + JAVA Library로 구성된다.

## JDK(Java Development Kit)
- 자바 프로그래밍에 필요한 개발툴 + JRE 이다.
- JAVAC(컴파일러) 등이 포함된다.

![dd](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc00klf%2FbtqAjMzLyF2%2F6sU1VGp5vqAYIPLsXpakpK%2Fimg.png)

### JVM 구조

![ff](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile22.uf.tistory.com%2Fimage%2F9973563D5ACE0315215FF6)

Java Source File (.java) —> JAVAC —> Java Byte Code (.class File)

### Class Loader (클래스 로더)

1. Java Byte Code를 읽어 메모리의 적당한 위치에 저장
2. static 값들 초기화 및 변수에 할당
3. 래퍼런스 연결
4. Heap 영역에 클래스 들의 Class객체를 생성

### Memory(메모리)

1. **Stack(스택)**
    1. Thread마다 생성된다.
    2. Primitive Type이 저장된다.
    3. Heap 에 할당되는 참조에 대한 값(주소)들이 저장된다.
    4. Method 실행에 필요한 Method Call Stack

2. Method (메소드)
    1. 공유자원
    2. Class 정보 (FQCN ,클래스 이름, 부모클래스 이름, 메소드 정보,변수 정보) 저장

3. Heap(힙)
    1. 공유자원
    2. 객체(Reference Type) 저장
    3. Garbage Collection의 대상

4. PC(Program Counter)

    1. Thread마다 생성
    2. 현재 실행할 스택 프레임을 가르키는 포인터

5. Native Method Stack(네이티브 메소드 스택)
    1. Thread마다 생성
    2. Native Method(Java가 아닌 다른 언어로 작성된 Method) 실행을 위해 Stack에 쌓음

### Execute Engine(실행엔진)
1. Interpreter가 Java Byte Code를 기계어로 바꿔서 한줄 씩 실행
2. 인터프리터 방식의 문제점 (중복되는 코드여도 한줄 씩 읽어서 실행)을 JIT컴파일러로 보완 (자주사용되는 코드(기계어)를 캐싱)
3. GC가 사용하지 않는 객체의 자원을 회수
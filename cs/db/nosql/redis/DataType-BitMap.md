# DataType-BitMap
- BitMap 말 그대로, 0과 1만을 저장한다.
  - Redis 내부적으로는 Bit 문자열로 저장된다.
- 많은 양의 데이터가 있을 경우, 다른 자료구조에 비해서 상당한 Memory를 절약 할 수 있다.
  - 실시간으로 단순한 데이터를 쌓는 용도로 사용하면 좋다.
  - 일일 접속자
  - 방문자 체크 
  - ...
- Element를 찾을 수 있는 Bit의 Key값을 Offset이라고 한다.
  - offset은 0부터 시작한다.


## Set과의 비교
| 자료구조   | Element당 비트 크기 | 지정한 Element 수 | Memory 소모               |
|:-------|:---------------|:--------------|:------------------------|
| BitMap | 1bit           | 5백만           | 1 * 5000000bits = 625KB |
| Set    | 32bit          | 2백만           | 32 * 2000000bits = 8MB  |

### [1] SETBIT
- Offset에 해당하는 Bit를 저장하는 명령어이다.
- 기존에 갖고있던 Bit값을 리턴한다.
  - 기본 값은 0
  - 기존에 offset에 지정한 값이 없고, 새롭게 지정했더라도 0이 리턴된다.
- O(1) 이다.
```shell
SETBIT KEY OFFSET VALUE

> setbit visitor:2023-01-27 0 1
(integer) 0
```

### [2] GETBIT
- Offset에 해당하는 Bit를 가져오는 명령어이다.
- 저장된 값이 없는 offset을 가져오면 0이리턴된다. (default)
- O(1)이다.
```shell
```shell
GETBIT KEY OFFSET

> getbit visitor:2023-01-27 0
(integer) 1
```
```
# Thrashing (쓰레싱)
- PageFault가 과도하게 발생하는 상황이다.
- 실제 실행보다 Page 교체에 더 많은 시간을 소모한다.
- CPU의 처리 효율이 급격히 감소하며 심각한 성능 저하를 유발한다.

## Page
- 가상 메모리의 고정분할 크기 블록

## Frame
- Page크기만큼 물리 메모리에서 분할한 블록

## 원인
1. CPU의 이용률을 높이기위해서 CPU는 Process를 추가한다. (MultiProgramming)
2. 프로세스마다 가질수 있는 Page의 개수가 줄어든다.
3. PageFault가 일어나면 Paging (Page 교체)이 필요하다.
    - 활성 Page가 교체되면서 연쇄적인 Page Fault가 발생할 수 있다
4. 전체적인 Paging이 일어나며, 심각한 성능저하를 일으킨다.


## 해결방법

### 1. Working Set Model
- 지역성을 활용한다.
  - Process가 실행되는 동안 참조지역이 특정 부분에 집중되는 현상을 이용한다.  
- 가장 많이 사용되는 페이지를 메모리공간에 계속 상주시켜 Thrashing을 예방한다.

### 2. Page Fault Frequency
- PageFault의 빈도를 조절하는 방법이다.
- PageFault > 상한선 --> Frame 추가
- PageFault < 하한선 --> Frame 회수
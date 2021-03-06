# JPA (Java Persistence API)
- Java ORM의 표준 명세
- java.persistence 하위 패키지에 존재한다.
- 명세이기 떄문에 구현을 갖고있지 않으며, Interface, Enum등이 대부분이다.
   - 대표적으로 EntityManager가 JPA의 인터페이스이다.

## JPA 등장 배경
1. **SQL 의존적개발**
```
   아무리 간단한 기능이라도, 모든 쿼리를 작성해야했으며,
   엔티티의 변경(칼럼추가/삭제, 연관관계 추가) 등에 취약했다.
```
2. **패러다임의 불일치**
```
    Java코드는 객체지향에 기반을 두고있고, 추상화 상속 등을 이용해서 코드를 작성하며,
    연관관계 또한 참조라는 기능을 통해서 연관관계를 표현 할 수 있었다.
    
    하지만 관계형 데이터베이스는 데이터 중심으로 구조화 되어있었으며,
    연관관계에 FK를 사용했으며, 상속 추상화등을 사용하지 않았다.
    
    이러한 패러다임의 불일치문제는 개발자의 부담을 가중 시켰다.
```
***
## JPA란
1. **정의**
```
    ORM (Object Relatinal Mapping)으로, 
    코드는 코드대로 객체지향에 맞게 설계하고, DB는 DB대로 설계해도
    중간에서 자동으로 Mapping을 수행해준다.
    
    Java Application과 JDBC사이에 위치한다.
    (JPA가 JDBC API를 사용한다)
```

2. **구현체**
    1. Hibernate (main)
    2. EclipseLink
    3. DataNucleus
    

3. **장점**
    1. 객체지향에 기반한 개발 가능
    2. 생산성 및 유지보수 효율 증대
    3. 패러다임의 불일치 문제 해결 (상속, 객체탐색 그래프, 연관관계 ...)
    4. 성능
    5. 자바 표준

## SpringDataJPA
- JPA를 사용한다.
- Repository개념을 사용한다.
- JPA를 쉽게 사용하기 위해서 한단계 더 추상화 한 것이다.
- JPA라는 인터페이스에 의존하므로, 세부 구현체에 대해서는 알지 못한다.
    

## 계층도
![SpringDataJpa PSA](https://user-images.githubusercontent.com/57896918/158820591-088dc7a8-be16-4a42-a537-98bd7de0f6c8.png)

# Cloud
```text
가상화와 같은 추상화 기술 또는 프로세스는 물리적 하드웨어에서 리소스를 분리하여
사용자가 필요할 떄 원하는 기능에 엑세스 할 수 있게 동작한다.
```

## Public Cloud
- AWS, Azure, GCP ...
```text
특정 기업이나 사용자를 위한 서비스가 아닌, 인터넷에 접속가능한 모든 사용자를 위한 클라우드 서비스 모델이다.
클라우드 서비스 제공자가 하드웨어, 소프트웨어를 관리하게 된다.

데이터나 기능, 서버 같은 자원은 각 서비스에서 사용자별로 관리되거나 격리되어있다.
클라우드 서비스 제공자에게 전적으로 의존하기 떄문에, 장애시에 서비스에 문제가 생길 수 있다.
```

## Private Cloud
- OpenStack, CloudStack, VM ware ...
```text
제한된 네트워크 상에서, 특정 기업이나 특정 사용자만을 대상으로 하는 클라우드 서비스
기업이 자원의 제어권을 갖고있다.
보안성이 매우 뛰어낙, 개별고객의 상황에 맞게 클라우드의 기능을 커스터 마이징 할 수 있다.
제한적인 자원을 활용하기 떄문에, 일시적인 이벤트에 대한 비용부담이 높다.
```


## Hybrid Cloud
- Public Cloud와 Private Cloud를 병행해서 사용하는 방식
```text
Public Cloud의 유연성,신속성과 Private Cloud의 보안성, 안정성을 함께 취할 수 있는 방법이다.
주요 데이터에 관한 서버들은 On-Premise환경에 두고, 
이벤트 또는 신규서비스 처럼 트래픽을 예측 할 수 없는 워크로드는 Cloud로 이전하는 방식이다.
```

## Multi Cloud
- 서비스 환경에 적합한 리소스 활용을 위해서 사용되는 방안이다.
```text
기존의 개념은, 하나의 Public Cloud에 의존하고 있을 때,
해당 서비스 제공자의 장애상황에 서비스가 영향을 받는 것을 방지하고자,
일종의 이중화였으나, 두 클라우드 서비스 제공자간의 서비스 이전은 현실적으로 어려움이 있었다.

최근에는 장애 대응의 일환이라기 보다는
서비스의 특성에 맞는 Public Cloud를 사용하는 방안으로 변화 하였다.

ex) 웹/게임 -> AWS, Big Data -> GCP 
```
# CIDR (Classless-Inter-Domain-Routing)
- IP Class를 대체하였다. (1993년 부터 표준 IP주소 할당 방식)
- Class 체계보다 더 유연하게 IP를 나눌 수 있게 되었다.
    - 클래스로 고정되어 있지 않으니, Host를 유동적으로 나눌 수 있다.
    - Network 영역과, Host 영역을 유연하게 나눈다.
- IP와 서브넷 마스크를 함께 표기한다.
  - IP/서브넷 마스크 (210.77.8.155/26)
  - 서브넷은 앞에서부터 1의 개수이고, 네트워크 영역이다.
  - 나머지 0의 개수는 Host 영역 이다.

### SubnetMask (서브넷 마스크)
- 해당 IP의 Network주소를 알아내기 위해서 사용한다.
  - IP주소와 SubnetMask를 &연산을 하면 Network 주소를 얻을 수 있다.
- Network 영역을 모두 1로, Host영역을 모두 0으로 표시한다.
- /이후에 Network영역을 가지는 비트의 숫자를 표기하는 것으로도 표현한다.

![CIDR](https://user-images.githubusercontent.com/57896918/160597364-d9b69939-c71b-4b73-9eb6-43a249696542.png)

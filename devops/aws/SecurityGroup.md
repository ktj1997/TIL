# SecurityGroup
- Ec2 Network 보안의 핵심이다. (Firewall)
- Ec2인스턴스의 Inbound/Outbound 트래픽에 대해 정의한다.
  - 기본적으로 모든 Outbound트래픽은 허용되며, Inbound 트래픽은 차단된다.
- **SecurityGroup은 서로를 참조할 수 있다.**
- 하나의 SecurityGroup이 여러 Instance에 설정 될 수 있다.
- **Region과 VPC조합에만 국한된다.**
  - 다른 Region에 세팅할 경우 새로만들어야 한다.
- 정책
  - Port 접근허용
  - 허용할 IP 대역
    - CIDR 사용 가능
  - ipv4/ipv6 여부
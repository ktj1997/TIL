# CDN (Content Delivery Network)
- 지리적으로 분산된 여러개의 서버
- 정적 컨텐츠(웹 페이지, 이미지, 비디오)를 사용자와 가까운 프록시 서버에 캐싱한다.
  - 물리적으로 가까운 서버와의 통신으로 인해 빠른 응답을 기대할 수 있다.
- 원본 서버에 대한 부하분산이 가능하다.
- CDN 서버에 요청한 컨텐츠가 없는 경우 원본서버에서 가져와 저장 한후 컨텐츠를 리턴해준다.(Read-Thorugh)
- 적절한 만료시간을 설정해야 한다.
  - 너무 짧은 시간은 원본서버에 잦은 접속으로 인해 캐싱의 효과가 떨어진다.
  - 너무 긴 시간은 컨텐츠의 신선도에 영향을 끼치게 된다.
 
![afaf](https://user-images.githubusercontent.com/57896918/167162853-c484c0d5-6941-4d5d-9c66-352eb5cec0d3.png)
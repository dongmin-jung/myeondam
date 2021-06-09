# HyperCAS 현황과 계획

## HyperCAS 큰 그림

- 구조에 대한 모니터링
- 성능에 대한 모니터링
- 보안에 대한 모니터링 -> HyperESM

## 시행착오

- 초기에는 Operator SDK - Go 로 하다가 Java로 바꾸었음
  - Operator SDK에서 제공하는 기능은 Custom Resource에 대한 Controller 기능에 집중되어 있음
    - CR들을 watch하고, 생성/변경/삭제가 감지된 CR의 namespace와 name을 전달받아 CR과 연관된/소유하는 리소스에 대한 reconcile
    - 목적에 맞게 watch 대상들을 특정하기 어렵고, watch한 리소스의 namespace와 name을 받아도 그게 어느 리소스였는지 알 수가 없음
  - ODBC를 통한 Tibero와의 연동이 잘 안된다???

## 현황

- HTTP POST 요청을 통해 watch 대상 kube서버주소와 리소스URI를 받아 등록한다
  - 이때 bearer token과 ca cert도 같이 받는다.
- kube 서버에 리소스 정보를 요청하고 chunked response으로 받는다. (json)
  - 왜 안 웹소켓?
  - 커넥션 많은거 문제 안 될지?
- json을 xml로 변환하여, 해당 리소스의 히스토리를 DB에 삽입한다.
  - Tibero가 지원하는 xpath query를 사용하기 위함

## 계획, 고민

- hibernate 도입 / xml 말고 그냥 json으로 하면?
- rule, relation rule DB 만들어야 하고 get할 때 쿼리 고민해야 한다.

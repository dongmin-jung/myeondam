# HyperCAS CMDB 설계 방향

## HyperCAS는 무엇인가

- HyperCloud를 사용/운영하면서 발생할 수 있는 위험을 모니터링, 예방, 대응해줄 제품
- 4단계 계획 중 현재 1단계
  1. CMDB만 -> 잘못된 설정을 방지
  2. SysMaster WAS, SysMaster DB 각각으로부터 문제를 체크
  3. SysMaster, HyperData -> 연동 문제를 체크, 머신러닝으로 이상치 탐지
  4. 확장된 CMDB + SysMaster + HyperData
- 1단계에서의 시나리오
  - 사용자가 Tibero 설정에 대한 검증을 요청하는 경우 - '바람직한 설정과 관계 룰' DB 필요
  - 사용자가 Pod label 바꾸면서 검증을 요청하면, Service와의 연결이 끊어질 것을 경고 - '실제 환경 구성 및 실존하는 리소스 간 관계들' DB 필요
- HyperCAS 아키텍처
  - API서버가 있어서, 사용자가 어떤 action을 취하기 전에 검증을 요청하면, CMDB를 조회하여 validate 수행
  - CMDB는 CI Space와 Config Space로 구성
    - CI Space는 HyperCloud의 '실제 생성된' 모든 리소스의 구성과 환경의 설정 정보를 가지고 있음
    - Config Space는 HyperCloud 리소스들에 적용될 모든 구성과 설정 관련 룰 및 관계 정보를 가지고 있음

## CI Space

- HyperCloud 설정값, 리소스들의 설정/상태값 저장 및 업데이트하는 데이터베이스
- RDB만으로 CI Space 구현하는 경우
  - k8s 리소스들, jeus, tibero 등 수많은 CI 타입에 대하여 모두 테이블 설계가 필요
    - 대부분 depth와 hierarchy를 가지고 있어 매우 복잡하다.
- 값을 xml로 저장하는 방식으로 CI Space 구현하는 경우
  - 모든 CI 타입에 대해 전부 테이블을 설계할 필요 없이, xml 파일 형태로 저장
    - 이렇게 해도 tibero에서 xml 쿼리 조회가 가능
    - jeus, tibero 설정값을 xml로 변환하여 저장
    - etcd로부터 받는 json 데이터도 xml로 변환하여 저장
      - 이유 : tibero에서 json path에 따라 쿼리 결과를 추출하지 못한다. 반면 xpath query는 사용할 수 있음.
  - 빠른 개발이 가능하지만, 순수한 RDB에 비해 쿼리 성능이 안 좋을 수 있음
- 일단 xml 기반으로 구현하여 성능을 테스트하고, 필요하다고 판단되면 CI 타입 별로 table을 만든다.

## Config Space

- HyperCloud의 바람직한 상태를 유지하기 위해 필요한 rule set (필수 property가 뭔지, 각 값의 바람직한 range가 뭔지)
- RDB로 구현한다면 Tibero를 사용할 수 있지만, 수많은 k8s 리소스 간 관계와 룰을 표현하기는 어려움
- OWL2 기반의 Ontology KDB(??)로 구현하고자 함
  - Subject, Predicate, Object 형식을 사용하므로 관계의 표현에 적합함
  - 기존에 정의한 relation들을 이용해서 새로운 relation을 정의하거나, inverse relation을 정의하거나, Transitivity, Symmetry 같은 성질에 의해 파생되는 relation을 정의하기가 용이하다.
  - Expression(Reasoning Rule??)
- 하지만 KDB로만 할 것은 아님 -> KDB의 쿼리 성능이 좋지 않으니, 주기적으로 싱크 맞추는 RDB를 둔다.

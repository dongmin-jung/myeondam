# HyperCAS CMDB 설계 방향

## HyperCAS는 무엇인가

- HyperCloud를 사용/운영하면서 발생할 수 있는 위험을 모니터링, 예방, 대응해줄 제품
- 4단계 계획 중 현재 1단계
  1. CMDB만 -> 잘못된 설정을 방지
  2. SysMaster WAS, SysMaster DB 각각으로부터 문제를 체크
  3. SysMaster, HyperData -> 연동 문제를 체크, 동작 중의 이상치는 머신러닝으로 체크
  4. 확장된 CMDB + SysMaster + HyperData
- 시나리오
  - Tibero 설정에 대한 검증 - '바람직한 설정'에 대한 DB 필요
  - Pod label 바꿀 때 Service와 연결이 끊어질 것을 경고 - '현재 리소스 간 형성되어 있는 관계들'에 대한 DB 필요
- HyperCAS 아키텍처
  - API서버가 있어서, 사용자가 설정을 test 하고자 하면 CMDB를 조회하여 이를 validate
  - CMDB는 CI Space와 Config Space로 구성
    - Config Space는 HyperCloud 리소스들에 적용될 모든 구성과 설정 관련 룰 및 관계 정보를 가지고 있음
    - CI Space는 HyperCloud의 '실제 생성된' 모든 리소스의 구성과 설정 정보를 가지고 있음
  - SysMaster 별 log DB, k8s log DB -> HyperData로 분석

## CI Space

- HyperCloud 설정값, 리소스들의 설정/상태값 저장 및 업데이트하는 데이터베이스
- k8s 리소스들은 설정값들이 depth와 hierarchy를 가진 tree 구조
- 모든 hierarchy를 RDB만으로 CI Space 구현하는 경우
  - 쿼리 성능은 우수하겠지만,
  - k8s, jeus, tibero 등 수많은 CI 타입에 대하여 모두 DB 설계가 필요
- xml 기반으로 CI Space 구현하는 경우
  - 모든 CI 타입에 대해 전부 테이블을 설계할 필요 없이, xml 파일 형태로 저장
    - 이렇게 해도 tibero에서 xml 쿼리 조회가 가능
  - jeus, tibero 설정값을 xml로 저장, etcd로부터 받는 json 데이터도 xml로 만들어 저장
  - 빠른 개발이 가능하지만, xpath 쿼리 성능이 안 좋을 수 있음
- 일단 xml 기반으로 구현하여 성능을 테스트하고, 필요하다고 판단되면 CI별 table을 만든다

## Config Space

- HyperCloud의 바람직한 상태를 유지하기 위해 필요한 rule set (필수 property가 뭔지, 각 값의 바람직한 range가 뭔지)
- RDB로 구현한다면 Tibero를 사용할 수 있지만, 수많은 k8s 리소스 간 관계와 룰을 표현하기는 어려움
- OWL2 기반의 Ontology KDB(??)로 구현하고자 함
  - Subject, Predicate, Object 형식을 사용하므로 관계의 표현에 적합함
  - Expression(Reasoning Rule??)으로 조건을 걸어 필터링(??)
- 하지만 KDB로만 할 것은 아님 -> KDB의 쿼리 성능이 좋지 않으니, 주기적으로 싱크 맞추는 RDB를 둔다

# HyperCAS CMDB 설계 방향

## HyperCAS는 무엇인가

- HyperCloud를 사용/운영하면서 발생할 수 있는 위험을 모니터링, 예방, 대응해줄 제품
- 프로토타입부터 만든 후 점차 기능을 확장하려 함 - 4단계 계획
  1. CMDB만 -> 잘못된 설정을 방지 -> 프로토타입
  2. SysMaster WAS, SysMaster DB 각각으로부터 문제를 체크
  3. SysMaster, HyperData -> 연동 문제를 체크, 머신러닝으로 이상치 탐지
  4. 확장된 CMDB + SysMaster + HyperData
- 1단계 프로토타입에서의 시나리오
  - 사용자가 Tibero 설정에 대한 검증을 요청하는 경우 - '바람직한 설정 및 관계들의 rule set' DB 필요
  - 사용자가 Pod label 바꾸면서 검증을 요청하면, Service와의 연결이 끊어질 것을 경고 - 'rule set' + '실제 리소스 간 관계' DB 필요
- HyperCAS 아키텍처
  - API서버가 있어서, 사용자가 어떤 action을 취하기 전에 검증을 요청하면, CMDB를 조회하여 validate 수행
  - CMDB는 CI Space와 Config Space로 구성
    - CI Space는 instance들의 공간 - 실제 설정과 관계들
    - Config Space는 class들의 공간 - 바람직한 설정과 관계 룰들

## CI Space

- instance들의 공간
  - ci instance table - ci instance id / ci type / description(-> xml file)
  - ci type table - ci type / detail table
- 이렇게 순수 RDB로 구현하면
  - k8s 리소스들, jeus, tibero 등 수많은 CI 타입에 대하여 모두 테이블 설계가 필요
    - 대부분 depth와 hierarchy를 가지고 있어 매우 복잡하다.
- 값을 xml로 저장하는 방식으로 CI Space 구현하는 경우
  - 모든 CI 타입에 대해 전부 테이블을 설계할 필요 없이, xml 파일 형태로 저장
    - 이렇게 해도 tibero에서 xml 쿼리 조회가 가능
    - 첫 번째 타겟인 etcd로부터 받는 json 데이터도 xml로 변환하여 저장
      - 이유 : tibero에서 json path에 따라 쿼리 결과를 추출하지 못한다. 반면 xpath query는 사용할 수 있음.
  - 빠른 개발이 가능하지만, 순수한 RDB에 비해 쿼리 성능이 안 좋을 수 있음
- xml 기반으로 구현, 성능 테스트, 필요시 CI 타입 별 table 설계

## Config Space

- HyperCloud의 바람직한 상태를 유지하기 위해 필요한 rule set (필수 property가 뭔지, 각 값의 바람직한 range가 뭔지)
- RDB로 구현한다면 Tibero를 사용할 수 있지만, 수많은 k8s 리소스 간 관계와 룰을 표현하기는 어려움
- KDB로 구현하고자 함 ???
  - Subject, Predicate, Object 형식을 사용하므로 관계의 표현에 보다 적합

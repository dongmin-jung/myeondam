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
  - CMDB는 instance들의 공간인 CI Space + class들의 공간인 Config Space로 구성
    - CI Space - 실제 리소스들에 대한 설정과 서로간의 관계들
    - Config Space - 바람직한 설정 정보 및 리소스 간 관계 룰들

## CI Space

- instance들의 공간
  - ci instance table - ci instance id / ci type / description(-> xml file)
  - ci type table - ci type / detail table
- 이렇게 순수 RDB로 구현하면
  - k8s 리소스들, jeus, tibero 등 수많은 CI 타입에 대하여 모두 테이블 설계가 필요
    - 대부분 depth와 hierarchy를 가지고 있어 매우 복잡하다.
- 값을 xml로 저장하면
  - 모든 CI 타입에 대해 전부 테이블을 설계할 필요가 없어지고 빠른 개발이 가능
    - 이렇게 해도 tibero에서 xml 쿼리 조회가 가능
    - 예를 들어 우리의 첫 번째 타겟인 etcd로부터 받는 데이터도 json->xml 변환하여 저장
      - 이러는 이유 : tibero에서 json path에 따라서는 쿼리 결과를 추출하지 못하지만 xpath query는 사용할 수 있음.
  - 쿼리 성능이 안 좋을 수 있음
    - xml 기반으로 구현, 성능 테스트, 필요시 CI 타입 별 table 설계

## Config Space

- HyperCloud의 바람직한 상태를 유지하기 위해 필요한 rule set (필수 property가 뭔지, 각 값의 바람직한 range가 뭔지)
- KDB로 구현하는 방안 검토를 위해 AS본부 민서준 팀장님과 논의해보았는데,
  - KDB가 여러 node 간의 관계 표현에 더 적합한 것은 맞지만...
  - KDB는 잘 변하지 않는 진리를 표현하는 데 적합 - 한 번 내용 입력 후 사전처럼 사용
    - 말론 브란도와 알 파치노가 출연한 영화 -> '영화' class의 instance 중에서, '배우' class의 instance 중 말론 브란도, 알 파치노로부터 동시에 '출연한' relation으로 연결되어 있는 것을 찾는다.
    - class 및 axiom이 동적으로 추가될 일이나 instance가 수정될 일이 적음
  - HyperCloud 리소스들은 계속 변하는 데다, KDB로 표현하기 어려운 관계도 많음
    - 예를 들어 service의 selector가 name=console, app=hypercloud, version!=old 를 찾는다면 어떻게 할 건가
- 현시점에서는 Tibero로 구현하고, 추후 KDB 기능 필요시 AS본부와 협의하여 진행

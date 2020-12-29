# HyperCAS CMDB 설계 방향

## HyperCAS는 무엇인가

- HyperCloud를 사용/운영하면서 발생할 수 있는 위험을 모니터링, 예방, 대응해줄 제품
- 교수님께서 제시해주신 4단계 계획
  1. CMDB만 -> 잘못된 설정을 방지 -> 프로토타입
  2. SysMaster WAS, SysMaster DB 각각으로부터 문제를 체크
  3. SysMaster, HyperData -> 연동 문제를 체크, 머신러닝으로 이상치 탐지
  4. 모 ~ 든 ~ 것 ~
- 1단계 프로토타입에서의 시나리오
  - 사용자가 action을 취하기 전에 검증을 요청하면, API서버에서 CMDB를 조회하여 validate 수행
    - 사용자가 Tibero 설정에 대한 검증을 요청하는 경우 - '바람직한 설정 및 관계들의 rule set' DB 필요
    - 사용자가 Pod label 바꾸면서 검증을 요청하면, Service와의 연결이 끊어질 것을 경고 - 'rule set' + '실제 리소스 간 관계' DB 필요
  - CMDB는 instance들의 공간인 CI Space + class들의 공간인 Config Space로 구성
    - CI Space - 실제 리소스들에 대한 설정과 서로간의 관계들
    - Config Space - 바람직한 설정 정보 및 리소스 간 관계 룰들

## CI Space

- instance들의 공간
  - 리소스들의 설정값은 depth와 hierarchy를 가지는 복잡한 구조
  - ci instance table - ci id / ci type / xml file
    - 빠른 진행이 가능하고 구조가 유연함
    - query시 xml parsing을 해야 하므로 성능이 좋지 않을 것임
    - 결국에는 depth와 hierarchy를 커버할 테이블을 전부 설계해야 하나?
  - Tibero에서 지원하는 function based index (w/xpath) 사용 - ci id (=ref) / ci type / xpath / key / value
    - key와 value를 각각 column으로 둔 것은 유연성을 높이기 위함
      - 운영 중인 환경에서 DDL을 변경하지 않아도 됨
- 교수님이 히스토리는 어디갔냐? 하시면 여기 CI Space에서 할거다
 
## Config Space

- class들의 공간
  - rule id / condition / expression
    - 각 condition에 대해서 true/false expression을 각각 주면 row가 줄어들 수 있으나, 이분법을 피하는 편을 택함
    - 새로운 rule이 추가될 때 새로운 condition을 작성해서 추가하게 해도 되지만, 기존 rule들을 조합해서도 표현할 수 있게 하려 함
- 교수님이 ontology는 왜 안 쓰냐? 하시면 장황한 썰 ㄱㄱ

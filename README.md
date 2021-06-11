# CMDB 아키텍쳐 설계

## CMDB의 목적과 역할

1. HyperCloud 구성모듈에서 발생하는 설정변경을 DB에 모아 중앙에서 관리하는 것
2. 해당 DB를 HyperData에 제공하여 분석에 활용할 수 있게 하는 것

## 설계내용

- 서브시스템들에서 이벤트를 수집해야 하고, 모두 모아서 DB에 적재해야 한다.
- DB에 적재하는 부분은 java로 프로토타입을 만들어봤다.
  - go로 테스트해봤는데, ODBC 표준 API를 통해 Tibero를 써보려 했으나
    - go 자체가 다양한 DB access에 한계가 있었고
    - Tibero ODBC도 안정성이 약간 덜 확보가 된 것 같다.
    - java를 선택하면 이런 것들은 모두 해결된다.
  - DB 적재를 위해서 변경이벤트를 수신해야 하는데,
    - 멀티클러스터나, 여러 서브시스템(ovirt, ceph) 등이 들어올 것을 대비해야 함
    - 인터페이스 단일화를 위해 비동기 메시지 큐로 Kafka를 도입하려 함
- K8s에서 발생하는 설정 변경들을 watch하고 이벤트 발생시켜 동기화시켜주는 부분
  - Operator SDK를 사용하려 했으나 원하는 리소스를 컴파일 없이 특정해서 가져오기 어렵다.
  - watch 대상을 사용자가 런타임에 설정할 수 있어야 할테니, 리소스 설정 변경을 감지 부분은 operator sdk 대신 kube client를 사용한다.
  - go나 java나 kube-client가 잘 되어 있으므로 언어는 별로 상관 없을 것으로 보인다. 프로토타입은 java로 만들었고 kube-exporter로 명명했다.
  - 이런 에이전트들은 kube마다 하나씩 떠서 kafka로 보내줄 것이고
  - 다른 서브시스템이 들어오면 그에 맞는 agent가 개발되어서 마찬가지로 kafka로 보내주기만 하면 된다.
- DB에는 비정형으로 xmltype으로 저장을 하려 했다.
  - 비정형 데이터 안에서 키를 가지고 search를 지원하기 위해 xpath를 사용하려 한 것.
  - 그런데 json에서 xml로 변환시 이슈가 있음. json에서 허용되는 key가 xml element name으로는 못쓴다.
  - DB들이 jsontype, jsonpath 지원하는지 조사해보니 mysql이나 postgresql는 지원하고, 티베로는 곧 도입예정
  - jsonpath를 이미 지원 중인 mysql 기반으로 개발하되, DB 디펜던시를 없애기 위해 hibernate를 쓸 생각이다.
- 분석파트는 하이퍼데이터에서 알아서
  - CMDB 외에도 다양한 Event DB(장애/보안) ElasticSearch? Ozone? 연계하여
  - 장애발생시점 전후에 어떤 설정변경이 있었는지 추적한다든지 할 수 있겠다.

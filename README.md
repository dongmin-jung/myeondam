# HyperCAS

## 면담 스크립트

### 무엇을 할 것인가

- Rule 기반의 Config Control

- Runtime에서의 Dynamic Analysis
  - 리소스 간 연관성 알아내기
    - CMDB로부터 연관성을 알아내려면 사람이 봐야 할 게 너무 많은데다,
    - Application 레벨까지 들어가면 CMDB에 나타나지 않는 연관성도 있을 수 있음
    - 그러니, 시계열들의 correlation을 보고 리소스 간 연관성을 잡아낸다
  - 이상 현상 탐지 (Anomaly Detection)
    - 시계열, 로그, config 변경을 감지하여 event를 생성
    - 근본 원인 분석 (Root Cause Analysis)
      - 문제 발생 시점에 어떤 액션이 있었는지 보고 추적
      - 제품에서 발생한 에러 코드로부터 설정 문제 추적
      - 시나리오
        - JEUS-WebtoB 연동 이슈 발생
        - rule 에 기반해 연동 관련 static config 확인
        - 설정에 이상이 있다면 정상 기동을 위한 static config 제안
        - 이상이 없다면 각 제품 환경 상의 문제 확인
        - 환경 상의 문제 확인될 시 k8s 모듈 config 수정 제안

---

## 노정완 팀장님 아이디어

- HyperCAS = ADS + APS
- ADS는 모니터링을 통한 비정상 디텍션
  - 모니터링 : 통합 모니터링
    - SysMaster(자사제품), Prometheus(HC Application), ETCD(HC resource), ???(infraData 호스트)
      - 각각의 제품이 어떤 정보를 줄 수 있는지 간단 정리
  - 이벤트/로깅 : ELK의 EL을 활용한 데이터 (3팀 이승원 연구원)
    - Elasticsearch: DB&SearchEngine / Logstash: DataExporter&Pipeline / Kibana: dashBoard
    - 어케뜨는지 보고싶다
  - CMDB : 모니터링, 로깅데이터 통합 저장소
    - Static (config) : sync 맞춰줘야함
      - 인스턴스 정보를 담기 위한 CI Table (설정값?)
        - instanceType : k8sResource, tmaxProduct(app), serverOS,
      - CI 관계를 표현하기 위한 CI Relation Table
      - 정상동작 가능한 CI 설정값 정보 ConfigSpace
    - Dynamic
      - 실시간 리소스 사용량
      - 로그/이벤트 데이터 (어케 저장할지는 좀더 생각필요...)
        - stdout / stderr 에서 stderr만 따로 tagging 관리
        - error, warning etc... 단어가 들어간 로그/이벤트 tagging 관리
    - CI - dynamic 맵핑 정보
      - A라는 인스턴스는 b라는 dynamic정보를 뿌린다.
  - 분석 : 유의미한 인사이트 제공을 위한 위의 데이터 분석
    - 룰&릴레이션 분석
      - 새로 만드려는 인스턴스가 configSpace의 validation에서 벗어남
        - Tibero를 새로 만들려하는데, k8s pod에 설정된 quota 보다 더 큰 memory를 요구하고 있음.
      - A 인스턴스 설정 변경이 B와의 relation을 끊어버림
        - A pod의 label을 변경하면 B svc와의 연결이 끊어져 통신이 원활하지 않을 수 있음
      - a라는 에러로그를 A라는 인스턴스가 뱉고있다
      - a라는 에러는 A가 뱉고있는데, 이런에러는 BorCorD 인스턴스와 밀접한 관계가 있다.
        - Tibero A에서 ~~한 에러를 뱉고 있는데, 이는 2번 JEUS, 6번 Pod과 관련이 있다.
      - A인스턴스에서 나오는 a라는 에러는 error-no.27이며, 이에대한 해결방법은 ${A제품 메뉴얼에 적혀있는 err-27 텍스트} 이다.
    - 상관관계 분석
      - a라는 로그가 나오면 높은 확률로 b라는 로그가 따라온다
      - 실시간 리소스 사용량 a와 b의 추세가 비슷한걸 보니, 이와 맵핑된 A와 B 인스턴스는 밀접한 관계가 있다.
        - tensorflow inference server의 network 사용량이 많은데 동일 시간에 istio-ingressgateway의 network사용량도 치솟는다. 이를보아 inferServ와 ist-Ingress의 인스턴스간 밀접한 관계가 있음을 알 수 있다.
    - 시계열 데이터 분석 anomaly detect
      - A 인스턴스는 월요일마다 a라는 로그를 남기는데, 1/4일(월)에는 a가 없고 b라는 로그를 남긴다
      - A 인스턴스가 화요일만 사용량이 많았는데, 수요일에도 많다
    - 회귀분석을 활용 anomaly detect
      - A 인스턴스의 이전 리소스 사용량을 보았을때 지금은 2단계 수준의 사용량이 나와야 하는데, 지금은 1단계 사용량이네?
  - 알림 : rule기반 / anomaly detect기반 특정상황에서 여러 체널로 알림
- APS는 비정상 프리벤션 (advanced)
  - ADS의 후속 조치, 관리자가 특정 상황에서 미리 정의해둔 action을 자동 실행
    - 특정상황 : rule / anomaly...
    - ex) auto scaling, roll back, 미리 정의해둔 코드뭉치 실행
  - advanced 하게는 권장 action 인사이트 까지도 제공

---

## 이승진 연구원님 도움

- 모니터링
  - network, storage, cpu 등의 기본적인 metric 수집 (prometheus)
  - 각 제품의 기동 및 연동 (WebtoB - Jeus - Tibero) 과 관련된 static config 는 config DB 에 수집
  - 제품의 연동 상태 정보나 app-specific metric 등의 runtime variable (편의상 호칭) 은 master (agent..?, 제품 별 마스터인지 sysmaster 인지 마스터가 설계 상에 존재하는지는 모르겠지만..) 를 통해 prometheus 에 노출

- 문제 발생하면 이를 찾고 -> 연관성 있는 이벤트들 찾아내고 -> 원인 알아내고 (rule based)
  - static config 의 이상 유무 확인을 통해 기동 및 연동 상의 문제 예측 가능
  - static config 와 runtime variable, runtime variable 과 metric 간의 상관 관계를 rule 로 정의 -> 본부 간 협의 필요
    - ex. 특정 static config 는 특정 runtime variable 과 관계가 있다, 특정 config 가 잘못되면 특정 문제가 발생하고 특정 metric 또는 특정 runtime variable 이 이렇게 변할 수 있다 등
  - config 에 이상이 있다는 것을 나타내는 metric 값이나 runtime variable  값을 event 로 정의

- 대응 방법 추천
  - event 가 발생했을 시 특정 config 를 조정하거나 특정 환경을 확인해보는 것을 제안
  - 경고 및 rule 에 따른 대응 방법 추천

- 시나리오 예시 (아키텍쳐가 어떻게 나올지 잘모르겠어서 위 정도 아키텍쳐로 생각해봤을 때...)
  - 사용자가 web-was-db 구조의 web application 에 http request 를 보냈는데 에러 발생
  - 각 제품의 기동 관련 이슈는 static config (env, arg 등), quota 정도..??
  - webtob - jeus 연동 이슈 -> rule 에 기반해 연동 관련 static config 확인 (ip, port, thread pool 등) -> 설정에 이상이 있다면 정상 기동을 위한 static config 제안 -> 이상이 없다면 각 제품 pod 의 network, cpu 등의 metric 과 log 를 통해 환경 상의 문제 확인 -> 환경 상의 문제 확인될 시 calico, metallb config 등 kubernetes 모듈 config 수정 제안까지..?
  - jeus - tibero 연동 이슈도 위와 유사 (datasource 관련 설정)
  - 사실 jeus, tibero 같은 제품을 standalone 형태로 kube 에 띄우는 시나리오를 생각하는 건지도 잘 모르겠음..

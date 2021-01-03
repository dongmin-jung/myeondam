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
        - tensorflow inference server의 network 사용량이 많은데 동일 시간에 istio-ingressgateway의 network사용량도 치솟는다.
          이를보아 inferServ와 ist-Ingress의 인스턴스간 밀접한 관계가 있음을 알 수 있다.
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

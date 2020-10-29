# HyperAuth 생체인증과 FIDO 표준

---

- FIDO 란 무엇인가?
  - Fast IDentity Online
  - 어원은 라틴어 fido - trust, rely upon, put confidence in - 링컨 대통령이 키우던 개의 이름
  - 생체인증과 동의어처럼 많이 사용되지만, FIDO가 곧 생체인증인 것은 아님
  - Password의 불편함과 취약성 때문에 등장하게 되었음
    - 불편함
      - 온갖 사이트마다 다 다른 정책 → 기억하기 어렵고 번거로움
    - 보안이 약함
      - DB Leak, Server Code Injection, MITM, Malware
      - 재사용성이 높음 → 하나 뚫리면 다 털림 (구글 비번 털린 실제 경험 - 이집트, 중국, 대만)
      - 서버를 가장한 제3자에게 비밀번호를 털리는 경우
        - 중고나라 안전결제 유도하여 네이버페이 모방 페이지를 통해 ID/PW 탈취하는 방식 성행 중

---

- FIDO 역사와 구성
  - 2012년 FIDO Alliance 설립
    - 편리하고 강력한 보안, password 없고 궁극적으로는 ID도 없는 세상
    - 회원사
      - 구글, 마소, 아마존, 페북 / 애플, 레노버, 인텔 / 비자, 마스터, 비씨, 페이팔 / 삼성, 라인, 라온시큐어 ...
      - 이상은 보드레벨의 일부, 스폰서레벨은 훨씬 많음
  - 2014년 12월 FIDO 1.0 표준 - UAF & U2F
    - UAF (Universal Authentication Framework) - 모바일 앱
      - 모바일 기기의 빌트인 인증장치를 사용하여 앱 서버에의 인증을 수행하는 데 사용되는 표준
      - 예시 - 은행 앱 로그인, 삼성페이 결제 등
    - U2F (Universal 2nd Factor) - 2nd Factor로만 사용
      - ID/PW에 추가적인 인증수단을 등록하여 보안을 강화하는 것
      - 특수한 인증장치 필요 (USB Security Key)
      - 예시 - Google 계정에서 옵션 활성화
  - 2019년 3월 FIDO2 표준 (WebAuthn & CTAP) - 브라우저
    - WebAuthn (Web Authentication)
      - W3C 표준이 되어 주요 브라우저에 내장 API가 추가됨 (Windows, Android, Mac / Chrome, Edge, FireFox)
        - 추가적으로 확인해본 브라우저
          - Windows - Opera, Brave, Whale
          - Ubuntu 20.04 LTS - FireFox
        - IE에서는 안됨
      - 정확히는 브라우저의 Credential Management API에 store/get 외에 create가 추가된 것
    - CTAP (Client To Authenticator Protocol)
      - 플랫폼(브라우저+OS)이 인증장치와 소통하는 데 사용되는 프로토콜
      - 플랫폼에서 USB/NFC/블루투스로 연결된 인증장치를 찾고, 특성을 알아내고, 사용자 등록/인증을 시도하고, 인증장치가 응답 데이터나 에러를 반환

---

- FIDO 등록/인증 동작 방식
  - Recall - FIDO 인증은 단말에서 사용자 확인 후 전자서명 값을 서버로 보내 인증을 완료하는 방식
  - 등록 - credential key pair를 생성하고, 공개키를 FIDO 서버에 등록하는 절차
    - 클라이언트에서 서버에 등록을 요청하면, 서버에서 인증장치로 챌린지와 추가적인 매개변수 전달
      - 추가적인 매개변수란 RP Name/ID(domain), 유저 Name/ID, 지원하는 암호화의 종류, 그외 선택적 정보들
        - 선택적 정보란 인증 수행 방식에 대한 선호나 강요 같은 것
          - user verification - required/prefered/discouraged
          - authenticator - platform/cross-platform
      - 클라이언트에서 서버 응답으로 받은 저 정보들을 가지고, WebAuthn API 요청
      - 브라우저에서 WebAuthn API가 불리면 플랫폼(OS와 브라우저)가 인증장치와 CTAP으로 통신 시작
    - 인증장치는 user verification이나 user presence를 체크하고, 새로운 key pair를 생성
    - 인증장치는 assertion을 생성하고 개인키로 서명하여 서버로 전송
    - 서버는 assertion을 검증하고, 공개키를 저장
  - 인증 - 로그인/거래요청 시
    - challenge를 받아서 assertion을 생성하고 개인키로 서명하여 서버로 보내고, 등록되어 있는 공개키로 검증
  - FIDO 인증의 장점
    - 기억력에 의존X
    - challenge-response 방식이므로, 누군가가 인증 데이터를 가로채더라도 재사용 불가
    - 서버에는 공개키만 보관되므로, 누군가가 서버 DB를 털거나 코드를 수정해서 정보를 입수해도 부정사용 불가
    - 서비스-specific key pair를 사용, https만 지원, 인증 과정에서 URL 도메인 확인하는 절차로 MITM 방지
    - 사족
      - 물리적 외압이나 국가 상대로는 덜 안전할 수 있음 - 묵비권(진술거부권) 보호X
      - Password와 달리 생체정보는 바꿀 수 없으니, 위조를 판별해낼 수 있게 인식기술이 계속 발전해야 함

---

- 인증장치란?
  - 하드웨어 인증장치 - USB Secure Key
    - secure key store라고 불리는, 사용자 정보와 개인키를 저장할 '안전한 공간'을 가지고 있어야 하고, 새로운 key pair를 계속 만들어낼 수 있어야 함
  - 소프트웨어 인증장치 - Windows Hello 같은 것 - 얼굴인식/지문인식/PIN을 등록해두고 택1하여 인증
    - 생체인식 데이터는 로컬에 저장, 외부로 전송X, 생체인식 데이터가 탈취되어도 원시샘플로 변환 불가
  - 지문인식/얼굴인식 과정 (교수님이 궁금해하실 경우)
    - 지문인식
      - 지문인식 하드웨어 (광학식/정전용량식/초음파식) + 분석/판독 소프트웨어
      - Windows Hello에서 어떻게 지문을 분석/판독하는지는 공개X
      - Ubuntu 20.04 LTS부터 빌트인으로 지원되는 지문인식은 NIST(미국 상무부 산하의 국립표준기술연구소) 배포한 코드를 사용
      - NBIS (NIST Biometric Image Software) 중 지문 관련된 것들
        - MINDTCT - 특징점 위치 추출 알고리즘
          - 특징점 = 끝점(융선이 끊기는 곳), 분기점(융선이 나뉘어지는 곳)
          - 처리 과정은 추후 보완 (이미지 강화, , 이진화, 의사특징점 제거)
        - BOZORTH3 - FBI에서 만들고 NIST에서 수정, MINDTCT 데이터로 지문 비교 판독하는 알고리즘
          - 이미지에서 특징점들의 거리, orientation과 연결선 사이의 각도를 테이블로 저장하여 비교
        - 기타1 - 지문 4개를 한번에 찍어둔 이미지에서 개별 지문을 추출하고 여백 제거
        - 기타2 - 지문 패턴 분류 시스템 - 이거는 뉴럴 네트워크 기반
          - 카테고리 - arch, tented arch, whorl, left loop, right loop, scar
    - 얼굴인식
      - Windows Hello의 얼굴 인식 과정은 간략하게만 공개되어 있음
        - 적외선 카메라 필요 - 조명과 관계없이 일정한 흑백이미지로 input control
        - 얼굴개수1개 → 얼굴각도15도이내 → 눈코입 등 landmark/alignment points 명암차 수치로 representation vector 수천개 생성 (이미지 저장X) → 저장된 representation과의 유사도가 threshold 넘는지 봄
        - 내가 시도하면 95%↑ / 5%↓, 남이 시도하면 1/10만↓, 일란성 쌍둥이도 구별
      - Debian/Ubuntu에서 얼굴인식 로그인을 지원하기 위해 만들어진 오픈소스 Howdy는 dlib C++ 라이브러리(github 즐겨찾기 1만 / k8s가 7만)의 모델을 사용함
        - 2016년에 출판된, 인용수 6만 논문의 ResNet-34를 조금 간소화하고 3백만 개 얼굴로 학습시킨 것
        - 위 논문의 Residual Network는 2015년 각종 대회 이미지 분류 분야 우승을 휩쓸었음
        - 보통 Network는 layer가 많아질수록 학습이 오래 걸리고 overfitting 위험성이 있고 training error 증가하지만,
        - Residual Network는 layer가 많아도 error가 감소하는 데다가, 같은 layer 개수에서 성능이 더 좋고 훈련 속도도 더 빠르다 함

---

- Keycloak의 FIDO2 지원 현황
  - webauthn4j('j'ava? 'j'apan?)라는 FIDO2 오픈소스 서버 개발하는 사람들이 Keycloak 플러그인 형태로 개발해오던 것이, Keycloak으로의 pull request가 merge되어 2019년 11월 Keycloak 8.0.0 부터 passwordless, 2-factor 등록/인증 지원 시작
  - 직접 구축하고 테스트해보았는데 아직은 최소한의 기능만 제공되고 있음
  - 2가지 한계
    - 키 등록이 강제됨 - 인증장치 없는 사용자, IE 사용하는 사용자는 사용 불가
    - 키 조회/추가/삭제 불가 - 최초에 등록했던 바로 그 인증장치가 있어야만 passwordless 로그인 가능 (패스워드로 로그인하거나 관리자가 직접 초기화해주면 되긴 함)
  - 추후에는 다 지원할 계획이라는데, 그때쯤 DB 구조나 UI가 대폭 바뀔 수 있다고 함

---

- 결론
  - FIDO2는 아직 대중화되지 않았음. 생체인증은 모바일 앱이나 윈도우 잠금해제에만 많이 쓰이는 중. 심지어 구글에서 "naver fido2" 검색하면 티맥스블로그가 맨 위에 나옴
  - 그러나 분명 웹에서도 생체정보 기반 인증은 점점 더 많이 사용될 것임
    - 중저가 랩탑에도 점점 '지문인식장치나 적외선카메라 중 적어도 하나'가 탑재된 기종이 많아지는 추세
    - Ubuntu도 20.04 LTS부터 지문인식 로그인을 빌트인으로 지원
    - 공인인증서 사용 의무화 폐지 이후 스마트폰 금융 앱에서 생체인증이 많이 활성화 → Why not 인터넷 뱅킹?
  - 지금 지원되는 곳이 적은 이유는, 아직 하드웨어/소프트웨어 보급이 덜 되었고, FIDO2가 표준이 된 지 얼마 안 되었기 때문
  - FIDO2가 더 대중화되면, Keycloak에서의 지원도 개선되어 있을 것
  - HyperAuth 생체인증도 이에 따라 추후 Keycloak 설정을 통해 지원하면 어떨까?

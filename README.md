# HyperAuth 생체인증과 FIDO 표준

---

- FIDO 란 무엇인가?
  - Fast IDentity Online
  - 어원은 라틴어 - 신뢰하다, 의지하다 (링컨 대통령 개의 이름)
  - 생체인증과 동의어처럼 많이 사용되지만, FIDO 자체가 생체인증인 것은 아님
  - Password의 불편함과 취약성 때문에 등장하게 되었음
    - 불편함
      - 온갖 사이트마다 다 다른 정책 → 기억하기 어렵고 번거로움
    - 보안이 약함
      - 재사용성이 높음 → 하나 뚫리면 다 털림 (실제로 네이버, 구글 계정 털려봤음)
      - 중고나라 거래 시 네이버페이 모방 페이지를 통해 ID/PW 탈취하는 범죄 성행 중

---

- FIDO 역사와 구성
  - 2012년 FIDO Alliance 설립
    - 편리하고 강력한 보안, password 없고 궁극적으로는 ID도 없는 세상
    - 회원사
      - SW - 구글, 마소, 아마존, 페북, 애플 / HW - 인텔, 레노버 / 카드 - 비자, 마스터 / 국내 - 삼성, 라인, 라온시큐어 ...
      - 이상은 보드레벨의 일부이고, 스폰서레벨은 훨씬 많음
  - 2014년 12월 FIDO 1.0 표준 - UAF & U2F
    - UAF
      - Universal Authentication Framework
      - 모바일 앱에서 생체인증을 사용하기 위한 표준
      - 예시 - 은행 앱 로그인, 삼성페이 결제 등
    - U2F
      - Universal 2nd Factor
      - ID/PW에 추가적인 인증수단을 등록하여 보안을 강화하는 것
      - 특수한 하드웨어 인증장치 필요 (USB Security Key)
      - 예시 - Google 계정에서 옵션 활성화
  - 2019년 3월 FIDO2 표준 (WebAuthn & CTAP)
    - PC에서 브라우저를 통해 생체인증을 사용하기 위한 표준
    - WebAuthn (Web Authentication)
      - W3C 공인 웹 표준이 되어, 주요 브라우저에 내장 JavaScript API 추가됨 (Windows, Android, Mac / Chrome, Edge, FireFox)
      - 브라우저의 Credential Management API에 store/get 외에 create가 추가된 것
      - IE는 해당X
    - CTAP (Client To Authenticator Protocol)
      - 플랫폼(브라우저+OS)이 인증장치와 소통하는 데 사용되는 low level 프로토콜
      - 플랫폼에서 인증장치를 찾고, 특성을 가져오고, 인증을 수행하고, 응답이나 에러 반환받음

---

- FIDO 등록/인증 동작 방식
  - FIDO 인증은 단말에서 사용자 확인 후 전자서명 값을 서버로 보내 인증을 완료하는 방식
  - 등록 - key pair를 생성하고 공개키를 FIDO 서버에 등록
    - 서버에서 클라이언트로 챌린지, 서버주소, 사용자이름 등을 보내고, 클라이언트(브라우저)에서 WebAuthn API가 불리면 CTAP으로 인증장치와 소통
    - 인증장치는 user verification이나 user presence를 체크하고, 새로운 service-specific key pair 생성
    - 인증장치는 assertion을 생성하고 개인키로 서명하여 서버로 전송. 이때 생체정보와 개인키 등은 인증장치를 떠나지 않음
    - 서버는 assertion을 19단계에 걸쳐 검증하고, 공개키를 저장
  - 인증 - 로그인/거래요청 시
    - challenge를 받아서 assertion을 생성하고 개인키로 서명하여 서버로 보내고, 등록되어 있는 공개키로 검증
  - FIDO 인증의 장점
    - 기억력에 의존X
    - challenge-response 방식이므로, 누군가 인증 데이터를 가로채더라도 재사용 불가
    - 서버에는 공개키만 보관되므로, 누군가 서버 DB를 털어도 부정사용 불가
    - 서비스-specific key pair, https만 지원, 인증 과정에서 서버주소 재확인하여 MITM 방지
    - 한계
      - 묵비권(진술거부권) 보호X
      - 생체정보는 바꿀 수 없으니, 위조에 속지 않도록 인식기술이 계속 발전해야 함

---

- 인증장치란?
  - 하드웨어 인증장치
    - USB Secure Key - 버튼/터치/PIN 중 택1하여 인증
    - 새로운 key pair를 계속 만들어낼 수 있어야 하고, 사용자 정보와 개인키를 저장할 '안전한 공간'을 가지고 있어야 함
  - 소프트웨어 인증장치
    - Windows Hello - 얼굴인식/지문인식/PIN을 등록해두고 택1하여 인증
    - 생체인식 데이터는 외부로 전송X, 기기에서 생체인식 데이터를 탈취해도 원시샘플로는 변환 불가
  - 지문인식/얼굴인식 과정 (교수님이 궁금해하실 경우)
    - 지문인식
      - 인식 하드웨어 + 분석 소프트웨어 필요
      - Windows Hello에서 어떻게 지문을 분석/판독하는지는 공개X
      - Ubuntu 20.04 LTS부터 지원되는 지문인식은 NIST(미 상무부 산하 국립표준기술연구소) 코드 사용
        - 특징점 추출 알고리즘
          - 끝점(융선이 끊기는 곳), 분기점(융선이 나뉘어지는 곳)
          - 각 특징점에서의 융선의 orientation도 함께 저장
        - 지문 비교 판독 알고리즘
          - BOZORTH3 - FBI에서 만들고 NIST에서 수정, 초기 버전은 bozorth98
          - 추출된 특징점 이미지에서, 각 특징점 2개의 쌍에 대해 특징점 사이의 거리, 특징점들을 연결하는 직선과 각 특징점에서의 융선 orientation이 이루는 각도를 테이블로 저장하여 비교
            - rotation과 translation에 대해 invariant함
          - 테이블을 비교하여 산출한 점수는 두 지문에서 일치하는 특징점 개수와 근사적으로 같음
          - 대부분, 지문 하나에 특징점은 80개 미만이고, 일치하는 특징점이 40개를 넘으면 같은 지문으로 봄
        - 기타1 - 지문 4개를 한번에 찍은 이미지에서 개별 지문 추출, 여백 제거
        - 기타2 - 지문 패턴 분류 - 뉴럴 네트워크 기반
          - 카테고리 - arch, tented arch, whorl, left loop, right loop, scar
    - 얼굴인식
      - Windows Hello 얼굴인식은 IR카메라를 사용함
        - 얼굴을 찾고, landmark 찾음 → 얼굴각도 정면인지 확인 → landmark point들에서의 명암차를 수치화하여 representation vector 생성 → 저장된 representation과의 유사성이 threshold 넘는지 봄
          - threshold 계산에 머신러닝으로 학습된 모델이 사용됨
          - 모델은 MS에서 개발, 학습시켜 Windows에 포함시킨 것이고, 실행은 각 로컬 기기에서 일어남 (생체 인증 데이터는 로컬 기기를 떠나면 안 된다는 philosophy와 부합)
        - 본인이 시도하면 95%↑ / 5%↓, 타인이 시도하면 1/10만↓, 일란성 쌍둥이도 구별
      - Debian/Ubuntu에서 얼굴인식 로그인을 지원하기 위해 만들어진 오픈소스는 dlib C++ 라이브러리의 모델을 사용함 (dlib 즐겨찾기 1만 / k8s가 7만)
        - 2016년에 출판된, 인용수 6만 논문의 ResNet-34를 조금 간소화하고 3백만 개 얼굴로 학습시킨 것
        - 위 논문의 Residual Network는 이미지 분류 분야에서 2015년 여러 대회에서 우승함
        - Normal Network는 layer가 어느 이상 많아지면 학습이 너무 오래 걸리고 training error도 증가하는데,
        - Residual Network는 layer가 많아도 training error가 감소하고, layer 개수가 같을 때 성능이 더 좋고, 학습 속도도 더 빠르다 함

---

- Keycloak의 FIDO2 지원 현황
  - webauthn4j('j'ava? 'j'apan?)라는 FIDO2 오픈소스 서버 개발하는 (주로 Hitachi 소속) 사람들이 Keycloak 플러그인 형태로 개발해오던 것이, Keycloak으로의 pull request가 merge되어 2019년 11월 Keycloak 8.0.0 부터 FIDO2 등록/인증 지원 시작
  - 아직 최소한의 기능만 제공되고 있어서 2가지 한계가 있음
    - 키 등록이 로그인 시점에 강제됨 - 인증장치 없는 사용자, IE 사용하는 사용자는 사용 불가
    - 키 조회/추가/삭제 불가 - 최초에 등록한 바로 그 인증장치로만 passwordless 로그인 가능
  - 추후에는 다 지원할 계획이라는데, 그때에 DB 스키마, 기존 데이터, UI가 지금과 같을 것을 보장하지 않음

---

- 결론
  - FIDO2는 아직 대중화되지 않음. 생체인증은 아직까지 모바일 앱에서나 윈도우 잠금해제에만 많이 쓰이고 있음.
  - 아직 하드웨어/소프트웨어 보급이 부족하고, FIDO2가 표준이 된 지도 얼마 안 되었기 때문
  - 그러나 웹에서도 생체정보 기반 인증은 점점 더 많이 사용될 것임. 3가지 이유
    - 하드웨어 - 중저가 랩탑에도 점점 '지문인식장치나 적외선카메라 중 적어도 하나'가 탑재된 기종이 많아지는 추세
    - 소프트웨어 - Ubuntu 20.04 LTS부터 지문인식 로그인 지원
    - 제도적 - 공인인증서 사용 의무화 폐지 이후 스마트폰 금융 앱에서 생체인증이 많이 활성화 → Why not 인터넷 뱅킹?
  - FIDO2가 대중화될 때면, Keycloak에서의 지원도 개선되어 있을 것
  - HyperAuth 생체인증도 추후에 FIDO2 기능이 강화된 Keycloak에서 Realm Authentication Flow 설정을 통해 지원하면 어떨까?

OWASP 고급 서버 측 공격 및 공급망 취약점 실습 

학습일: 2026.07.30
주제: WebGoat 기반 XXE, SSRF 취약점 원리 파악 및 오픈소스 의존성 스캔(Dependency-Check)
목표: 외부에서 직접 접근할 수 없는 서버 내부망 및 시스템 파일을 타격하는 취약점의 메커니즘을 이해하고 방어 대책을 수립한다.

단순한 입력값 검증 누락을 넘어, 서버의 파싱(Parsing) 설정 결함이나 외부 통신 기능의 허점을 노리는 고급 취약점들에 대해 실습을 진행했다.

1. XXE (XML External Entity) Injection 실습 및 분석

서버가 클라이언트로부터 전달받은 XML 데이터를 파싱할 때, DTD(Document Type Definition) 내에서 외부 리소스 참조 기능이 비활성화되어 있지 않아 발생하는 취약점을 테스트했다.

실습 과정 (WebGoat XXE 4, 7번 과제):

프록시 툴을 거치지 않고 브라우저 개발자 도구의 콘솔에서 fetch 함수를 이용해 악성 XML 페이로드를 직접 구성하여 전송했다.

XML 선언부 직후에 <!ENTITY xxe SYSTEM "file:///"> 구문을 삽입하여 서버 로컬의 루트 디렉터리를 가리키는 외부 엔티티를 생성하고, 이를 댓글 데이터 영역에 바인딩했다.

서버가 JSON 형태의 데이터를 기대하는 상황(7번 과제)에서도, HTTP 헤더의 Content-Type을 application/xml로 강제 변조하여 전송하면 서버의 XML 파서가 동작하여 동일한 취약점이 트리거됨을 확인했다.

결과: * 서버 응답(댓글 목록)에 서버 내부의 /bin, /etc, /boot 등 중요 디렉터리 및 파일 목록이 그대로 유출되는 것을 확인했다.

<img width="1193" height="947" alt="image" src="https://github.com/user-attachments/assets/eaf33489-3f67-4022-828d-6bb0e91bca3c" />


2. SSRF (Server-Side Request Forgery) 실습 및 분석

웹 서버가 클라이언트가 지정한 URL로 데이터를 가져오는 기능을 악용하여, 방화벽 내부에 위치한 사내망이나 로컬 시스템으로 서버가 직접 통신하게 만드는 취약점이다.

실습 과정 (WebGoat SSRF 2, 3번 과제):

이미지를 불러오는 기능을 분석한 결과, HTML 내부에 숨겨진 input 필드(<input type="hidden" name="url"...>)를 통해 이미지의 경로를 서버에 전달하고 있음을 확인했다.

F12 개발자 도구를 통해 해당 요소의 value 값을 강제로 조작하였다.

서버의 내부 자원을 찌르거나, http://ipconfig.pro와 같은 임의의 외부 악성 주소로 값을 변경한 뒤 폼을 전송했다.

결과:

웹 서버가 조작된 URL을 맹목적으로 신뢰하여, 해커가 지시한 외부 도메인으로 HTTP 통신을 수행한 결과를 반환하는 것을 확인했다.

<img width="1628" height="966" alt="image" src="https://github.com/user-attachments/assets/e56ed007-b703-4c3b-ba69-218f8a28ec40" />

3. 공급망 취약점 (Vulnerable Components) 점검

애플리케이션의 비즈니스 로직에 결함이 없더라도, 프로젝트에 포함된 오픈소스 라이브러리에 취약점이 존재하면 전체 시스템이 장악될 수 있다. 이를 방어하기 위한 인프라 점검을 실습했다.

실습 과정:

Node.js 환경에서 빈 프로젝트를 생성하고, 의도적으로 취약점이 포함된 과거 버전의 라이브러리(lodash 4.17.4)를 설치했다.

패키지 매니저에 내장된 npm audit 명령어를 실행하여 현재 프로젝트의 의존성 트리를 스캔했다.

결과:

스캔 완료 후 터미널에 High/Critical 등급의 CVE 취약점(Prototype Pollution 등) 내역과 문제가 되는 패키지가 경고 리포트로 출력되는 것을 시각적으로 식별할 수 있었다.

<img width="958" height="356" alt="image" src="https://github.com/user-attachments/assets/fd7d1db8-f339-45e7-924e-7655110ce1de" />


종합 회고 및 방어 대책

XXE 방어: XML을 처리하는 백엔드 파서(Parser) 설정 시, 외부 엔티티(External Entity) 확인 및 DTD 참조 기능을 코드 레벨에서 명시적으로 비활성화(Disable)해야 한다. 가급적 데이터 통신 포맷을 JSON으로 통일하는 것이 안전하다.

SSRF 방어: 파라미터로 URL이나 IP를 전달받아 서버가 외부 통신을 수행해야 할 경우, 해당 대상이 127.0.0.1, localhost 등 사내망 내부 대역이 아닌지 검사하는 엄격한 화이트리스트/블랙리스트 검증 로직이 필수적이다.

공급망 보안: CI/CD 파이프라인 빌드 단계에 의존성 취약점 스캔 도구를 통합하여, 취약점이 존재하는 라이브러리가 운영 서버에 배포되는 것을 사전에 차단해야 한다.

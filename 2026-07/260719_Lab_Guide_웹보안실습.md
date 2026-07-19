[Week 2] 웹 보안 실습 및 아키텍처 학습 기록 (TIL)

학습일: 2026.07.19
주제: SQLi 실습(Dreamhack), React XSS 검증, L4/L7 로드밸런서 분석

오늘은 지난 이론 스터디에서 다뤘던 웹 취약점들을 직접 눈으로 확인해 보고, 안전한 아키텍처 설계의 기초가 되는 로드밸런서 개념을 정리하는 시간을 가졌다.

1. SQL Injection 실습 (Dreamhack: simple_sqli)

가장 기본이 되는 웹 취약점인 SQL Injection을 체화하기 위해 드림핵(Dreamhack) 워게임의 simple_sqli 문제를 풀고 인증 우회를 실습했다.

서버 소스코드 분석 (app.py)
제공된 파이썬 백엔드 코드를 살펴보니 데이터베이스 조회 로직에 치명적인 취약점이 있었다.

res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')


사용자의 입력값(userid)을 검증 없이 f-string으로 쿼리에 직접 꽂아 넣고 있었고, 홑따옴표가 아닌 큰따옴표(")를 사용하고 있었다.

<img width="512" height="431" alt="image" src="https://github.com/user-attachments/assets/dda56dd6-d833-4377-a542-deb73f831628" />

공격 페이로드 작성 및 우회 성공
따라서 쿼리 구조를 탈옥하고 뒷부분의 비밀번호 검증 로직을 무력화시키기 위해 다음과 같은 페이로드를 작성해 아이디 칸에 주입했다.

Payload: admin"--  (맨 뒤 공백 포함)

입력 결과, 서버에서 실행되는 실제 쿼리는 다음과 같이 조작되었다.
select * from users where userid="admin"--" and userpassword="..."
뒤의 패스워드 검증 로직이 -- 기호로 인해 주석 처리되면서 무사히 관리자(admin) 계정으로 로그인에 성공했고, 플래그를 획득할 수 있었다. 방어만 외우는 것보다 직접 코드를 보고 뚫어보니 원리가 훨씬 와닿았다.
(※ 워게임 규정에 따라 스포일러 방지를 위해 정답 플래그 DH{...} 원본은 캡처에서 가림 처리함)

<img width="146" height="128" alt="image" src="https://github.com/user-attachments/assets/0f28a8d8-d454-478e-9e63-39b9578074ae" />

2. React XSS 취약점 직접 검증하기

React는 프론트엔드 단에서 발생하는 XSS(Cross-Site Scripting) 공격을 막기 위해 훌륭한 자체 방어 메커니즘을 가지고 있다. 하지만 예외적으로 DOM을 직접 조작하는 API를 사용할 때 발생하는 취약점을 눈으로 확인하고자 CodeSandbox 환경(https://codesandbox.io/s/react-new)에서 테스트를 진행했다.

실습 환경 구성 및 테스트 코드 작성
App.js에 텍스트 입력창을 하나 만들고, 사용자의 입력값(상태값)이 화면에 렌더링되는 두 가지 구역을 나누어 구성했다. 하나는 React의 기본 데이터 바인딩 방식({input})을 사용했고, 다른 하나는 React에서 제공하는 DOM 직접 주입 속성인 dangerouslySetInnerHTML을 사용했다.

이후 악의적인 자바스크립트가 포함된 페이로드 <img src="x" onerror="alert('XSS 해킹 성공!')" /> 를 입력창에 주입하여 브라우저의 렌더링 차이를 비교 분석했다.

<img width="1870" height="957" alt="image" src="https://github.com/user-attachments/assets/024eb3df-56ad-4a27-acd2-3cbf2e8006b6" />


결과 분석 및 동작 원리 파악

1번 구역 (안전한 기본 바인딩): 화면에 <img src="x"... 라는 문자열 자체가 그대로 출력되었다. React는 내부적으로 중괄호 안에 들어온 데이터를 렌더링할 때 HTML 태그를 해석하지 않고 단순 텍스트 노드로 취급한다. 즉, 괄호 < 와 > 를 &lt; 와 &gt; 와 같은 이스케이프 문자로 치환하기 때문에 스크립트가 무력화된다.

2번 구역 (dangerouslySetInnerHTML 사용): 코드를 입력하자마자 즉시 브라우저 화면에 'XSS 해킹 성공!' 이라는 팝업 경고창(Alert)이 발생했다. 이 속성은 이름 그대로 입력받은 문자열을 어떠한 필터링도 없이 브라우저의 DOM에 innerHTML 방식으로 꽂아 넣는다. 브라우저는 이를 실제 이미지 태그로 인식하여 파싱하고, "x"라는 잘못된 경로의 이미지를 불러오다 실패하여 onerror 이벤트 핸들러에 작성된 자바스크립트를 그대로 실행해 버린 것이다.

이 실습을 통해 서비스 내에서 웹 에디터(WYSIWYG) 등을 통해 작성된 사용자 정의 HTML 문자열을 렌더링해야 할 경우, 단순히 React의 기본 기능만 믿어서는 안 되며 반드시 DOMPurify와 같은 서드파티 라이브러리를 사용해 XSS 유발 태그를 서버와 클라이언트 양측에서 무해화(Sanitization) 해야 함을 확실히 깨달았다.

<img width="1873" height="994" alt="image" src="https://github.com/user-attachments/assets/20da08c7-b340-4065-9c3b-41b85854456c" />


3. L4/L7 로드밸런서 아키텍처 학습

단일 서버로 감당할 수 없는 대규모 트래픽을 처리하기 위한 스케일 아웃(Scale-out) 환경에서, 네트워크 트래픽을 분산시키는 로드밸런서의 동작 원리를 인프라 아키텍처 관점에서 학습했다. AWS의 ELB 구조 및 국내 IT 기업의 아키텍처 기술 영상을 참조하여 다이어그램을 분석했다.

계층별 로드밸런서의 특징과 트래픽 라우팅 원리

L4 로드밸런서 (Network Layer - TCP/UDP):
L4 스위치는 클라이언트의 트래픽이 들어왔을 때 오직 IP 주소와 포트 번호만을 확인하여 백엔드 서버로 요청을 넘긴다. 패킷의 캡슐화를 뜯어 데이터 내용(Payload)을 확인하지 않기 때문에 연산 리소스 소모가 매우 적고 트래픽 처리 속도가 압도적으로 빠르다. 단순한 분산 처리가 필요한 대규모 서비스의 1차 관문 역할을 한다.

L7 로드밸런서 (Application Layer - HTTP/HTTPS):
L7 스위치는 L4와 달리 HTTP 헤더, 쿠키(Cookie), URI 파라미터 등 애플리케이션 계층의 데이터까지 모두 까보고 분석한다. 따라서 클라이언트가 /api 경로로 요청을 보냈는지, /images 경로로 보냈는지 판단하여 마이크로서비스(MSA)의 각기 다른 타겟 서버로 지능적인 라우팅을 수행할 수 있다.

<img width="1267" height="733" alt="image" src="https://github.com/user-attachments/assets/20a78bc0-c8ec-4f8e-946e-53a567b33680" />

출처: https://www.youtube.com/watch?v=40y26urYM-M

보안 관점에서의 아키텍처 고찰 (느낀 점)

L7 로드밸런서의 트래픽 내용 분석 기능은 라우팅뿐만 아니라 '보안' 영역에서 핵심적인 역할을 수행한다는 것을 알게 되었다. L7 단에 WAF(웹 방화벽)를 연동하면, 오늘 실습한 SQL Injection이나 XSS 공격 페이로드가 포함된 비정상적인 HTTP 요청을 백엔드(애플리케이션) 서버에 도달하기 전에 네트워크 앞단에서 1차적으로 차단할 수 있다.

결과적으로 안전한 서비스 환경을 구축하려면 프론트엔드/백엔드 코드 레벨에서의 시큐어 코딩은 물론이고, L7 로드밸런서와 방화벽을 활용한 인프라 레벨의 방어 체계가 겹겹이 구성되는 심층 방어(Defense in Depth) 아키텍처가 필수적이라는 시야를 얻게 되었다.

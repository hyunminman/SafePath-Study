[Network] 웹의 동작 원리와 HTTP 통신 분석

학습일: 2026.07.26
주제: URL 구조 분해, GET/POST 차이, HTTP 상태 코드 및 헤더 분석
목표: 보안 실습 전 브라우저와 서버 간 HTTP 통신 구조 파악

웹 취약점은 브라우저와 서버 간 HTTP 통신의 구조적 특성에 기인하는 경우가 많다. 따라서 본격적인 보안 실습에 앞서 개발자 도구(F12)를 활용하여 HTTP 통신의 동작 과정을 분석하고 기초 개념을 정리하였다.

1. URL과 URI 구조 분석

특정 URL을 기준으로 각 구성 요소를 분해하여 분석하였다.

예시 URL: https://www.google.com:443/search?q=web+hacking

Protocol: https:// (보안 통신 프로토콜)

Domain: www.google.com (서버 주소)

Port: :443 (HTTPS 기본 포트)

Path: /search (서버 내 자원 경로)

Query String: ?q=web+hacking (서버에 전달하는 파라미터. ?로 시작하여 &로 구분)

[분석 결과]
SQL 인젝션 공격 시 쿼리 스트링 영역에 특수문자(' 1=1--)를 주입하는 구조적 이유를 확인하였다. 쿼리 스트링 값은 백엔드 로직의 변수로 매핑되므로, 해당 경로를 통한 입력값 검증이 필수적이다.

2. GET vs POST 통신 방식 비교

네트워크 탭을 통해 GET과 POST 메서드의 차이를 대조하였다.

GET 방식: 데이터 조회(Read) 시 주로 사용된다. 요청 데이터가 URL의 쿼리 스트링에 포함되어 전송된다.

POST 방식: 데이터 생성 및 제출(Create) 시 사용된다. 데이터가 HTTP 패킷의 본문(Payload) 영역에 포함되어 전송된다.

[분석 결과]
로그인 요청 확인 결과, POST 메서드가 사용되며 비밀번호 등 민감한 데이터는 Payload 탭에 포함되어 전송됨을 확인하였다.
만약 로그인 폼이 method="GET"으로 구현될 경우, https://example.com/login?id=admin&pw=mySecret 형태처럼 브라우저 주소창, 방문 기록, 서버 접근 로그 등에 비밀번호가 평문으로 노출되는 정보 유출 취약점이 발생한다.

<img width="347" height="63" alt="image" src="https://github.com/user-attachments/assets/9d893369-7e7b-4a14-8b7a-2f095ff1034b" />


3. HTTP 상태 코드 (Status Code) 분석

클라이언트 요청에 따른 서버 응답 상태 코드를 확인하였다.

200 (OK): 정상적으로 로딩된 페이지 및 리소스의 성공 응답 코드.

302 (Found / Redirect): 로그인 성공 직후 메인 페이지 이동 시 확인. 서버가 클라이언트에게 다른 URL로 리다이렉션을 지시할 때 사용된다.

404 (Not Found): 존재하지 않는 URL 경로(/asdfasdf) 요청 시 발생. 클라이언트가 요청한 자원이 서버에 없음을 의미한다.

500 (Internal Server Error): 서버 내부 로직 처리 과정에서 예외가 발생했을 때 반환되는 서버 측 에러 코드.

4. HTTP 헤더 (Headers) 데이터 분석

네트워크 탭의 Request Headers(요청 헤더)를 통해 클라이언트가 서버로 전송하는 메타데이터를 확인하였다.

User-Agent: 사용자의 OS 환경, 브라우저 종류 및 버전을 나타낸다. (해당 값 변조 시 접속 기기 위장 가능)

Host: 클라이언트가 요청을 전송한 목적지 도메인을 명시한다.

Cookie: 브라우저가 현재 도메인에 대해 보유 중인 세션 쿠키 정보. 브라우저가 요청 헤더에 자동으로 포함하여 서버에 인증 상태를 전달한다.
<img width="319" height="18" alt="image" src="https://github.com/user-attachments/assets/ac20cf5f-1bbe-4099-a058-aa410102dde7" />

<img width="324" height="31" alt="image" src="https://github.com/user-attachments/assets/0b72ebfe-e6ec-430c-b548-aa649e9ff0c3" />


[학습 결론]
브라우저 네트워크 탭(F12)을 통한 패킷 흐름 관찰은 웹 통신 구조 파악에 효과적이다. 클라이언트에서 전송되는 쿼리 스트링과 헤더 데이터가 백엔드 서버에서 어떻게 파싱되고 처리되는지 아키텍처 관점에서 고려해야 한다.

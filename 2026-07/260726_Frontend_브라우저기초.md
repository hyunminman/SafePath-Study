브라우저 저장소 및 DOM, SOP 원리 분석

학습일: 2026.07.26
주제: 쿠키와 로컬 스토리지 구조 비교, DOM 트리 이해, SOP/CORS 개념 파악
목표: 클라이언트(브라우저) 환경에서의 데이터 저장 방식과 보안 정책 구조 

웹 서비스 취약점(XSS, CSRF 등)의 주된 타겟은 클라이언트인 브라우저 환경이다. 효율적인 방어 아키텍처 설계를 위해 개발자 도구(F12)를 활용하여 브라우저의 데이터 저장 구조와 렌더링 조작 원리를 분석하였다.

1. 브라우저 내장 저장소 분석 (Cookie vs Local Storage)

애플리케이션(Application) 탭을 통해 브라우저가 데이터를 저장하는 두 가지 핵심 저장소를 확인하고 차이점을 대조하였다.

Cookie (쿠키):

특징: 서버와 클라이언트가 통신할 때 HTTP 헤더에 자동으로 포함되어 전송된다. 주로 세션 ID 등 인증 상태를 유지하는 데 사용된다.

보안: HttpOnly 속성을 부여할 경우 자바스크립트(document.cookie)를 통한 접근이 차단되므로 XSS 공격에 의한 세션 탈취를 방어할 수 있다.

Local Storage (로컬 스토리지):

특징: 브라우저에 영구적으로 데이터를 저장하는 Key-Value 형태의 저장소이다. HTTP 요청 시 서버로 자동 전송되지 않는다.

취약점: 자바스크립트를 통해 언제든지 자유롭게 접근이 가능하다. 따라서 로그인 토큰(JWT 등)을 이곳에 저장할 경우, XSS 공격 발생 시 탈취당할 위험이 높다.

<img width="563" height="311" alt="image" src="https://github.com/user-attachments/assets/4176876c-d8bd-416d-abec-e662ada8c885" />
<img width="548" height="297" alt="image" src="https://github.com/user-attachments/assets/097a2e1d-65ee-4db8-9431-bb341ef51d9c" />


2. DOM (Document Object Model) 구조 및 조작

서버로부터 전달받은 HTML 코드가 브라우저 화면에 렌더링되는 원리를 파악하고 동적 조작을 실습하였다.

DOM의 개념: 브라우저가 HTML 문서를 파싱하여 트리(Tree) 구조의 객체 모델로 변환한 결과물이다.

콘솔(Console) 조작 실습:

콘솔 탭에서 document.body.style.backgroundColor = 'black'; 등의 자바스크립트 명령어를 입력하여 화면 구조를 실시간으로 변조하는 테스트를 진행하였다.

[분석 결과] 자바스크립트를 통해 DOM을 임의로 조작할 수 있다는 특성을 악용하여, 정상적인 웹 페이지 구조를 피싱 사이트 폼으로 덮어씌우거나 악성 팝업을 띄우는 것이 XSS 공격의 동작 메커니즘임을 체감하였다.

<img width="1855" height="1031" alt="image" src="https://github.com/user-attachments/assets/51c726b6-199d-4e15-b703-0a10219cf412" />


3. SOP (동일 출처 정책) 및 CORS 원리 파악

브라우저가 내장하고 있는 가장 기본적인 클라이언트 사이드 보안 정책의 개념을 정리하였다.

SOP (Same-Origin Policy): 현재 접속 중인 웹 사이트(Origin)와 출처(도메인, 포트, 프로토콜)가 다른 외부 사이트의 데이터를 자바스크립트로 임의로 읽어오지 못하게 차단하는 브라우저의 기본 보안 규칙이다.

CORS (Cross-Origin Resource Sharing): 프론트엔드와 백엔드 서버가 분리된 아키텍처에서는 SOP를 우회한 교차 출처 통신이 필수적이다. 이때 백엔드 서버에서 HTTP 응답 헤더(Access-Control-Allow-Origin)를 통해 특정 출처의 접근을 명시적으로 허용하는 과정이 CORS이다. 콘솔에서 강제로 외부 도메인 API를 Fetch하여 CORS 블락 에러를 직접 확인하였다.

[학습 결론]
브라우저는 단순한 뷰어(Viewer) 역할을 넘어, 데이터를 저장하고 코드를 실행하는 독립적인 클라이언트 환경이다. 프론트엔드 개발 시 클라이언트 측 저장소의 한계와 브라우저 자체 보안 정책(SOP)을 명확히 인지하고 데이터 바인딩 로직을 설계해야 한다.

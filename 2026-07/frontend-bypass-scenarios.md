OWASP 프론트엔드 취약점 분석 및 브라우저 신뢰 우회 실습

1. 프로젝트 개요

실습 목적: HTTP 통신 구조 이해, 클라이언트 사이드 취약점 증명 및 방어 대책 수립

사용 기술 스택: WebGoat (Docker), Burp Suite (Community Edition), Chrome DevTools

2. 실습 환경 구성

대상 서버: Docker를 이용한 WebGoat 구동 (로컬 포트 8080)

프록시 도구: Burp Suite 내장 브라우저를 활용하여 127.0.0.1 환경의 통신 패킷 인터셉트 구성 (포트 충돌 방지를 위해 리스너 포트 8081로 변경하여 환경 구축 완료)

3. 상세 실습 내용 및 결과

3.1. HTTP 패킷 인터셉트 및 파라미터 변조

웹 서버와 클라이언트 간에 오가는 HTTP Request 및 Response 통신 과정을 분석하고, 클라이언트의 입력값을 서버로 전송하기 전에 임의로 조작하여 서버의 비정상적인 응답을 유도하였습니다.

과정:

Burp Suite의 Proxy 기능을 활성화하여 브라우저에서 WebGoat 서버로 향하는 통신을 가로채기(Intercept) 하였습니다.

HTTP Request 패킷 내의 특정 파라미터 값을 공격자 의도대로 변조한 후 서버로 전송(Forward)하였습니다.

실습 증적:

<img width="1573" height="881" alt="image" src="https://github.com/user-attachments/assets/e2eeb20a-e77b-425d-b275-2fc354f11acf" />
<img width="918" height="930" alt="image" src="https://github.com/user-attachments/assets/0aefb971-6f1c-4e80-9435-34f8bd63586d" />


3.2. XSS 공격 및 세션 하이재킹

사용자 입력값에 대한 서버의 필터링이 미흡한 취약점을 이용하여, 악의적인 자바스크립트를 삽입하고 다른 사용자의 세션 권한을 탈취하는 실습을 진행하였습니다.

과정:

Reflected XSS: URL 파라미터가 화면에 즉시 반사되어 출력되는 구간에 자바스크립트 페이로드를 삽입하여 실행을 증명하였습니다.

Stored XSS: 게시판 댓글 입력란에 악성 스크립트를 영구적으로 주입하여, 해당 게시글을 열람하는 모든 사용자의 브라우저에서 스크립트가 강제 실행되도록 구현하였습니다.

Session Hijacking: 삽입한 스크립트를 통해 사용자 브라우저의 document.cookie 값을 읽어 들인 후, 이를 활용해 관리자 권한 로그인을 우회하는 시나리오를 시연하였습니다.

실습 증적:

<img width="1112" height="522" alt="image" src="https://github.com/user-attachments/assets/41981a2b-66ae-4726-b9f4-9f9a79f07137" />

<img width="1114" height="571" alt="image" src="https://github.com/user-attachments/assets/a21e9a34-ca1f-4494-befe-6ec2c05c5055" />

3.3. CSRF 공격 및 인프라 취약점 탐색

사용자가 시스템에 로그인하여 획득한 인증된 세션을 악용하여, 사용자 의도와 무관한 조작을 유발하는 구조를 분석하였습니다.

과정:

CSRF: 인증된 세션을 유지하고 있는 사용자가 악의적으로 조작된 폼이나 링크를 클릭하도록 유도하여 비밀번호 강제 변경 등의 패킷을 전송시켰습니다.

설정 오류 탐색: URL 경로 조작(../)을 통해 상위 디렉토리 접근을 시도하고, 웹 미들웨어(Tomcat) 및 OS 버전이 노출되는 에러 페이지를 확인하였습니다.

실습 증적:

<img width="1119" height="508" alt="image" src="https://github.com/user-attachments/assets/4fd0dca8-b44c-4cf0-ad86-81af1b2eb1cb" />

<img width="1117" height="471" alt="image" src="https://github.com/user-attachments/assets/f2170ab4-dc9e-4771-be06-91496fce265d" />

4. 공격 시나리오 시퀀스 다이어그램

본 실습에서 진행한 Stored XSS 및 Session Hijacking 공격의 흐름도입니다.

sequenceDiagram
    participant Attacker as 공격자
    participant Server as 웹 서버
    participant Victim as 일반 사용자
    
    Attacker->>Server: 악성 페이로드(JS)가 포함된 게시글 작성 (Stored XSS)
    Server-->>Server: 데이터베이스에 페이로드 영구 저장
    Victim->>Server: 해당 게시글 열람 요청
    Server-->>Victim: 악성 스크립트가 포함된 웹 페이지 응답
    Victim-->>Victim: 브라우저에서 악성 스크립트 자동 실행
    Victim->>Attacker: 세션 쿠키(document.cookie) 유출
    Attacker->>Server: 탈취한 세션을 이용해 관리자 권한 도용


5. 코드 수준의 방어 대책 (Mitigation)

본 실습을 통해 확인한 클라이언트 사이드 취약점들을 근본적으로 해결하기 위한 웹 애플리케이션 보안 대책입니다.

XSS 취약점 방어

입력값 치환 (HTML Entity Encoding): 사용자의 입력값이 HTML 태그로 해석되지 않도록, 서버 단에서 <를 &lt;로, >를 &gt;로 치환하여 안전하게 출력해야 합니다.

HttpOnly 쿠키 설정: XSS 공격을 당하더라도 세션이 탈취되는 것을 막기 위해, 인증 쿠키 발급 시 자바스크립트에서 접근할 수 없도록 HttpOnly 속성을 부여해야 합니다.

CSRF 취약점 방어

CSRF Token 도입: 클라이언트에게 예측 불가능한 임의의 난수 토큰을 발급하고, 폼 데이터 전송 시 해당 토큰의 일치 여부를 서버에서 검증해야 합니다.

SameSite 쿠키 속성: 타 도메인에서 시작된 요청에 대해서는 쿠키가 전송되지 않도록 SameSite=Lax 또는 Strict 속성을 적용해야 합니다.

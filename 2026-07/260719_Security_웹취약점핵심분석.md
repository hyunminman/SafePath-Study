[Security] 주요 웹 서비스 취약점 분석 (SQLi, XSS, CSRF)

작성일: 2026.07.19
주제: Dreamhack 기반 웹 해킹 공격 벡터 분석 및 시큐어 코딩(Secure Coding) 방어 대책

1. SQL Injection (SQLi)

사용자의 입력값이 데이터베이스 쿼리의 일부로 실행되게 만드는 치명적인 공격.

공격 원리: 서버가 사용자의 입력을 적절히 이스케이프(Escape) 처리하지 않고 문자열 연결(String Concatenation) 방식으로 쿼리를 조립할 때 발생한다.

공격 시나리오 (Bypass Authentication):

-- 서버의 원래 쿼리
SELECT * FROM users WHERE userid = '{사용자입력}' AND password = '{사용자입력}';

-- 공격자의 입력값: admin' --
-- 조작된 쿼리 (비밀번호 검증 로직이 주석 처리되어 무력화됨)
SELECT * FROM users WHERE userid = 'admin' --' AND password = '';


대응 방안 (Secure Coding):

Prepared Statement 적용: 쿼리의 로직(구문)과 데이터(입력값)를 철저히 분리한다. 입력값은 쿼리의 명령어로 해석되지 않고 순수 텍스트 데이터로만 바인딩된다.

ORM 사용 시 대부분 내장되어 있으나, Raw Query 작성 시 각별한 주의가 필요하다.

2. XSS (Cross-Site Scripting)

목표: 타겟 사용자의 브라우저에서 악성 자바스크립트를 실행시켜 세션 쿠키, 토큰 등을 탈취한다.

종류:

Stored XSS: 게시판 등에 악성 스크립트가 DB에 저장되어 발생.

Reflected XSS: 악성 URL 클릭 시 요청에 포함된 스크립트가 즉시 반사되어 실행.

방어: 프론트엔드 및 백엔드 단에서 출력값 무해화(Sanitization) 필수. <script> 등의 HTML 특수문자를 &lt;script&gt; 형태의 엔티티 코드로 치환한다.

3. CSRF (Cross-Site Request Forgery)

목표: 사용자가 자신의 의지와 무관하게 공격자가 의도한 행위(송금, 비밀번호 변경 등)를 서버에 요청하도록 만듦. (사용자의 인증된 세션을 도용).

방어: * CSRF Token: 상태를 변화시키는 요청(POST, PUT, DELETE 등) 시마다 서버에서 발급한 난수 토큰을 함께 전송하여 유효성을 검증한다.

SameSite Cookie 속성: 쿠키 설정 시 SameSite=Lax 또는 Strict를 적용하여 타 도메인에서의 악의적인 요청에 쿠키가 포함되지 않도록 원천 차단한다.

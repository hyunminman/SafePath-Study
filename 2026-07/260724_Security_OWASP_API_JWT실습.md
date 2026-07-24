[Week 3] 공통 스터디: 백엔드 API 인가 결함 및 JWT 검증 우회 분석

학습일: 2026.07.24
주제: Boolean-based Blind SQLi, API 인가 결함(BOLA), JWT 위변조 취약점 분석
도구: Docker WebGoat, Postman, jwt.io

본 실습에서는 웹 애플리케이션의 백엔드 API 설계 시 발생할 수 있는 인가(Authorization) 결함과, JWT(JSON Web Token)의 구조적 취약점을 이용한 인증 우회 기법을 테스트하고 방어 대책을 정리했다.

1. Blind SQL Injection (Boolean-Based) 트러블슈팅

데이터베이스 쿼리 결과나 에러 메시지가 화면에 노출되지 않는 환경에서, 서버의 '응답 메시지 차이(True/False)'를 이용해 내부 데이터를 유추하는 기법을 실습했다.

실습 및 분석 결과
WebGoat의 회원가입(REGISTER) API에서 취약점을 테스트했다.

취약점 증명 (성공): tom' AND '1'='1 주입 시 논리 연산이 참(True)이 되어 "User already exists" 응답이 반환됨을 확인했다.

데이터 추출 (실패): tom' AND (SELECT LENGTH(password) FROM users WHERE userid='tom')=23 페이로드를 주입하여 타겟 계정의 패스워드 길이를 추출하려 시도했다. 그러나 현재 실습 환경의 데이터베이스 버전 및 문법 제한으로 인해 쿼리가 정상 실행되지 못하고 에러(Something went wrong)가 발생하여 실패했다.

결론: 세부 데이터 추출은 환경적 제약으로 실패했으나, 참/거짓 반응의 차이('1'='1' vs '1'='2')만으로 내부 상태를 유추할 수 있는 Boolean-Based Blind SQLi의 근본 원리는 성공적으로 증명했다.

<img width="748" height="766" alt="image" src="https://github.com/user-attachments/assets/da7f8880-cf74-42f8-8e63-be8906c38eab" />


2. BOLA / IDOR (API 인가 결함)

BOLA(Broken Object Level Authorization)는 클라이언트가 요청하는 리소스의 식별자(ID)에 대해 백엔드가 적절한 소유권 검증을 수행하지 않을 때 발생하는 API 취약점이다.

실습 및 동작 원리
Postman을 활용하여 웹 프론트엔드를 거치지 않고 서버의 API 엔드포인트에 직접 HTTP 요청을 전송하여 인가 로직을 테스트했다.

요청 변조: 정상적인 프로필 조회 API(예: GET /WebGoat/IDOR/profile/2342384)의 URL 파라미터를 타인의 식별자로 추정되는 /2342388로 임의 변조하여 전송했다.

분석 결과: 백엔드에서 현재 요청을 보낸 세션의 주체와 요청된 리소스의 실제 소유자가 일치하는지 교차 검증하는 로직이 누락되어 있었다. 그 결과, HTTP 200 상태 코드와 함께 타 사용자의 개인정보가 JSON 형태로 응답되는 것을 확인했다.

<img width="543" height="279" alt="image" src="https://github.com/user-attachments/assets/42a5a6a1-f12c-4a4e-98ad-b1c8627a8222" />
<img width="1316" height="821" alt="image" src="https://github.com/user-attachments/assets/62db2773-9b8c-4d5c-93c2-99b360e57675" />


3. JWT 토큰 위변조 및 소스코드 취약점 분석

무상태(Stateless) 인증 방식인 JWT의 구조적 특징과, 백엔드 라이브러리의 잘못된 서명(Signature) 검증 코드 구현 시 발생하는 권한 탈취 취약점을 실습했다.

실습 및 동작 원리
JWT 검증 로직을 수행하는 소스코드 스니펫을 비교 분석하여 none 알고리즘 처리 시의 예외 예방 조치를 검증했다.

안전한 코드 구조: 토큰 내부의 Header 파트 알고리즘이 "alg": "none"으로 조작되어 들어올 경우, 서명이 없음을 감지하고 코드 라인에서 명시적인 Exception(예외 에러)을 발생시켜 요청을 차단한다.

취약한 코드 구조: 알고리즘이 none일 때 분기문이 서명 검증 로직 자체를 스킵(Skip)하도록 설계된 경우, 서명이 없는 토큰임에도 무사히 통과되어 관리자 권한 메서드(removeAllUsers)가 강제로 호출(Invoked)된다.

분석 결과: 백엔드 검증 라이브러리 설정 시 none 알고리즘을 명시적으로 거부하지 않으면, 공격자가 헤더와 페이로드를 임의 변조(admin: true)하여 서명 없이 제출하더라도 비즈니스 로직이 수행되는 치명적인 취약점이 발생함을 확인했다.

<img width="1496" height="444" alt="image" src="https://github.com/user-attachments/assets/c339c838-b248-4595-abd7-fd536b710757" />


4. 학습 결론 및 시사점

이번 실습을 통해 API 설계 및 토큰 기반 인증 시스템에서 백엔드 보안 로직의 중요성을 확인했다.

객체 수준 인가(BOLA 방어): API가 민감한 리소스를 반환하거나 수정할 때는, 단순 인증 여부(로그인 상태)만 확인하는 것을 넘어 현재 사용자(User)가 해당 리소스(Object)의 실제 소유자가 맞는지 교차 검증하는 로직이 반드시 포함되어야 한다.

안전한 JWT 검증 로직 구현: JWT 라이브러리 사용 시 취약한 "alg": "none"이 허용되지 않도록 유의해야 하며, 백엔드에서 허용된 알고리즘(예: HS256, RS256 등)만을 처리하도록 명시적인 화이트리스트 검증을 강제해야 한다.

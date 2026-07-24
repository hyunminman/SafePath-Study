[Week 3] WebGoat 기반 SQL Injection 실습 및 취약점 분석

학습일: 2026.07.24
주제: 로컬 Docker 환경에서 WebGoat 구동 및 SQLi 원리 분석

이론으로 학습했던 SQL 인젝션 취약점을 OWASP WebGoat 환경에서 직접 테스트하고 동작 원리를 확인했다.

1. WebGoat 도커 환경 세팅

안전한 테스트를 위해 로컬 PC 대신 Docker 컨테이너를 활용하여 격리된 환경을 구축했다.

명령어: docker run -p 8080:8080 -p 9090:9090 -e TZ=Asia/Seoul webgoat/webgoat

컨테이너 실행 후 localhost:8080/WebGoat에 접속하여 실습용 계정을 생성하고 로그인을 완료했다.

<img width="1236" height="599" alt="image" src="https://github.com/user-attachments/assets/22ff6f3c-2bff-4aa8-8ef9-ed469f1f04c9" />


2. Normal SQL Injection (인증 우회)

Injection Flaws -> SQL Injection (intro) 메뉴에서 실습을 진행했다.

목표: 패스워드를 모르는 상태에서 특정 계정으로의 인증 우회 테스트.

입력 폼에 admin' OR '1'='1 페이로드를 주입했다. 이 페이로드가 삽입되면 백엔드에서 실행되는 쿼리는 다음과 같이 조작된다.

SELECT * FROM users WHERE id='admin' OR '1'='1' AND pw=''

'1'='1' 조건이 수학적으로 항상 참(True)이 되므로, 뒤따르는 패스워드 일치 여부와 무관하게 WHERE 조건절이 성립하여 인증이 우회됨을 확인했다.

<img width="873" height="785" alt="image" src="https://github.com/user-attachments/assets/cf27d868-97aa-464d-bb38-e82caac88328" />


3. Blind SQL Injection (회원가입 폼 취약점 악용)

SQL Injection (advanced) 메뉴에서 로그인 폼을 직접 우회하려 했으나 실패했다. 원인을 분석해보니, 로그인 로직은 PreparedStatement가 적용되어 SQL 인젝션이 불가능한 안전한 상태였다. 반면, REGISTER(회원가입) 탭의 '아이디 중복 체크' 로직에는 문자열 결합이 사용되어 취약점이 존재했다.

이를 악용하여 시간 기반 및 응답 기반의 Blind SQL 인젝션을 수행할 수 있음을 확인했다.

공격 원리: REGISTER 탭에서 tom' AND (SELECT LENGTH(password) FROM users WHERE userid='tom')=23 AND '1'='1 과 같은 참/거짓 쿼리를 던져, "이미 존재하는 계정"이라는 응답이 오는지 확인하는 방식으로 데이터를 한 글자씩 유추해낸다.

탈취 결과: 위와 같은 공격을 반복 수행하여 tom의 패스워드 길이가 23자리이며, 실제 패스워드가 thisisasecretfortomonly임을 알아낼 수 있다.

최종 로그인: 알아낸 패스워드로 안전한 로그인 폼에 정상적으로 인증하여 타겟 계정(tom) 탈취에 성공했다.

<img width="888" height="677" alt="image" src="https://github.com/user-attachments/assets/f80ab679-2850-4c6d-abbb-7b2c91c11ab9" />

(설명: Blind SQLi로 알아낸 패스워드를 사용하여 tom 계정 로그인에 성공한 화면)


💡 실습 회고

이번 실습을 통해 입력값 검증 부재가 가져오는 보안 위협을 직접 확인했으며, 시큐어 코딩의 중요성을 인지했다.

Prepared Statement 적용: 사용자 입력값이 쿼리의 논리 구조로 해석되지 않도록, 반드시 바인딩 변수 처리를 통해 데이터로만 취급되게 해야 한다.

안전한 예외 처리: DB 쿼리 실행 중 에러가 발생하더라도 스택 트레이스나 쿼리 원문이 클라이언트에 노출되지 않도록, 서버 단에서 에러 응답을 정제(예: 500 Internal Server Error)하여 반환하는 처리가 필수적이다.

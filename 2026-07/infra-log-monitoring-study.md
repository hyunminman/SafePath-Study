[주간 학습 보고서] 인프라 인지 및 실시간 로그 관측 실습

학습 일자: 2026년 7월 14일

작성자: 최현민

1. 학습 개요 및 목표

학습 주제: 인프라 인지 및 실시간 로그 관측 메커니즘 이해

학습 목표: 실제 해킹 공격이나 시스템 오류 발생 시, 웹 서버(Nginx)와 앱 서버(Spring Boot)가 각각 어떤 로그를 남기는지 관측하고 데이터 구조를 분석한다. 이를 통해 실시간 보안 관제 및 트러블슈팅의 기반을 다진다.

2. 활용 기술 스택 및 학습 자료

기술 스택: Docker, Nginx, Java / Spring Boot, Logback (SLF4J)

학습 소스: * Docker Desktop 공식 가이드 문서

Nginx access log 분석 및 Spring Boot Logback 설정 관련 스터디

인프런 강의: 스프링부트 REST API, 스프링 시큐리티 스터디 내용 접목

3. 실습 진행 내용

3.1. Docker 환경 구성 및 연동

docker-compose를 활용하여 Nginx 웹 서버 컨테이너와 Spring Boot 애플리케이션 서버 컨테이너를 각각 구동.

Nginx를 Reverse Proxy로 설정하여 클라이언트의 요청이 Nginx를 거쳐 Spring Boot로 전달되도록 아키텍처 구성.

3.2. 실시간 로그 관측 (정상 흐름)

브라우저를 통해 http://localhost/api/users 에 접속하며 로그가 쌓이는 과정을 확인.

관측 방법: 터미널에서 docker logs -f [nginx_container] 및 docker logs -f [springboot_container] 명령어를 통해 두 컨테이너의 로그를 실시간 스트리밍으로 동시 관측.

3.3. 비정상 요청 및 에러 유발 관측 (해킹 시뮬레이션)

의도적 오류 발생: 일부러 존재하지 않는 주소인 http://localhost/hack 경로로 강제 접근 시도.

결과 확인: 정상적인 HTTP 200 OK 로그와 달리, 404 Not Found 발생 시 두 계층(Web, App)의 로그 포맷이 어떻게 달라지는지 대조 관측.

4. 실습 결과물 (로그 관측 캡처)

4.1. 서버별 정상/에러 로그 포맷 구조 분석

 [Case 1] 정상 요청 시 (HTTP 200 OK)

Nginx는 클라이언트의 IP, 시간, HTTP 메서드, 상태 코드(200)를 기록함.

Spring Boot는 Logback을 통해 비즈니스 로직에 따른 INFO 레벨의 로그를 성공적으로 출력함.

<img width="1370" height="890" alt="image" src="https://github.com/user-attachments/assets/5cdf4415-c0de-4f7f-838c-3e401c4a8236" />



 [Case 2] 비정상 요청 / 해킹 시도 시 (HTTP 404 Not Found)

Nginx는 비정상적인 URI(/hack) 접근 시도와 404 상태 코드를 즉각 기록함.

Spring Boot는 맵핑되지 않은 주소에 대한 경고(WARN / No mapping for GET)를 띄워, 어플리케이션 계층에서의 방어 및 기록 상태를 보여줌.

<img width="1373" height="940" alt="image" src="https://github.com/user-attachments/assets/d1f95ab1-9480-40b5-b928-340a0c14ee8f" />



4.2. 실시간 관제를 위한 핵심 식별자 추출 정의 테이블

실시간 로그 모니터링 시, 빠른 위협 탐지와 장애 대응을 위해 파싱(Parsing)하여 반드시 추출해야 할 핵심 식별자 4가지를 다음과 같이 정의함.

식별자 (Identifier)

설명 (Description)

추출 목적 및 보안/운영 관점의 의미

Client IP

요청을 보낸 클라이언트 IP

특정 IP의 과도한 요청(DDoS), 악의적 크롤링 차단 및 공격자 추적.

Timestamp

요청이 발생한 정확한 시간

공격 타임라인 구성, 에러 발생 시점 특정, Nginx-SpringBoot 로그 간 연관 분석(Correlation).

Request URL

클라이언트가 요청한 URI 경로

SQL Injection, Path Traversal 등 악의적 페이로드가 포함된 스캐닝 요청 파악.

Status Code

HTTP 응답 상태 코드

400번대(비정상 접근/스캐닝 시도), 500번대(서버 내부 로직 에러) 등 서버 상태 즉각 모니터링.

5. 결론 및 인사이트

이번 실습을 통해 Nginx(웹 서버)의 앞단 로그와 Spring Boot(앱 서버)의 뒷단 로그가 어떻게 유기적으로 연결되는지 눈으로 확인했다. 특히, 에러를 고의로 유발했을 때 각 서버 계층에서 남는 흔적의 차이를 비교해 보며 인프라 가시성(Visibility)의 중요성을 깨달았다. 인프라를 블랙박스로 두지 않고 로그를 체계적으로 관측함으로써, 실제 운영 환경에서 해킹 공격이나 장애 발생 시 데이터에 기반하여 신속하게 원인을 추적하고 대응할 수 있는 기초 역량을 확보했다.

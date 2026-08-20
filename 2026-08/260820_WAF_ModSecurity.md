[Week 3] WAF (ModSecurity + OWASP CRS) 구축 및 방어 실습

학습일: 2026.08.20
주제: Nginx 기반 ModSecurity WAF 구축 및 실시간 웹 공격(SQLi, XSS) 차단 관제
목표: OWASP A09(보안 로깅 및 모니터링 실패) 대응을 위한 인바운드 트래픽 방어 인프라 구축

웹 애플리케이션 프레임워크(Spring Boot)에 트래픽이 도달하기 전, 네트워크 앞단(L7)에서 악의적인 공격을 선제적으로 차단하기 위해 ModSecurity와 OWASP CRS(Core Rule Set) 기반의 WAF(Web Application Firewall)를 구축하고 방어 성능을 검증했다.

1. WAF 인프라 환경 구축 (Docker)

실무에서 자주 사용되는 Nginx + ModSecurity 조합을 복잡한 컴파일 과정 없이 즉각적으로 배포하기 위해, OWASP CRS 공식 Docker 이미지를 활용하여 격리된 컨테이너 환경을 구성했다.

실행 명령어:
docker run -d --name waf-test -p 8080:8080 owasp/modsecurity-crs:nginx

초기 구동 확인:
컨테이너 구동 후 http://localhost:8080으로 접속하여 502 Bad Gateway 또는 기본 Nginx 에러 페이지가 노출되는 것을 확인했다. 이는 WAF 뒤에 연동된 백엔드(Upstream) 서버가 아직 없어서 발생하는 정상적인 에러로, Nginx 엣지 서버가 트래픽을 정상적으로 수신하고 있음을 의미한다.

<img width="1110" height="364" alt="image" src="https://github.com/user-attachments/assets/3e88e6a6-6def-4125-87e8-7f9364389eaa" />


2. 웹 공격 패턴 차단 테스트 (SQLi & XSS)

구축된 WAF의 룰셋(OWASP CRS)이 정상적으로 동작하는지 확인하기 위해, 브라우저를 통해 정상 요청과 악의적인 페이로드가 포함된 요청을 대조하여 발송했다.

2.1. 정상 요청 테스트 (Bypass)

요청: http://localhost:8080/?name=test

결과: WAF가 안전한 파라미터로 인식하여 통과시켰고, 백엔드가 없으므로 초기와 동일한 Nginx 에러 페이지가 출력되었다.

<img width="1116" height="478" alt="image" src="https://github.com/user-attachments/assets/b5219dfb-1ac8-4613-ba82-341f265f2ad4" />

2.2. SQL Injection 공격 테스트 (Block)

요청: http://localhost:8080/?id=1' OR '1'='1

결과 (차단 성공): 즉시 403 Forbidden 상태 코드가 반환되었다. WAF 엔진이 파라미터 내의 SQL 논리식(OR '1'='1')을 악성 페이로드로 탐지하여 연결을 강제 종료했다.

<img width="1110" height="384" alt="image" src="https://github.com/user-attachments/assets/765e2026-5241-40d6-bc60-d0702b47bb9a" />

2.3. XSS (Cross-Site Scripting) 공격 테스트 (Block)

요청: http://localhost:8080/?q=<script>alert('hack')</script>

결과 (차단 성공): 역시 403 Forbidden으로 즉각 차단되었다. <script> 태그를 통한 스크립트 인젝션 시도를 CRS 룰이 성공적으로 방어함을 확인했다.

<img width="1122" height="378" alt="image" src="https://github.com/user-attachments/assets/f4582425-c48c-4da3-8c8b-3e3eb9ad5729" />

3. 실습 회고 및 다음 단계

애플리케이션 코드(백엔드) 단에서의 방어(시큐어 코딩)도 중요하지만, 인프라 레벨에 WAF를 배치함으로써 SQLi나 XSS와 같은 널리 알려진 패턴 공격(Zero-day 이전의 공격)을 대규모로 필터링할 수 있는 '심층 방어(Defense in Depth)' 아키텍처의 중요성을 체감했다.

다음 단계로는 애플리케이션 내부 로깅 시스템에 MDC(Mapped Diagnostic Context)를 도입하여, WAF를 통과한 요청들의 흐름을 고유 Trace-ID로 추적하는 시큐어 로깅 파이프라인을 구축할 예정이다.

<img width="1122" height="378" alt="image" src="https://github.com/user-attachments/assets/21def760-bc1c-43b7-bbd5-f9948333d4ce" />

(설명: 공격 테스트 시 브라우저에 출력된 403 Forbidden 차단 화면 스크린샷)

[Week 3] 애플리케이션 시큐어 코딩: MDC 기반 Trace-ID 로그 추적 체계 구축

학습일: 2026.08.20
주제: 분산 환경에서의 요청 추적성 확보를 위한 MDC(Mapped Diagnostic Context) 기반 로깅 필터 구현
목표: 백엔드(Spring Boot)로 인입되는 수많은 요청을 식별하고, WAF(ModSecurity)에서 통과된 요청의 전체 흐름을 추적(Trace)하기 위한 내부 로깅 아키텍처 구축

앞선 Step 1에서 인프라 네트워크 앞단(L7)의 보안(WAF)을 강화했다면, 이번 단계에서는 애플리케이션(Spring Boot) 내부로 들어온 정상적인 트래픽들이 어떤 로직을 타고 흘러가는지 추적하기 위한 시큐어 로깅 체계를 구축했다.

1. MDC (Mapped Diagnostic Context) 도입 배경

수많은 클라이언트 요청이 Tomcat 스레드 풀에서 병렬로 처리될 때, 일반적인 로그 파일만으로는 어떤 로그가 어떤 사용자의 요청인지 식별하는 것이 불가능하다.
이를 해결하기 위해, 각 스레드(Thread)별로 고유한 컨텍스트(Map 구조)를 유지할 수 있는 SLF4J MDC를 도입하여, 요청의 진입점부터 응답까지 동일한 식별자(Trace-ID)를 공유하도록 아키텍처를 설계했다.

2. Trace-ID 발급 필터(Filter) 구현

요청이 컨트롤러(Controller)에 도달하기 전, 가장 최전방인 Filter 단에서 고유 ID를 발급하고 MDC에 바인딩하는 로직을 구현했다.

구현 핵심 (MdcLoggingFilter.java):

OncePerRequestFilter를 상속받아 HTTP 요청당 한 번만 필터가 동작하도록 보장.

UUID.randomUUID()를 통해 고유한 Trace-ID 생성.

사용자의 Client-IP를 함께 추출하여 MDC 객체에 보관.

[보안/메모리 관리 포인트]: finally 블록에서 MDC.clear()를 명시적으로 호출. 스레드 풀 환경에서 스레드가 반환되고 재사용될 때, 이전 요청의 식별자 데이터가 남아있어 로그가 오염되는 크리티컬한 버그를 원천 차단함.

3. Logback 설정 고도화 (PatternLayout)

MDC에 저장된 데이터(traceId, clientIp)를 실제 애플리케이션 로그에 자동으로 삽입하기 위해 logback-spring.xml의 설정 포맷을 변경했다.

변경 전 포맷: %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n

변경 후 포맷 (MDC 연동): %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [TraceID: %X{traceId}] [IP: %X{clientIp}] %logger{36} - %msg%n

동작 원리: %X{키} 문법을 사용하여 필터에서 MDC.put("traceId", ...)로 넣었던 값을 호출해, 모든 logger.info() 출력 시마다 자동으로 꼬리표가 붙도록 자동화했다.

4. 실습 결과 및 트러블슈팅

테스트 컨트롤러(http://localhost:8080/test)를 호출하여 로깅 결과를 검증하는 과정에서 인프라 레벨의 충돌을 경험했다.

트러블슈팅 (Port Conflict):
Spring Boot 서버 기동 중 Web server failed to start. Port 8080 was already in use. 에러 발생.
원인: 앞선 실습에서 구동해둔 Docker WAF 컨테이너가 8080 포트를 점유하고 있어 발생한 충돌.
해결: docker stop waf-test 명령어로 컨테이너를 중지(또는 서버 포트 8081로 변경)하여 인프라 충돌을 해결하고 정상 기동 확인.

결과 검증:
여러 차례 새로고침을 수행한 결과, 콘솔 창에 각 요청마다 고유한 [TraceID: 550e8400-...]가 정상적으로 찍히는 것을 확인했다. 이를 통해 추후 ELK(Kibana) 관제 시, 특정 에러가 발생한 TraceID만 검색하면 해당 요청이 거쳐 간 모든 서비스 로직을 한눈에 역추적할 수 있는 기반이 완성되었다.

<img width="1230" height="141" alt="image" src="https://github.com/user-attachments/assets/783206ed-2b14-4115-99eb-488479a649aa" />

(설명: VS Code 터미널에서 TraceID와 IP가 찍혀 있는 로그 실행 화면 스크린샷)

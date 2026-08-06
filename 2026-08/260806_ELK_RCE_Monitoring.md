ELK 스택 기반 RCE 공격 탐지 파이프라인 구축

학습일: 2026.08.06
주제: Logstash 및 Kibana를 활용한 악성 페이로드 실시간 모니터링
기술 스택: ELK(Elasticsearch, Logstash, Kibana), Docker

개발 단계의 시큐어 코딩을 넘어, 인프라 운영 관점에서 애플리케이션으로 유입되는 RCE(원격 코드 실행) 공격 시도를 실시간으로 탐지하기 위한 로그 관제 파이프라인을 구축했다.

1. Logstash 탐지 패턴 구현

웹 서버(Nginx/Spring Boot)의 접근 로그를 수집하고 분석하기 위해 logstash.conf 필터 로직을 구성했다. 공격자가 주로 사용하는 Java Runtime 및 프로세스 실행 관련 키워드를 정규표현식으로 정의하여 탐지하도록 설정했다.

Logstash 필터 설정:
Runtime, ProcessBuilder, exec, 그리고 템플릿 인젝션에 사용되는 달러 기호 인코딩 패턴(%24{)이 로그에 포함될 경우, 해당 로그에 malicious_rce_attempt 태그를 부착하여 Elasticsearch로 전송하도록 파이프라인을 구성했다.

filter {
  if [message] =~ /(?i)(Runtime|ProcessBuilder|exec\(|%24\{)/ {
    mutate {
      add_tag => ["malicious_rce_attempt"]
    }
  }
}


2. 공격 시뮬레이션 및 Kibana 대시보드 관제

탐지 시스템의 정상 동작 여부를 검증하기 위해 로컬 웹 서버에 악성 페이로드 파라미터가 포함된 HTTP 요청을 다수 발생시켰다.

이후 Kibana 대시보드에 접속하여 security-logs-* 인덱스 패턴을 대상으로 KQL(Kibana Query Language) 검색을 수행했다.

검색 쿼리: tags: "malicious_rce_attempt"

관제 결과: 필터링 태그를 기반으로 검색한 결과, 앞서 시도했던 RCE 및 SSTI 공격 로그들이 정상적으로 식별되어 리스트업되는 것을 확인했다.

<img width="1515" height="495" alt="image" src="https://github.com/user-attachments/assets/8a0e1392-50c9-4458-8d11-14e0e663e900" />

(설명: Kibana Discover 탭에서 태그 조건으로 악성 로그들을 필터링하여 찾아낸 대시보드 캡처 화면)

3. 회고

단일 애플리케이션의 방어 로직만으로는 고도화되는 우회 공격을 모두 막아낼 수 없다. ELK 스택과 같은 중앙 집중식 로그 관제 시스템을 통해 비정상적인 트래픽 패턴을 지속적으로 모니터링하고 가시화하는 것이 엔터프라이즈 환경 보안의 핵심임을 학습했다.

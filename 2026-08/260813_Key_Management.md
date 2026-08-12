암호화 키(Secret Key) 하드코딩 방지 및 환경변수 분리

학습일: 2026.08.13
주제: OWASP A02 방어를 위한 안전한 Key Management 적용

1. 취약점 분석 (Key 하드코딩)

소스코드 내부에 String TEMP_KEY = "123456...;" 와 같이 비밀키를 직접 하드코딩하는 것은 가장 전형적인 암호화 실패(Cryptographic Failure) 취약점이다.
코드가 GitHub 등 형상관리 시스템에 업로드되거나, 퇴사한 직원에 의해 소스코드가 유출될 경우 암호화된 모든 데이터가 즉시 복호화되는 대형 사고로 이어진다.

2. 안전한 키 관리 (환경변수 주입) 적용

이를 방지하기 위해 운영체제(OS)의 환경변수(Environment Variables) 기능을 활용하여, 서버가 실행되는 시점에만 메모리로 키 값을 주입받도록 아키텍처를 개선했다.

2.1. application.yml 설정

Spring Boot의 설정 파일에 OS 환경변수 ENCRYPTION_SECRET_KEY를 참조하도록 선언했다.

app:
  security:
    aes-secret-key: ${ENCRYPTION_SECRET_KEY}


2.2. 컨버터 리팩토링 (AesGcmConverter)

Spring의 @Value 어노테이션을 사용하여 의존성 주입(DI)을 통해 키 값을 동적으로 할당받도록 수정했다. JPA Converter의 생명주기 특성을 고려하여, 주입받은 값을 static 필드에 매핑하는 방식으로 구현하여 하드코딩된 문자열을 소스코드에서 완벽히 제거했다.

<img width="935" height="247" alt="image" src="https://github.com/user-attachments/assets/1e27eb7a-0f73-45e1-985b-b3ddcb28b278" />

    // ... 이하 동일


3. 회고

이러한 분리 구조를 통해 소스코드가 유출되더라도, 실제 서버(인프라)에 접근하여 환경변수를 탈취하지 않는 이상 데이터의 기밀성은 완벽하게 유지된다. 또한, JUnit 단위 테스트를 통해 암호화 및 복호화가 정상적으로 이루어짐을 눈으로 직접 검증하였다.

<img width="1111" height="358" alt="image" src="https://github.com/user-attachments/assets/cf4cb5b9-a929-4813-8e06-1af3ca250eb7" />

(설명: IntelliJ의 Run/Debug Configurations 창에서 Environment variables에 키 값을 세팅한 화면 또는 JUnit 테스트 성공 화면 캡처)

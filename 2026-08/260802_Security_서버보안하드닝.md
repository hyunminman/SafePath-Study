[보안 스터디] 미들웨어 및 HttpOnly 쿠키를 적용한 서버 하드닝 (Step 4)

학습 일자: 2026년 8월 2일

분류: Security / Backend

주제: 취약한 Node.js 서버에 방어 로직(Helmet, CORS, HttpOnly Cookie) 적용

1. 실습 개요 및 목표

이전 단계(Step 3)에서 발견된 스니핑 및 XSS 취약점을 해결하기 위해 서버 코드를 리팩토링(Hardening)한다. 핵심 보안 미들웨어를 장착하고 토큰의 전달 방식을 근본적으로 변경하여 안전한 인증 서버를 구축한다.

2. 보안 하드닝(Hardening) 적용 내역

2.1. XSS 방어: HttpOnly & SameSite 쿠키 적용

문제점: 기존에는 JWT 토큰을 HTTP Response Body(JSON)로 평문 반환하여 브라우저의 로컬 스토리지 등에 저장해야 했으며, 이는 XSS 공격 시 자바스크립트에 의해 탈취될 위험이 컸다.

해결책: res.cookie()를 사용하여 토큰을 쿠키로 전달하되, httpOnly: true 옵션을 부여했다. 이로 인해 브라우저의 자바스크립트(document.cookie)로는 해당 토큰을 읽을 수 없어 XSS 공격을 원천 차단했다. 또한 sameSite: 'strict' 옵션을 통해 CSRF 공격의 위험도 함께 완화했다.

2.2. HTTP 헤더 보안: Helmet 미들웨어

해결책: helmet 패키지를 적용하여 X-Powered-By(서버 정보 노출) 헤더를 숨기고, X-Frame-Options(클릭재킹 방어) 등 다양한 HTTP 보안 헤더를 자동으로 설정하도록 구성했다.

2.3. 비정상적 접근 차단: CORS 정책 설정

해결책: cors 패키지를 적용하여 백엔드 API에 접근할 수 있는 Origin(출처)을 신뢰할 수 있는 도메인(예: 프론트엔드 서버)으로 명시적 제한을 두었다. 이를 통해 허가되지 않은 외부 사이트에서의 API 직접 호출을 차단했다.

3. 남은 보안 과제 (Production 환경 기준)

현재 로컬(localhost) 실습 환경의 한계로 HTTP를 사용 중이나, 실제 프로덕션 환경에서는 반드시 HTTPS(TLS/SSL) 암호화 통신을 적용하고 쿠키 옵션에 secure: true를 추가해야 한다. 이를 통해 Step 3에서 확인했던 네트워크 패킷 스니핑(MITM)을 통한 탈취까지 완벽하게 방어할 수 있다.

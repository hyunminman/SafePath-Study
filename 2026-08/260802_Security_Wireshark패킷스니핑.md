[보안 스터디] 네트워크 패킷 스니핑을 통한 평문 데이터 노출 실증 (Step 3)

학습 일자: 2026년 8월 2일

분류: Security / Network

주제: Wireshark를 활용한 HTTP 통신 구간 중간자 공격(MITM) 및 데이터 스니핑 실증

1. 실습 개요 및 목표

보안 미들웨어 및 암호화 프로토콜(HTTPS)이 적용되지 않은 웹 서버 환경에서 데이터 전송 시 발생하는 보안 위협을 실증 분석한다. 네트워크 패킷 분석 도구인 Wireshark를 사용하여 로컬 통신 과정(Loopback)을 캡처하고, 로그인 시도 시 계정 정보와 JWT 토큰이 평문으로 유출되는 과정을 증명한다.

2. 실습 환경 구성

Target Server: Node.js (Express) 기반 로그인 API 서버 (Port: 3000, HTTP)

Client: cURL (CLI 기반 HTTP 요청 도구)

Analyzer: Wireshark (Network Protocol Analyzer)

Target Interface: Loopback Traffic Capture (내부망 통신 캡처)

3. 패킷 캡처 및 취약점 분석 결과

Wireshark에서 tcp.port == 3000 필터를 적용한 상태에서 클라이언트가 서버로 로그인 POST 요청을 전송했다. 캡처된 트래픽을 분석한 결과는 다음과 같다.

3.1. 요청(Request) 패킷 분석: 계정 정보 평문 노출

클라이언트가 전송한 POST 요청 패킷을 분석한 결과, HTTP 헤더 하단의 Payload(JSON Body) 영역에서 사용자의 민감한 인증 정보가 암호화 없이 그대로 노출되는 것을 확인했다.

<img width="1019" height="296" alt="image" src="https://github.com/user-attachments/assets/7df8b263-1849-4120-8c10-25654992c961" />


분석: HTTPS 통신 부재로 인해 네트워크 구간에서 패킷을 훔쳐보는 중간자(MITM) 공격에 무방비 상태임을 시사한다. 누군가 동일한 와이파이 대역에서 스니핑을 수행한다면 아이디와 비밀번호가 즉각 탈취된다.

3.2. 응답(Response) 패킷 분석: JWT 토큰 유출

서버가 인증 성공 후 반환하는 200 OK 응답 패킷을 분석한 결과, 서버가 발급한 JWT(JSON Web Token) 문자열이 JSON 객체 내에 평문 상태로 전달됨을 확인했다.

<img width="786" height="408" alt="image" src="https://github.com/user-attachments/assets/88c10a29-92de-4042-95a5-08658851c96e" />

분석: 토큰이 응답 본문(Body)으로 전송되며 네트워크 구간에 노출되었다. 발급된 JWT는 탈취될 경우 유효기간 동안 공격자가 정상 사용자로 위장할 수 있는 마스터키 역할을 하므로 치명적인 보안 결함이다.

4. 결론 및 방어 대책 (Next Step)

HTTP 환경에서는 아무리 서버의 로직이 견고해도 전송 구간(In-Transit)에서의 탈취를 막을 수 없다는 것을 패킷 수준에서 실증했다. 이를 방어하기 위해 다음 단계(Step 4)에서는 아래의 보안 하드닝(Hardening) 조치를 적용할 예정이다.

전송 구간 암호화: TLS/SSL 인증서를 적용하여 HTTPS 프로토콜로 전환. (패킷 스니핑 시 데이터 암호화)

토큰 보호: JWT를 단순 JSON Body가 아닌, 자바스크립트에서 접근 불가능한 HttpOnly 쿠키에 담아 전송하여 XSS 공격을 통한 탈취 원천 차단.

보안 헤더 추가: Helmet 미들웨어를 통해 기본 HTTP 보안 헤더 적용.

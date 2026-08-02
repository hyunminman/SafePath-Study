[보안 스터디] 취약한 JWT 인증 서버 구현 및 테스트 (Step 2)

학습 일자: 2026년 8월 2일

분류: Security / Backend

주제: 보안 미들웨어가 결여된 취약한 Node.js 웹 서버 구축 및 토큰 발급 테스트

1. 실습 개요 및 목표

네트워크 패킷 스니핑(Sniffing) 시 데이터가 어떻게 유출되는지 관측하기 위해, 보안 로직이 전혀 적용되지 않은 뼈대 상태의 로그인 서버를 로컬 환경에 구축한다.

구현 목표 1: HTTPS 암호화가 적용되지 않은 HTTP 평문 통신 환경 구성

구현 목표 2: HttpOnly 쿠키를 사용하지 않고, JSON Body를 통해 평문으로 JWT 토큰을 반환하는 취약한 API 엔드포인트 작성

2. 환경 구성 및 패키지 설치

Node.js 환경에서 웹 서버 프레임워크인 express와 토큰 발급을 위한 jsonwebtoken 라이브러리를 설치하여 프로젝트를 구성했다.

# 프로젝트 초기화
npm init
# 필수 라이브러리 설치
npm install express jsonwebtoken


3. 취약한 서버 코드 구현 (server.js)

server.js 파일을 생성하고 아래와 같이 라우터를 구성했다. 이 코드는 실제 운영 환경에서는 절대 사용해서는 안 되는 취약점들을 포함하고 있다.

const express = require('express');
const jwt = require('jsonwebtoken');

const app = express();
const PORT = 3000;
const SECRET_KEY = 'super_secret_key_for_study'; 

app.use(express.json());

// [취약점 1] HTTPS 미적용 (평문 전송)
// [취약점 2] 토큰을 안전한 쿠키가 아닌 JSON 본문으로 반환
app.post('/api/login', (req, res) => {
    const { username, password } = req.body;

    if (username === 'admin' && password === '1234') {
        const token = jwt.sign({ username, role: 'admin' }, SECRET_KEY, { expiresIn: '1h' });
        
        console.log(`[서버 로그] ${username} 로그인 성공 및 토큰 발급`);
        
        // 평문으로 토큰 반환
        return res.json({ 
            message: '로그인 성공', 
            token: token 
        });
    }

    return res.status(401).json({ message: '인증 실패' });
});

app.listen(PORT, () => {
    console.log(`[서버 실행] http://localhost:${PORT} 포트 열림`);
});


4. 서버 실행 및 API 테스트 결과

터미널에서 node server.js 명령어로 서버를 구동한 뒤, 클라이언트(cURL/Postman)를 통해 로그인 API(POST /api/login)에 인증 정보를 전송하여 정상 작동 여부를 확인했다.

4.1. 서버 구동 로그

아래는 서버가 정상적으로 3000번 포트에서 실행된 화면이다.

<img width="1114" height="438" alt="image" src="https://github.com/user-attachments/assets/7552d052-3143-4e0c-9591-1ad09f15b69a" />


4.2. 토큰 평문 발급 확인

클라이언트에서 admin 계정 정보를 전송했을 때, 서버가 JWT 토큰 문자열을 캡슐화 없이 평문 JSON 형태로 그대로 반환하는 것을 확인했다.

<img width="403" height="154" alt="image" src="https://github.com/user-attachments/assets/177284cd-d070-4cb3-8bae-d91d92531ea5" />


5. 결론 및 Next Step

이로써 외부 공격에 무방비로 노출된 인증 서버 구성을 완료했다. 브라우저나 클라이언트가 이 응답을 받게 되면, XSS 공격 등을 통해 토큰이 쉽게 탈취될 수 있다.

다음 단계(Step 3)에서는 이 취약한 통신 구간을 Wireshark로 캡처하여, 아이디/비밀번호와 발급된 JWT 토큰이 네트워크상에서 어떻게 노출되는지 패킷 수준에서 실증 분석한다.

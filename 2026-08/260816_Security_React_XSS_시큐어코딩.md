React XSS 취약점 및 시큐어 코딩 실습 가이드

본 가이드는 로컬 환경에서 React 프로젝트를 생성하고, XSS 취약점을 고의로 발생시킨 뒤 DOMPurify를 통해 방어하는 실증 실습 절차입니다.

1. 실습 환경 세팅 (React 프로젝트 생성)

터미널을 열고 실습을 진행할 폴더로 이동한 뒤, 가장 빠르고 가벼운 Vite를 이용해 React 기본 프로젝트를 생성합니다.

프로젝트 생성 명령어 입력:

npm create vite@latest react-xss-study -- --template react


생성된 폴더로 이동 후 필수 패키지 및 방어용 라이브러리(DOMPurify) 설치:

cd react-xss-study
npm install
npm install dompurify


2. 취약점 실증 (공격 테스트)

<img width="545" height="423" alt="image" src="https://github.com/user-attachments/assets/f03a14cc-cdbb-4bf2-bc6d-1ec42ddc4696" />




터미널에 npm run dev를 입력하여 로컬 서버를 켜고, 브라우저에서 http://localhost:5173으로 접속합니다.
접속하자마자 화면에 '치명적인_XSS_공격_성공!' 이라는 경고창(Alert)이 강제로 뜨는 것을 확인합니다. 

<img width="544" height="435" alt="image" src="https://github.com/user-attachments/assets/73557707-74e2-4372-affd-410a0e86c6bd" />


3. 시큐어 코딩 적용 (방어 테스트)

이제 취약점을 막기 위해 코드를 수정합니다. src/App.jsx 파일을 아래와 같이 변경합니다.

<img width="555" height="483" alt="image" src="https://github.com/user-attachments/assets/2e232e7a-a4b1-4da7-8c9b-909fb50296fc" />




파일을 저장하고 브라우저를 새로고침 해봅니다.

<img width="548" height="432" alt="image" src="https://github.com/user-attachments/assets/36c6e8d5-d81c-49e7-b43c-bb212127d44b" />

이번에는 경고창(Alert)이 절대 뜨지 않고, 악성 코드가 무력화된 안전한 화면만 렌더링되는 것을 확인할 수 있습니다.

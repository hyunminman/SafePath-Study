OS Command Injection 취약점 분석 및 시큐어 코딩 적용

학습일: 2026.08.06
주제: 외부 입력값 검증 누락으로 인한 RCE 취약점 분석 및 화이트리스트 필터링 적용

서버의 상태를 확인하는 Ping 테스트 로직을 구현하는 과정에서, 사용자의 입력값이 검증 없이 운영체제 명령어로 전달될 때 발생하는 OS Command Injection 취약점을 실습하고 방어 코드를 적용했다.

1. 취약한 코드 분석 (Vulnerable)

클라이언트로부터 전달받은 IP 주소를 Runtime.getRuntime().exec() 메서드에 문자열 결합 방식으로 직접 전달하는 백엔드 로직을 작성했다.

@GetMapping("/ping")
public String ping(@RequestParam String ip) throws Exception {
    Process process = Runtime.getRuntime().exec("ping -c 4 " + ip);
    return "Ping command executed for: " + ip;
}


공격 검증 결과:
파라미터로 8.8.8.8; cat /etc/passwd를 전달하여 API를 호출했다.
명령어 구분자인 세미콜론(;) 뒷부분의 명령어가 기존 ping 명령어와 분리되어 서버의 운영체제에서 독립적으로 실행되는 원격 코드 실행(RCE) 취약점이 발생함을 확인했다.

2. 안전한 코드로의 개선 (Secure Coding)

입력값 검증과 명령어 실행 구조를 변경하여 취약점을 방어했다.

화이트리스트 기반 필터링: 정규표현식을 사용하여 숫자와 점(.)으로만 구성된 정상적인 IP 주소 형태가 아닐 경우 예외를 발생시키도록 처리했다.

ProcessBuilder 사용: Runtime.exec() 대신 ProcessBuilder를 도입하여 실행할 프로그램과 인자(Argument)를 리스트 형태로 분리했다. 이를 통해 특수문자가 섞이더라도 하나의 인자로 묶이게 하여 명령어로 해석되는 것을 원천 차단했다.

@GetMapping("/ping/secure")
public String securePing(@RequestParam String ip) throws Exception {
    if (!ip.matches("^[0-9]{1,3}(\\.[0-9]{1,3}){3}")) {
        return "Error: Invalid IP format.";
    }

    ProcessBuilder pb = new ProcessBuilder("ping", "-c", "4", ip);
    Process process = pb.start();
    
    return "Secure Ping command executed for: " + ip;
}


<img width="853" height="392" alt="image" src="https://github.com/user-attachments/assets/bbdc5635-8c3c-4da0-8947-3dd98ee577f2" />
<img width="1018" height="613" alt="image" src="https://github.com/user-attachments/assets/1d549a7d-83a1-409c-8b14-61aedb5c4277" />

(설명: 실습 사이트와 IntelliJ에서 개선된 코드를 캡처 화면)

3. 회고

시스템 명령어를 호출해야 하는 비즈니스 로직에서는 사용자 입력값에 대한 엄격한 화이트리스트 검증이 필수적이다. 또한 쉘(Shell)을 거치지 않고 인자를 분리하여 실행하는 ProcessBuilder와 같은 안전한 API 사용이 기본이 되어야 함을 확인했다.

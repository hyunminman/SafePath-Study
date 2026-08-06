SSTI 및 자바 역직렬화 취약점 분석과 아키텍처 개선

학습일: 2026.08.06
주제: Thymeleaf 템플릿 엔진 오설정 및 ObjectInputStream 결함 분석

사용자 입력값이 실행 가능한 코드로 해석되거나, 객체로 재조립되는 과정에서 발생하는 SSTI 및 역직렬화 취약점을 분석하고 구조적인 개선을 진행했다.

1. SSTI (Server-Side Template Injection) 방어

웹 템플릿 엔진이 사용자 입력을 단순 텍스트가 아닌 표현식(Expression)으로 파싱할 때 발생하는 취약점을 테스트했다.

취약한 구조:
사용자 입력값을 뷰 템플릿의 변수 처리 기능에 직접 할당할 경우, __[달러기호]{T(java.lang.Runtime).getRuntime().exec("id")}__ 와 같은 페이로드가 서버 내부에서 실행되는 문제가 발생한다.

개선된 구조:
템플릿 파싱 로직과 데이터를 엄격히 분리했다. 데이터는 Spring의 Model 객체에 속성(Attribute)으로 바인딩하여 전달하고, 뷰 영역에서는 정해진 텍스트로만 렌더링되도록 수정하여 취약점을 방어했다.

@GetMapping("/hello/secure")
public String secureHello(@RequestParam String name, Model model) {
    model.addAttribute("username", name);
    return "helloView"; 
}


2. 자바 역직렬화 (Insecure Deserialization) 방어

네트워크를 통해 전달받은 바이트 스트림을 자바 객체로 역직렬화하는 과정의 위험성을 분석했다.

취약한 구조:
자바 기본 기능인 ObjectInputStream.readObject()를 사용할 경우, 조작된 바이트 스트림이 객체화되는 과정에서 악성 가젯(Gadget) 체인이 동작하여 원격 코드 실행(RCE)이 발생할 수 있다.

개선된 구조:
자바 네이티브 역직렬화 사용을 전면 배제하고, Jackson 라이브러리를 활용하여 구조화된 JSON 텍스트 포맷으로 데이터를 교환하도록 아키텍처를 변경했다. JSON 데이터는 단순 DTO로만 매핑되므로 악성 메서드가 호출될 통로가 차단된다.

import com.fasterxml.jackson.databind.ObjectMapper;

public UserDTO secureDeserialize(String jsonData) throws Exception {
    ObjectMapper mapper = new ObjectMapper();
    return mapper.readValue(jsonData, UserDTO.class); 
}


<img width="989" height="395" alt="image" src="https://github.com/user-attachments/assets/cb0a526c-15b6-410a-aad7-41811a80ddc4" />

(설명: Jackson ObjectMapper를 사용하여 안전하게 역직렬화 로직을 개선한 코드 캡처 화면)

3. 회고

프레임워크나 언어가 제공하는 기본 기능(템플릿 엔진, 직렬화)이라도 내부 동작 원리를 이해하지 못하고 사용하면 치명적인 결함이 된다. 데이터와 코드가 혼재되지 않도록 구조를 분리하고, 순수한 데이터 포맷(JSON)을 사용하는 것이 설계 단계에서의 가장 확실한 보안 대책임을 체감했다.

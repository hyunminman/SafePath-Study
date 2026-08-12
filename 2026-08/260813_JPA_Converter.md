JPA AttributeConverter를 활용한 DB 민감정보 자동 암복호화

학습일: 2026.08.13
주제: OWASP A02 (Cryptographic Failures) 방어를 위한 JPA 컨버터 적용

데이터베이스에 민감정보(주민등록번호, 계좌번호 등)를 저장할 때 애플리케이션 레벨에서 암호화를 강제하기 위해, JPA의 AttributeConverter를 도입하여 자동 암복호화 파이프라인을 구축했다.

1. 도입 배경

개발자가 비즈니스 로직(Service Layer)에서 명시적으로 암호화 유틸리티를 호출하도록 설계하면, 휴먼 에러(호출 누락)로 인해 평문 데이터가 DB에 적재되는 대형 보안 사고가 발생할 수 있다.
이를 방지하기 위해 ORM(JPA) 계층에서 DB 컬럼과 엔티티 필드가 매핑되는 순간에 투명하게(Transparently) 암복호화가 수행되도록 아키텍처를 개선했다.

2. AesGcmConverter 및 Entity 적용 로직

AesGcmConverter 클래스를 작성하여 Step 1에서 만든 AesGcmUtil을 호출하도록 했다. 이를 통해 DB에 Insert/Update 될 때는 암호화를, Select 될 때는 복호화를 자동으로 수행하도록 구현했다.
또한, UserMember 엔티티를 생성하고 민감정보를 담을 rrn 컬럼에 @Convert(converter = AesGcmConverter.class) 어노테이션을 부착하여 컨버터가 정상적으로 작동하도록 매핑했다.

3. 회고

JPA의 @Convert 기능을 활용함으로써 비즈니스 로직(Service) 코드는 전혀 수정할 필요 없이, 보안 계층만 독립적으로 분리해 낼 수 있었다. 데이터베이스 침해 사고가 발생해 덤프(Dump) 파일이 유출되더라도, 해당 컬럼은 안전한 AES-256-GCM 암호문으로 덮여있어 기밀성이 철저히 보장된다.

<img width="1186" height="900" alt="image" src="https://github.com/user-attachments/assets/0c7caecb-629f-4b13-8517-a49e55ca8235" />
<img width="709" height="209" alt="image" src="https://github.com/user-attachments/assets/02ca57bd-962e-4e1d-b0e8-7bb723b778dc" />

<img width="816" height="651" alt="image" src="https://github.com/user-attachments/assets/ffbc614f-0edc-429b-863a-83232c2ddb86" />


(설명: IntelliJ에서 작성한 AesGcmConverter.java 와 UserMember.java 코드가 나란히 보이는 화면 캡처)

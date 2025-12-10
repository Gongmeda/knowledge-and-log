# Hexagonal Architecture

## 헥사고날 아키텍처란?

2005년 Alistair Cockburn이 제안한 아키텍처 패턴이다.

'Ports and Adapters 아키텍처' 라고도 불리는데, 이는 애플리케이션의 핵심 로직이 외부 세계와 '포트'라는 인터페이스를 통해 소통하고, 외부 기술은 '어댑터'를 통해 이 포트에 연결되는 구조적 특징을 표현한 것이다.

### 목적

> 애플리케이션이 사용자, 프로그램, 자동화된 테스트, 또는 배치 스크립트에 의해 동등하게 구동될 수 있도록 하고, 애플리케이션이 실행 시 사용될 장치나 데이터베이스로부터 격리된 상태에서 개발 및 테스트될 수 있도록 한다.

- UI나 데이터베이스 없이도 애플리케이션에 대한 자동화된 회귀 테스트를 실행할 수 있다.
- 데이터베이스를 사용할 수 없게 될 때도 작업을 계속할 수 있다.
- 사용자의 개입 없이 애플리케이션들을 서로 연결할 수 있다.

### 해결하는 문제

- **UI 코드로의 비즈니스 로직 침투**: 테스트해야 할 로직이 UI와 강결합되어 자동화된 테스트가 어려워지는 문제
- **외부 서비스에 묶이는 애플리케이션 로직**: 애플리케이션 로직이 외부 DB나 서비스에 묶이게 되면, DB가 다운되거나 크게 변경될 때 지연 비용이 큰 문제

> 외부 기술·UI와 분리된 도메인 중심 구조를 통해 애플리케이션을 쉽게 테스트하고, 컴포넌트 단위로 유연하게 교체·확장할 수 있도록 하는 접근

### 핵심 원칙과 구성 요소

전통적인 계층형 아키텍처(Layered Architecture)가 시스템을 표현(Presentation), 비즈니스(Business), 데이터 접근(Data Access) 등 상하(up-down) 계층으로 구분하는 반면, 헥사고날 아키텍처는 시스템을 다음과 같이 두 영역으로 나눈다.

- 내부 (Inside)
  - 순수한 비즈니스 로직과 도메인 규칙을 담고 있는 애플리케이션의 핵심부
  - 이 영역은 외부 세계의 그 어떤 기술에도 의존하지 않음
- 외부 (Outside)
  - UI, 데이터베이스, 외부 API, 메시지 큐 등 외부 세계와 상호작용하는 모든 기술적인 부분을 의미

이러한 구분은 비즈니스 로직이 데이터베이스 같은 하위 계층의 세부 구현에 의존하게 되는 계층형 아키텍처의 고질적인 문제를 근본적으로 방지한다.

<img width="330" height="330" alt="image" src="https://github.com/user-attachments/assets/213b9ff3-5a6b-45fa-9a55-a6ba6f96fc0a" />

> 애플리케이션은 '포트(Port)'를 통해 외부 에이전시와 통신하며, 각 포트에는 '어댑터(Adapter)'가 연결됨

#### 포트 (Port)

- 애플리케이션이 외부 세계와 상호작용하기 위해 정의하는 인터페이스(API)
- 운영체제나 전자 기기의 포트처럼, 프로토콜을 준수하는 모든 장치가 연결될 수 있도록 하는 개념
- 특정 기술에 종속되지 않은 추상화된 규약
- 애플리케이션과의 유일한 통신 통로 역할

#### 어댑터 (Adapter)

- 포트 인터페이스의 구체적인 구현체
- 외부 세계의 기술과 포트를 연결하는 접착제 역할
- 특정 기술(예: REST API 컨트롤러, DB 리포지토리)에 종속된 코드로 구성

#### 의존성 역전 원칙 (DIP)

- 헥사고날 아키텍처의 가장 중요한 규칙은 의존성의 방향
- 모든 의존성은 외부에서 내부를 향해야 하며, 그 반대는 절대 허용되지 않음
- 이 원칙 덕분에 외부 기술이 변경되어도 애플리케이션은 전혀 영향을 받지 않게 됨

## 헥사고날 아키텍처 vs 레이어드 아키텍처

### 레이어드 아키텍처

<img width="540" height="384" alt="image" src="https://github.com/user-attachments/assets/f2640209-e29f-4625-ac4c-61944afd0ba2" />

'**레이어드 아키텍처**'는 애플리케이션을 3가지 계층으로 분리하는 방식이다.

1. **프레젠테이션 계층 (Presentation Layer - UI)**: HTTP 요청을 처리하고 HTML을 렌더링하는 등 사용자 인터페이스(UI)에 대한 지식을 포함하는 웹 계층과 같이 UI 동작에 초점을 맞춤
2. **도메인 로직 계층 (Domain Logic Layer - Business Logic)**: 유효성 검사(validations) 및 계산(calculations) 등 비즈니스 로직을 포함
3. **데이터 접근 계층 (Data Access Layer)**: DB나 원격 서비스에서 영구 데이터를 관리하는 방법을 처리하는 계층

각각을 간단하게 프레젠테이션 레이어, 도메인 레이어, 데이터 레이어로 부르겠다.

핵심 원칙은 상위 레이어는 바로 아래 레이어에만 의존한다는 것이다.

### 헥사고날 아키텍처

<img width="540" height="466" alt="image" src="https://github.com/user-attachments/assets/0698ab5b-689b-4bf5-94d2-5b812d3646c1" />

[마틴 파울러의 글](https://martinfowler.com/bliki/PresentationDomainDataLayering.html)에서는 레이어드 아키텍처의 일반적인 변형 중 도메인 계층이 데이터 소스에 의존하지 않도록 도메인과 데이터 레이어 사이에 매퍼(mapper)를 도입하는 접근 방식을 '**헥사고날 아키텍처**'라고 한다고 언급된다.

그림에서 Data Mapper는 데이터 레이어이고 Data Access를 의존하는 것으로 보아 헥사고날 아키텍처의 'Adapter' 에 대응되는 것으로 보인다.

그림상으로는 Service가 Data Mapper를 의존하는 구조이지만, Port(인터페이스)가 생략된 것으로 보이며 이게 추가되면 Repository 인터페이스가 될 것이다.

그렇다면 결과적으로 아래처럼 대응이 된다.
- Presentation = Controller
- Service = Service
- Domain Object = Entity
- Port(생략됨) = Repository
- Data Mapper & Data Access = Repository Impl (매핑 + 데이터 액세스)

즉, 일반적으로 Spring MVC + Spring Data JPA 스택으로 구성되는 Controller + Service + Repository 구조로 헥사고날 아키텍처를 구현할 수 있는 것이다.

하지만 우리는 해당 구조를 레이어드 아키텍처라 부른다.

그렇다면 코드 구조상 동일함에도 서로 아키텍처가 다른건 무엇이 원인일까?

나는 레이어드 아키텍처와 헥사고날 아키텍처는 '**Entity가 어느 레이어 모델을 표현하는가?**'에 대한 관점의 차이라고 생각한다.

### Entity가 속하는 레이어에 따른 분류

#### 1. Entity가 데이터 레이어 모델

Entity가 데이터 레이어 모델이라면, Repository는 자연스럽게 데이터 레이어에 속한 인터페이스로 인식될 것이다. (이미 데이터 레이어 모델에 의존하므로 도메인 레이어 내에서의 Repository 추상화는 의미가 약해짐)

이 관점에서는 레이어별 요소 분류는 아래와 같다.

```
[Presentation Layer]
- Controller

[Domain Layer]
- Service
- Domain Model

[Data Layer]
- Repository (인터페이스)
- Data Model (Entity)
- ORM / DB 접근 코드, SQL (Repository 구현체)
```

전통적인 레이어드 아키텍처의 원칙에 따르면 관심사 분리를 위해 도메인 레이어의 모델과 데이터 레이어의 모델은 분리가 되어 있기 때문에 도메인 레이어에는 별개의 모델이 존재한다.

이때, 아키텍처상 도메인 레이어는 데이터 레이어 모델을 의존할 수 있고, 도메인 모델로의 매핑을 강제하지 않기 때문에 자연스럽게 **도메인 모델을 별도로 만들지 않고 데이터 레이어의 모델을 직접 사용**하는 것이다.

따라서 이런 구조는 결과적으로 도메인 레이어와 데이터 레이어 모델이 사실상 하나로 섞여버리는 구조로 이어진다.

이 모델은 데이터 레이어 모델이라는 인식 때문에 모델을 DB 구조와 1:1로 매핑되도록 구현할 확률이 높고, 데이터 중심적인 관점이 되어버린다.

이러한 모델은 빈약한 도메인 모델, 트랜잭션 스크립트 구조의 코드를 유발하게 된다.

#### 2. Entity가 도메인 레이어 모델

Entity가 도메인 레이어 모델이라면, Repository는 자연스럽게 도메인 레이어에 속한 인터페이스로 인식될 것이다. (DIP를 위한 인터페이스 역할)

이 관점에서는 레이어별 요소 분류는 아래와 같다.

```
[Presentation Layer]
- Controller

[Domain Layer]
- Service
- Domain Model (Entity)
- Repository (인터페이스)

[Data Layer]
- Data Model (코드상에는 존재X)
- 모델 매핑, ORM / DB 접근 코드, SQL (Repository 구현체)
```

이 구조에서는 도메인 모델을 데이터 레이어로 전달하는 구조라 의존성 방향은 도메인을 향한다.

데이터 레이어 내에서는 전달된 도메인 모델을 매핑하기 위한 데이터 레이어 모델이 따로 필요하다.

하지만 일반적으로는 Spring Data 기술 덕분에 데이터 레이어 모델을 별도로 구현할 필요는 없다.

Spring Data는 Repository 호출에서 도메인 레이어 모델을 넘기면 알아서 DB 구조에 맞게 매핑하고 적절한 쿼리를 날려주는 역할을 한다.

이때, 개발자는 어노테이션 등을 통해 사실상 Mapper 스펙을 구현해둔 것이고 Spring Data는 이에 맞는 Repository 구현체 즉, Adapter를 생성해주는 것이라 볼 수 있다.

결과적으로 이런 관점을 고수하는 레이어드 아키텍처에서는 약간의 재해석만으로도 헥사고날식으로 정리할 수 있다. (주로 패키징, 네이밍 차이)

## 참고

- https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)
- https://alistair.cockburn.us/hexagonal-architecture
- https://martinfowler.com/bliki/PresentationDomainDataLayering.html
- https://www.inflearn.com/course/%ED%86%A0%EB%B9%84-%ED%81%B4%EB%A6%B0%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8F%84%EB%A9%94%EC%9D%B8%EB%AA%A8%EB%8D%B8%ED%8C%A8%ED%84%B4-%ED%97%A5%EC%82%AC%EA%B3%A0%EB%82%A0-part1
- https://tech.kakaopay.com/post/home-hexagonal-architecture/
- https://medium.com/naverfinancial/%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90-%EC%83%88%EB%A1%9C%EC%9A%B4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-99d70df6122b
- https://medium.com/@rlaeorua369/%ED%97%A5%EC%82%AC%EA%B3%A0%EB%82%A0-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EB%A5%BC-%EC%84%A4%EA%B3%84%ED%95%98%EB%A9%B0-%EB%A7%88%EC%A3%BC%ED%95%9C-%ED%98%84%EC%8B%A4%EC%A0%81%EC%9D%B8-%EA%B3%A0%EB%AF%BC%EB%93%A4-6142ddb53bbb
- https://youtu.be/Y8gX49FGLtw?si=yZMiElHNPAqtIbjL

---
title: "Annotation 사용시 주의할 것들"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - annotation
---

개발자의 불필요한 코드 작성을 줄여주기도 하고, 개발자의 퍼포먼스를 높혀주는 다양한 어노테이션이 있습니다.

이런 장점 때문에 어노테이션의 정확한 의미를 모르고 사용하다 보면, 오히려 예상치 못한 오류를 만들어 내는 경우가 있습니다.

저도 최근에 알게된 몇가지 어노테이션의 주의점에 대해 간략히 적어 보고자 합니다.

<!--more-->

### 어노테이션(Annotation) 이란?

JEE5 (Java Platform, Enterprise Edition 5) 부터 새로 추가된 요소로 기본적으로는 **프로그램에 추가적인 정보를 제공 해주는 메타데이터(Meta Data : 데이터를 위한 데이터)**라고 볼 수 있다.

그외 아래와 같은 용도로 많이 사용되고 있습니다.

> 1. 컴파일러에게 코드 작성 문법 에러를 체크하도록 정보를 제공
> 1. 소프트웨어 개발툴이 빌드나 배치시 코드를 자동으로 생성할 수 있도록 정보를 제공
> 1. 실행시(런타임시) 특정 기능을 실행하도록 정보를 제공

자바에서 제공하는 기본적인 어노테이션에 대해 알아보도록 하겠습니다.

#### 1. @Override
> 선언한 메서드가 오버라이드 되었다는 것을 나타냅니다.  
> 만약 상위(부모) 클래스(또는 인터페이스)에서 해당 메서드를 찾을 수 없다면 컴파일 에러를 발생 시킵니다.

#### 2. @Deprecated
> 해당 메서드가 더 이상 사용되지 않음을 표시합니다.  
> 만약 사용할 경우 컴파일 경고를 발생 키십니다.

#### 3. @SuppressWarnings
> 선언한 곳의 컴파일 경고를 무시하도록 합니다.

#### 4. @Autowired
> 스프링에서 제공하며, 필요한 의존 객체의 "타입"에 해당하는 빈을 찾아 주입한다.  
> 생성자, setter, 필드 에 사용할 수 있음

#### 5. @Data
> 스프링에서 제공하며, 필요한 의존 객체의 "타입"에 해당하는 빈을 찾아 주입한다.  
> 생성자, setter, 필드 에 사용할 수 있음  
> @Data = @NoArgsConstructor + @Getter + @Setter + @ToString + @EqualsAndHashCode

### 어노테이션 사용 시 주의점

#### @Autowired

인텔리제이에서 @Autowired 사용 시 아래와 같은 경고 메시지가 노출된다.
> Spring Team recommends: “Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies”.

#### @Autowired 보다 생성자 주입을 권장하는 이유

- 순환 참조를 방지할 수 있다.

 개발을 하다보면 여러 컴포넌트 간에 의존성이 생긴다. 그중에서 A가 B를 참조하고, B가 다시 A를 참조하는 순한 참조가 발생할 수 있다. 
이런 경우 @Autowired 를 사용할 경우 컴파일시나 어플리케이션 구동 시 오류없이 정상적으로 구동된다. (해당 메소드 호출시 StackOverflowError 발생)
   
하지만, 생성자 주입을 사용할 경우 어플리케이션 구동 시 아래와 같은 오류가 발생하여, 순환 참조로 발생할 수 있는 잠재적 이슈를 사전에 알 수 있다.
```shell
Description:
The dependencies of some of the beans in the application context form a cycle:
┌─────┐
|  madLifeService defined in file [~~~/MadLifeService.class]
↑     ↓
|  madPlayService defined in file [~~~/MadPlayService.class]
└─────┘
```

위와 같은 차이가 발생하는 사유는 생성자 주입과 필드, 수정자 주입과는 빈을 주입하는 순서가 다르기 때문이다.

[자세한 설명](https://madplay.github.io/post/why-constructor-injection-is-better-than-field-injection)

- 테스트에 용이하다

DI의 핵심은 관리되는 클래스가 DI 컨테이너에 의존성이 없어야 한다는 것이다. 즉, **독립적으로 인스턴스화가 가능한 POJO(Plain Old Java Ojbect)** 여야 한다는 것이다. DI 컨테이너를 사용하지 않고서도 단위 테스트에서 인스턴스화할 수 있어야 한다.

- 코드 속에 나쁜 냄새를 없앤다.

생성자 주입을 사용하게 되는 경우 생성자의 인자가 많아짐에 따라 복잡한 코드가 됨을 쉽게 알 수 있고 리팩토링하여 역할을 분리하는 등과 같은 코드의 품질을 높이는 활동의 필요성을 더 쉽게 알 수 있다.

- 불변성 (Immutability)

아쉽게도 필드 주입과 수정자 주입은 해당 필드를 final로 선언할 수 없다. 따라서 초기화 후에 빈 객체가 변경될 수 있지만 생성자 주입의 경우는 다르다. 필드를 final로 선언할 수 있다. 물론 런타임 환경에서 객체를 변경하는 경우가 있을까 싶지만 이로 인해 발생할 수 있는 오류를 사전에 미리 방지할 수 있다.

#### @Data

> @AllArgsConstructor, @RequiredArgsContstructor 사용 금지
> - @AllArgsConstructor : 객체 내부의 인스턴스멤버들을 모두 가지고 있는 생성자를 생성하는 Lombok 어노테이션
> - @RequiredArgsConstructor : 객체 내부의 final, @Notnull 이 붙은 인스턴스멤버들을 가지고 있는 생성자를 생성하는 Lombok 어노테이션

두 어노테이션은 인스턴스 멤버의 순서에 따라 생성자 파라미터 순서가 정해진다. 만일, 리팩토링과 같은 단계에서 같은 타입의 인스턴스 멤버 변수의 순서가 바뀌게 되는 경우 컴파일 오류 발생하지 않지만, 논리적인 오류가 발생한다.

> 무분별한 @EqualsAndHashCode 사용금지
> - @EqualsAndHashCode : equals 메소드와 hashcode 메소드를 생성하는 Lombok 어노테이션

불변객체의 경우 사용해도 무방하지만, 동일함 객체임에도 Set 에 저장한뒤 필드 값을 변경하면 hashCode 가 변경되면서 찾을 수 없게 된다.

> @ToString 사용시 StackOverflowError 가 발생할 수 있다.
> - @ToString : toString 메소드를 생성해주는 Lombok 어노테이션

두 객체가 서로를 멤버변수로 가지고 있는 경우, 서로 참조를 하기 때문에, print 시 StackOverflowError 가 발생할 수 있다.

> @Data 어노테이션에서는 하지말라는 것이 포함되어 있다.
> @Data = @NoArgsConstructor + @Getter + @Setter + @ToString + @EqualsAndHashCode 이다. 하지말라는 것, 조심해야하는 것이 포함되있다.

### 마무리

이와 같이 잘 사용하면 편하지만, 무분별하게 사용하면 독이 되는 어노테이션에 대해서 알아 보았습니다.

개발을 하면서 내가 사용하는 기능에 대해 자세히 알지 못한채 사용하는건 늘 위험이 따르는 것 같네요. 공부할게 참 많은 것 같습니다. 

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://honeyinfo7.tistory.com/56](https://honeyinfo7.tistory.com/56)
[https://madplay.github.io/post/why-constructor-injection-is-better-than-field-injection](https://madplay.github.io/post/why-constructor-injection-is-better-than-field-injection)  
[https://woodcock.tistory.com/8](https://woodcock.tistory.com/8)
[https://lkhlkh23.tistory.com/159](https://lkhlkh23.tistory.com/159)
[https://kwonnam.pe.kr/wiki/java/lombok/pitfall](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)

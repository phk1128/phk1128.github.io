---
title: Spring 컨테이너와 빈 (1)
date: 2024-07-24
categories: [ SpringBoot ]
tags: [ SpringBoot,Spring Bean, Spring Container ]
image: assets/posts/spring_logo.png
---

## Intro

> Spring 컨테이너와 빈에 대해 알아봅시다.

### ✅ Spring 컨테이너

- 스프링 컨테이너는 스프링 어플리케이션 내에서 자바 객체를 관리하는 공간이며 IoC 개념을 구현한 도구 입니다
- 스프링 컨테이너에 관리하는 자바 객체를 스프링에서는 `빈(Bean)`이라고 표현합니다.
- 스프링에서 제공하는 실제 구현체로서, 빈의 생성, 의존성 주입, 생명 주기 관리 등을 담당합니다.
- 어플리케이션 동작 중 빈이 필요한 시점에 개발자 대신 DI를 해줍니다.
- `BeanFactory` 인터페이스와 그의 구현체인 `ApplicationContext`의 구조로 이루어져 있습니다.
- `BeanFactory`은 지연로딩, `ApplicationContext`은 즉시로딩 전략을 사용합니다. 즉시로딩은 애플리케이션 실행 시점에 모든 싱글톤 빈을 로드합니다.
  반연에 지연로딩은 필요한 시점에 로드하기 때문에 런타임 시점에서 지연시간이 발생할 수 있습니다.
- 이 외에 `ApplicationContext`은 `BeanFactory` 보다 더 많은 기능을 제공하므로 Spring 공식문서에서는 `ApplicationContext`사용을 권장하고 있습니다.

### ✅ Spring Bean
스프링에서 **빈(Bean)**은 스프링 컨테이너에 의해 관리되는 자바 객체를 의미합니다.
애플리케이션에서 필요한 객체들을 스프링 컨테이너에 등록하고, 이 객체들이 필요할 때 스프링이 자동으로 의존성을 주입해줍니다.
빈은 스프링의 IoC(Inversion of Control) 원칙에 따라 생성되고 관리됩니다.

- 스프링 컨테이너는 어플리케이션이 시작될ㄷ 때 지정된 빈 설정을 참고하여 빈을 생성하고, 이 빈들 간의 의존성을 자동으로 주입(DI) 합니다.
- `@Component`, `@Service`, `@Repository`, `@Controller` 등의 애너테이션을 사용하거나, 자바 설정 파일에서 `@Bean` 애너테이션을 사용해 직접 빈을 정의할 수 있습니다.

### ✅ Spring Bean Scope
스프링 빈 스코프는 `싱글톤` 과 `프로토타입`으로 나뉘며 이에 따라 빈의 생명 주기도 달라지게 됩니다. 디폴트는 싱글톤으로 설정 되어 있습니다.

#### 싱글톤
디폴트 스코프, 애플리케이션 전체에서 하나의 슨트런스만 생성됩니다. 컨테이너가 처음 빈을 생성하면 그 이후로는 동일한 객체를 계속해서 재사용합니다.

- 메모리 효율적이고, 객체를 공유하는 방식입니다.
- 애플리케이션 전반에서 동일한 상태를 유지하는 빈이 필요할 때 유용합니다.
- 일반적인 서비스 레이어나 데이터 접근 객체(DAO)는 싱글톤으로 사용하는 것이 좋습니다.
- 예시
  ```java
  // 한번만 생성 됨
  @Service
  public class MyService {
  }
  ```

#### 프로토타입
프로토타입 스코프는 빈을 요청할 때마다 새로운 인스턴스를 생성하는 방식입니다. 매번 다른 객체를 반환합니다.

- 매번 새로운 객체가 필요할 때 유용하지만, 메모리 사용량이 증가할 수 있습니다.
- 빈의 상태가 호출될 때마다 달라지길 원할 때 사용합니다.
- 요청마다 새로운 상태를 유지해야 하는 객체나, 상태를 공유하면 안되는 경우
- 예시
  ```java
  // 요청할 때마다 새로운 객체가 생성됨
  @Component
  @Scope("prototype")
  public class MyPrototypeBean {
    
  }
  ```

### 📋 빈 스코프 사용 예제

#### 싱글톤, 프로토타입 각각 사용
```java
class BeanTest {

  @Test
  @DisplayName("빈 스코프 테스트")
  void 빈스코프테스트() throws Exception {
    //given
    System.out.println("======= 싱글톤빈 인스턴스는 context가 열리는 순간 생성됨 =======");
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SingletonCounter.class, PrototypeCounter.class);
    System.out.println("======================================================");

    System.out.println("======= 프로토타입빈 인스턴스는 빈을 호출하는 시점에 생성됨 =======");
    final SingletonCounter singletonCounter1 = context.getBean(SingletonCounter.class);
    final SingletonCounter singletonCounter2 = context.getBean(SingletonCounter.class);
    final PrototypeCounter prototypeCounter1 = context.getBean(PrototypeCounter.class);
    final PrototypeCounter prototypeCounter2 = context.getBean(PrototypeCounter.class);
    System.out.println("======================================================");
    //when

    singletonCounter1.addCount();
    singletonCounter2.addCount();
    prototypeCounter1.addCount();
    prototypeCounter2.addCount();

    final int singletonCount = singletonCounter2.getCount();
    final int prototypeCount = prototypeCounter2.getCount();

    System.out.println("====== 싱글톤빈은 사용이 끝나는 순간 소멸됨 ============");
    context.close();
    System.out.println("===============================================");

    System.out.println("====== 프로토타입빈은 직접 소멸해줘야함 ======");
    prototypeCounter1.destroy();
    prototypeCounter2.destroy();
    System.out.println("======================================");


    //then
    assertAll(
      () -> assertThat(singletonCounter1).isEqualTo(singletonCounter2),
      () -> assertThat(prototypeCounter1).isNotEqualTo(prototypeCounter2),
      () -> assertThat(singletonCount).isEqualTo(2),
      () -> assertThat(prototypeCount).isEqualTo(1)
    );


  }

  // 기본이 싱글톤
  static class SingletonCounter {

    private int count = 0;

    public void addCount() {
      count++;
    }

    public int getCount() {
      return count;
    }

    @PostConstruct
    public void init() {
      System.out.println("Singleton.init " + this);
    }
    @PreDestroy
    public void destroy() {
      System.out.println("Singleton.destroy");
    }
  }

  @Scope("prototype")
  static class PrototypeCounter {
    private int count = 0;

    public void addCount() {
      count++;
    }

    public int getCount() {
      return count;
    }

    @PostConstruct
    public void init() {
      System.out.println("Prototype.init " + this);
    }
    @PreDestroy
    public void destroy() {
      System.out.println("Prototype.destroy");
    }
  }
}
```
```text
======= 싱글톤빈 인스턴스는 context가 열리는 순간 생성됨 =======
Singleton.init com.example.springstudy_2024_06_02.BeanTest$SingletonCounter@db56ff78
======================================================
======= 프로토타입빈 인스턴스는 빈을 호출하는 시점에 생성됨 =======
Prototype.init com.example.springstudy_2024_06_02.BeanTest$PrototypeCounter@954f6a5b
Prototype.init com.example.springstudy_2024_06_02.BeanTest$PrototypeCounter@6459a607
======================================================
====== 싱글톤빈은 사용이 끝나는 순간 소멸됨 ============
Singleton.destroy
===============================================
====== 프로토타입빈은 직접 소멸해줘야함 ======
Prototype.destroy
Prototype.destroy
======================================
```
![img.png](/assets/posts/img_16.png)

- 싱글톤빈은 스프링 컨텍스트가 로드되는 시점에 생성됩니다.
- 프로토타입빈은 해당 빈이 호출되는 시점에 생성됩니다.
- 싱글톤빈은 1개의 인스턴스만 생성됩니다. 따라서 모든 스레드에서 공유를 하게 되므로 필드에 상태를 갖는 변수를 두는것을 주의해야합니다.
- 프로토타입빈은 호출 될 때마다 빈을 생성합니다.
- 싱글톤빈은 사용이 종료되는 시점(컨텍스트 종료)에 소멸이 됩니다. (컨텍스트 종료 != 애플리케이션 종료)
- 프로토타입빈은 사용이 종료되는 시점에 소멸되지 않으므로 개발자가 직접 소멸시켜줘야 합니다.
- 싱글톤빈은 1개의 인스턴스이므로 `singletonCounter1` 과 `singletonCounter2`는 동일한 인스턴스 입니다.
  따라서 동일한 count값을 참조하므로 각 인스턴스가 addCount()를 호출하면 count는 2번 증가하게 됩니다.
- 프로토타입빈은 서로 다른 인스턴스이므로 `prototypeCounter1`과 `prototypeCounter2`는 각각 다른 count를 참조하게 됩니다.
  따라서 각 인스턴스가 addCount()를 호출하면 서로 다른 count가 1번씩 증가하게 됩니다.

### Reference
> [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)

### Continue
> 싱글톤빈과 프로토타입빈을 함께 사용하는 예제
>
> [Spring 컨테이너와 빈 (2)](/posts/Spring_Bean(2))




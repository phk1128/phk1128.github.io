---
title: Spring 컨테이너와 빈 (2)
date: 2024-07-29
categories: [ SpringBoot ]
tags: [ SpringBoot,Spring Bean, Spring Container ]
image: assets/posts/spring_logo.png
---

## 🎬Intro
> 싱글톤빈과 프로토타입빈을 함께 사용하는 예제를 알아봅시다.

### 📋예제

#### 싱글톤빈과 프로토타입빈을 함께 사용

```java
class BeanTest2 {

  @Test
  @DisplayName("빈 스코프 테스트2")
  void 빈스코프테스트2() throws Exception {
    //given
    final AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(
      BeanTest2.Calculator.class, BeanTest2.Counter.class);

    //when

    final Calculator calculator1 = context.getBean(Calculator.class);
    final Calculator calculator2 = context.getBean(Calculator.class);

    calculator1.addCounter();
    calculator2.addCounter();

    final Counter counter1 = calculator1.getCounter();
    final Counter counter2 = calculator2.getCounter();

    final int count1 = counter1.getCount();
    final int count2 = counter2.getCount();
    //then
    assertAll(
      () -> assertThat(counter1).isEqualTo(counter2),
      () -> assertThat(count1).isNotEqualTo(1),
      () -> assertThat(count2).isNotEqualTo(1),
      () -> assertThat(count1).isEqualTo(2),
      () -> assertThat(count2).isEqualTo(2)
    );

  }


  @Component
  static class Calculator {

    private final Counter counter;

    @Autowired
    Calculator(final Counter counter) {
      this.counter = counter;
    }

    public void addCounter() {
      counter.addCount();
    }

    public Counter getCounter() {
      return counter;
    }
  }

  @Component
  @Scope("prototype")
  static class Counter {

    private int count = 0;

    public int getCount() {
      return count;
    }

    public void addCount() {
      count++;
    }

    @PostConstruct
    public void init() {
      System.out.println("PrototypeBean.init " + this);
    }

    @PreDestroy
    public void destroy() {
      System.out.println("PrototypeBean.destroy");
    }

  }
}
```
![img.png](/assets/posts/img_17.png)

- 싱글톤빈 `Calculator`에 프로토타입빈 `Counter`를 주입받아서 사용합니다.
- 싱글톤빈이 호출될때마다 프로토타입빈이 새로 주입받기를 기대하지만, 싱글톤빈이 생성되는 시점에 프로토타입빈의 참조를 저장하기 때문에
  `calculator1`과 `calculator2` 는 같은 `Counter` 객체를 참조하게 됩니다.
- 따라서 `addCounter()` 를 통해 동일한 count가 2번 증가하게 됩니다.
- 싱글톤빈이 생성될때마다 프로토타입빈을 새로 주입받을려면 리팩토링이 필요합니다.

#### Calculator 리팩토링

```java
@Component
static class Calculator {

  private final ApplicationContext applicationContext;
  private final Counter counter;

  @Autowired
  Calculator(final ApplicationContext applicationContext) {
    this.applicationContext = applicationContext;
    counter = applicationContext.getBean(Counter.class); // 생성시점에 컨텍스트에서 빈을 직접 찾는다
  }

  public void addCounter() {
    counter.addCount();
  }

  public Counter getCounter() {
    return counter;
  }
}
```
- 생성시점에 스프링 컨테이너(컨텍스트)에서 빈을 직접 찾아서 등록합니다. 이처럼 외부에서 의존성 주입을 하지 않고 내부에서 등록하는 방식을 DL(Dependency Lookup)
  이라고 합니다.
- DL 방식은 스프링 컨테이너에 종속적이기 때문에 Mocking이 어려워 테스트하기 쉽지 않습니다. 따라서 의존성을 주입받을 수 있는 형태로 리팩토링이 필요합니다.

```java
@Component
static class Calculator {

  private final ObjectProvider<Counter> counterProvider;
  private final Counter counter;

  @Autowired
  Calculator(final ObjectProvider<Counter> counterProvider) {
    this.counterProvider = counterProvider;
    counter = counterProvider.getObject();
  }

  public void addCounter() {
    counter.addCount();
  }

  public Counter getCounter() {
    return counter;
  }
}
```
| 특징      | ObjectProvider                       | ApplicationContext               |
|---------|--------------------------------------|----------------------------------|
| 역할      | 특정 타입의 빈을 조회하고, 필요한 경우 빈을 생성하는 기능 제공 | 스프링 컨테이너 자체를 나타내며, 모든 빈 관리 기능 제공 |
| 기능 범위   | 빈 조회 및 생성에 특화                        | 빈 조회, 생성, 관리, 이벤트 처리 등 다양한 기능    |
| 의존성     | ObjectProvider 자체만으로 사용 가능           | ApplicationContext 에 대한 의존성 필요   |
| 테스트 용이성 | Mocking 및 Stubbing 용이                | Mocking 및 Stubbing 어려움           |

- ObjectProvider는 스프링 컨테이너에 대신 조회해주는 기능이 있습니다.
- ObjectProvider는 인터페이스이므로, Mockito와 같은 Mocking 프레임워크를 사용하여 쉽게 Mocking 할 수 있습니다.
  이를 통해 테스트 대상 클래스와 의존성을 제어하고 검증할 수 있습니다.

```java
class BeanTest3 {

  @Test
  @DisplayName("빈 스코프 테스트3")
  void 빈스코프테스트3() throws Exception {
    // given
    final AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(
      BeanTest3.Calculator.class, BeanTest3.Counter.class);

    // when

    final Calculator calculator = context.getBean(Calculator.class);

    calculator.setCounter();
    final Counter counter1 = calculator.getCounter();
    counter1.addCount();
    final int count1 = counter1.getCount();

    calculator.setCounter();
    final Counter counter2 = calculator.getCounter();
    counter2.addCount();
    final int count2 = counter2.getCount();

    counter1.destroy();
    counter2.destroy();

    // then
    assertAll(
      () -> assertThat(counter1).isNotEqualTo(counter2),
      () -> assertThat(count1).isEqualTo(1),
      () -> assertThat(count2).isEqualTo(1)
    );

  }

  @Component
  static class Calculator {

    private final ObjectProvider<Counter> counterProvider;
    private Counter counter;

    @Autowired
    Calculator(final ObjectProvider<Counter> counterProvider) {
      this.counterProvider = counterProvider;
    }

    public void setCounter() {
      counter = counterProvider.getObject();
    }

    public Counter getCounter() {
      return counter;
    }

  }

  @Component
  @Scope("prototype")
  static class Counter {

    private int count = 0;


    public int getCount() {
      return count;
    }

    public void addCount() {
      count++;
    }

    @PostConstruct
    public void init() {
      System.out.println("PrototypeBean.init " + this);
    }

    @PreDestroy
    public void destroy() {
      System.out.println("PrototypeBean.destroy");
    }

  }
}
```
![img.png](/assets/posts/img_18.png)

- `Calculator`의 `setCounter()`를 통해 `Counter`를 새로 주입 받습니다. 즉 싱글톤빈에 새로운 프로토타입빈을 주입 받습니다.
- 따라서 `counter1` 과 `counter2`는 서로 다른 객체가 됩니다. `addCount()` 역시 서로 다른 객체에 적용 됩니다.
- 결과적으로 `count1` 과 `count2` 는 한번씩 증가하여 각각 1이 됩니다.

### Reference
> [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)

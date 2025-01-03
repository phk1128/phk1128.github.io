---
title: Spring DI 및 IoC (with 추상화, 다형성)
date: 2024-07-19
categories: [ SpringBoot ]
tags: [ SpringBoot,Spring DI,Spring IoC,객체 지향,추상화,다형성 ]
image: assets/posts/spring_logo.png
---

## 🎬Intro

> Spring DI, IoC와 추상화 및 다형성에 대해 알아봅시다.

### ✅ Spring DI

DI는 Bean으로 등록된 객체를 `@Autowired` 애노테이션을 이용해 해당 객체가 필요한 곳에 스프링이 자동으로 주입해주는 개념입니다.<br>
쉽게 말해, 필요한 객체를 직접 생성하지 않고, 스프링이 대신 만들어 주입해주는 방식입니다.

정확히 말하자면 프로젝트 로드시 SpringBoot가 컴포넌트 스캔을 통해 `@Component`애노테이션이 붙은 객체를 찾아 Bean으로 자동 등록합니다.

### ✅ Spring IoC

IoC는 프로그램의 흐름을 제어하는 권한이 개발자가 아닌 스프링 프레임워크에 있다는 개념입니다.<br>
전통적으로 객체의 생성과 의존성 관리를 개발자가 직접 했지만, IoC에서는 이 역할을 스프링이 대신합니다.

### ✅ 추상화

추상화는 특정 기능이나 개념을 일반화하여 핵심적인 부분만 노출하고, 세부 사항은 숨기는 것을 말합니다.<br>
주로 인터페이스나 추상 클래스를 통해 이루어지며, 구현체가 어떤 방식으로 동작하는지는 추상화된 인터페이스를 통해서만 노출됩니다.

### ✅ 다형성

다형성은 동일한 인터페이스나 메서드가 여러 가지 형태로 동작할 수 있도록 하는 기능입니다.<br>
이는 주로 상속과 인터페이스를 통해 구현됩니다. 예를 들어, 부모 클래스의 메서드를 자식 클래스에서 오버라이딩하여 각각 다른 방식으로 동작하게 하거나,<br>
같은 인터페이스를 구현한 여러 클래스가 같은 메서드를 다른 방식으로 구현할 수 있습니다.

### 📝 예제

그럼 이제 Spirng에서 추상화와 다형성을 어떻게 활용하는지 예시를 살펴보겠습니다.

예제에 사용될 SpringBoot의 애노테이션 부터 설명하겠습니다.

```java

@SpringBootApplication
public class SpringBootApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringBootApplication.class, args);
  }

}
```

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
  @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class)})
```

- `@SpringBootApplication`을 타고 들어가보면 `@ComponentScan`을 감싸고 있는것을 확인할 수 있습니다.
- 따라서 프로젝트를 run하면 컴포넌트 스캔이 `@Component`애노테이션을 붙은 클래스들을 빈으로 자동 등록합니다.

```java

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {

  /**
   * Alias for {@link Component#value}.
   */
  @AliasFor(annotation = Component.class)
  String value() default "";

}
```

```java

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

  /**
   * Alias for {@link Component#value}.
   */
  @AliasFor(annotation = Component.class)
  String value() default "";

}

```

- `@Repository`,  `@Service`를 타고 들어가면 `@Component`를 감싸고 있습니다<br>
- 따라서 해당 애노테이션이 붙은 클래스는 컴포넌트 스캔에 의해서 빈으로 자동 등록되게 됩니다.

다음으로 예제 입니다

#### UserRepository

```java

@Repository
public interface UserRepository {
  User findUserById(Long id);
}

public class JdbcUserRepository implements UserRepository {
  @Override
  public User findUserById(Long id) {
    // JDBC 코드로 데이터베이스에서 유저 정보를 가져옴
    return new User(id, "John Doe");
  }
}

public class JpaUserRepository implements UserRepository {
  @Override
  public User findUserById(Long id) {
    // JPA 코드로 데이터베이스에서 유저 정보를 가져옴
    return new User(id, "John Doe");
  }
}
```

- `JdbcUserRepository`와 `JpaUserRepository` 모두 `UserRepository`를 구현하고 있는 구현체 입니다.
- 각각 다른 방식으로 데이터를 가져오지만, `UserRepository` 인터페이스를 통해 이러한 차이를 추상화할 수 있습니다.
- 이를 통해 비즈니스 로직은 데이터 접근 방식을 특정 구현체에 의존하지 않고, `UserRepository` 인터페이스에 의존함으로써<br>
  유연성을 확보할 수 있습니다.(다형성)
- 즉, 외부에서는 `JdbcUserRepository`와 `JpaUserRepository`가 어떻게 구현 되어 있는지 알 수도 없고, 알 필요도 없습니다.(추상화)
- UserRepository는 `@Repository` 애노테이션이 붙어있어 프로젝트 run시 Bean으로 등록됩니다.

#### UserService

```java

@Service
public class UserService {

  private final UserRepository userRepository;

  // UserService는 UserRepository 인터페이스에 의존하고, 특정 구현체에 의존하지 않음
  @Autowired
  public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
  }

  public User getUserById(Long id) {
    return userRepository.findUserById(id);
  }
}
```

- `UserService`는 특정 구현체에 의존하지 않고, `UserRepository`에 의존하게 됩니다.(추상화, 다형성)
- 이를 통해서 DB 접근 방식이 `JdbcUserRepository`에서 `JpaUserRepository`로 변경되더라도<br>
  서비스에서는 수정이 일어나지 않고, 해당 서비스를 생성하는 Config에서 일어나게 됩니다. (다형성)
- `@Service`애노테이션이 붙어있어 프로젝트 run시 Bean으로 등록됩니다.
- `@Autowired`을 통해 필요한 의존성을 SpringBoot로 부터 자동으로 주입 받습니다. (DI, IoC)

SpringBoot가 자동으로 해주는 부분을 코드로 표현하자면 아래와 같습니다.

#### AppConfig

```java

@Configration
public class AppConfig {

  @Bean
  public UserRepository userRepository() {
    // 환경이나 필요에 따라 JdbcUserRepository 또는 JpaUserRepository로 수정
    return new JdbcUserRepository(); // 현재는 JDBC 사용
  }

  @Bean
  public UserService userService() {
    return new UserService(userRepository());
  }
}
```

- `userRepository()`는 `JdbcUserRepository`를 생성하여 반환합니다.<br>
  `UserService`는 해당 메서드를 통해 생성시 `JdbcUserRepository`를 주입(DI)하게 됩니다.(다형성)
- Bean의 기본 이름은 UserRepository일 경우 userRepository 이런식으로 정해지게 됩니다.<br>
  이름을 개발자가 설정할 수도 있는데 자세한 부분은 다른 포스트에서 다루도록 하겠습니다.
- 이제 이를 기반으로 SpringBoot가 프로젝트 run시, Bean으로 등록된 클래스간의 의존성을 분석하여 자동으로 필요한 객체를 주입해주게 되는 것 입니다.(DI, IoC)

## ✨ Summary

- Spring DI는 스프링이 필요한 객체를 자동으로 주입해주는 방식으로, 코드의 유연성과 재사용성을 높입니다.
- Spring IoC는 객체의 생성과 관리를 스프링이 담당함으로써, 개발자는 비즈니스 로직에만 집중할 수 있게 합니다.
- 추상화는 인터페이스를 사용해 구현의 세부 사항을 숨기고, 핵심 기능만 노출하는 방식입니다.
- 다형성은 동일한 인터페이스로 다양한 구현체를 유연하게 교체할 수 있게 합니다.


---
title: Spring Session JDBC 도입기
date: 2025-10-27
categories: [ ETC, Refactoring ]
tags: [ Refactoring, Session, Spring, JDBC ]
image: assets/posts/img_21.png
---

## 🎬 Intro
> 분산 환경에서 Session 관리를 위한 Spring Session JDBC를 도입하는 과정을 소개합니다.

## 결론
Spring Session JDBC를 사용하여 `분산환경에서 Session 불일치를 해결`하고, `EC2 추가 인프라 비용 월 약 20% 절감` 하였습니다.

## 현재 상황
- 단일 서버 환경에서 분산 환경으로 마이그레이션
- Session이 서버 메모리에서 관리

## 현재 상황이 왜 문제가 되는가?
서버 메모리에서 Session을 관리하고 있어 분산환경에서는 Session 불일치가 발생하게 됩니다. 때문에 로그인한 사용자가 세션이 만료되기전에 간헐적으로 로그아웃이
되는 상황이었습니다.

## 개선 과정

### ✅ Session 관리 방법
분산환경의 Session 불일치를 해결 하기 위해선 총 3가지의 선택지가 있었습니다.
#### 1. Sticky Session
로드 밸런서가 쿠키나 IP 기반으로 사용자를 식별하고, 해당 사용자의 요청을 항상 동일한 서버로 라우팅

**장점**: 구현이 매우 간단하고 기존 코드 변경이 거의 없습니다. 세션 일관성이 완벽하게 보장되죠.

**단점**: 특정 서버에 부하가 집중될 수 있고, 그 서버에 장애가 발생하면 해당 사용자들의 세션이 모두 사라짐. 또한 진정한 의미의 부하 분산이 이루어지지 않을 수 있습
#### 2. Session Replication
각 서버가 클러스터를 구성하여 세션 정보를 실시간으로 동기화
**장점**: 완전한 부하 분산이 가능하고 서버 장애에 강함

**단점**: 서버 간 네트워크 트래픽이 증가하고, 서버 수가 늘어날수록 복제 오버헤드가 기하급수적으로 커짐

#### 3. External Session Store
세션 정보를 서버의 메모리가 아닌 별도의 외부 저장소에 저장

**장점**: 확장성이 뛰어나고 서버를 자유롭게 추가하거나 제거할 수 있음. 또한 세션 데이터를 영속적으로 저장할 수도 있어서 서버 재시작 시에도 세션이 유지

**단점**: 외부 저장소로의 네트워크 지연이 발생하고, 추가 인프라 비용이 필요

결과적으로 저희는 `3번 방법`을 채택하였습니다. 그 이유는 `AutoScaling 기반의 분산 환경`으로 되어 있어 개별 `서버가 세션을 직접 관리하기 어려운 구조`이기 때문입니다.
또한 스케일 아웃이나 인스턴스 교체가 수시로 일어나는 상황에서 서버 메모리에 세션을 두게 되면, 사용자는 세션이 사라져 재로그인을 반복해야 하고 장애나 재배포 시 일관성을 보장하기도 어렵습니다.

### ✅ 외부 저장소 선택 MySQL vs Redis
외부 저장소로는 Redis(추가필요) vs MySQL(기존사용) 2가지 선택지가 있었습니다. 일반적으로 외부 세션 저장소로 Redis를 많이 사용하지만, 저희 서비스에 합당한지 한번 조사해봤습니다.

#### Redis 비용 체크
![img.png](/assets/posts/img_116.png)

저희 서비스에서는 Redis를 사용한적이 없기 때문에, 우테코의 다른 팀 중 Redis를 사용하는 팀의 인스턴스 비용을 검색해봤습니다.
특정 인스턴스로 비용을 검색할 경우 14일치의 비용만 계산이 됐었고, 비용은 약 `6$` 였습니다.

그럼 월 사용료는 약 `12$` 이고, 저희에게 제한된 월 사용료 `70$`의 약 `20%`가 Redis 사용 비용으로 나가게 됩니다.

#### Session이 필요한 API 호출 횟수 체크
![img.png](/assets/posts/img_118.png)

Session이 필요한 admin 사용자의 api 호출 횟수를 보면, 전체의 약 `25%` 정도 였습니다. 

위 상황을 종합했을때 Redis를 사용하는것은 비효율적이라고 판단을 했었고 기존 MySQL을 사용하는것으로 팀원들과 함께 결정했습니다.

### ✅ Spring Session 학습
![img.png](/assets/posts/img_119.png)
Spring에서 이러한 분산환경에서 Session을 어떻게 관리하는지 찾아보던 중, [Spring 공식 문서](https://docs.enterprise.spring.io/spring-session/reference/index.html?utm_source=chatgpt.com)에서 위와 같은 내용을 확인할 수 있었습니다.
즉, Spring Session을 사용하면 세션관리를 DB에서 할 수 있다는 의미입니다. 그 동작 원리를 한번 알아보겠습니다.

![img.png](/assets/posts/img_120.png)
Spring Session은 `SessionRepositoryFilter`를 사용하여 요청을 가로챕니다. 해당 필터는 `HttpServletRequest`, `HttpServletResponse` 의 구현체를 각각
`SessionRepositoryRequestWrapper`, `SessionRepositoryResponseWrapper` 로 생성하는데, 이를 Wrapping 이라고 합니다. 이렇게 래핑을 하는 이유는 ***Tomcat이 관리하는 메모리 세션을 Spring Session이 관리하도록*** 하기 위함입니다. 


```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
    throws ServletException, IOException {
  request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);
  // 1. Request와 Response를 Wrapping
  SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryRequestWrapper(request, response);
  SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryResponseWrapper(wrappedRequest,
      response);

  try {
    // 2. 다음 필터로 전달 (래핑된 request 사용)
    filterChain.doFilter(wrappedRequest, wrappedResponse);
  }
  finally {
    // 3. 요청 끝날 때 세션을 DB에 커밋
    wrappedRequest.commitSession();
  }
}
```
[Spring 공식 github 코드](https://github.com/spring-projects/spring-session/blob/93f713efa918ac7c636c9634014c12037a4c3e3b/spring-session-core/src/main/java/org/springframework/session/web/http/SessionRepositoryFilter.java#L132-L147)를 보면 위와 같이
`HttpServletRequest` 및 `HttpServletResponse`가 Wrapping 되는 것을 확인할 수 있습니다.

```java
class SessionRepositoryRequestWrapper extends HttpServletRequest {
    @Override
    public HttpSession getSession() {
        // Tomcat 세션 대신 Repository 세션 반환
        return repositorySession;
    }
}
```
또한 해당 구현체는 위와 같이 Session을 DB를 이용해서 관리하도록 메소드를 오버라이드 합니다. 따라서 결과적으로 Wrapping된 `HttpServletRequest`, `HttpServletResponse`는 `DispatcherServlet`을 통해 컨트롤러로 전달되게 되고,
최종적으로 다형성을 통해 Session을 DB를 통해 조회, 저장할 수 있게 됩니다.

### ✅ Spring Session 적용
Spring Session을 적용하는 방법은 정말 간단합니다. session 테이블 생성과 어노테이션 추가만 하면 됩니다.

#### `@EnableJdbcHttpSession` 어노테이션으로 Spring Session 활성화 
```java
@Configuration
@EnableJdbcHttpSession(
        maxInactiveIntervalInSeconds = 14 * 24 * 60 * 60  // 세션 만료 2주
)
public class SessionConfig {
}
```
`@EnableJdbcHttpSession`를 사용하게 되면 Spring은 `SessionRepositoryFilter` 빈을 생성하고 `Servlet Container`에 등록하게 됩니다.

#### Session 테이블 생성
```sql
CREATE TABLE IF NOT EXISTS SPRING_SESSION (
    PRIMARY_ID CHAR(36) NOT NULL PRIMARY KEY,
    SESSION_ID CHAR(36) NOT NULL,
    CREATION_TIME BIGINT NOT NULL,
    LAST_ACCESS_TIME BIGINT NOT NULL,
    MAX_INACTIVE_INTERVAL INT NOT NULL,
    EXPIRY_TIME BIGINT NOT NULL,
    PRINCIPAL_NAME VARCHAR(100),
    UNIQUE INDEX spring_session_ix1 (SESSION_ID),
    INDEX spring_session_ix2 (EXPIRY_TIME),
    INDEX spring_session_ix3 (PRINCIPAL_NAME)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
```sql
CREATE TABLE IF NOT EXISTS SPRING_SESSION_ATTRIBUTES (
    SESSION_PRIMARY_ID CHAR(36) NOT NULL,
    ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
    ATTRIBUTE_BYTES BLOB NOT NULL,
    PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
    FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
![img.png](/assets/posts/img_121.png)

Session 관리에 필요한 테이블은 2가지 입니다. 각 테이블의 역할은 다음과 같습니다.

- SPRING_SESSION : 세션의 생성시간, 만료시간 등 기본 정보를 저장하는 메인 테이블
- SPRING_SESSION_ATTRIBUTES : 세션에 저장된 실제 데이터를 저장하는 테이블

```sql
INSERT INTO SPRING_SESSION_ATTRIBUTES VALUES (
    'abc-123-def',           -- SESSION_PRIMARY_ID
    'user',                  -- ATTRIBUTE_NAME
    <직렬화된 User 객체>      -- ATTRIBUTE_BYTES
);
```

예를 들어 `session.setAttribute("user", user)`로 유저 세션 정보를 저장한다면, 위와 같은 쿼리가 발생하게 됩니다.


## 성과
### ✅ 추가 인프라 비용 20% 절감
### ✅ 분산환경 Session 불일치 문제 해결

## 마무리하며

팀원 모두 만족하는 결과라 뿌듯했고, 기술을 선택함에 있어 비용과 복잡도를 고려하는것이 얼마나 중요한지를 깨달았습니다. 단순히 좋다 라는 관점에서 Redis를 사용했다면, 관리 비용과 복잡도 모두 올라갔을 것입니다.

또한 이번 기회에 `Spring Session`의 원리에 대해서도 깊게 학습할 수 있었습니다. Wrapping 패턴을 통해 Tomcat의 세션 관리를 투명하게 대체하는 설계가 특히 인상적이었고, 어노테이션 하나로 세션 저장소를 변경할 수 있다는 점에서 추상화와 다형성의 중요성을 한번 더 깨달을 수 있었습니다.

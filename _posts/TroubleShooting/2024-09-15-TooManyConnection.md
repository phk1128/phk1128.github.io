---
title: mysql ERROR 1040 (HY000) Too many connections
date: 2024-09-15
categories: [ TroubleShooting ]
tags: [ mysql, ERROR 1040, Too many connections, RDS ]
image: assets/posts/img_15.png
---

## 🎬 Intro
> 사이드 프로젝트 중 발생한 mysql ERROR 1040 (HY000) Too many connections 에러에 대해 알아보겠습니다.

### ✅ max_connections
MySQL 서버는 동시에 처리할 수 있는 최대 연결 수를 설정으로 관리합니다. 이 설정 값은 max_connections라는 매개변수로 제어되며, 기본적으로 151로 설정되어 있습니다. 즉, MySQL 서버는 기본 설정에서는 최대 151개의 연결만 동시에 허용합니다.

### ✅ mysql ERROR 1040 (HY000): Too many connections
MySQL의 ERROR 1040 (HY000): Too many connections 에러는 커넥션의 수가 max_connections 값을 초과했을때 발생하는 오류 입니다.

1. 애플리케이션 서버에서 다수의 사용자가 접속하여 데이터베이스에 연결할 때
2. 데이터베이스에 대한 대량의 쿼리를 동시에 처리할 때
3. 데이터베이스 연결이 제대로 종료되지 않고 계속 열린 상태로 유지될 때(커넥션 누수)

주로 이 3가지 상황에서 발생하게 됩니다.

### ✅ 문제
현재 사이드프로젝트는 `@CommandLineRunner`와 flyway를 통해 프로젝트에 필요한 초기 데이터를 인서트하고 있습니다.
따라서 빌드 단계에서 다수의 DB 커넥션이 발생하게 됩니다.

```sql
show status like 'Max_used_connections';
```
![img.png](/assets/posts/img_22.png)

- RDS(mysql)의 최대로 사용된 커넥션이 263으로 조회됩니다.
- 하지만 mysql의 경우 max_connections이 151이 디폴트로 설정되어 있습니다.
  ```sql
  # max_connections 조회 명령어
  show variables like 'max_connections';
  ```
- 따라서 커넥션이 151이 초과할경우 `mysql ERROR 1040 (HY000) Too many connections`가 발생하게 됩니다.

### ✅ 해결방법

`max_connections`를 적절하게 설정하면 됩니다. 
```sql
# max_connections 설정 명령어
set global max_connections=500;
```
![img_1.png](/assets/posts/img_23.png)
설정 값 선택 기준은 Prometheus + Grafana와 같은 모니터링 도구를 통해서 Max Used Connections을 체크할 수 있습니다.
또한 스프링 부트의 `spring.datasource.hikari.maximum-pool-size` 보다 큰 값으로 설정해야 합니다.
해당 옵션은 스프링 부트 어플리케이션에서 db에 동시에 커넥션할 수 있는 값을 의미합니다. 하지만 주의 할 점은 max_connections 값을 높이면 그만큼 메모리를 더 많이 할당하게 됩니다.
서버 메모리가 부족해지면 swap이 발생하게 되는데 이때 데이터를 하드 디스크에 저장하게 됩니다. 일반 적으로 하드 디스크는 RAM보다 훨씬 느리기 때문에, 데이터 처리를 하드 디스크에 맡기면 속도가 크게 떨어지고 그 결과 DB의 성능도 느려지게 됩니다.
따라서 max_connections값을 적절하게 조절할 필요가 있습니다.





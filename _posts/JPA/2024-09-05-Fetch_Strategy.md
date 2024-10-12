---
title: JPA 연관관계 Fetch 전략
date: 2024-10-12
categories: [ JPA ]
tags: [ JPA, ORM, Fetch, LAZY, EAGER, 연관관계 ]
image: assets/posts/jpa_logo.jpeg
---

## 🎬 Intro
> JPA 연관관계 Fetch 전략에 대해 알아봅시다.


### ✅ 지연 로딩(Lazy Loading)
필요한 시점에 DB에서 데이터를 불러오는 기능입니다. JPA는 엔티티를 우선 프록시 객체로 가져온 뒤 필요한 시점에 DB에 SELECT 쿼리를 날려
가져오게 됩니다.

여기서 주의 할 점은 이로 인해서 원치 않게 프록시 객체만큼 SELECT 쿼리가 날라가게 되는데요. 이를 N + 1 문제라고 합니다.
자세한 내용은 다른 포스트에서 다루도록 하겠습니다.

### ✅ 즉시 로딩(Eager Loading)
즉시 로딩은 그 연관된 엔티티들을 즉시 로딩하여, 추가적인 DB 접근을 최소화합니다. 따라서 연관된 데이터를 한 번에 처리하기에 적합하지만,
필요하지 않는 데이터를 로딩하기 위해 불필요한 Join등이 발생할 수 있어 비효율적일 수 있습니다.


### 📝 예제

#### Lazy 전략
```java
@Entity
@Table
@NoArgsConstructor
@Getter
@Setter
public class Member {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  @Column(name = "member_id")
  private Long id;

  private String name;


  // Join Column을 기준으로 Team의 객체를 찾아오게 된다.
  // 만약 Member 테이블에 team_id가 null 이면 Team 객체도 null이다.
  // @ManyToOne은 Fetch 기본 전략이 EAGER
  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "team_id")
  private Team team;

  @Builder
  public Member(final Long id, final String name, final Team team) {
    this.team = team;
    this.id= id;
    this.name = name;
  }


}



@Entity
@Table
@NoArgsConstructor
@Getter
@Setter
public class Team {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  @Column(name = "team_id")
  private Long id;

  private String name;

  // mappedBy에는 Member에 존재하는 Team의 객체 이름을 넣어줍니다.
  // @OneToMany 는 Fetch 기본전략이 LAZY
  @OneToMany(mappedBy = "team", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
  private List<Member> members = new ArrayList<>();

  @Builder
  public Team(final Long id, final String name) {

    this.id = id;
    this.name = name;
  }

  /*
  연관관계 편의 메서드, Member와 Team 중 한곳에서만 선언한는것이 좋습니다.
   */
  public void addMember(Member member) {
    member.setTeam(this);
    members.add(member);
  }

}

```

- Member, Team 은 양방향 연관관계 입니다.
- Member, Team 모두 Fetch는 Lazy 전략 입니다.

```java

@DataJpaTest
// 실제 DB 사용 옵션, 해당 애노테이션이 없을 시 @DataJpaTest는 스프링부트의 테스트 전용 DB를 사용하게 됨
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class JpaTest7 {

  @Autowired
  MemberRepository memberRepository;

  @Autowired
  TeamRepository teamRepository;


  @Test
  @DisplayName("Lazy 전략 테스트")
  void lazyTest() throws Exception {

    //given
    //when
    final Member member = memberRepository.findById(1L).orElse(null);
    final Team team = member.getTeam();

    //then
    assertThat(team).isInstanceOf(HibernateProxy.class);
    System.out.println("========= 초기화 ==========");
    Hibernate.initialize(team);
    assertThat(team).isInstanceOf(HibernateProxy.class);

  }
}

```
```text
[Hibernate] 
    select
        m1_0.member_id,
        m1_0.name,
        m1_0.team_id 
    from
        member m1_0 
    where
        m1_0.member_id=?
========= 초기화 ==========
[Hibernate] 
    select
        t1_0.team_id,
        t1_0.name 
    from
        team t1_0 
    where
        t1_0.team_id=?

```
- Member의 Fetch 전략이 Lazy이므로 영속성 1차 캐시에 team객체가 프록시 객체 형태로 저장되어 있습니다.
- Hibernate.initialize() 메서드를 통해 team 프록시 객체를 초기화 합니다. 이때 DB에 SELECT 쿼리가 실행됩니다. 
- 초기화 되더라도 team객체는 여전히 프록시 객체로 남아있습니다.그러나 데이터는 채워져 있는 상태입니다.


#### Eager 전략
```java
@Entity
@Table
@NoArgsConstructor
@Getter
@Setter
public class Member {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  @Column(name = "member_id")
  private Long id;

  private String name;


  // Join Column을 기준으로 Team의 객체를 찾아오게 된다.
  // 만약 Member 테이블에 team_id가 null 이면 Team 객체도 null이다.
  // @ManyToOne은 Fetch 기본 전략이 EAGER
  @ManyToOne(fetch = FetchType.EAGER)
  @JoinColumn(name = "team_id")
  private Team team;

  @Builder
  public Member(final Long id, final String name, final Team team) {
    this.team = team;
    this.id= id;
    this.name = name;
  }


}

@Entity
@Table
@NoArgsConstructor
@Getter
@Setter
public class Team {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  @Column(name = "team_id")
  private Long id;

  private String name;

  // mappedBy에는 Member에 존재하는 Team의 객체 이름을 넣어줍니다.
  // @OneToMany 는 Fetch 기본전략이 LAZY
  @OneToMany(mappedBy = "team", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
  private List<Member> members = new ArrayList<>();

  @Builder
  public Team(final Long id, final String name) {

    this.id = id;
    this.name = name;
  }

  /*
  연관관계 편의 메서드, Member와 Team 중 한곳에서만 선언한는것이 좋습니다.
   */
  public void addMember(Member member) {
    member.setTeam(this);
    members.add(member);
  }

}


```

- Member의 Fetch 전략을 EAGER로 변경하였습니다.

```java
@DataJpaTest
// 실제 DB 사용 옵션, 해당 애노테이션이 없을 시 @DataJpaTest는 스프링부트의 테스트 전용 DB를 사용하게 됨
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class JpaTest8 {

  @Autowired
  MemberRepository memberRepository;

  @Autowired
  TeamRepository teamRepository;

  @Test
  @DisplayName("Eager 전략 테스트")
  void eagerTest() throws Exception {

    //given
    //when
    final Member member = memberRepository.findById(1L).orElse(null);
    final Team team = member.getTeam();

    //then
    assertThat(team).isNotInstanceOf(HibernateProxy.class);

  }
}

```
```text
[Hibernate] 
    select
        m1_0.member_id,
        m1_0.name,
        t1_0.team_id,
        t1_0.name 
    from
        member m1_0 
    left join
        team t1_0 
            on t1_0.team_id=m1_0.team_id 
    where
        m1_0.member_id=?

```
- Member의 Team 매핑 전략이 Eager이므로 조회 시 left join으로 team 정보도 함께 가져옵니다.
- 따라서 team은 프록시 객체가 아니라 실제 객체가 됩니다.


### ✅ 지연 로딩 vs 즉시 로딩
결론적으로, 즉시 로딩은 모든 연관된 엔티티를 즉시 로드하여 추가적은 DB 접근을 줄이는 장점이 있지만,
경우에 따라 불필요한 데이터까지 로드하게 되어 성능 저하가 발생할 수 있습니다. 반면에 지연 로딩은 필요한 시점에만 데이터를 로드할 수 있지만 N + 1문제가 존재합니다.
어떤 로딩 방식을 사용할지는 애플리케이션의 요구 사항과 성능 고려 사항에 따라 결정해야 합니다.


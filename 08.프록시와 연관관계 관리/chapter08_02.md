# 8장 프록시와 연관관계 관리 
## 8.2 즉시 로딩과 지연로딩
- 프록시 객체는 주로 연관된 객체를 지연로딩할 때 쓰인다. 
- JPA는 개발자가 연관된 엔티티의 조회시점을 선택할 수 있도록 즉시 로딩과 지연 로딩 두 가지 방법을 제공한다. 

### 8.2.1 즉시로딩 
- 엔티티를 조홰할 때 연관된 엔티티도 함께 조회한다. 
- EAGER LOADING을 사용하려면 @ManyToOne의 fetch속성을 FetchType.EAGER로 지정한다. 
```java
@Entity
public class Member{
    @ManyToOne(fecth = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;     
}

Member member = em.find(Member.class, "member1");
Team team = member.getTeam();
```

- 회원과 팀을 즉시 로딩으로 설정하고 em.find(Member.class, "member1")로 회원을 조회하는 순간 팀도 함께 조회한다. 
- 두 개의 테이블을 조회해야 하므로 쿼리를 2번 실행할 것 같지만 JPA 구현체는 즉시 로딩을 최적화하기 위해서 조인 쿼리를 사용한다. 

#### NULL제약 조건과 JPA조인 전략 
- 현재회원 테이블에 TEAM_ID 외래키는 NULL값을 허용하고 있고 팀에 소속되지 않은 회원이 있을 경우에 내부 조인을 사용하면 팀도 회원도 조회할 수 없다. 
- JPA는 위의 상활을 고려해서 외부 조인을 사용한다. 하지만 외부 조인보다 내부 조인이 성능과 최적화에 더 유리하다. 
- 내부 조인을 사용하기 위해서는 외래키에 NOT NULL 제약 조건을 설정하면 값이 있는 것을 보장한다. 따라서 이 때는 내부 조인만 사용해도 된다. 
- JPA에 위의 사실을 알려주기 위해서 @JoinColumn(nullable = false)을 설정해서 이 외래키는 NULL값을 허용하지 않는다고 알려주면 JPA는 외부 조인 대신에 내부 조인을 사용한다. 


### 8.2.2 지연로딩 
- 연관된 엔티티를 실제 사용할 때 조회한다. 
- 지연로딩을 사용하려면 @ManyToOne의 fetch속성을 FetchType.LAZY로 설정한다. 
```java
@Entity
public class Member{
    @ManyToOne(fecth = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;     
}

Member member = em.find(Member.class, "member1");
Team team = member.getTeam();
```
- 회원과 팀을 지연로딩을 설정했고 em.find(Member.class, "member1")을 호출하면 회원만 조회하고 팀을 조회하지 않는다. 대신에 조회한 회원의 팀 멤버변수에 프록시 객체를 넣는다. 
- 반환된 팀 객체를 프록시 객체이다. 이 프록시 객체는 사용될 때까지 데이터 로딩을 미룬다. 


###  8.2.3 즉시로딩, 지연로딩 정리 
- 연관된 엔티티를 즉시 로딩하는 것이 좋은지 아니면 실제 사용할 때까지 지연해서 로등하는 것이 좋은지는 상황에 따라 다르다. 
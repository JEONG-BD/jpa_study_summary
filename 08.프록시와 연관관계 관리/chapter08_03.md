# 8장 프록시와 연관관계 관리 
## 8.3 지연로딩 활용 
#### 사내 주문 관리 시스템 개발 
- 회원(Member)은 하나의 팀(Team)에만 소속할 수 있다.(N:1) 
- 회원(Member)은 여러 주문내역(Order)을 가진다. (1:N) 
- 주문내역(Order)은 상품정보(Product)를 가진다. (N:1)

```java
@Entity 
public class Member{
    
    @Id
    @GeneratedValue 
    @Column(name = "MEMBER_ID")
    private String id; 

    @ManyToOne(fetch = fetchType.EAGER)
    private Team team; 

    @OneToMany(mappedBy="member", fecth = fetchType.LAZY) 
    private List<Order> orders = new ArrayList<Order>();
}


Member member = em.find(Member.class. "member1");
// 호출해도 컬렉션은 초기화되지 ㅇ낳는다. 컬렉션의 경우에는 member.getOrders().get(0) 처럼 컬렉션에 실제 데이터를 조회할 때 데이터베이스에서 초기화 한다. 
List<Order> orders =member.getOrders();

```

### 8.3.1 프록시와 컬렉션 레퍼 
- 지연로딩을을 설정하면 실제 엔티티 대신에 프록시 객체를 사용한다. 
- 프록시 객체는 자신이 사용될 때 까지 데이터베이스를 조회하지 않는다. 
- 하이버네이트는 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 추적하고 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경하는데 이것을 컬렉션 레퍼라고 한다. 
- 엔티티를 지연로딩하면 프록시 객체를 사용해서 지연 로딩을 수행하지만 주문 내역 같은 컬렉션은 컬렉션 레퍼가 지연 로딩을 처리해준다. 

### 8.3.2 JPA 기본 페치 전략 
- @ManyToOne, @OneToOne : 즉시 로딩 
- @OneToMany, @ManyToMany : 지연 로딩 
- JPA의 기본 페치 전략은 연관된 엔티티가 하나면 즉시 로딩을 컬렉션일 경우 지연 로딩을 사용한다. 
- 모든 연관관계에 지연로딩을 사용하는 것을 권장한다. 개발이 어느 정도 완료 단계에 왔을 때 상황을 보고 꼭 필요한 곳에만 즉시 로딩을 사용한다. 


### 8.3.3 컬렉션에 FetchType.EAGER 사용시 주의점 
- 컬렉션을 하나 이상 즉시 로딩 하는 것은 권장하지 않는다. 
- 컬렉션과 조인한다는 것은 데이터베이스 테이블로 보면 일대다 조인이다. 
- 일대다 조인은 결과가 데이터 다쪽에 있는 수 만큼 증가하게 된다. 
- 컬렉션 즉시 로딩은 항상 외부 조인을 사용한다.

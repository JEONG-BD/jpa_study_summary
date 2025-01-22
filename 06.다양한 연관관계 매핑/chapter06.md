# 6장 다양한 연관관계 매핑 
## 6.1 다대일 
- 다대일 관계의 반대방향은 항상 일대다 관계, 일대다 관계의 반대방향은 항상 다대일 관계이다. 
### 6.1.1 다대일 단방향 
```java
@Entity
public class Member{

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id; 

    private String username; 

    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team; 


}

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id; 

    private String name; 
}

```
- 회원은 Member.team으로 팀 엔티티를 참조할 수 있지만 반대로 팀에서는 회원을 참조할 수 있는 필드가 없다. 
- 회원과 팀은 다대일 단방향 연관관계이다. 
### 6.1.2 다대일 양방향 
```java
@Entity
public class Member{

    @Id
    @GeneratedValue 
    @Column(name="MEMBER_ID")
    private Long id; 

    private String username; 


    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team; 


    public void setTeam(Team team){
        this.team  = team; 

        if (!team.getMembers().contains(this)){
            team.getMembers().add(this);
        }
    }
}

@Entity
public class Team {

    @Id
    @GeneratedValue 
    @Column(name= "TEAM_ID")
    private Long id; 

    private Strint name; 

    @OneToMany(mappedBy="team")
    private List<Member>members = new ArrayList<Member>(); 

    public void addMember(Member member){
        this.members.add(member);
        if(member.getTeam() != this){
            member.setTeam(this);
        }
    }

}

```
- 양방향은 외래키가 있는 쪽이 연관관계의 주인이다. 
- 일대다와 다대일 연관관계는 항상 다(N)에 외래키가 있다. 
- JPA는 외래키를 관리할 때 연관관계의 주인만 사용하고 주인이 아닌 Team.members는 조회를 위한 JPQL이나 객체 그래프 탐색할 때 사용한다. 
- 양방향 연관관계는 항상 서로를 참조해야 한다. 어느 한쪽만 참조하면 양방향 연관관계가 성립하지 않는다. 항상 서로 참조하게 하려면 연관관계 편의 메소드를 작성하는 것이 좋고, 편의 메소드는 한 곳에서만 작성하거나 양쪽 다 작성할 수 있는데 양쪽에 다 작성하면 무한루프에 빠지므로 주의해야 한다. 

## 6.2 일대다 
### 6.2.1 일대다 단방향[1:N]
- 하나의 팀은 여러 회원을 참조할 수 있는데 이런 관계를 일대다 관계라한다. 

```java
@Entity 
public class Team{
    @Id 
    @GeneratedValue 
    @Column(name="TEAM_ID")
    private Long id; 

    private String name; 

    @OneToMany
    @JoinColumn(name="TEAM_ID")
    private List<Member> members = new ArrayList<Member>();

}

@Entity
public class Member{
    @Id 
    @GeneratedValue 
    @Column(name="MEMBER_ID")
    private Long id; 

    private String username; 

}

```
#### @JoinColumn
- 일대다 단방향 관계를 매핑할 때에는 @JoinColumn을 명시해야 한다. 그렇지 않으면 JPA는 연결 테이블을 중간에 두고 연관관계를 관리하는 조인 테이블 전략을 기본으로 사용해서 매핑한다. 
#### 일대다 단방향 매핑의 단점 
- 일대다 단방향 매핑의 단점은 매핑한 객체가 관리하는 외래키가 다른 테이블에 있다는 점이다. 
- 본인 테이블에 외래키가 있으면 엔티티의 저장과 연관관계를 Insert SQL한번으로 끌 낼 수 있지만 다른 테이블에 연관관계가 있으면 연관관계 처리를 위한 Update SQL을 추가로 실행해야 한다. 
```java
public void testSave(){
    Member member1 = new Member("member1");
    Member member2 = new Member("member2"); 

    Team team1 = new Team("team1"); 

    team1.getMembers().add(member1); 
    team1.getMembers().add(member2); 

    em.persist(member1);
    em.persist(member2); 
    em.persist(team1);

    transaction.commit();
}

```
- 일대다 단방향 매핑을 사용하면 엔티티를 매핑한 테이블이 아닌 다른 테이블의 외래키를 관리해야 한다. 성능의 문제도 있지만 관리도 부담스럽기 때문에 일대다 단방향 매핑 대신에 다대일 양방향 매핑을 사용하는 것을 권장한다. 

### 6.2.2 일대다 양방향 [1:N, N:1] 
- 일대다 양방향 매핑은 존재하지 않는다. 대신 다대일 양방향 매핑을 사용해야 한다. 
- 양방향 매핑에서 @OneToMany는 연관관계의 주인이 될 수 없다. 왜냐하면 관계형 데이터베이스의 특성상 일대다, 다대일 관계는 항상 다쪽에 외래키가 있다. 따라서 @OneToMany, @ManyToOne 둘중에 연관관계의 주인은 항상 다 쪽인 @ManyToOne을 사용한 곳이다. 따라서 @ManyToOne에는 mappedBy 속성이 없다. 
```java
@Entity 
public class Team{
    @Id
    @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id; 

    private String name; 

    @OneToMany
    @JoinColumn(name="TEAM_ID")
    private List<Member> members = new ArrayList<Member>();
}

@Entity 
public class Member{
    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id; 

    private String username; 

    @ManyToOne 
    @JoinColumn(name="TEAM_ID", insertable=false, updateable=false)
    private Team team; 

}
``` 
## 6.3 일대일
- 일대일 관계는 양쪽이 서로 하나의 관계만 가진다.
- 일대일 관계는 그 반대로 일대일 관계이다. 
- 테이블 관계에서 일대다, 다대일은 항상 다(N)쪽이 외래키를 가진다. 반명에 일대일 관계는 주 테이블이나 대상 테이블 둘 중 어느 쪽도 외래키를 가질 수 있다. 
### 6.3.1 주 테이블에 외래 키 
- 일대일 관계를 구성할 때 객체지향 개발자들은 주 테이블에 외래키가 있는 것을 선호한다. 
- JPA도 주 테이블에 외래키가 있으면 좀 더 편리하게 매핑할 수 있다. 
#### 단방향 
```java
@Entity 
public class Member {
    @Id
    @GeneratedValue 
    @Column(name="MEMBER_ID")
    private Long id; 

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker; 
}

@Entity
public class Locker{
    @Id
    @GeneratedValue 
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;  

}

```
#### 양방향 
```java
@Entity 
public class Member {
    @Id
    @GeneratedValue 
    @Column(name="MEMBER_ID")
    private Long id; 

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker; 
}

@Entity
public class Locker{
    @Id
    @GeneratedValue 
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;  

    @OneToOne(mappedBy="locker")
    private Member member;

}

```

### 6.3.2 대상 테이블에 외래 키 
#### 단방향
- 일대일 관계 중 대상 테이블에 외래키가 있는 단방향 연관 관계는 JPA에서는 지원하지 않고 이런 모야으로 매핑할 수 있는 방법도 없다. 

```java
@Entity 
public class Member {
    @Id
    @GeneratedValue 
    @Column(name="MEMBER_ID")
    private Long id; 

    private String username;

    @OneToOne(mappedBy="member")
    private Locker locker; 
}

@Entity
public class Locker{
    @Id
    @GeneratedValue 
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;  

    @OneToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;
}
```
- 일대일 매핑에서 대상 테이블에 외래키를 두고 싶으면 이렇게 양방향으로 매핑한다. 
## 6.4 다대다 
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다. 보통 다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다. 
- 객체는 테이블과 다르게 객체 2개로 다대다 관계를 만들 수 있다. 
### 6.4.1 다대다 단방향 
```java

@Entity 
public class Member{

    @Id
    @Column(name="MEMBER_ID")
    private String id; 

    private String username; 

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT", 
    joinColumns = @JoinColumn(name="MEMBER_ID"), 
    inverseJoinColumn = @JoinColumn(name="PRODUCT_ID"))
    private List<Product> products = new ArrayList<Product>(); 

}

@Entity 
public class Product {

    @Id
    @Column(name="PRODUCT_ID")
    private String id; 

    private String name; 
}

```
- 회원 엔티티와 상품엔티티를 @ManyToMany로 매핑했다 
#### @JoinTable.name 
- 연결테이블을 지정한다. MEMBER_PRODUCT 테이블을 선택했다. 
#### @JoinTable.joinColumns
- 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다. MEMBER_ID로 지정했다. 
#### @JoinTable.inverseJoinColumns 
- 반대방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.PRODUCT_ID로 지정했다. 
```java
// 저장 

public void save(){
    Product productA = new Product(); 
    productA.setId("productA"); 
    productA.setName("상품A");
    em.persist(productA);

    Member member1 = new Member();
    member1.setId("member1");
    member1.setUsername("회원1");
    member1.getProducts().add(productA); 
    em.persist(member1); 

}

// 탐색 
public void find(){
    Member member = em.find(Member.class, "member1"); 
    List<Product> products = member.getProducts();

    for(Product product : products){
        System.out.println("product.name = " + product.getName()); 
    }
}
```
### 6.4.2 다대다 양방향 
```java
@Entity 
public class Product {

    @Id
    private String id; 

    @ManyToMany(mappedBy="products")
    private List<Member> members; 

}

```
### 6.4.3 다대다: 매핑의 한계와 극복, 연결 엔티티 사용 
- @ManyToMany를 사용하면 연결 테이블을 자동으로 처리해주므로 도메인 모델이 단순해지고 여러가지로 편리하다. 
- 하지만 이 매핑을 실무에서 사용하기에는 한계가 있다.
```java 
// 회원 엔티티 
@Entity
public class Member {
    @Id 
    @Column(name="MEMBER_ID")
    private String id; 


    @OneToMany(mappedBy="member")
    private List<MemberProduct> memberProducss; 
}

// 상품 엔티티 
@Entity 
public class Product {
    @Id
    @Column(name="PRODUCT_ID")
    private String id; 

    private String name; 
}

//회원 상품 엔티티 
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {
    @Id
    @ManyToOne 
    @JoinColumn(name="MEMBER_ID")
    private Member member; //MemberProductId.member와 연결 

    @Id
    @ManyToOne 
    @JoinColumn(name="PRODUCT_ID")
    private Product product; // MemberProductId.product와 연결 

    private int orderAmount; 

}

public class MemberProductId implements Serializable {

    private String member; 
    private String product; 

    @Override 
    public boolean equals(Object o){...}

    @Override 
    public int hashCode(){...} 

}

```
#### 복합 기본키 
- 회원상품 엔티티는 기본키가 MEMBER_ID와 PRODUCT_ID로 이루어진 복합 키본키다. 
- JPA에서는 복합키를 사용하려면 별도의 식별자 클래스를 만들고, 엔티티에 @IdClass 를 사용해서 식별자 클래스를 지정하면 된다. 
#### 복합 기보키를 위한 식별자 클래스 
- 복합 키는 별도의 식별자 클래스로 만들어야 한다. 
- Serializable을 구현해야 한다. 
- equals와 hashCode메소드를 구현해야 한다. 
- 식별자 클래스는 public이어야 한다. 

#### 식별관계 
- 회원 상품은 회원과 상품의 기본키를 받아서 자신의 기본 키로 사용한다. 
- 이렇게 부모테이블의 기본키를 받아서 자신의 기본 키 + 외래키로 사용하는 것을 데이터베이스 용어로 식별 관계라 한다. 
```java
//회원 저장 
public void save(){


    Member member1 = new Member(); 
    member1.setId("member1");
    member1.setUsername("회원1"); 
    em.persist(member1); 

    Product productA = new Product(); 
    productA.setId("productA"); 
    productA.setName("상품A"); 
    em.persist(productA);

    MemberProduct memberProduct = new MemberProduct();

    memberProduct.setMemeber(member1);
    memberProduct.setProduct(productA);
    memberProduct.setOrderAmount(2); 
    
    em.persist(memberProduct);
}

public void find(){
    MemberProductId memberProductId = new MemberProductId(); 

    memberProductId.setMember("member1");
    memberProductId.setProduct("productA");

    MemberProduct memberProduct = em.find(MemberProduct.class, memberProductId);

    Member member = memberProduct.getMember();
    Product product = memberProduct.getProduct();
}
```
- 복합키는 항상 식별자 클래스를 만들어야 하고 복합키를 사용하는 방법은 복잡하다. 단순히 컬럼 하나만 기본 키로 사용하는 것과 비교해서 복합 키를 사용하면 처리할 일이 많아진다. 

### 6.4.3 다대다 : 새로운 기본 키 사용 
- 추천하는 기본 키 생성 전략은 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 값으로 사용하는 것이다. 
- 간편하고 영구히 쓸 수 있으며 비즈니스에 의존하지 않고 ORM매핑 시 복합키를 만들지 않아도 되므로 간단히 매핑이 가능하다. 

```java
@Entity 
public classs Order{

    @Id
    @GeneratedValue 
    @Column(name="ORDER_ID")
    private Long id; 

    @ManyToOne 
    @JoinColumn(name="MEMBER_ID")
    private Member member; 

    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product; 

    private int orderAmount; 


}

@Entity 
public class Member{

    @Id
    @Column(name="MEMBER_ID")
    private String id; 

    private String username; 

    @OneToMany(mapped = "member")
    private List<Order> orders = new ArrayList<Order>(); 

}

@Entity 
public class Product {

    @Id
    @Column(name="PRODUCT_ID")
    private String id; 

    private String name; 
}

public void save(){
    Member member1 = new Member();
    member1.setId("member1");
    member1.setUsername("회원1");
    em.persist(member1); 

    Product productA = new Product();
    productA.setId("productA");
    productA.setName("상품1");
    em.persist(productA); 

    Order order = new Order();
    order.setMember(member1);
    order.setProduct(productA);
    order.setOrderAmount(2);
    em.persist(order);
}

public void find(){
    Long orderId = 1L;

    Order order = em.find(Order.class, orderId); 
    Member member = order.getMember(); 
    Product product = order.getProduct()l

}

```
### 6.4.5 다대다 연관관계 정리 
- 다대다 관계를 다대일 관계로 풀어내기 위해 연결 테이블을 만들 때 식별자를 어떻게 구성할지 선택해야 한다. 
#### 식별 관계 
- 받아온 식별자를 기본키 + 외래키로 사용한다. 
#### 비식별관계 
- 받아온 식별자를 외래키로만 사용하고 새로운 식별자를 추가한다. 


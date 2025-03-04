# 14장 컬렉션과 부가 기능 
## 14.1 컬렉션 
- JPA는 Java에서 기본 제공하는 Collection, List, Set, Map 컬렉션을 지원하고 아래의 경우에 이 컬렉션을 사용할 수 있다. 
- @OneToMany, @ManyToMany를 사용해서 일대다 다대다 엔티티 관계를 매핑할 때 
- @ElementCollection을 사용해서 값 타입을 하나 이상 보관할 때 

### 14.1.1 JPA와 컬렉션 
- 하이버네이트는 엔티티를 영속 상태로 만들 때 컬렉션 필드를 하이버네이트에서 준비한 컬렉션을 감싸서 사용한다. 

|컬렉션 인터페이스|내장컬렉션|중복허용|순서보관|
|---|---|---|---| 
|Collection, List|PersistenceBag|O|X|
|Set|PersistenceSet|X|X|
|List + @OrderColumn|PersistentList|O|O|

### 14.1.2 Collection, List
-  Collection, List 인터페이스는 중복을 허용하는 컬렉션이고 PersistenceBag을 래퍼 컬렉션을 사용한다. 
-  Collection, List는 엔티티를 추가할 때 중복된 엔티티가 있는지 비교하지 않고 단순히 저장만 하면 된다. 따라서 엔티티를 추가해도 지연로딩된 컬렉션을 초기화 하지 않는다. 


### 14.1.3 Set
- Set은 중복을 허용하지 않는 컬렉션이다. 하이버네이트는 PersistentSet을 컬렉션 레퍼로 사용한다. 
- Set은 엔티티를 추가할 떼 중복된 엔티티가 있는지 비교해야 한다. 따라서 엔티티를 추가할 때 지연로딩된 컬렉션을 초기화해야 한다. 

### 14.1.4 List + @OrderColumn 
```java
@Entity 
public class Board {
    @Id 
    @GeneratedValue 
    private Long id; 
    ...
    @OneToMany(mappedBy="board")
    @OrderColumn(name="POSITION")
    private List<Comment> comments = new ArrayList<>(); 

}

```

- 순서가 있는 컬렉션은 데이터베이스에 순서 값도 함께 관리한다.

#### OrderColumn의 단점 
- @OrderColumn을 Board 엔티티에서 매핑하므로 POSITION의 값을 Comment에서는 알 수 없다. 
- POSITION은 Board.comments의 위치 값이므로 이 값을 사용해서 POSITION의 값을 UPDATE하는 SQL이 추가로 발생한다. 
- List를 변경하면 연관된 많은 위치 값을 변경해야 한다. 
- 중간에 POSITION 값이 없으면 List에는 null이 보관되고, 다른 POSITION 값을 수정하지 않으면 컬렉션 순회시 NullPointerException이 발생한다. 
### 14.1.5 @OrderBy 
- @OrderBy는 데이터베이스 ORDER BY절을 사용해서 컬렉션을 정렬하기 때문에 순서용 컬럼을 매핑하지 않아도 되고 @OrderBy는 모든 컬렉션에 사용이 가능하다. 

## 14.2 @Converter
- 컨버터를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장 가능 하다. 
```java
CREATE TABLE MEMBER (
    ...
    VIP VARCHAR(1) NOT NULL 
    ...
)

@Entity
@Convert(convert=BooleanToYnConverter.classm attributeName = "vip")
public class Member{
    ...
    @Convert(convert=BooleanToYNConverter.class)
    private boolean vip;

    ...
}

@Converter 
public class BooleanToYNConverter implements AttributeConverter<Boolean, String>

    @Override 
    public String convertToDatabaseColumn(Boolean arrtibute){
        return (arrtibute != null && attribute) ? "Y" : "N"
    }

    @Override
    public Boolean convertToEntityAttribute(String dbData){
        return "Y".equals(dbData);
    }

public interface AttributeConverter<X, Y>{
    public Y convertToDatabaseColumn(X attribute);

    public X convertToEntityAttribute(Y dbData)
}
```
### 14.2.1 글로벌 설정 
- 모든 Boolean 타입에 컨버터를 적용하려면 @Converter(autoApply = true) 옵션을 적용하면 된다. 
## 14.3 리스너 
- 모든 엔티티를 대상으로 어떤 사용자가 삭제를 요청했는지 모두 로그로 남겨야 한다면 애플리케이션 삭제 로직을 찾아서 하나씩 로그를 남기는 것은 너무 비효율적이기 때문에 JPA 리스너 기능을 사용하면 엔티티의 생명 주기에 따른 이벤트를 처리 가능하다. 
- PostLoad : 엔티티가 영속성 컨텍스트에 조회된 직후 또는 refresh 를 호출 후 수행된다. 
- PrePersist : persist() 메소드를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출된다. 식별자 생성 전략을 사용한 경우에는 엔티티의 식별자는 존재하지 않는 상태이다. 새로운 인스턴스를 merge 할 때도 수행된다.
- PreUpdate : flush나 commit을 호출해서 엔티티를 데이터베이스에 수정하기 직전에 호출된다.
- PreRemove : remove 메소드를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출된다. 또한 삭제 명령어로 영속성 전이가 일어날 때도 호출된다. orphanRemoval 에 대해서는 flush나 commit 시에 호출된다.
- PostPrsist : flush나 commit을 호출해서 엔티티를 데이터베이스에 저장한 직후에 호출된다. 식별자가 항상 존재한다. 참고로 식별자 생성 전략이 IDENTITY인 경우 식별자를 생성하기 위해 persist()를 호출하면서 데이터베이스에 해당 엔티티를 저장하므로, 이때는 persist()를 호출한 직후에 바로 PostPersist가 호출된다.
- PostUpdate : flush나 commit을 호출해서 엔티티를 데이터베이스에 수정한 직후에 호출된다.(persist 시에는 호출되지 않는다)
- PostRemove : flush나 commit을 호출해서 엔티티를 데이터베이스에 삭제한 직후에 호출된다.

### 14.3.2 이벤트 적용 위치 
- 이벤트는 엔티티에서 직접 받거나 별도의 리스너를 등록해서 받을 수 있다. 
#### 엔티티에 직접 적용 

```java
@Entity
public class Duck {

    @Id @GeneratedValue
    private Long id;

    private String name;
    
    @PostLoad
    public void PostLoad(){
        System.out.println("PostLoad");
    }

    @PrePersist
    public void prePersist(){
        System.out.println("prePersist");
    }

    @PreUpdate
    public void PreUpdate(){
        System.out.println("PreUpdate");
    }

    @PreRemove
    public void PreRemove (){
        System.out.println("PreRemove ");
    }

    @PostPersist
    public void PostPersist (){
        System.out.println("PostPersist ");
    } 

		@PostUpdate
    public void PostUpdate (){
        System.out.println("PostUpdate ");
    }

    @PostRemove
    public void PostRemove(){
        System.out.println("PostRemove ");
    }
}

```
#### 별도의 리스너 등록 

```java
@Entity
@EntityListeners(DuckListener.class)
public class Duck {
		...
}

public class DuckListener {
    @PostLoad
    public void PostLoad(Object obj){
        System.out.println("PostLoad obj = "+obj);
    }

    @PrePersist
    public void prePersist(Object obj){
        System.out.println("prePersist"+obj);
    }

    @PostPersist
    public void PostPersist (Object obj){
        System.out.println("PostPersist ");
    } 
}

```
- 여러 리스너를 등록했을 때 이벤트 호출 순서는 아래와 같다. 
1. 기본 리스너 
2. 부모 클래스 리스너 
3. 리스너 
4. 앤티티 

#### 더 세밀한 설정 
- javax.persistence.ExcludeDefaultListeners: 기본 리스너 무시 
- javax.persistence.ExcludeSuperListeners : 상위 클래스 이벤트 리스너 무시 

## 14.4 엔티티 그래프 
- 엔티티를 조회할 때 연관된 엔티티를 함께 조회하려면 글로벌 fetch옵션을 FetchType.EAGER로 설정한다. 
- 글로벌 fetch 옵션은 애플리케이션 전체에 영향을 주고 변경할 수 없는 단점이 있기 때문에 글로벌 fetch 옵션은 LAZY로 사용하고, 엔티티를 조회할 때 연관된 엔티티를 함깨 조회할 필요가 있으면 JPQL의 페치 조인을 사용한다. 
- JPQL의 페치조인을 사용하면 같은 JPQL을 중복해서 작성하는 경우가 많다. 
- 엔티티 그래프 기능을 사용하면 엔티티를 조회하는 시점에 함께 조회할 엔티티를 선택가능 하다. 
- 엔티티 그래프는 정적으로 정의하는 Named 엔티티 그래프와 동적으로 정의하는 엔티티 그래프가 있다. 

### 14.4.1 Named 엔티티 그래프 
- Named 엔티티 그래프는 @NamedEntityGraph로 정의한다. 
```java
@NamedEntityGraph(name = "Order.withMember", attributeNodes = {
    @NamedAttributeNode("member")
})
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```
### 14.4.2 em.find()에서 엔티티 그래프 사용 
- Named 엔티티 그래프를 사용하려면 위에서 정의한 엔티티 그래프를 em.findEntityGraph()를 통해서 찾아오면 된다. 

```java
EntityGraph graph = em.getEntityGraph("Order.withMember");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);

```
### 14.4.3 subgraph 
- Order -> OrderItem -> Item 조회 
```java
@NamedEntityGraph(name = "Order.withAll", attributeNodes = {
    @NamedAttributeNode("member"),
    @NamedAttributeNode(value = "orderItems", subgraph = "orderItems")
    },
    subgraphs = @NamedSubgraph(name = "orderItems", attributeNodes = {
        @NamedAttributeNode("item")
    })
)
@Entity
public class Order {
    ...
}

```
### 14.4.4 JPQL에서 엔티티 그래프 사용
- JPQL에서 엔티티 그래프를 사용하는 방법은 em.find()와 동일하게 힌트만 추가 하면 된다. 
```java
List<Order> resultList =
    em.createQuery("select o from Order o where o.id = :orderId",
        Order.class)
        .setParameter("orderId", orderId)
        .setHint("javax.persistence.fetchgraph", em.getEntityGraph("Order.withAll"))
        .getResultList();

```
### 14.4.5  동적 엔티티 그래프 
- 엔티티 그래프를 동적을 구성하기 위해서는 createEntityGraph() 메소드를 사용하면 된다. 

### 14.4.6 엔티티 그래프 정리 
- 엔티티 그래프는 항상 조회하는 엔티티의 ROOT에서 시작해야 한다. 
- 해당 컨텍스트에 해당 엔티티가 이미 로딩되어 있으면 엔티티 그래프가 적용되지 않는다. 
- fetchgraph는 선택한 속성만 함께 조회하고  loadgraph속성은 엔티티 그래프에 선택한 속성 뿐만 아니라 글로버 fetch모드가 EAGER로 설정된 연관관계도 포함해서 함께 조회한다. 


# 7장 고급매핑 

## 7.3 복합키와 식별 관계 매핑 
### 7.3.1 식별관계 vs 비식별관계 
- 데이터베이스 테이블 사이에 관계는 외래키가 기본키에 포함되는지 여부에 따라 식별관계와 비식별관계로 구분한다. 
#### 식별관계 
- 식별관계는 부모 테이블의 기본키를 내려받아서 자식 테이블의 기본키 + 외래키로 사용하는 관계이다. 

#### 비식별관계 
- 비식별관계는 부모 테이블의 기본키를 내려받아서 자식 테이블의 외래키로만 사용하는 관계이다. 
- 비식별관계는 필수적 비식별관계와 선택적 비식별관계로 나눌 수 있다. 

### 7.3.2 복합키 : 비식별 관계 매핑 
- 기본키를 구성하는 컬럼이 하나면 단순하게 매핑을 한다. 
- 둘이상의 컬럼으로 구성된 복합키를 사용하기 위해서는 별도의 식별자 클래스를 만들어야 한다. 
- JPA는 영속성 컨텍스트에 엔티티를 보관할 때 엔티티의 식별자를 키로 사용한다. 
- 식별자를 구분하기 위해서 equals와 hashCode를 사용해서 동등성 비교를 하는데 식별자가 하나 일때는 보통의 자바 기본 타입을 사용하므로 문제가 없지만, 식별자 필드가 2개 이상이면 별도의 식별자 클래스를 만들고 그곳에 equals와 hashCode를 구현해야 한다. 

#### @IdClass 
```java
@Entity 
@IdClass(ParentId.class) 
public class Parent {
    @Id 
    @Column(name= "PATENT_ID")
    private String id; 
    
    @Id 
    @Column(name= "PATENT_ID2")
    private String id2; 

}

public class ParentId implements Serializable{
    private String id;
    private String id2; 

    public ParentId(){

    }
    public ParentId(String id, String id2){
        this.id = id; 
        this.id2 = id2; 
    }
    @Override 
    public boolean equals(Object o){

    }
    @Override
    public int hashCode(){

    }
}
```
- @IdClass를 사용할 때 식별자 클래스는 다음 조건을 만족해야 한다. 
- 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 동일해야 한다. 
- Serializable 인터페이스를 구현해야 한다. 
- equals, hashCode를 구현해야 한다. 
- 기본 생성자가 있어야 한다. 
- 식별자 클래스는 public 여야 한다. 
```java
Parent parent = new Parent(); 
parent.setId("myId");
parent.setId2("myId2");
parent.setName("parentName"); 
em.persist(parent);
```
- em.persist()를 호출하면 영속성 컨텍스트에 엔티티를 등록하기 직전에 내부에 Parent.id, Parent.id2 값을 사용해서 식별자 클래스인 ParentId를 생성하고 영속성 컨텍스트의 키로 사용한다. 

```java
ParentId parentId = new ParentId("myId", "myId2");
Parent parent = em.find(Parent.class, parentId); 
 
```
```java

@Entity 
public class Child {
    @Id
    private String id; 

    @ManyToOne 
    @JoinColumns({
        @JoinColumn(name="PARENT_ID1", referencedColumnName="PARENT_ID1"), 
        @JoinColumn(name="PARENT_ID2", referencedColumnName="PARENT_ID2"), 
    private Parent parent;
    })
}
```
- 부모 테이블의 기본키 컬럼이 복합키 이므로 자식 테이블의 외래키도 복합키다. 
- 여러 컬럼을 매핑해야 하므로 @JoinColumns어노테이션을 사용하고 각각의 외래키 컬럼을 @JoinColumn으로 매핑한다. 
#### EmbeddedId
- @IdClass가 데이터베이스에 맞춘 방법이라면 @EmbeddedId는 좀 더 객체지향적인 방법이다. 
```java
@Entity 
public class Parent{
    @EmbeddedId
    private ParentId id;

}

@Embeddable 
public class ParentId implements Serializable {
    @Column(name= "PARENT_ID1")
    private String id; 

    @Column(name= "PARENT_ID2")
    private String id2; 
}
```
```java
// 저장 
Parent parent = new Parent(); 
ParentId parentId = new ParentId("myId", "myId2");
parent.setId(parentId);
parent.setName("parentName"); 
em.persist(parent);

// 조회 
ParentId parentId = new ParentId("myId", "myId2");
Parent parent = em.find(Parent.class, parentId); 
 
```
#### 복합키와 equals(), hashCode() 
- 복합키는 equals()와 hashCode()필수로 구현해야 한다. 

```java
ParentId id1 = new ParendId();
id1.setId("myId");
id1.setId("myId"); 

ParentId id2 = new ParendId();
id2.setId("myId");
id2.setId("myId"); 

id1.equals(id2);

```
- equals()를 적절하게 오버라이딩 했다면 참이지만 equals()를 적절히 오버라이딩 하지 않았다면 결과는 거짓이다. 
- Java의 모든 클래스는 기본으로 Object 클래스를 상속받는데 이 클래스가 제공하는 기본 equals()는 인스턴스의 참조값을 비교하는 동일성 비교를 하기 때문이다. 
- 영속성 컨텍스트는 엔티티의 식별자를 키로 사용해서 엔티티를 관리하고 식별자를 비교할 때 equals()와 hashCode()를 사용한다. 
- 따라서 식별자 객체의 동등성이 지켜지지 않으면 예상과 다른 엔티티가 조회되거나 엔티티를 찾을 수 없는 등 영속성 컨텍스트가 관리하는데 심각한 문제가 발생하므로 복합키는 반드시 equals()와 hashCode()를 구현해야 한다. 

#### @IdClass vs @EmbeddedId 
- @IdClass와 @EmbeddedId는 각각의 장단점이 있으므로 본인의 취향에 맞는 것을 일관성 있게 사용하면 된다. 

### 7.3.3 복합기 : 식별 관계 매핑 
- 식별관계에서 자식 테이블은 부모테이블의 기본키를 포함해서 복합키를 구성해야 하므로 @IdClass, @EmbeddedId를 사용해서 식별자를 매핑해야 한다. 
#### @IdClass와 식별관계 
```java
// 부모 
@Entity 
public class Parent {
    @Id
    @Column(name = "PARENT_ID")
    private String id; 

}

// 자식 
@Entity
@IdClass(ChildId.class)
public class Child{
    @Id
    @ManyToOne
    @JoinColumn(name="PARENT_ID")
    public Parent parent; 

    @Id
    @Column(name="CHILD_ID")
    private String childId; 

}
// 자식 Id
public class ChildId implements Serializable {
    private Stirng parent; 
    private String childId; 

    //equals, hashCode
}

//손자 
@Entity
@IdClass(GrandChildId.class)
public class GrandChild{
    @Id
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name="PARENT_ID"), 
        @JoinColumn(name="CHILD_ID")
    })
    private Child child; 

    @Id
    @Column(name="GRANDCHILD_ID")
    private String id; 

}
// 손자 Id 
public class GrandChildId implements Serializable {
    private ChildId Child; 
    private String id; 

    //equals, hashCode
}

```
- 식별관계는 기본키와 외래키를 같이 매핑해야 한다. 


#### EmbeddedId와 식별관계 
- EmbeddedId로 식별관계를 구성할 때는 @MapsId를 사용해야 한다. 

```java
// 부모 
@Entity 
public class Parent {
    @Id
    @Column(name = "PARENT_ID")
    private String id; 

}

// 자식 
@Entity
public class Child{
    @EmbeddedId
    private ChildId id;
    
    @MapsId("parentId")
    @ManyToOne
    @JoinColumn(name="PARENT_ID")
    public Parent parent; 

}
// 자식 Id
@Embeddedable
public class ChildId implements Serializable {
    private Stirng parentId;

    @Column(name = "CHILD_ID")
    private String id; 

    //equals, hashCode
}

//손자 
@Entity
public class GrandChild{
    
    @EmbeddedId
    private GrandChildId id; 

    @MapsId("childId")
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name="PARENT_ID"), 
        @JoinColumn(name="CHILD_ID")
    })
    private Child child; 

    @Id
    @Column(name="GRANDCHILD_ID")
    private String id; 

}
// 손자 Id 
@Embeddable
public class GrandChildId implements Serializable {
    private ChildId ChildId;

    @Column(name="GRANDCHILD_ID") 
    private String id; 

    //equals, hashCode
}

```
- @IdClass 와 다른 점은 @Id대신에 @MapsId를 사용한다는 점이다. 
- @MapsId는 외래키를 매핑한 연관관계를 기본키에도 매핑한다는 뜻이다. 

### 7.3.4 비식별 관계로 구현 
```java
// 부모 
@Entity 
public class Parent {
    @Id
    @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id; 

}

// 자식 
@Entity
public class Child{
    @Id
    @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    
    @ManyToOne
    @JoinColumn(name="PARENT_ID")
    public Parent parent; 

}
 
//손자 
@Entity
public class GrandChild{
    
    @Id 
    @GeneratedValue
    @Column(name= "GRANDCHILD_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name="CHILD_ID")
    private Child child; 
}


```
- 식별 관계의 복합키를 사용한 코드와 비교하면 매핑도 쉽고 코드도 단순하며 복합키 클래스가 별도로 필요하지 않다. 
### 7.3.5 일대일 식별관계 
- 일대일 식별관계는 자식 테이블의 기본키 값으로 부모 테이블의 기본키 값만을 사용한다. 
- 부모테이블의 기본키가 복합키가 아니면 자식테이블의 기본키는 복합키로 구성하지 않아도 된다. 
```java
@Entity 
public class Board {
    @Id
    @GeneratedValue 
    @Column(name= "BOARD_ID")
    private Long id; 

    private String title; 

    @OneToOne(mappedBy = "board")
    private BoardDetail boardDetail;
}

@Entity 
public class BoardDetail{
    @Id
    private Long boardId; 

    @MapsId
    @OneToOne
    @JoinColumn(name="BOARD_ID")
    private Board board; 

}

```

### 7.3.6 식별 비식별 관계의 장단점 
#### 단점  
- 식별관계는 부모 테이블의 기본키를 자식테이블로 전파, 자식 테이블의 기본키 컬럼이 점점 늘어난다. 
- 식별관계는 2개 이상의 컬럼을 합해서 복합 기본키를 만들어야 하는 경우가 많다. 
- 식별관계를 사용할 때는 기본키로 비즈니스 의미가 있는 자연키 컬럼을 조합하는 경우가 많지만 비식별 관계는 기본 키는 비즈니스와 관계없는 대리키를 주로 사용한다. 
- 식별관계의 자연키 컬럼이 자식, 손자까지 전파되면 변경하기 힘들어진다. 
- 식별 관계는 부모 테이블의 기본키를 자식 테이블의 기본키로 사용하므로 비식별관계보다 테이블 구조가 유연하지 못하다. 

#### 장점  
- 비식별관계의 기본키는 주로 대리키를 사용하는데 JPA는 @GeneratedValue처럼 대리키를 생성하기 위한 편리한 방법을 제공한다. 
- 식별관계는 기본키 인덱스를 활용하기 좋고 상위 테이블들의 기본키 컬럼을 자식, 손자들이 가지고 있으므로 특정 상황에서 조인 없이 하위테이블 검색이 가능하다. 

 

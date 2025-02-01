# 8장 프록시와 연관관계 관리 
## 8.4 영속성 전이: CASCADE 
- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 영속성 전이(transitive persistence)기능을 사용하면 된다. 

```java
@Entity
public class Parent{
    @Id 
    @GeneratedValue 
    private Long id; 

    @OneToMany(mappedBy = "parent")
    private List<Child> children = new ArrayList<Child>();

}

@Entity 
public class Child{
    @Id 
    @GeneratedValue 
    private Long id; 

    @ManyToOne
    private Parent parent;
}

private static void saveNoCascase(EntityManager em){

    Parent parent = new Parent(); 
    em.persist(parent); 

    // 1번 자식 저장 
    Child child1 = new Child();
    child1.setParent(parent); 
    parent.getChild().add(child1); 
    em.persist(child1);


    //2번 자식 저장 

    Child child2 = new Child(); 
    child2.setParent(parent);
    parent.getChild().add(child2);
    em.persist(child2);
}

```
- JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 하기 때문에 부모 엔티티를 영속 상태로 만들고 자식 엔티티도 각각 영속 상태로 만든다. 이럴 때 영속성 전이를 사용하면 부모만 영속 상태로 만들면 연관된 자식까지 한 번에 영속 상태로 만들 수 있다. 

### 8.4.1 영속성 잔전이 : 저장 
```java 
@Entity 
public class Parent{
    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    private List<Child> children = new ArrayList<Child>(); 
}
```
- 부모를 영속화할 떼 연관된 자식들도 함께 영속화하라고 cascade = CascadeType.PERSIST옵션을 설정했다. 
```java
private static void saveWithCascade(EntityManager em){

    Child child1 = new Child();
    Child child2 = new Child(); 

    Parent parent = new Parent(); 
    child1.setParent(parent);
    child2.setParent(parent); 
    parent.getChild().add(child1);
    parent.getChild().add(child2);

    em.persist(parent);
}
``` 
- 영속성 전이는 연관관계를 매핑하는 것과는 아무 관련이 없다. 단지 엔티티를 영속화 할 때 관련된 엔티티도 같이 영속화하는 편리함을 제공할 뿐이다. 

### 8.4.2 영속성 전이 : 삭제 
- 저장한 부모와 자식 엔티티를 모두 제거하려면 각각의 엔티티를 하나씩 제거해야 한다. 

```java
Parent parent = em.find(Parent.class, 1L);
Child child1 = em.find(Child.class, 1L);
Child child2 = em.find(Child.class, 2L);
```
- 영속성 전이는 엔티티를 삭제할 때도 사용 가능 하다. 
```java 
@Entity 
public class Parent{
    
    @OneToMany(mappedBy = "parent", cascade = CascadeType.REMOVE)
    private List<Child> children = new ArrayList<Child>(); 
}
```

### 8.4.3 CASCADE의 종류 
```java
public enum CascadeType {
    ALL, 
    PERSIST, 
    MERGE, 
    REMOVE, 
    REFRESH, 
    DETCH
}
```
- 여러 속성을 같이 사용하는 것이 가능하다. 
- PERSIST, REMOVE는 em.persist(), em.remove()를 실행할 때 바로 전이가 발생하지 않고 플러시를 호출할 때 전이가 발생한다. 

#### 8.5 고아 객체 
- JPA는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제공하는데 이것을 고아 객체(ORPHAN)제거라 한다. 
- 이 기능을 사용해서 부모 엔티티의 컬렉션에서 자식 엔티티의 참조만 제거하면 자식 엔티티가 자동으로 삭제 된다. 
```java
@Entity 
public class Parent{

    @Id 
    @GeneratedValue 
    private Long id; 

    @OneToMany(mappedBy = "parent", orphanRemovel = true)
    private List<Child> children = new ArrayList<Child>();

}

Parent parent1 = em.find(Parent.class, id);
parent1.getChild().remove(0); //자식 엔티티를 컬렉션에서 제거 

// DELETE FROM CHILD WHERE ID = ?

```
- 고아 객체는 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하느 기능이다. 따라서 이 기능은 참조하는 곳이 하나일 때만 사용해야 한다. 
- orphanRemovel은 @OneToOne , @OneToMany에만 사용이 가능하다. 


### 8.6 영속성 전이 + 고아 객체, 생명주기 
- CascadeType.ALL 과 orphanRemoval = true를 동시에 사용하면 어떻게 될까? 
- 일반적으로 엔티티는 EntityManager.persit()를 통해 영속화되고 EntityManger.remove()를 통해 제거된다. 
- 이것은 엔티티 스스로 생명주기를 관리한다는 뜻인데 두 옵션을 모두 활성화하면 부모 엔티티를 통해 자식의 생명 주기를 관리할 수 있다. 
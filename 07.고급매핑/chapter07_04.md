# 7장 고급매핑 
 
## 7.4 조인 테이블 
- 데이터베이스 테이블의 연관관계를 설계하는 방법은 조인 컬럼을 사용하는 방법과 조인 테이블을 사용하는 방법으로 구분할 수 있다. 

#### 조인 컬럼 사용 
- 테이블 간에 관계는 조인 컬럼이라고 부르는 외래키 컬럼을 사용해서 관리한다. 

#### 조인 테이블 사용 
- 조인 테이블이라는 별도의 테이블을 사용해서 연관관계를 관리한다. 
- 조인 컬럼을 사용하는 방식은 단순히 외래키 컬럼만 추가해서 연관관계를 맺지만 조인 테이블을 사용하는 방법은 연관관계를 관리하는 조인 테이블을 추가하고 여기에 두 테이블의 외래키를 가지고 연관관계를 관리한다. 
- 조인 테이블의 가장 큰 단점은 조인 테이블을 하나 추가해야 한다. 


### 7.4.1 일대일 조인 테이블 
- 일대일 관계를 만들려면 조인테이블에 외래키 컬럼에 각각 총 2개의 유니크 제약 조건을 걸어야한다. 
```java
@Entity 
public class Parent{
    @Id
    @GeneratedValue 
    @Column(name="PATENT_ID")
    private Long id; 
    private String name; 

    @OneToOne
    @JoinTable(name="PARENT_CHILD", 
                joinColumn = @JoinColumn(name="PARENT_ID"), 
                inverseJoinColumn = @JoinColumn(name="CHILD_ID"))
    
    private Child child;
}

@Entity 
public class Child {
    @Id
    @GeneratedValue 
    @Column(name="CHILD_ID")
    private Long id; 

    
}

```
### 7.4.2 일대다 조인 테이블 
- 일대다 관계를 만들려면 조인 테이블의 컬럼 중 다와 관련된 컬럼인 CHILD_ID에 유니크 제약 조건을 걸어야 한다. 
```java
@Entity 
public class Parent {
    @Id
    @GeneratedValud 
    @Column(name="PARENT_ID")
    private Long id; 

    @OneToMany 
    @JoinTable(name="PARENT_CHILD", 
                joinColumns= @JoinColumn(name="PARENT_ID"). 
                inverseJoinColumns = @JoinColumn(name="CHILD_ID"))
    private List<Child>child = new ArrayList<Child>();

}

＠Entity 
public class Child {

    @Id
    @GeneratedValue 
    @Column(name = "CHILD_ID")
    private Long id; 

}
```
### 7.4.3 다대일 조인 테이블 
```java
@Entity 
public class Parent{
    @Id
    @GeneratedValud 
    @Column(name="PARENT_ID")
    private Long id; 

    @OneToMany(mappedBy = "parent") 
    private List<Child>child = new ArrayList<Child>();

}

＠Entity 
public class Child {

    @Id
    @GeneratedValue 
    @Column(name = "CHILD_ID")
    private Long id; 

    @ManyToOne(optional = false)
    @JoinTable(name = "PARENT_CHLD", 
                joinColumn = @JoinColumn(name = "CHILD_ID"), 
                inverseJoinColumn = @JoinColumn(name = "PARENT_ID"))
    
    private Parent parent;

}

```

### 7.4.4 다대다 조인 테이블 
- 다대다 관계를 만들려면 조인 테이블의 두 컬럼을 합쳐서 하나의 복합 유니크 제약 조건을 걸어야 한다. 
```java
@Entity
public class parent{


    @Id
    @GeneratedValud 
    @Column(name="PARENT_ID")
    private Long id; 

    @ManyToMany 
    @JoinTable(name="PARENT_CHILD", 
                joinColumns= @JoinColumn(name="PARENT_ID"). 
                inverseJoinColumns = @JoinColumn(name="CHILD_ID"))
    private List<Child>child = new ArrayList<Child>();

}

＠Entity 
public class Child {

    @Id
    @GeneratedValue 
    @Column(name = "CHILD_ID")
    private Long id; 

}


```

## 7.5 엔티티 하나에 여러 테이블 매핑 
- @SecondaryTable을 사용하면 한 엔티티에 여러 테이블을 메핑할 수 있다. 
```java
@Entity 
@Table(name= "BOARD")
@SeconaryTable(name = "BOARD_DETAIL",
                pkJoinColumn = @PrimaryKeyJoinColumn(name= "BOARD_DETAIL_ID"))
public class Board{
    @Id
    @GeneratedValud 
    @Column(name="BOARD_ID")
    private Long id; 
    
    @Column(table = "BOARD_DETAIL")
    private String content; 
}

``` 
- Board 엔티티는 @Table를 사용해서 BOARD테이블과 매핑하고 @SecondaryTable를 사용해서 BOARD_DETAIL테이블을 추가로 매핑했다. 
#### @SecondaryTable.name 
- 매핑할 다른 테이블의 이름 
#### @SecondaryTable.pjJoinColumns 
- 매핑할 다른 테이블의 기본 키 컬럼 속성 
#### @SecondaryTables 
- 더 많은 테이블을 매핑하려면 @SecondaryTables를 사용하면 된다. 
```java 
@SecondaryTables({
    @SecondaryTable(name = "BOARD_DETAIL"), 
    @SecondaryTable(name = "BOARD_FILE") 
    
}) 
```

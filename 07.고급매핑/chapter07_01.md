# 7장 고급매핑 

## 7.1 상속관계 매핑 
- 관계형 데이터베이스에는 객체 지향 언어에서 다루는 상속이라는 개념이 없다. 
- 대신에 Super-Type Sub-Type라는 모델링 기법이 객체의 상속 개념과 가장 유사하다. 
- ORM에서 이야기하는 상속 관계 매핑은 객체의 상속구조와 데이터베이스의 슈퍼타입 서브 타입 관계를 매핑하는 것이다. 
- 슈퍼 타입 - 서브 타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때는 3가지 방법을 선택 가능하다. 

### 7.1.1 조인 전략(Joined Strategy)
- 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본키를 받아서 기본키 + 외래키로 사용하는 전략이다. 
- 조회할 때는 조인을 자주 사용한다. 
- 이 전략을 사용할 때는 객체는 타입으로 구분할 수 있지만 테이블은 타입 개념이 없기 때문에 타입을 구분하는 컬럼을 추가 해야 한다. 
```java
@Entity 
@Inheritance(strategy=InheritanceType.JOINED)
@DiscriminatorValue(name="DTYPE")
public abstract class Item{
    @Id
    @GeneratedValue
    @Column(name="ITEM_ID")
    private Long id; 

    private String name; 
    private int price; 
}
@Entity
@DiscriminatorValue("A")
public class Album extends Item{
    private String artist; 
    ...
}

@Entity 
@DiscriminatorValue("M")
public class Movie extends Item{
    private String director;
    private String actor;  
    ...
}

```
#### @Inheritance(strategy=InheritanceType.JOINED)

- 상속 매핑은 부모 클래스에 @Inheritance 를 사용해야 하고 매핑전략을 지정해야 하는데 여기서는 조인 젼략을 사용하기 때문에 InheritanceType.JOINE를 사용했다. 

#### @DiscrimanatorColumn("DTYPE")
- 부모 클래스에 구분 컬럼을 지정한다. 이 컬럼으로 저장된 자식 테이블을 구분 가능하다. 기본 값이 DTYPE이므로 @DiscrimanatorColumn으로 줄여서 사용해도 된다. 

#### @DiscrimanatorValue("A")
- 엔티티를 저장할 때 구분 컬럼에 입력값을 지정한다. 

#### @PrimaryKeyJoinColumn 
- 기본 값으로 자식 테이블은 부모 테이블의 ID 컬럼명을 그대로 사용하는데 만약 자식 테이블의 기본키 컬럼명을 변경하고 싶으면 @PrimaryKeyJoinColumn을 사용하면 된다. 

```java
@Entity 
@DiscriminatorValue("B")
@PrimaryKeyJoinColumn(name = "BOOK_ID")
public class Book extends Item{
    private String autor;
    private String isbn;  
    ...
}
```    
#### 장점 
- 테이블이 정규화 된다. 
- 외래키 참조 무결성 제약 조건을 활용할 수 있다. 
- 저장 공간을 효율적으로 사용한다. 
#### 단점 
- 조회할 때 조인이 많이 사용되므로 성능이 저하될 수있다. 
- 조회 쿼리가 복잡하다.  
- 데이터를 등록할 때 INSERT SQL을 두번 실행한다. 
#### 특징 
- JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 Hibernate를 포함한 몇몇 구현체는 구분 컬럼 없이도 동작한다. 

### 7.1.2 단일 테이블 전략 
- 이름 그대로 테이블을 하나만 사용한다. 그리고 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분한다. 
- 조회할 때 조인을 사용하지 않으므로 일반적을 가장 빠르다. 
- 이 전략을 사용할 때 주의점으로 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다. 

```java
@Entity 
@Inheritance(strategy= Inheritance.SINGLE_TABLE)
@DiscriminatorColumn(name="DTYPE")
publc abstract class Item {
    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id; 
    private String name; 
    private int price; 

}
@Entity
@DiscriminatorValue("A")
public class Album extends Item{
    private String artist; 
    ...
}

@Entity 
@DiscriminatorValue("M")
public class Movie extends Item{
    private String director;
    private String actor;  
    ...
}

```
- InheritanceType.SINGLE_TABLE로 지정하면 단일 테이블 전략을 사용한다.
- 테이블 하나에 모든 것을 통합하므로 구분 컬럼을 필수로 사용해야 한다.

#### 장점
- 조인이 필요없으므로 일반적으로 조회성능이 빠르고 단순하다. 


#### 단점 
- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다. 
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있고, 상황에 따라 조회성능이 느려질 수 있다. 

#### 특징 
- 구분 컬럼을 반드시 사용해야 한다. 따라서 @DiscriminatorColumn을 꼭 설정해야 한다. 
- @DiscrimatorValue를 지정하지 않으면 기본 엔티티를 이름으로 사용한다. 

### 7.1.3 구현 클래스마다 테이블 전략 
- 구현 클래스 마다 테이블 전략은 자식 엔티티 마다 테이블을 만들고 자식 테이블에 각각 필요한 컬럼이 모두 있다. 
```java

@Entity 
@Inheritance(strategy= Inheritance.TABLE_PER_CLASS)
publc abstract class Item {
    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id; 
    private String name; 
    private int price; 

}
@Entity
public class Album extends Item{
    private String artist; 
    ...
}

@Entity 
public class Movie extends Item{
    private String director;
    private String actor;  
    ...
}

```
- @Inheritance(strategy= Inheritance.TABLE_PER_CLASS)를 선택하면 구현 클래스 마다 테이블 전략을 사용한다. 
- 이 전략은 자식 엔티티마다 테이블을 만든다. **일반적으로 추천하지 않는 전략이다.**

#### 장점 
- 서브타입을 구분해서 처리할 때 효과적이다. 
- not null 제약 조건을 사용할 수 있다. 

#### 단점 
- 여러 자식 테이블을 함께 조회할 때 성능이 느리다.
- 자식 테이블을 통합해서 쿼리하기 어렵다. 

#### 특징 
- 구분 컬럼을 사용하지 않는다. 


# 7장 고급매핑 

## 7.2 @MappedSuperclass 
- 부모 클래스를 테이블과 매핑하지 않고 부모 클래스를 상속 받는 자식 클래스에게 매핑 정보만 제공하고 싶으면 @MappedSuperclasss를 사용하면 된다. 
- @MappedSuperclass는 추상 클래스와 비슷한데 @Entity는 실제 테이블과 매핑되지만 @MappedSuperclass는 실제 테이블과 매핑되지 않는다. 
- 이것은 단순히 매핑 정보를 상속할 목적으로만 사용된다. 

```java
@MappedSuperclass 
public abstract class BaseEntity {
    @Id 
    @GeneratedValue 
    private Long id; 
    private String name; 
}

@Entity 
public class Member extends BaseEntity {
    private String email;
    
}

@Entity 
public class Member extends BaseEntity {
    private String email;
    
}

@Entity 
public class Member extends BaseEntity {
    private String email;
}

@Entity 
public class Seller extends BaseEntity{
    private String shopName; 

}
```
- BaseEntity에는 객체들이 주로 사용하는 공통 매핑 정보를 정의했다. 
- 자식 엔티티들은 상속을 통해 BaseEntity의 매핑 정보를 물려 받았다. 
- 부모로 물려받은 매핑 정보를 재정의하려면 @AttributeOverrides나 @AttributeOverride를 사용한다. 
- 연관관계를 재정의 하려면 @AssociationOverrides 나 @AssociationOverride를 사용한다. 

```java
@Entity 
@AttributeOverride(name="id", column= @Column(name= "MEMBER_ID"))
public class Member extends BaseEntity{


}

@Entity 
@AttributeOverrides({
    @AttribureOverride(name="id", column= @Column(name= "MEMBER_ID")), 
    @AttribureOverride(name="name", column= @Column(name= "MEMBER_NAME")), 
    })
public class Member extends BaseEntity{

}

```
- 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해서 사용한다. 
- @MapeedSuperclass로 지정한 클래스는 엔티티가 아니므로 em.find나 JPQL에서 사용이 불가능하다. 
- 이 클래스를 직접 생성해서 사용할 일은 거의 없으므로 추상 클래스로 만드는 것을 권장한다. 

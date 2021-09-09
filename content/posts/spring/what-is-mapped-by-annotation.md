---
author: Han
title: Mapped By
date: 2021-09-08
tags: ['JPA', 'Spring']
ShowToc: false
---

# 1. 어디까지 알고 있는가?
- `mapped by` 보통 연관관계 매핑에서 사용되며, 연관 관계에서 어느쪽이 주도권을 가지고 있는 지 나타날 떄 쓰이는 어노테이션.
- 어노테이션에 이 속성이 붙어있는 경우, 해당 필드는 주도권을 가지고 있지 않음.
  - 이 속성이 없는 쪽에서, 연관 관계를 관리한다고 보면 된다.
  - 즉 이 속성이 없는 쪽에서 생성, 업데이트를 해줘야 적용된다.


# 2. 내가 생각한게 맞는가?
- 아래와 같이 Entity 가 존재한다고 할 때..

```java
@Entity
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;
    private String name;
    @OneToMany(mappedBy = "person", cascade = CascadeType.ALL)
    private List<Address> addresses
}

@Entity
public class Address {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;
    private String street;
    private int houseNumber;
    private String city;
    private int zipCode;
    @ManyToOne(fetch = FetchType.LAZY)
    private Person person;
}
```
- 우선 알고 있는 바를 토대로..
- Person - Address 는 1대 다 관계
- `List<Address> addresses` 에 mappedBy 속성 붙어 있으므로, 해당 관계에서 주도권은 `Address`가 갖는다
- 즉 `Address` Entity 생성 시에, Address.person에 알맞은 Entity를 set 하고 생성 및 업데이트 해야만, `Person.addresses` 에서 조회가 가능할 것임.
  - 외래키 (forien key) 는 `Address` 에서 관리될 것이고..

- `mapped by` 되었다라는 의미는, 
   - `...it tells hibernate not to map this field. it's already mapped by this field [name="field"]...`
   - hibernate한테, 이 필드에 대해 map 하지 말라. 이 필드는 이미 `person` 이라는 이름으로 맵핑된 상태다. 
   - 위 예시를 보면, `List<Address> addresses` 필드는 이미 `Address` Entity 에서 `person` 이라는 이름으로 맵핑된 상태다.

- 참고
    - https://kok202.tistory.com/138
    - https://stackoverflow.com/questions/9108224/can-someone-explain-mappedby-in-jpa-and-hibernate

# 3. OneToMany의 다른 속성들

 > *cascade*
 ```java
     /** 
     * (Optional) The operations that must be cascaded to 
     * the target of the association.
     * <p> Defaults to no operations being cascaded.
     *
     * <p> When the target collection is a {@link java.util.Map
     * java.util.Map}, the <code>cascade</code> element applies to the
     * map value.
     */
    CascadeType[] cascade() default {};
 ```

- [cascade](https://en.dict.naver.com/#/entry/enko/404e7935c8494e448e4991c5096de246) , 폭포, 매달리다, 흐르다..
  - 아마도 뭔가 연결성, 전파성, 종속성을 의미하는 단어인듯.
- `optional` 하고,
- `default` 값은 어떠한 행동도 cascade 되지 않을 것.
- `CascadeType` 은 ALL, PERSIST, MERGE, REMOVE, REFRESH, DETACH
    - ALL : 모든 작업(ALL 이하) 상위에서 하위 엔티티로 전파됨.
    - PERSIST : 상위 엔티티가 Persist 되면 (저장) 하위 엔티티도 Persist 됨. (변경 감지 x)
    - MERGE : 변경 감지가 가능한 상태인 상위 엔티티 MERGE (update) 되면 하위 엔티티의 변경 사항도 update 되는듯
    - REMOVE : 상위 엔티티를 삭제하면, 이에 연결된 하위 엔티티도 삭제 되는 것. JPA에서 제공하는 `CascadeType.DELETE` 과 차이가 없는 듯.
    - REFRESH : reread the value of a given instance from the database, 즉 데이터베이스에서, 인스턴스에 해당하는 값을 다시 읽어오는 것. 즉 부모 엔티티 값이 갱신되면, 자식 엔티티도 갱신되어짐이 기대된다.
    - DETACH : 상위 엔티티가 `persistent context` 에서 제거 되면, 하위 엔티티도 제거된다.

- 참고
    - https://www.baeldung.com/jpa-cascade-types



# 4. 궁금한 점

> *그럼 mappedBy 된 필드에, 추가 혹은 업데이트 시 반영되지 않는가?*
- 생각해보면, 우선 `mappedBy` 되어 있으면, 추가 업데이트 시에 반영되지 않을 것 같음.
- 그런데, `cascade` 옵션이 추가적으로 적혀 있으면 반영될 것 같음.
- [CascadeType.PERSIST를 함부로 사용하면 안되는 이유](https://joont92.github.io/jpa/CascadeType-PERSIST%EB%A5%BC-%ED%95%A8%EB%B6%80%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-%EC%95%88%EB%90%98%EB%8A%94-%EC%9D%B4%EC%9C%A0/) 라는 포스팅을 보면, 그에 대한 답이 있는듯.
  - cascdeType이 적용되어 있으면, 해당 타입에 따라 mappedBy 관계로 맺어진 엔티티라도, 부모 엔티티에 적용된 operation 이 연쇄적으로 퍼질 수 있는듯.
  - 즉 `mappedBy` 로 의존 관계의 owner 를 설정해뒀어도, cascde option 에 따라, 해당 엔티티의 추가 및 업데이트가 실행될 수 있는듯.


# 5. 테스트
- Person, Address Entity.
- 관계는 1번에 정의된 1 : 다 관계.
- Person이 가지고 있는 `List<Address> addresses` 에 `mappedBy` + `CascadeType` 을 조정하면서 실행.

## 5.1 persist operation

```java
    @Test
    public void testPersist() {
        Person person = new Person();
        Address address = new Address();
        person.setAddresses(Arrays.asList(address));
        session.persist(person);
        session.flush();
        session.clear();
    }
```

> *CascadeType.ALL*
```txt
...
Hibernate: insert into Person (name, id) values (?, ?)
Hibernate: insert into Address (city, houseNumber, person_id, street, zipCode, id) values (?, ?, ?, ?, ?, ?)
```
- Person.address 는 `mappedBy` 로 묶여 있지만, `CascadeType.ALL` 로 옵션이 있기에, 부모 엔티티 (Person) 의 persist operation 이 자식 엔티티 (Address) 로 전파됨을 확인.

> *CascadeType.MERGE*
```txt
Hibernate: insert into Person (name, id) values (?, ?)
```
- Persist 외 상황에서는 적용되지 않음을 확인
- 하나만 더보자.

## 5.2 merge operation

```java
    @Test
    public void testMerge() {
        int pId;
        Person person = buildPerson("devender");
        Address address = buildAddress(person);
        person.setAddresses(Arrays.asList(address));
        address.setPerson(person);

        session.persist(person);
        session.persist(address);
        session.flush();
        pId = person.getId();
        session.clear();

        Person savedPersonEntity = session.find(Person.class, pId);
        Address savedAddressEntity = savedPersonEntity.getAddresses().get(0);
        savedPersonEntity.setName("devender kumar");
        savedAddressEntity.setHouseNumber(24);
        session.merge(savedPersonEntity);
        session.flush();
    }

    private Address buildAddress(Person person) {
      Address address = new Address();
      address.setCity("Berlin");
      address.setHouseNumber(23);
      address.setStreet("Zeughofstraße");
      address.setZipCode(123001);
      address.setPerson(person);
      return address;
    }

    private Person buildPerson(String name) {
        Person person = new Person();
        person.setName(name);
        return person;
    }
```

> *CascadeType.MERGE*

```txt
Hibernate: update Person set name=? where id=?
Hibernate: update Address set city=?, houseNumber=?, person_id=?, street=?, zipCode=? where id=?
```
- `savedPersonEntity` 만 merge를 실행했지만, 이 operation이 Address에도 전파됨을 알 수 있다.
- 만약에 `savedAddressEntity.setPerson(..)` 으로 업데이트가 아닌 Persist operation 을 유도하면?

```java
        Person savedPersonEntity = session.find(Person.class, pId);
        Address savedAddressEntity = savedPersonEntity.getAddresses().get(0);
        savedPersonEntity.setName("devender kumar");
//        savedAddressEntity.setHouseNumber(24);
        savedAddressEntity.setPerson(buildPerson("newPerson"));
        session.merge(savedPersonEntity);
        session.flush();
```

- 결과적으로 에러로그를 볼 수 있다.

```txt
java.lang.IllegalStateException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing: com.baeldung.cascading.domain.Person
```
- Object 의 참조값이, 저장되지 않은 transient instance 이다. 
- 아마도 Persistent context 에서 관리되지 않는 객체라는 의미인듯 싶다.

## 5.3 Only mappedBy
- 만약에 `mappedBy` 만 정의되어 있다면?

> *persist*
```txt
Hibernate: insert into Person (name, id) values (?, ?)
```
- Person 엔티티만 생성됨을 알 수 있다 (persist operation 이 전파가 안된다.)

> *merge*
```txt
Hibernate: update Person set name=? where id=?
Hibernate: update Address set city=?, houseNumber=?, person_id=?, street=?, zipCode=? where id=?
```
- 로그 상으로는 Person 엔티티 업데이트 시, Address로 전파된다. 
- 전파가 되지 않을 것을 기대했는데, 전파된다. 이게 맞나? (좀 더 찾아봐야 할듯)
- [JPA Managed entities merge operations without cascade options](https://stackoverflow.com/questions/18377055/jpa-managed-entities-merge-operations-without-cascade-options) 이 글로 살펴보았을 때, 전파가 된다기 보다는, Person이 Persistent context에 들어가면서, 그 내부에 있는 객체들도 모두 관리 대상이 되었고, 그 관리 대상 중에 업데이트가 일어나서 자연스럽게 업데이트가 쳐진듯?

> *remove*

```java
    @Test
    public void testRemove() {
        int personId;
        Person person = buildPerson("devender");
        Address address = buildAddress(person);
        person.setAddresses(Arrays.asList(address));
        address.setPerson(person);

        session.persist(person);
        session.persist(address);
        session.flush();
        personId = person.getId();
        session.clear();

        Person savedPersonEntity = session.find(Person.class, personId);
        session.remove(savedPersonEntity);
        session.flush();
    }
```

- 하나만 더보자.
- 만약에 전파가 일어난다고 하면 어떻게 될까?
- `savedPersonEntity`를 삭제하려면, address에서 해당 person id를 가지고 있는 address entity를 모두 삭제하고 그리고 savePersonEntity가 삭제되어야하나? 
- 너무 생각할게 많아지고 복잡한 느낌. 아마 안될듯.

```txt
Hibernate: delete from Person where id=?
[2021-09-09 22:57:24,493]-[main] WARN  org.hibernate.engine.jdbc.spi.SqlExceptionHelper - SQL Error: 23503, SQLState: 23503
[2021-09-09 22:57:24,493]-[main] ERROR org.hibernate.engine.jdbc.spi.SqlExceptionHelper - Referential integrity constraint violation: "FKDU13RL17O4H24M9GT7B2BDOBO: PUBLIC.ADDRESS FOREIGN KEY(PERSON_ID) REFERENCES PUBLIC.PERSON(ID) (1)"; SQL statement:
```

- 에러 내용으로 보아, `savedPersonEntity` 를 삭제하려 했는데, Address 테이블의 Forein Key(PERSON ID) 를 참조하고 있는게 발견되어서, 삭제하지 못했다는 뜻.
- 전파..되는 것 같진 않다.

- 참고
  - [eugenp/tutorials/persistence-modules/jpa-hibernate-cascade-type/](https://github.com/eugenp/tutorials/tree/master/persistence-modules/jpa-hibernate-cascade-type)


# 6. 정리
- 정리해보자면, `mappedBy`는 외래키를 어디서 관리하냐 정도 의미 인듯. 
- `mappedBy`가 붙어 있으면 해당 엔티티에서 관리하진 않고, 상대편 엔티티에서 관리하는듯
- `mappedBy` + `cascadeType` 이 같이 있으면, 해당 엔티티에 추가, 업데이트 될 시 트랜잭션 끝나는 시점에 쿼리 날라갈 수 있음.
- 즉 `mappedBy` 는 엔티티 간 관계만 나타낼 뿐, Persistent Context에서 관리하는 건 똑같으니까, 이 객체에 변화(merge) 가 있을 경우 업데이트가 날라갈 수도 있다.
- 조심해서 사용해야할듯함.



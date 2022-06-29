---
title:  "[SPRING] Spring Data Jpa 학습"
excerpt: "Spring Data Jpa 학습"

categories: Java
tags:
  - [other]

toc: true
toc_sticky: true
 
date: 2022-06-30
last_modified_at: 2022-06-30
author_profile: false     
---


## 스프링부트 프로젝트 생성

> JPA용 프로젝트와 SpringDataJpa용 프로젝트 2가지를 생성하여 개발/테스트 하였다. 

- 의존성 추가는 동일하다. <br>
 1. spring-boot-starter-web
 2. spring-boot-starter-data-jpa
 3. lombok
 4. spring-boot-starter-test

> persistence.xml 생성

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">

    <persistence-unit name="sk_jpa">

        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="oracle.jdbc.driver.OracleDriver"/>
            <property name="javax.persistence.jdbc.user" value="********"/>
            <property name="javax.persistence.jdbc.password" value="**********"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:oracle:thin:@*********"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.Oracle10gDialect" />

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />
			<!--Created-->
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
 
</persistence>
```

 - 심플한 Member 객체를 생성한다.

```java
@Getter
@Entity
public class Member {
	
	@Id
    private Long id;
	
	private String name;
	
	public void setId(Long id) {
		this.id = id;
	}
	
	public void setName(String name) {
		this.name = name;
	}
}
```

 - 테스트 코드를 생성한다. 

```java
class MemberTest {

	@Test
	void test() {
		EntityManagerFactory emf = Persistence.createEntityManagerFactory("sk_jpa");
		EntityManager em = emf.createEntityManager();
		
		EntityTransaction tx = em.getTransaction();
		try {
			tx.begin();
			
			businessLogic(em);
			
			tx.commit();
		}catch(Exception e) {
			tx.rollback();
		}finally {
			em.close();
		}
		emf.close();
	}

	private void businessLogic(EntityManager em) {
		
		Member m = new Member();
		m.setId(1L);
		m.setName("sk");
		em.persist(m);
	}

}
```

 - 앞으로 테스트시 businessLogic 부분을 제외하면, 나머지는 중복될 것이므로 템플릿 콜백 패턴으로 추출한다. 

```java
public enum JpaTemplate {
	
	INSTANCE;
	
	public void persist(BusinessCallback callback) {
		EntityManagerFactory emf = Persistence.createEntityManagerFactory("sk_jpa");
		EntityManager em = emf.createEntityManager();
		
		EntityTransaction tx = em.getTransaction();
		try {
			tx.begin();
			
			callback.progress(em);
			
			tx.commit();
		}catch(Exception e) {
			tx.rollback();
		}finally {
			em.close();
		}
		emf.close();
	}

}

public interface BusinessCallback {

	void progress(EntityManager em);

}

``` 

 - 테스트 코드를 변경한다. 

```java
class MemberTest {

	@Test
	void test() {
		
		JpaTemplate.INSTANCE.persist(new BusinessCallback() {
			
			@Override
			public void progress(EntityManager em) {
				Member m = new Member();
				m.setId(1L);
				m.setName("sk2");
				em.persist(m);
				
			}
		});
		
	}

}
```

 - `<property name="hibernate.hbm2ddl.auto" value="create" />` 옵션 때문에 테이블에 drop-create 된다. 

## 기본키 정책

 - 이전 예제에서는 id 값을 직접 넣어주었다면, 이번엔 DBMS에서 제공해주는 기능을 사용하여 넣어주는 것을 해본다. 
 - Long 타입 외에 UUID 타입으로 기본키를 설정할수도 있다.

```java
	@Id
	@GeneratedValue
    private Long id;
```

 - Member 에 id 값을 설정하지 않는다. 설정하면, 오류 발생.
 - @GeneratedValue의 기본값은 @GeneratedValue(strategy = GenerationType.AUTO)으로 생략해도 된다.
 - 오라클은 auto_increament 를 제공하지 않고, 시퀀스 사용한다. Mysql은 auto_increament 를 제공한다.
 - 오라클의 경우, 별도 시퀀스를 지정해주지 않으면 hibernate_sequence 시퀀스가 자동 생성된다. 

>  GenerationType.AUTO 와 GenerationType.IDENTITY 차이 

 - GenerationType.AUTO <br>
  : 시퀀스 오브젝트를 사용한다. Mysql 은 시퀀스 오브젝트가 없으므로 Table Generator 를 사용한다. 오라클은 hibernate_sequence라는 시퀀스를 생성하고, Mysql 은 hibernate_sequence라는 키 생성 전용 테이블을 생성되어 전역으로 사용되어진다. <br>
  : hibernate_sequence 외에 별도의 제너레이터(시퀀스,테이블)를 지정할 수 있다. 지정할때는 GenerationType.IDENTITY로 적용해야 할듯

 - GenerationType.IDENTITY <br>
  : Mysql 은 auto_cincreament 동작이 수행된다. <br>
  : 아래와 같이 오라클은 시퀀스명을 지정할수도 있다.

```java
@Getter
@Entity
@SequenceGenerator(name="MEMBER_SEQ_GEN",
sequenceName = "SEQ_T_MEMBER",
initialValue = 1)
public class Member {
	
	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator ="MEMBER_SEQ_GEN")
    private Long id;
	
	private String name;
	
	public void setId(Long id) {
		this.id = id;
	}
	
	public void setName(String name) {
		this.name = name;
	}
}
```

> 기본키 생성 전략

 - IDENTITY
  1. id값을 null 로 하면 DB가 알아서 AUTO_INCREMENT 해준다(Mysql, PostgreSql, Mssql등)
  2. IDENTITY는 Insert Query 를 수행하는 순간 ID가 정해진다. 영속성 컨텍스트에서 객체가 관리되려면 무조건 PK값이 있어야 한다. 때문에, IDENTITY에서는 예외적으로 persist() 가 호출되는 시점에 insert 쿼리를 수행하여, DB에서 식별자를 조회해 영속성 컨텍스트의 1차 캐시에 값을 넣는다. 

- SEQUENCE
  1. 시퀀스 오브젝트 사용(Oracle, PostgreSql, DB2, H2)
  2. 시퀀스는 메모리상에 할당할 범위를 정할수 있다. 기본값은 50인데, 1로 설정할수 있지만, 시퀀스 조회를 위한 네트웍 통신이 계속 발생하게 된다. 

- IDENTITY는 persist 하면서 쿼리를 수행하고, SEQUENCE는 commit 하면서 쿼리를 수행한다.

> 갑자기 오류 발생

```java
java.lang.IllegalArgumentException: Unknown entity: org.hibernate.internal.SessionImpl
	at org.hibernate.internal.SessionImpl.firePersist(SessionImpl.java:723)
	at org.hibernate.internal.SessionImpl.persist(SessionImpl.java:706)
	at com.sk.jpa.started.ch01.MemberTest$1.progress(MemberTest.java:25)
	at com.sk.jpa.started.JpaTemplate.persist(JpaTemplate.java:22)
	at com.sk.jpa.started.ch01.MemberTest.test(MemberTest.java:17)
```

 - 가끔 Unknown entity 가 발생하는 경우가 있다고 한다. 저 경우는 persistence.xml 에 entity 클래스를 선언해주면 된다고 함. 

```xml
<persistence-unit name="sk_jpa">
		<class>com.sk.jpa.started.ch01.Member</class>
        <properties>
```
 - 하지만 저 경우는,, 테스트 코드 실수로 발생함.

```java
	@Override
	public void progress(EntityManager em) {
		Member m = new Member();
		//m.setId(1L);
		m.setName("sk");
		System.out.println("=========before persist===============");
		em.persist(em); //em 이 아니라 m이어야 함. 
		System.out.println("=========after persist===============");
	}

	/*요렇게 변경함*/
	public void progress(EntityManager entityManager) {
		Member member = new Member();
		//m.setId(1L);
		member.setName("sk");
		System.out.println("=========before persist===============");
		entityManager.persist(member);
		System.out.println("=========after persist===============");
	}
```

 - persist 에 m이 아닌 em 변수를 넣어벼렸다. ㅡㅡ; 변수명은 잘 정의해야할것 같다. 혼란스럽지 않도록!

## 생성시간 및 enum 

 - LocalDateTime으로 필드 선언시, 오라클에서는 TimeStamp 타입 컬럼이 생성된다. 
 - @Enumerated는 Enum 객체를 컬럼 값으로 사용할 수 있다. 
 - EnumType.STRING 은 enum 타입 문자열로, EnumType.ORDINAL은 enum 순서로 적용된다.

```java
@ToString
@Getter
@NoArgsConstructor
@Entity
public class Member{
	
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
	
	private String name;
	
	@Column(name = "create_date")
    private LocalDateTime createDate;
	
	@Enumerated(EnumType.STRING)
    private Grade grade;
	
	public void setId(Long id) {
		this.id = id;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	@Builder
	public Member(String name, LocalDateTime now, Grade grade) {
		this.name = name;
		this.createDate = now;
		this.grade = grade;
	}
}
```

 - enum 을 별도의 컨버터를 통해 커스텀하게 DB에 저장되는 데이터를 변환할 수 있다. 

```java
@Convert(converter = GradeConverter.class)
private Grade grade;

...
public class GradeConverter implements AttributeConverter<Grade, Integer>{

	//enum 이 integer 타입으로 변환되므로, DB 컬럼은 number 타입이 된다. 
	@Override
	public Integer convertToDatabaseColumn(Grade attribute) {
		if(attribute==Grade.S1) {
			return 100;
		}else if(attribute==Grade.S2) {
			return 101;
		}
		return 102;
	}

	@Override
	public Grade convertToEntityAttribute(Integer dbData) {
		if(dbData == 100) {
			return Grade.S1;
		}else if(dbData == 101) {
			return Grade.S2;
		}
		return Grade.S3;
	}
}
``` 


## Embedded

 - Embedded를 사용하여, 새로운 값 타입을 정의해서 사용할 수 있다. 
 - 아래 Employee 에서는 Address 라는 타입을 정의해서 사용하였다. 
 - 또한 ID는 UUID 를 적용해보았다. `@Type(type = "uuid-char")`를 사용해야지, UUID가 텍스트로 저장된다. 그렇지 않으면 바이너리러 저장됨
 - address 가 home 과 work 2가지가 있는데, 중복되지 않도록 workAddress 의 컬럼을 재정의한다. 

```java
package com.sk.jpa.started.ch03;

import java.util.UUID;

import javax.persistence.Embedded;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import org.hibernate.annotations.GenericGenerator;
import org.hibernate.annotations.Type;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

@ToString
@Getter
@NoArgsConstructor
@Entity
public class Employee {
	
	@Id
	//@Column(columnDefinition = "VARCHAR(255)")
	@Type(type = "uuid-char")
	@GeneratedValue(generator = "uuid2")
	@GenericGenerator(name="uuid2", strategy = "org.hibernate.id.UUIDGenerator")
	private UUID id;
	
	@Embedded
	private Address homeAddress;

	@AttributeOverrides({
        @AttributeOverride(name = "address1", column = @Column(name = "waddr1")),
        @AttributeOverride(name = "address2", column = @Column(name = "waddr2")),
        @AttributeOverride(name = "zipcode", column = @Column(name = "wzipcode"))
	})
	@Embedded
	private Address workAddress;

	public void setHomeAddress(Address homeAddress) {
		this.homeAddress = homeAddress;
	}
	
	public void setWorkAddress(Address workAddress) {
		this.workAddress = workAddress;
	}

	public void setAddress(Address homeAddress) {
		this.homeAddress = homeAddress;
	}
}

```

```java
@ToString
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Embeddable
public class Address {
	
	private String address1;
	private String address2;
	private String zipcode;
	
}

- 테스트 코드

```java
@Test
void test() {
	JpaTemplate.INSTANCE.persist((EntityManager em) ->{
		
		Address homeAddress = new Address("kyungkido", "namyan", "1001");
		Address workAddress = new Address("kyungkido2", "namyan2", "1002");
		Employee employee = new Employee();
		employee.setHomeAddress(homeAddress);
		employee.setWorkAddress(workAddress);
		
		em.persist(employee);
		
		Employee findEmployee = em.find(Employee.class, employee.getId());
		System.out.println(findEmployee);
		}
	);
}
```

```java
//테스트 결과 후 DB 조회 결과
/*
ID	ADDRESS1	ADDRESS2	ZIPCODE	WADDR1	WADDR2	WZIPCODE
702894e4-e42d-43fa-bf3b-31523c04f476	kyungkido	namyan	1001	kyungkido2	namyan2	1002
*/
```

## 엔티티 하나에 여러 테이블 매핑

 - @SecondaryTables 을 사용하여, 하나의 엔티티에서 여러개의 테이블을 사용할 수 있다. 아래 예제에서는 BOARD_DETAIL과 BOARD_FILE 2 개의 테이블을 정의했고, Board 엔티티안에 필드를 선언하였는데, @Embedded를 사용해도 된다.
 - @SecondaryTables 는 자주 사용되지 않는다고 하는데, 유용하게 사용될 수 있을것 같다.

```java
@Getter
@NoArgsConstructor
@Entity
@Table(name="BOARD")
@SecondaryTables({
		@SecondaryTable(name="BOARD_DETAIL", pkJoinColumns = @PrimaryKeyJoinColumn(name="BOARD_DETAIL_ID")),
		@SecondaryTable(name="BOARD_FILE", pkJoinColumns = @PrimaryKeyJoinColumn(name="BOARD_FILE_ID"))
})
public class Board {
	
	@Id
	@GeneratedValue
	private Long id;
	
	private String title;
	
	@Column(table = "BOARD_DETAIL")
	private String content;
	
	@Column(table="BOARD_FILE")
	private String filename;
	
	@Column(table="BOARD_FILE")
	private String filepath;

	public void setTitle(String title) {
		this.title = title;
	}

	public void setContent(String content) {
		this.content = content;
	}

	public void setFilename(String filename) {
		this.filename = filename;
	}
	
	public void setFilepath(String filepath) {
		this.filepath = filepath;
	}
}
```

- embedded 사용

```java
    @Embedded
	private BoardFile boardFile;
...

@ToString
@Getter
@Setter
@NoArgsConstructor
@Embeddable
public class BoardFile {
	
	@Column(table="BOARD_FILE")
	private String filepath;
	
	@Column(table="BOARD_FILE")
	private String filename;
}

```

 - 아래와 같이 find 를 수행하면, 쿼리는 left 조인이 생성된다. 

```java
Board findBoard = em.find(Board.class, 1L);
System.out.println(findBoard);
```

```sql
select
        board0_.id as id1_0_0_,
        board0_.title as title2_0_0_,
        board0_1_.content as content1_1_0_,
        board0_2_.filename as filename1_2_0_,
        board0_2_.filepath as filepath2_2_0_ 
    from
        BOARD board0_ 
    left outer join
        BOARD_DETAIL board0_1_ 
            on board0_.id=board0_1_.BOARD_DETAIL_ID 
    left outer join
        BOARD_FILE board0_2_ 
            on board0_.id=board0_2_.BOARD_FILE_ID 
    where
        board0_.id=?
```

> Embedded를 List 타입으로 할때 & LAZY

 - 위 쿼리처럼 @SecondaryTable에 적용된 테이블은 left outer join 으로 한번에 쿼리가 수행된다. FetchType.LAZY를 적용하면, Board 테이블을 먼저 조회한 후, boardFiles가 참조되는 시점에 BOARD_FILE에 있는 데이터를 조회하는 쿼리가 수행된다. 

 - @SecondaryTable 는 주석처리 해야 한다. 주석처리 하지 않으면, FetchType.LAZY으로 선언해도 계속 left outer join이 수행된다.
 - Collection 은 필드에서 초기화 하는 것이 안전하다. 하이버네이트는 엔티티를 영속화 할 때, 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다. 컬렉션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생할 수 있다. 필드레벨에서 생성하는 것이 가장 안전하고 코드도 간결하다. 

```java
@ToString
@Getter
@NoArgsConstructor
@Entity
@Table(name="BOARD")
@SecondaryTables({
		@SecondaryTable(name="BOARD_DETAIL", pkJoinColumns = @PrimaryKeyJoinColumn(name="BOARD_DETAIL_ID"))
		//@SecondaryTable(name="BOARD_FILE", pkJoinColumns = @PrimaryKeyJoinColumn(name="BOARD_FILE_ID"))
})
public class Board {
	
	@Id
	@GeneratedValue
	private Long id;
	
	private String title;
	
	@Column(table = "BOARD_DETAIL")
	private String content;
	
	@ElementCollection(fetch = FetchType.LAZY)
    @CollectionTable(
            name = "BOARD_FILE",
            joinColumns = @JoinColumn(name = "BOARD_FILE_ID")
    )
    @OrderColumn(name = "IDX")
	private List<BoardFile> boardFiles = new ArrayList<>();

	public void setTitle(String title) {
		this.title = title;
	}

	public void setContent(String content) {
		this.content = content;
	}

	public void setBoardFiles(List<BoardFile> boardFiles) {
		this.boardFiles = boardFiles;
	}
}
```

## 연관관계 1 대 1 단방향

 - Member 와 MembershipCard 에 대한 1:1 관계

```java
@ToString
@Getter
@Setter
@NoArgsConstructor
@Entity
public class Member {
	
	@Id
	@GeneratedValue
    private Long id;
	
	private String name;
	
	public void setId(Long id) {
		this.id = id;
	}
	
	public void setName(String name) {
		this.name = name;
	}
}
``` 

 - EAGER 는 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행할 때 N+1 문제가 자주 발생한다. 
 - 실무에서 모든 연관관계는 지연로딩(LAZY)으로 설정해야 한다. 
 - 연관된 엔티티를 함께 DB에서 조회해야 하면, fetch join 또는 엔티티 그래프 기능을 사용한다. 

```java
@ToString
@Getter
@Entity
@NoArgsConstructor
@Table(name = "MEMBERSHIP_CARD", uniqueConstraints = {@UniqueConstraint(name="MEMBERSHIP_CARD_UNIQUE", columnNames = { "CARD_NUMBER" })})
public class MembershipCard {

	@Id
	@GeneratedValue
    private Long id;
	
	@Column(name = "CARD_NUMBER", nullable = false)
	private String cardNumber;
	
	@OneToOne(fetch = FetchType.LAZY)
	@JoinColumn(name="member_id")
	private Member member;
	
	public MembershipCard(String cardNumber, Member member) {
		this.cardNumber = cardNumber;
		this.member = member;
	}
}
```

 - unique 한 인덱스를 생성하려면, 위에처럼 제약사항으로 정의하거나 아래와 같은 속성을 정의한다. 

```java
@Table(name = "MEMBERSHIP_CARD", 
	   indexes = @Index(name="unique_idx_card_number", columnList = "CARD_NUMBER", unique = true)
)
```

## 연관관계 N대1, 1대N 양방향

 - Member 와 Team 은 서로 1대N , N대1의 양방향 구조를 가질수 있다. 
 - 양방향으로 관리하기 위해 단방향 2개를 만들어줬다.
 - 연관관계에서는 주인이라는 것이 있다. 연관관계의 주인만이 외래키를 관리할 수 있다. 외래키를 관리하는 테이블은 Member(team_id를 가지고 있음)이므로, Member 가 주인이고, 주인이 아닌쪽에 mappedBy를 작성한다. 
 - 주인만이 등록과 수정이 가능하다. 따랏거, `member1.setTeam(team);` 이 필요하다. 

```java
@ToString
@Getter
@Setter
@NoArgsConstructor
@Entity
public class Member {
	
	@Id
	@GeneratedValue
    private Long id;
	
	private String name;
	
	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;
	
	public void setId(Long id) {
		this.id = id;
	}
	
	public void setName(String name) {
		this.name = name;
	}
}
```

 - Team

```java
@ToString
@Setter
@Getter
@NoArgsConstructor
@Entity
public class Team {
	
	@Id
	@GeneratedValue
	private Long id;
	
	private String name;
	
	@OneToMany(mappedBy = "team")
	List<Member> member = new ArrayList<>();

}
```

 - Test Code 

```
	Team team = new Team();
	team.setName("인사팀");
	em.persist(team);
	
	Member member1 = new Member();
	member1.setName("이상국");
	member1.setTeam(team);
	em.persist(member1);
```

 - 아래와 같은 상황에서는 결과가 나오지 않는데, 그 이유는 DB에 반영이 안됐기 때문이다. 1차 캐시에 저장만 된 상태기 때문에 영속성 컨텍스트에서는 둘의 관계를 알지 못한다. 

```java

	Team team = new Team();
	team.setName("인사팀");
	em.persist(team);
	
	Member member1 = new Member();
	member1.setName("이상국");
	member1.setTeam(team); //  주인입장에서 연관관계 생성
	em.persist(member1);
	
	Member findMember = em.find(Member.class, member1.getId());
	List<Member> members = findMember.getTeam().getMember();
	
``` 

 - flush로 DB에 쿼리를 날려주고 캐시를 참고하지 않도록 clear 까지 해주면, members 는 DB에서 가져오게 된다.

```java

	Team team = new Team();
	team.setName("인사팀");
	em.persist(team);
	
	Member member1 = new Member();
	member1.setName("홍길동");
	member1.setTeam(team);
	em.persist(member1);
	
	Member member2 = new Member();
	member2.setName("이상국");
	member2.setTeam(team);
	em.persist(member2);
	
	em.flush();
	em.clear();
	
	Member findMember = em.find(Member.class, member1.getId());
	List<Member> members = findMember.getTeam().getMember();
	
	for(Member m : members) System.out.println(m.getName());

```

- 하지만 매번 flush 와 clear 를 하는것은 번거롭고 실수할 가능성이 많다. 아래와 같이 양쪽에 모두 적용을 시켜준다. 

```java

	Team team = new Team();
	team.setName("인사팀");
	em.persist(team);
	
	Member member1 = new Member();
	member1.setName("홍길동");
	member1.setTeam(team);
	em.persist(member1);
	
	Member member2 = new Member();
	member2.setName("이상국");
	member2.setTeam(team);
	em.persist(member2);
	
	team.getMember().add(member1);
	team.getMember().add(member2); //객체지향적 설계
	
//		em.flush();
//		em.clear();
	
	System.out.println("member1.getId()::" + member1.getId());
	
	Member findMember = em.find(Member.class, member1.getId());
	List<Member> members = findMember.getTeam().getMember();
	
	for(Member m : members) System.out.println(m.getName());

```

- 여기서 조금만 더 최적화를 하면, Member 엔티티에 아래와 같은 메서드를 추가한다. 

```java
public void changeTeam(Team team) {
	this.team = team;
	team.getMember().add(this);//연관관계 편의 메서드
}
```

 - 최종적으로 아래와 같은 테스트 코드

```java
Team team = new Team();
		team.setName("인사팀");
		em.persist(team);
		
		Member member1 = new Member();
		member1.setName("홍길동");
		member1.changeTeam(team); // 편의메서드 사용
		em.persist(member1);
		
		Member member2 = new Member();
		member2.setName("이상국");
		member2.changeTeam(team);
		em.persist(member2);
		
//		team.getMember().add(member1);
//		team.getMember().add(member2); //객체지향적 설계
		
//		em.flush();
//		em.clear();
		
		System.out.println("member1.getId()::" + member1.getId());
		
		Member findMember = em.find(Member.class, member1.getId());
		List<Member> members = findMember.getTeam().getMember();
		
		for(Member m : members) System.out.println(m.getName());
```


 - `em.find(Team.class, 1L);` 를 실행하면, 아래와 같은 쿼리가 수행된다. 

```sql
select
        team0_.id as id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        Team team0_ 
    where
        team0_.id=?
``` 

- `em.find(Member.class, 2L)` 를 실행하면, 아래와 같은 쿼리가 수행된다. 
- Member 가 주인이고, 팀에 대한 외래키를 가지고 있으므로, Team 정보까지 조회.
- 

```sql
select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_,
        member0_.TEAM_ID as team_id3_0_0_,
        team1_.id as id1_1_1_,
        team1_.name as name2_1_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.TEAM_ID=team1_.id 
    where
        member0_.id=?
```

 - `@ManyToOne(fetch = FetchType.LAZY)` 로 설정하면, 

```sql
select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_,
        member0_.TEAM_ID as team_id3_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
```


# Reference

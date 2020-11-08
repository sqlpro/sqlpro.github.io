---
title: Spring Bean(2) - 빈을 참조하는 N가지 방식
layout: post
date: '2020-01-06 10:19:00 +0000'
author: sqlpro
categories:
- 스프링
- 스프링부트
- Beans 
- Autowired 
- Bean 


---

이전 시간에는 스프링 컨테이너에서 빈(Bean)을 생성하는 방식에 대해 알아보았다. 이번 시간에는 생성된 빈을 참조하는 방법에 대해 알아보자.

# 빈을 참조하는 방식

빈을 참조하는 방식에는 크게 세 가지가 있다.

1. @Autowired 어노테이션을 사용하는 방식
2. ApplicationContext::getBean() 메소드를 사용하여 참조하는 방식
3. 생성자 주입 방식

### 1. @Autowired

빈을 참조하는 첫번째 방식은 @Autowired 어노테이션을 사용하는 것이다. 멤버 변수를 선언하고 @Autowired 어노테이션을 붙이면 스프링이 그 변수에 맞는 빈을 찾아 대입해 준다. 이를 전문 용어로 '의존성을 주입한다'라고 한다.

기본 매핑 룰은 객체의 타입이다. 해당 변수의 타입과 일치하는 빈을 찾아 매핑하는 것이 원칙이다. 동일한 타입이 여러 개 있을 경우에는 이름으로 매핑하거나 우선 순위를 부여하는 등 추가 룰을 적용할 수 있는데, 이에 대해서는 이 글의 후반부에 살펴볼 예정이다.

설명을 위해 다음과 같은 빈이 정의되어 있다고 가정한다.

```java
@Service
public class MemberService {

	public void int count() {
    int cnt = ... 
    return cnt; 
  }

}
```

`@Service` 어노테이션은 내부적으로 `@Component` 어노테이션을 품고 있기 때문에, `MemberService`는 빈으로 생성된다. `@Component` 외에 빈 생성시 사용할 수 있는 또다른 어노테이션으로는 `@Controller`, `@Configuration`, `@Repository` 등이 있다.

이 빈을 참조하는 구문은 아래와 같다.

```java
@Component
public class MemberComponent  {
  
  @Autowired 
  private MemberService memberService; // 'new MemberService()' 구문은 필요없다. 
  
  public List<Member> memberList() {
    int count = memberService.count();
  }
}
```

`MemberService` 타입으로 변수를 선언하고 `@Autowired`를 붙여 주었다. 그것으로 끝이다. 'new  XXX()' 같은 인스턴스 생성 구문은 필요하지 않다. 멤버 변수에 `@Autowired` 어노테이션이 붙어있으므로, `MemberComponent`가 생성될 때 의존성이 주입된다. 메소드 내에서는 원하는 대로 사용할 수 있다.

다만, 이 방식은 편리하긴 하지만 다음과 같은 제약이 있다.

1. 빈이 아닌 일반 클래스에서는 사용할 수 없다.
2. 멤버 변수에만 빈을 주입할 수 있다.

### 2. ApplicationContext::getBean()

방금 살펴본 것처럼, `@Autowired`는 빈 내에서만 사용이 가능하다. 만약 빈이 아닌 일반 객체에서 빈을 참조하려면 `getBean()` 메소드를 사용해야 한다. 이 메소드는 `ApplicationContext` 객체의 정적 메소드이므로 스프링 컨테이너 내 어디서든 사용할 수 있다. 사용 방법은 다음과 같다.

```java
public class MemberUtils() {

  private MemberService memberService = ApplicationContext.getBean(MemberService.class); 

  public boolean paging() {
    int cnt = memberService.count(); 
    ... 
  }
}
```

`MemberUtils`는 빈으로 선언되지 않았기 때문에 `@Autowired` 어노테이션을 사용할 수 없다. 그래서 대신 `ApplicationContext::getBean()` 메소드를 사용하고 있다.

매핑할 빈을 찾아 해당 변수에 직접 주입해주는 `@Autowired`와 달리, `getBean()` 메소드의 역할은 제한적이다. 인자로 전달된 타입과 일치하는 빈을 찾아 반환하기만 할 뿐이다. 위 예제에서는 `MemberService.class`를 인자값으로 전달하고 있으므로, 이 타입과 일치하는 빈인 `MemberService`가 반환된다.

이 방식의 장점은 빈을 멤버 변수에 할당할 필요가 없다는 것이다. 즉, 다음과 같은 방식으로도 사용 가능하다.

```java
public class MemberUtils() {

	public boolean paging() {
    MemberService memberService = ApplicationContext.getBean(MemberService.class);
    int count = memberService.count();
    ... 
  }
}
```

### 3. 생성자 주입 방식

앞에서 알아본 두 가지 방식은 사실 그리 추천되지 않는다. `getBean()` 메소드는 어쨌거나 빈을 직접 변수에 대입한다는 점에서 `의존성 주입`이라는 스프링의 원칙에 반하는 면이 있으며, `@Autowired`는 두 개의 빈 객체 사이에 상호 참조가 발생하는 경우에 문제가 발생하기 때문이다(아래와 같이 빈 A와 빈 B가 서로 @Autowired로 참조하는 경우가 이에 해당한다)

```java
// BeanA.java 
@Component 
public class BeanA {
  @Autowired 
  private BeanB b; 
}

// BeanB.java 
@Component 
public class BeanB {
  @Autowired 
  private BeanA a; 
}
```

이 때문에, 최근에는 생성자의 인자값으로 빈을 주입하는 방식이 추천되고 있다. 예제를 살펴보자.

```java
@Component 
public class MemberUtils {

  private final MemberService memberService; 
 
  @Autowired
  public MemberUtils(MemberService memberService) {
    this.memberService = memberService;
  } 
}
```

앞서의 예제에서 `memberService` 변수에 붙어 있던 `@Autowired` 어노테이션을 제거하고, 대신 생성자에다 `@Autowired`를 정의하였다. 이를 통해 스프링은 해당 클래스의 인스턴스 실행시 자동으로 타입에 맞는 빈을 찾아 생성자의 인자로 전달한다. 따라서 빈이 생성되면 `memberService` 변수에도 자동으로 빈이 주입된다.

이 구조에서 빈을 주입할 멤버 변수에 final 키워드를 붙이는 것은 좋은 습관이다. 의도치 않은 참조 변경을 방지할 수 있기 때문이다. final 키워드를 붙였더라도 생성자 구문에서 해당 변수를 초기화하면 오류는 발생하지 않는다. 앞서 `@Autowired` 방식에서는 사용할 수 없는, 생성자 주입 방식에서만 허용되는 구문 특성이라고 할 수 있다.

**@RequiredArgsConstructor**

생성자 주입 방식은 클래스 내에서 참조할 빈이 많아지면 생성자 구문이 지저분해진다는 단점이 있다. 예를 들어 해당 클래스 내에서 다섯 개 정도의 빈을 참조해야 한다고 가정해보자. 빈 주입을 위해 우리는 다음과 같은 코드를 작성해야 한다.

```java
@Component 
public class MemberUtils {
 
  private final MemberService memberService; 
  private final BillingService billingService;
  private final MonitoringService monitoringService;
  private final RabbitMQTemplate rabbitmqTemplate; 
  private final RestTemplate restTemplate; 

  public MemberUtils(MemberService memberService, 
                     BillingService billingService, 
                     MonitoringService monitoringService,
                     RabbitMQTemplate rabbitmqTemplate,
                     RestTemplate restTemplate) {

    this.memberService = memberService;
    this.billingService = billingService;
    this.monitoringService = monitoringService;
    this.rabbitmqTemplate = rabbitmqTemplate;
    this.restTemplate = restTemplate; 
  }

}
```

클래스 내에서 사용해야 할 빈이 늘어날 수록 생성자 구문도 점점 복잡해진다. 만약 롬복(Lombok)을 사용하고 있다면, `@RequiredArgsConstructor` **어노테이션을 사용해서** 이 문제를 깔끔하게 처리할 수 있다.

```java
@Component 
@RequiredArgsConstructor
public class MemberUtils {
  
  private final MemberService memberService; 
  private final BillingService billingService;
  private final MonitoringService monitoringService;
  private final RabbitMQTemplate rabbitmqTemplate; 
  private final RestTemplate restTemplate; 

}
```

`@RequiredArgsConstructor` 어노테이션은 final 키워드가 붙은 멤버 변수를 인자로 하는 생성자 구문을 자동으로 생성해 준다. 여기서 우리가 할 것은 다음 두 가지 뿐이다.

1. 빈을 주입할 멤버 변수를 final로 선언한다.
2. 클래스 상단에 `@RequiredArgsConstructor`를 붙여준다.

나머지는 모두 롬복이 알아서 처리한다. 필요한 코드는 컴파일 타임에 생성되므로 눈으로 직접 확인하기는 어렵겠지만, 방금 전에 작성했던 생성자와 동일한 코드를 작성해 준다고 생각하면 될 것이다.

결론적으로, 생성자 주입 방식을 사용할 때에는 `@RequiredArgsConstructor` 어노테이션을 힘께 사용하는 것을 권장한다.

## 같은 타입의 빈이 두 개 이상일 때 참조 지정하기

지금까지는 각 타입별 빈이 유일한 경우에 대해 알아보았다. 그러나 필요에 따라서는 동일 타입의 빈이 두 개 이상 정의되어야 하는 경우가 존재한다. 대표적인 것이 DB 연결이다.

MySQL 기반의 대용량 애플리케이션 구축시, 부하 분산을 목적으로 Master DB와 Slave DB를 구분해서 사용하기도 한다. Master에는 쓰기 요청만, Slave에는 읽기 요청만 처리하도록 하는 것이다. 대다수의 서비스 부하는 데이터를 읽는 쪽에서 발생하므로, 위와 같이 구성해 두면 부하가 증가하더라도 Slave Replication 사이즈를 늘려 대처 가능하다는 장점이 있다. 다음 예제를 보자.

```java
@Configuration 
public class MyBatisConfig {
  @Bean 
  public SqlSessionFactory masterSqlSessionFactory() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(masterDataSource());
    return factoryBean.getObject();
  }

  @Bean 
  public SqlSessionFactory slaveSqlSessionFactory() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(slaveDataSource());
    return factoryBean.getObject(); 
  }
}
```

MyBatis의 핵심 객체인 SqlSessionFactory 빈을 정의하는 구문이다. 동일한 빈 두 개를 정의하여 하나는 마스터 DB 쪽을, 또다른 하나는 슬레이브 DB 쪽을 담당하도록 설정했다. 이 경우 타입이 중복되기 때문에, 아래와 같이 작성하면 스프링 컨테이너는 둘 중 어느 타입을 선택해야 할지 판단할 수 없어 오류가 발생한다.

```java
@Repository 
public MyBatisRepository {
  @Autowired 
  private SqlSessionFactory factory; 
}
```

이를 해결하기 위한 방법을 알아보자.

#### 4.1 이름으로 구분하기

동일한 타입의 빈이 두 개 이상일 때, 참조를 지정하기 위해 가장 쉽게 선택할 수 있는 전략은 빈에 이름을 부여하는 것이다. 이렇게 하면 객체의 타입 뿐만 아니라 이름으로도 구분이 가능하므로 원하는 빈을 적절히 선택할 수 있다. 빈에 이름을 정의하는 방법은 다음과 같다.

```java
@Repository("mybatisRepo")
public MyBatisRepository {
  @Autowired 
  private SqlSessionFactory factory; 
}
```

`MyBatisRepository` 빈을 정의하면서, 어노테이션의 인자값으로 "mybatisRepo"를 입력했다. 이제 이 빈의 이름은 mybatisRepo이므로, 스프링 컨테이너에서는 `MyBatisRepository`라는 타입 또는 "mybatisRepo"라는 이름으로 해당 객체를 읽어올 수 있다. 읽어 오는 방법은 잠시 후에 알아보기로 하고, 이름을 지정하는 또다른 경우를 살펴보자.

```java
@Configuration 
public class MyBatisConfig {
  @Bean("master") 
  public SqlSessionFactory masterSqlSessionFactory() throws Exception {
    ...
  }

  @Bean("slave") 
  public SqlSessionFactory slaveSqlSessionFactory() throws Exception {
    ...
  }
}
```

위 예제는 @Bean 메소드를 통해 빈을 생성하는 방식이다. 클래스 자체를 빈으로 선언하지 않고 위와 같이 생성 메소드를 사용하여 빈을 정의하는 경우에 얻을 수 있는 장점은, 원본 클래스가 빈이 아닌 경우에도 임의로 빈을 만들어 낼 수 있다는 것이다. 주로 라이브러리에 포함되어 있는 등 수정이 불가능한 클래스를 빈으로 다루어야 할 때 사용된다.

이 방식에서는 `@Bean(<빈 이름>)`으로 빈의 이름을 지정할 수 있다. 주어진 예제에서는 두 개의 빈을 각각 `master`, `slave`로 지정하고 있다. 이때, 각각의 빈은 @Qualifier 어노테이션 설정을 통해 참조할 수 있다.

```java
@Component
public class MemberService {
  
  @Autowired
  @Qualifier("master")
  private SqlSessionFactory masterFactory; 

  @Autowired
  @Qualifier("slave")
  private SqlSessionFactory slaveFactory; 

}
```

@Qualifier 어노테이션을 사용하기가 번거롭다면, 생략하는 대신 멤버 변수 명을 빈 이름과 일치시켜도 같은 결과를 얻을 수 있다. 다음 구문은 @Qualifier를 사용한 바로 위의 구문과 결과가 동일하다.

```java
@Component
public class MemberService {
  
  @Autowired
  private SqlSessionFactory masterSqlSessionFactory; 

  @Autowired
  private SqlSessionFactory slaveSqlSessionFactory; 

}
```

@Bean 메소드 형식을 사용했다면, 메소드 이름으로 빈 이름을 대신할 수 있다. 아래와 같이 작성했을 때 각각의 빈의 이름은 `master` ,  `slave`로 세팅된다.

```java
@Configuration 
public class MyBatisConfig {
  @Bean 
  public SqlSessionFactory master() throws Exception {
    ...
  }

  @Bean 
  public SqlSessionFactory slave() throws Exception {
    ...
  }
}
```

조금 더 발전시켜 보자. 앞서, 멤버 변수 명을 빈의 이름과 동일하게 선언하면 원하는 빈을 선택할 수 있다고 설명한 바 있다. 이에 더하여 빈 이름을 메소드 이름으로 대신할 수 있다는 것도 알았으므로, 이 두 가지를 합하면 다음과 같은 코드가 가능하다.

```java
// MybatisConfig.java 
@Configuration 
public class MyBatisConfig {
  @Bean 
  public SqlSessionFactory master() throws Exception {
    ...
  }

  @Bean 
  public SqlSessionFactory slave() throws Exception {
    ...
  }
}

// MemberService.java 
@Component 
public class MemberService {

  @Autowired SqlSessionFactory master; 
  @Autowired SqlSessionFactory slave; 
}
```

이 방식은 생성자 주입 방식에서도 동일하게 적용할 수 있다. `MemberService` 를 생성자 주입 방식으로 변경한 결과는 다음과 같다.

```java
// MemberService.java 
@Component 
@RequiredArgsConstructor
public class MemberService {

  @Autowired final SqlSessionFactory master; 
  @Autowired final SqlSessionFactory slave; 
}
```

final로 선언된 각각의 멤버 변수는 동일한 이름의 빈 생성 메소드에 매핑된다. 즉, master 변수에는 master() 메소드가, slave 변수에는 slave() 메소드가 매핑되는 것이다. 만약 빈 생성시 이름이 지정되어 있다면 마찬가지로 그 이름과 변수명을 동일하게 맞추어 주면 된다. 다만 이 방식에서는 멤버 변수에 @Qualifier 어노테이션이 적용되지 않으므로 주의해야 한다.

#### 4.2 @Primary

같은 타입의 빈이 두 개 이상이라면 @Primary 어노테이션을 통해 우선 순위 빈을 지정할 수 있다. 사용 방법은 간단하다. 빈을 정의할 때, 빈 중에서 가장 우선 순위가 높은 하나에 @Primary 어노테이션을 걸어주면 된다. 이름이 지정되지 않은 상태에서 빈을 참조하면 우선 순위가 높은, 다시 말해 @Primary가 지정된 빈이 참조된다.

```java
@Configuration 
public class MyBatisConfig {
  @Bean 
  @Primary // 이름 생략시 이 빈이 우선적으로 참조된다.
  public SqlSessionFactory masterSqlSessionFactory throws Exception {
    ...
  }

  @Bean // 이 빈은 명시적으로 이름이 지정될 때에만 참조된다. 
  public SqlSessionFactory slaveSqlSessionFactory throws Exception {
    ...
  }
}
```

`@Component`, `@Service`등의 어노테이션을 사용해서 빈을 정의할 때에도 사용법은 동일하다. 클래스 선언부에 명시해주면 된다.

```java
@Component
@Primary
public class MemberService {

	public void int count() {
    int cnt = ... 
    return cnt; 
  }
}
```


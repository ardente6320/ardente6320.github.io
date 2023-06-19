---
title:  "DI(의존성 주입)"
date:   2023-06-19 22:30:02+0900
categories: [Spring Boot]
tags: [SpringBoot, DI, Dependency Injection]
---

<br>

Spring의 핵심 기술중 하나인 DI(의존성 주입)에 대해 알아보자.

<br>

#### **<span style="color:#ef5369">필드 주입(Field Injection)</span>**

필드 주입은 필드에서 바로 주입하는 방법이다. 

과거에 많이 이용되었지만 외부에서 접근이 불가능하고, 테스트코드 작성 시 필드의 객체를 수정할 수 없게 되어 사용하지 않게 되었다.

즉 강한 결합으로 인해 외부에서 사용하기가 어려워진다.

```java
//Field Injection
public class MyController{
     @Autowired
     private MyService myService;
}
```
---

#### **<span style="color:#ef5369">수정자 주입(Setter Injection)</span>**

수정자 주입은 Setter Method을 통해 주입하는 방법이다. 

주입 받는 객체가 변경될 가능성이 있는 경우에 사용한다.

의존관계를 나타낼 수 있으나, 필수적으로 주입되어야 할 항목들을 빼먹어 null 처리될 수가 있다.

```java
//Setter Injection
public class MyController{
    private MyService myService;
    
    @Autowired
    public void setMyService(MyService myService){
    	this.myService = myService;
    }
}
```

---
#### **<span style="color:#ef5369">생성자 주입(Constructor Injection)</span>**

생성자에서 주입하는 방법이다. 최근 Spring에서 권장하는 방법인데 그 이유는 다음과 같다.


**<span style="color:#ef5369">1. 객체의 불변성 확보</span>**

생성자 주입을 통해 변경 가능성을 배제하고 불변성을 보장한다


**<span style="color:#ef5369">2. 테스트 코드 작성 용이</span>**

생성자 주입이 아닌 다른 주입으로 작성된 코드는 순수 자바 코드로 테스트 작성하는 것이 어렵다.

다음 코드는 필드 주입을 했을 때 나타나는 테스트 코드이다.

```java
//Field Injection
public class MyService{
     @Autowired
     private MyRepository myRepository;
     
     public User findById(String id){
     	return myRepository.findById(id);
     }
}
```

해당 클래스의 테스트 코드는 다음과 같다.

```java
public class MyServiceTest{
	
    @Test
    public void findByIdTest(){
    	//Given
        MyService myService = new MyService();
        String id = "testId";
        
        //When
        User user = myService.findById(id);
        
        //Then
        assertEquals(user.getId(),id);
    }
}
```

위와 같이 작성한 경우 DI 프레임워크 위에서 동작하지 않기 때문에 의존 관계가 주입되지 않아 의존성 주입 대상인 `myRepository`가 `null`이 되어 `findById`를 실행할 때 `NPE`가 발생한다. 

수정자 주입을 통해 처리할 수는 있지만, 변경 가능성을 열어두기 때문에 단점을 갖게 된다.

만약 Spring Bean을 올려서 하기위해서는 MockBean으로 생성해서 하거나 SpringBootTest를 통해 전체 테스트를 해야하는데 이것은 단위테스트가 아니기 때문에, 생성자 주입을 사용한다.

또한 생성자 주입을 사용했을 때 컴파일 시점에서 오류를 발견할 수 있다.


**<span style="color:#ef5369">3. Final 키워드 작성</span>**

생성자 주입을 하게 되면 final 키워드를 사용할 수 있고, 컴파일 단에서 누락된 부분을 확인 할 수 있다.

Spring에서 생성자가 1개인 경우에는 `@Autowired`를 생략해도 된다.

또는 Lombok의 `@RequiredArgsConstructor`을 활용해도 된다.

따라서 다음과 같이 작성이 가능하다. 다음 3개는 동일한 동작을 하게 된다.

```java
//Constructor Injection
public class MyController{
    private final MyService myService;
    
    @Autowired
    public MyController(MyService myService){
    	this.myService = myService;
    }
}
```

```java
//Constructor Injection
public class MyController{
    private final MyService myService;
    
    public MyController(MyService myService){
    	this.myService = myService;
    }
}
```

```java
//Constructor Injection
@RequiredArgsConstructor
public class MyController{
    private final MyService myService;
}
```


**<span style="color:#ef5369">4. 순환 참조 방지</span>**

주입하려는 대상이 서로 마주보고 있어 계속해서 참조하고 생성하는 것을 순환 참조라고 한다.

```java
@Component
public class MyServiceA{
    @Autowired
    private MyServiceB myServiceB;
    
    public void startB(){
        myServiceB.startA();
    }
}

@Component
public class MyServiceB{
    @Autowired
    private MyServiceA myServiceA;
    
    public void startA(){
        myServiceA.startB();
    }
}
```
위와 같은 상황이 발생하게 되면 `MyServiceA` 가 `MyServiceB`를 호출하고 `MyServiceB`가 `MyServiceA`를 호출하게되는데 

해당 객체를 필드 주입 또는 수정자 주입을 통해하게 되면 

런타임 중에 계속적으로 `new` 하게 되어 `StackOverFlow`가 발생하게 된다.

```
Caused by: java.lang.StackOverflowError: null
```

하지만 생성자 주입을 통해 생성하게 되면 다음과 같이 나타난다.

아래와 같이 하게 되면 객체가 생성될때 주입을 하게되기 때문에, 애플리케이션 컴파일 시점에서 순환 참조를 알아차릴 수 있다.

```java
@Component
@RequiredArgsConstructor
public class MyServiceA{
    private final MyServiceB myServiceB;
    
    public void startB(){
        myServiceB.startA();
    }
}

@Component
@RequiredArgsConstructor
public class MyServiceB{
    private final MyServiceA myServiceA;
    
    public void startA(){
        myServiceA.startB();
    }
}
```
```
Description: 
The dependencies of some of the beans in the application context form a cycle: 
┌─────┐ 
| myServiceA defined in file [.....class]
  ↑ ↓ 
| myServiceB defined in file [.....class]
└─────┘
```

---

#### **<span style="color:#ef5369">요약</span>**

DI(의존성 주입)을 할때 필드 주입, 수정자 주입 보다 생성자 주입을 하여 사용하여서, 순환 참조를 방지하고,

테스트 코드를 더욱 간편하게 작성할 수 있다.
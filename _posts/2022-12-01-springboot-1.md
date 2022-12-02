---
title:  "Test Case 생성하기"
date:   2022-12-01 23:59:20
categories: [Spring Boot]
tags: [SpringBoot, TestCase, JUnit]
---
<br>

Spring Boot에서 단위 테스트를 생성할 수 있다.

테스트를 하기 위해선 **JUnit 모듈**을 사용하여 테스트를 한다.

<br>

#### **<span style="color:#ef5369">단위 테스트(Unit Test)란?</span>**

소스코드의 특정 모듈이 의도된 대로 정확히 작동하는지 검증하는 절차다.

즉, 모든 함수와 메소드에 대한 테스트 케이스(Test case)를 작성하는 절차를 말한다.

<br>

#### **<span style="color:#ef5369">테스트 코드 작성하는 이유</span>**

##### **<span style="color:#ef5369">1. 빠른 피드백이 가능해진다.</span>**

- 테스트 케이스를 통해 오류 검출을 빠르게 하여 수정할 수 있다.

- 사전 오류를 검출할 수 있다.
 

##### **<span style="color:#ef5369">2. 리팩토링의 두려움이 없어진다.</span>**

- 검증된 테스트 케이스가 있다면 소스 코드를 변경하는데 무리가 없다.

---

#### **<span style="color:#ef5369">단위 테스트(Unit Test)</span>**

테스트는 기본적으로 Given, When, Then 방식을 사용한다.

- **Given : 주어진 환경**

- **When : 가정된 상황**

- **Then : 결과**

이렇게 세가지로 분류하게 되면 테스트는 굳이 복잡하게 생각할 필요가 없어진다.

코드를 작성하기 위해서 구문을 이해해야한다.


- **@RunWith**

JUnit에 내장된 실행자 외에 다른 실행자를 실행시키는데, 여기선 SpringRunner 실행자를 사용한다.

즉, 스프링 부터 테스트와 JUnit 사이에 연결자 역할을 한다.

- **@Autowired**

스프링이 관리하는 빈을 주입받는다.

- **@Test**

실제 테스트를 진행할 메소드를 나타낸다.
 
<br>

우선 간단한 예제를 통해 확인해보자.

##### **<span style="color:#ef5369">서비스 메소드 테스트</span>**

```java
@RunWith(SpringRunner.class)
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class Tests {

    @Autowired
    private TestService testService;

    /** 문자열 자르기 테스트  */
    @Test
    public void testSubStr() {
        //Given
        String givenStr = "This is Test"; //주어진 문자열
        String expected = "Test";         //예상 결과

        //When
        String result = testService.subStr(givenStr,8,givenStr.length()); //실제 결과

        //Then
        assertEquals(expected,result);
    }
}
```

코드 상에서 `"This is Test"` 라는 문자열을 잘라 `"Test"`라는 결과를 도출할 것이다 라고 가정했다.

그리고 `assertEquals`라는 메소드를 통해 실제 값과 비교한 결과를 나타낸다.

성공하면 Success라고 나타나고, 실패하면 왜 실패하였는지 이유를 나타낸다.

<br>

#### **<span style="color:#ef5369">Controller API 테스트</span>**

API 테스트를 하려면 다음 구문을 이해해야한다.

- **@WebMvcTest**
 
여러 스프링 테스트 어노테이션 중, Web에 집중할 수 있는 어노테이션이다.

이를 호출할 경우 @Controller, @ControllerAdvice 등을 사용할 수 있다.

- **MockMvc mvc**

웹API를 테스트할 때 사용한다.

이 클래스를 통해 HTTP GET, POST등에 대한 API 테스트를 할 수 있다.

- **mvc.perform(get("/test"))**

MockMvc를 통해 /hello 주소로 HTTP GET 요청을 한다.

- **andExpect(status().isOk())**

mvc.perform의 결과를 검증한다.

isOk()는 status가 200인지 아닌지를 검증한다.

- **andExpect(content().string(hello))**

응답 본분의 내용을 검증한다.

Controller에서 "hello"를 정상적으로 리턴하는지 검증한다.

```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void helloTest() throws Exception {
    	//Given
        String hello = "hello";

        //When And Then
        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}
```

예시 코드와 같이 API 또한 테스트할 수 있어 보다 안전하고 강력한 API를 개발하는데 도움을 줄 수 있다.

---


테스트 코드 작성을 귀찮아하면 안될 것 같다. 

여러가지 케이스를 생각하고, 작성을 하다보면 언젠가 방대하고 검증된 테스트 코드를 개발할 수 있고, 

조금 더 직관적인 코드를 작성하는데 큰 도움을 줄 수 있다.

앞으로 테스트 코드 작성하는 습관을 기르도록 노력해야겠다. 
---
title:  "Controller가 수많은 Request 처리하는 방법"
date:   2022-12-13 21:33:57
categories: [Spring Boot]
tags: [SpringBoot, Controller]
---
<br>

![main-img](/images/assets/springboot-3_controller_main.png)

**`Controller`**의 역할은 Model 객체를 만들어 데이터를 담고 View를 찾고, 

**`RestController`**는 객체만 반환하고 객체 데이터는 Http Response에 담아 전송한다.

<br>

우선 **`Controller`** 객체(Bean)은 **싱글톤(Singleton)**으로 생성된다. 

<br>

**`Controller`** 객체가 생성되면 **`JVM(Java Virtual Machine)`**의 **`Heap Area`**에 생성되며, 

해당 클래스의 정보는 **`Method Area`**에 생성된다.

---

#### **<span style="color:#ef5369">JVM 영역 참고</span>**

**<span style="color:#ef5369">1. Heap Area</span>**

new 키워드로 생성된 객체와 배열이 저장되는 영역

Method Area에 로드된 클래스만 생성이 가능

효율적인 GC를 위해 메모리 영역 분리

런타임 시 할당

**<span style="color:#ef5369">2. Method Area</span>**

클래스 정보(맴버 변수의 이름), 변수 정보(데이터 타입, 접근제어자 정보),

메소드 정보(메소드 이름, 반환 타입, 파라미버, 접근제어자 정보), static 변수, final class 변수,

Constant pool(싱수 풀 : 문자 상수, 타입, 필드, 객체참조가 저장됨)등을 분류해서 저장

JVM이 동작해서 클래스가 로딩될 때 생성

---

스레드간의 정보(상태)를 공유하기 위해서는 **동기화**가 필요하다.

<br>

하지만 **`Controller`**는 정보(상태)가 없다.(**stateless**)

내부적으로 상태를 가지고 있지않아 스레드 간의 동기화가 필요가 없어진다.

Request가 들어오면 메소드를 공유하며 사용하게 된다.

상태를 가지는게 없으니 메소드만 호출하게된다. 말그대로 처리 로직만 공유를 하게된다.

<br>

결론적으로 **싱글톤(Singleton)**으로 관리되는 객체의 경우 정보(상태)를 갖지 않기 때문에 

요청의 수와 상관없이 싱글턴 객체의 장점을 이용할 수 있다.
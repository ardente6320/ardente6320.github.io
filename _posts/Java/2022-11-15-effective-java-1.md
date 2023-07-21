---
title:  "Effective Java-ITEM 1 정적 팩토리 메소드"
date:   2022-11-15 01:20:23+0900
categories: [Java]
tags: [Java, Effective Java, Factory Method]
---
<br>

#### **<span style="color:#ef5369">전통적인 인스턴스 생성 방법</span>**

<br>

클래스의 인스턴스 생성 방식 중에 흔히 사용 하는 것이 public 생성자이다.

생성자를 통해 인스턴스를 생성하여 사용하게 되면 어떤 것을 반환하고자 하는지 찾기가 모호해질 수 있다.

```java
class Pizza{
private final String cheese;
	private final String topping;
	private final int size;

	public Pizza(String cheese, String topping, int size){
		this.cheese = cheese;
		this.topping = topping;
		this.size = size;
	}
    ...
}
```

위의 코드를 보면 Pizza 인스턴스를 생성하려면 다음과 같이 생성해야한다.

```java
Pizza cheesePizza = new Pizza("체다치즈","토마토소스",10);
```

만약 파라미터가 더욱더 많아지게 된다면 해당 순서의 파라미터가 무슨 일을 하는 것인지 구분하기가 어려워지게 되고, 개발자 또한 실수를 범하게 될수 있다.

---

#### **<span style="color:#ef5369">정적 팩토리 메소드</span>**

<br>

정적 팩토리 메소드는 객체를 생성을 담당하는 클래스 메소드이다.

전통적인 생성 방식과 다른 점은 `new`를 할때 직접적으로 하는 것이 아닌 메소드를 통해 간접적으로 생성하게 된다.

```java
//생성자 방식
String constructorStr = new String("Constructor");

//정적 팩토리 메소드 방식
String factoryStr = String.valueOf("Factory");
```

이런 정적 팩토리 메소드는 다음과 같은 장단점을 가지게 된다.

---

#### **<span style="color:#ef5369">장점</span>**

<br>

##### **<span style="color:#ef5369">1. 이름을 가질 수 있다.</span>**


생성자의 파라미터로는 해당 인스턴스가 어떤 역할을 하는지 특성을 제대로 설명하지 못한다.

반면 정적 팩토리 메소드는 네이밍만 올바르게 한다면 반환할 인스턴스의 특정을 제대로 설명할 수 있다.

```java
class Pizza{
	private final String cheese;
	private final String topping;
	private final int size;

	public Pizza(String cheese, String topping, int size){
		this.cheese = cheese;
		this.topping = topping;
		this.size = size;
	}

	public static Pizza cheesePizza(int size){
		return new Pizza("체다치즈", "토마토소스", size);
	}

	public static Pizza peperoniPizza(int size){
		return new Pizza("기본치즈", "페퍼로니", size);
	}
}

//생성자 방식
Pizza cheesePizza = new Pizza("체다치즈","토마토소스",10);

//정적 팩토리 메소드
Pizza cheesePizza = Pizza.cheesePizza(10);
```

<br>

##### **<span style="color:#ef5369">2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.</span>**


정적 팩토리 메소드를 사용하게 되면 미리 만들어진 인스턴스를 캐싱하여 재활용하는 식으로 사용하여 무분별한 객체 생성을 막을 수있다.

```java
Boolean value = Boolean.valueOf(true);
```

위와 같이 Boolean 객체를 new 하여 인스턴스화하지 않고 받아올 수 있다.

이렇게 미리 만들어진 객체를 사용하게 되면 인스턴스가 단 하나 뿐임을 보장할 수 있다.

<br>

#### **<span style="color:#ef5369">3. 반환 타입의 하위 타입 인스턴스를 반환 할수 있다.</span>**


이 능력으로 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 제공한다.

```java
class Mobility{
	public static Mobility of(String type){
		if(type.equalsIgnoreCase("car")){
			return new Car();
		}else if(type.equalsIgnoreCase("bike")){
			return new Bike();
		}
		...
	}
}
```

위 클래스를 보면 of 메소드를 통해 type 별로 필요한 모빌리티를 반환해준다.
 
<br>

##### **<span style="color:#ef5369">4. 입력 파라미터에 따라 매번 다른 클래스의 인스턴스를 반환한다.</span>**


반환 타입의 하위 타입이기만 하면 어떤 클래스를 반환하든 상관없다.

Java의 EnumSet클래스가 그 예가 될 수 있다.

아래 코드를 보면 파라미터 길이에 따라 다른 클래스를 반환하고 있다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
	Enum<?>[] universe = getUniverse(elementType);
	if (universe == null)
		throw new ClassCastException(elementType + " not an enum");

	if (universe.length <= 64)
		return new RegularEnumSet<>(elementType, universe);
	else
		return new JumboEnumSet<>(elementType, universe);
}
```

<br>

##### **<span style="color:#ef5369">5. 정적 팩토리 메소드를 작정하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.</span>**


이 유연함을 통해 서비스 제공자 프레임워크를 만들 수 있었다.

클래스가 존재해야 생성자가 존재할 수 있지만, 정적 팩토리 메소드는 메소드와 반환할 타입만 정해두고 실제 반환될 클래스를 나중에 구현하는게 가능하다.

```java
private Class<?> serviceClass(){
	return Class.forName("com.test.services.TestService");
}
```

---

#### **<span style="color:#ef5369">단점</span>**

<br>

##### **<span style="color:#ef5369">1. 정적 팩토리 메소드만 제공하는 경우에는 상속이 불가능하다.</span>**


상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메소드만 제공하면 하위 클래스를 생성할 수 없다.

<br>

##### **<span style="color:#ef5369">2. 개발자가 찾기 어렵다.</span>**


API설명에 명확히 드러나지 않다보니 다른 개발자가 찾기 어렵다.

하지만 문서화를 잘하고, 메소드 네이밍을 널리 알려진 규약으로 사용한다면 문제를 완화할 수 있다.

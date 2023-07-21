---
title:  "Effective Java-ITEM 2 Builder"
date:   2022-11-25 14:26:15+0900
categories: [Java]
tags: [Java, Effective Java, Builder]
---
<br>

#### **<span style="color:#ef5369">Builder 이전의 생성 방식</span>**

<br>

##### **<span style="color:#ef5369">점층적 생성자 패턴</span>**


정적 팩토리 메소드나 생성자의 경우 파라미터가 많은 경우 적절히 대응하기가 쉽지 않다.

다음은 생성자 또는 정적 팩토리 메소드에서 사용했던 **<span style="color:#ef5369">점층적 생성자 패턴</span>**이다.

```java
public class Car{
    private final int type;
    private final int doors;
    private final int oilType;
    
    public Car(int type){
        this(type, 4);
    }
    
    public Car(int type, int doors){
        this(type, doors, 0);
    }
    
    public Car(int type, int doors, int oilType){
        this.doors = doors;
        this.type = type;
        this.oilType = oilType;
    }
}
```

해당 클래스의 인스턴스를 생성할 때 필요한 파라미터를 포함한 생성자를 선택해서 호출하면 된다.

점층적 생성자 패턴은 불필요한 변수를 내부에서 설정하기 때문에 그리 나빠보이진 않겠지만, 파라미터가 늘어나면 클라이언트 코드를 작성하거나 읽기 어려워진다.

몇번째 변수가 어떤 값에 해당하는지 주의해야하고 원하는 생성자를 생성했는지 파라미터의 개수도 세어보아야한다.

<br>

##### **<span style="color:#ef5369">자바빈즈 패턴</span>**


선택해야할 파라미터가 많은 경우 활용할 수 있는 **<span style="color:#ef5369">자바빈즈 패턴</span>**이 있다.

파라미터가 없는 생성자를 생성 후, Setter 메소드를 호출하여 원하는 값을 직접 설정하는 방식이다.

```java
public Car{
    private int type;
    private int doors = 4; //필수값 설정
    private int oilType;
    
    public Car(){}
    
    public void setType(int type){
        this.type = type;
    }
    
    public void setDoors(int doors){
        this.doors = doors;
    }
    
    public void setOilType(int oilType){
        this.oilType = oilType;
    }
    
    public int getType(){
        return type;
    }
    public int getDoors(){
        return doors;
    }
    public int getOilType(){
        return oilType;
    }
}
```

Setter메소드를 활용하게 되면 점층적 생성자 패턴에서 나타나던 단점이 사라진다.

직접 Setter를 호출하면서 필요한 값을 설정하면 되기 때문에 어떤 값을 설정하는지 혼동되지 않는다.

하지만 객체 하나를 만들기 위해선 메소드를 여러번 호출해야하고, 객체가 생성되기 전까지는 일관성이 무너진 상태에 처하게 된다.(런타임에 버그로 나타날 수 있다.) 

또한 자바빈즈 패턴은 클래스를 불변하게 만들 수 없다.

---

#### **<span style="color:#ef5369">빌더 패턴</span>**

<br>

빌더 패턴의 경우 **<span style="color:#ef5369">점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성</span>**을 가졌다.


사용 방법은 다음과 같다.


**<span style="color:#ef5369">1. 필수 파라미터만으로 생성자 또는 정적 팩토리 메소드를 호출해 빌더 객체를 얻는다.</span>**

**<span style="color:#ef5369">2. 빌더 객체가 제공하는 일종의 Setter 메소드로 원하는 파라미터를 설정한다.</span>**

**<span style="color:#ef5369">3. 파라미터가 없는 build()를 호출하여 필요한 객체를 얻는다.(보통 불변 객체이다.)</span>**


빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는게 보통이다.

```java
//import 있다고 가정
enum CarType{SEDAN,SUV,CUV,SPORT,TRUCK}
enum OilType{GASOLINE,DIESEL,LPG}
public class Car{
    private final CarType type;
    private final int doors;
    private final OilType oilType;

    public static class Builder{
        //필수 파라미터
        private final CarType type;

        //선택 파라미터의 경우 Default 값으로 셋팅
        private int doors = 4;
        private OilType oilType = OilType.GASOLINE;

        public Builder(CarType type){
            this.type = type;
        }

        public Builder doors(int val){
            this.doors = val;

            return this;
        }

        public Builder oilType(OilType val){
            this.oilType = val;

            return this;
        }

        public Car build(){
            return new Car(this);
        }
    }
}
```

빌더의 Setter 메소드들은 자기 자신을 반환하기 때문에 연쇄적 호출을 할 수 있다. 

이런 방식을 **<span style="color:#ef5369">Fluent API</span>** 또는 **<span style="color:#ef5369">Method Chaining</span>** 이라고 한다.(메소드 호출이 흐르듯 연결된다는 뜻)


해당 빌더를 사용하는 클라이언트 코드는 다음과 같다.

예시로 세단 형태의 3개의 문을 가진 디젤 차량을 생성한다.

```java
Car car = new Car.Builder(CarType.SEDAN)
                 .doors(3)
                 .oilType(OilType.DIESEL)
                 .build();
```

현재 유효성 검사 로직은 제거하였지만, 필요하다면 유효성 검사 로직을 추가하여 잘못된 점을 발견하게 되면 메세지를 담아 IllegalArgumentException 을 던져주면 된다.

<br>

#### **<span style="color:#ef5369">추상 클래스를 활용한 빌더</span>**

<br>

빌더 패턴은 계층적으로 설계된 클래스와 사용하기 좋다.

추상클래스의 경우 추상 빌더, Concrete 클래스의 경우 Concrete 빌더를 갖게 한다.
 

다음은 추상 클래스를 활용한 추상 빌더 구현한 코드이다.

```java
abstract class Burger{
    public enum Topping{CHEEZE,PATTY,LETTUCE, TOMATO,MUSGROOM, ONION, PICKLE, EGG}
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>>{
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        protected abstract T self();

        public T addTopping(Topping topping){
            toppings.add(topping);
            return self();
        }

        public abstract Burger build();
    }

    protected Burger(Builder<?> builder){
        toppings = builder.toppings.clone();
    }
}
```

다음은 Burger 추상 클래스를 상속 받은 EggSlut 클래스를 생성해보겠다.

```java
class EggSlut extends Burger{
    public enum Side{NONE, FRENCH_FRIES, SLUT, COLA}
    private final Side side;

    public static class Builder extends Burger.Builder<Builder>{
        private final Side side;

        @Override
        protected Builder self(){
            return this;
        }

        public Builder(Side side){
            this.side = Objects.requireNonNull(side);
        }

        @Override
        public EggSlut build(){
            return new EggSlut(this);
        }
    }

    public EggSlut(Builder builder){
        super(builder);
        side = builder.side;
    }
}
```

EggSlut 인스턴스를 가지려면 동일하게 Builder를 호출하여 사용하면 된다.

```java
EggSlut eggSlut = new EggSlut.Builder(EggSlut.Side.SLUT)
                             .addTopping(Burger.Topping.EGG)
                             .addTopping(Burger.Topping.LETTUCE)
                             .addTopping(Burger.Topping.CHEEZE)
                             .addTopping(Burger.Topping.TOMATO)
                             .build();
```

위 코드를 보다시피 빌더 패턴은 매우 유연하다.

빌더 하나로 여러 객체를 만들 수 있고, 넘겨지는 파라미터에 따라 다른 객체를 만들 수 도 있다.


빌더 패턴의 단점이 있다면 빌더 생성 비용은 크진 않지만, 성능에 민감한 서비스라면 문제가 될수 있다.

코드가 장황하다 보니 파라미터가 4개 이상 되어야 값어치를 한다.


다만 **<span style="color:#ef5369">API는 시간이 지날 수록 파라미터가 많아지는 경향</span>**이 있으니 예방차원에서 빌더를 구현해도 좋다.

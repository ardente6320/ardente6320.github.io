---
title:  "Factory Pattern을 활용한 Service 관리"
date:   2023-07-08 19:50:58+0900
categories: [Spring Boot]
tags: [SpringBoot, Factory Pattern, Service Factory]
---

<br>

비즈니스 단 로직 구현할 때 입력 받은 특정 값에 따라 다르게 처리해야 하는 경우가 많아진다.

음식 주문 서비스를 예로 들어보자.

음식을 주문하는데 있어서, 포장 주문과 배달 주문, 그리고 매장 주문이 있다.

먼저 가장 쉽게 생각할 수 있는 방식은 `if`문이나 `switch`로 나눠서 처리하는 방식이다.

```java
@RestController
@RequestMapping("/order")
@RequiredArgsConstructor
public class OrderController{
    private final OrderServoce packageOrderServiceImpl;
    private final OrderServoce deliverOrderServiceImpl;
    private final OrderServoce storeOrderServiceImpl;

    @PostMapping(value = "/amount", produces = { MediaType.APPLICATION_JSON_VALUE })
    public ResponseEntity<?> order(@RequestBody OrderInfo orderInfo){
        Map<String,Object> response = new HashMap<>();

        int amount = 0;
        if(order.getOrderType() == OrderType.PACKAGE){
            amount = packageOrderServiceImpl.getOrderAmount(orderInfo());
        }else if(order.getORderType() == OrderType.DELIVER){
            amount = deliverOrderServiceImpl.getOrderAmount(orderInfo());
        }else if(order.getOrderType() == OrderType.STORE){
            amount = storeOrderServiceImpl.getOrderAmount(orderInfo());
        }

        response.put("amount", amount);
        return new ResponseEntity<>(response, HttpStatus.OK);
    }
}
```

#### **<span style="color:#ef5369">주문 서비스 인터페이스</span>**

```java
//주문 서비스
public interface OrderService{
    int getOrderAmount(OrderInfo order);
}
```

#### **<span style="color:#ef5369">포장 주문 서비스</span>**

```java
//포장 주문 서비스
@Service
public class PackageOrderServiceImpl implements OrderService{
    @Override
    int getOrderAmount(OrderInfo order){
        return order.getAmount() - 2000;
    }
}
```

#### **<span style="color:#ef5369">배달 주문 서비스</span>**

```java
//배달 주문 서비스
@Service
public class DeliveryOrderServiceImpl implements OrderService{
    @Override
    int getOrderAmount(OrderInfo order){
        return order.getAmount() + 3000;
    }
}
```
#### **<span style="color:#ef5369">매장 주문 서비스</span>**

```java
//매장 주문 서비스
@Service
public class StoreOrderServiceImpl implements OrderService{
    @Override
    int getOrderAmount(OrderInfo order){
        return order.getAmount() + 1000;
    }
}
```

위코드를 보면 `Controller`에서 각각의 `OrderType`에 따라 필요한 서비스를 호출하고 있다.

현재는 메소드가 하나지만 메소드가 여러개가 되고 각 메소드에서 분기로 나눈다면 유지보수하기 어려워진다.

만약 새로운 주문 서비스가 생겨버리게 되면 모든 `if`나 `switch`문을 손봐야할 것이다.

---

이 문제를 해결 하기위해선 우선 각 서비스에 타입 반환 메소드를 `Override`하여 추가해준다.

```java
public interface OrderService{
    int getOrderAmount(OrderInfo order);
    
    //타입 반환
    OrderType getType();
}
```

그리고 아래와 같이 주문 서비스를 상속받은 각각의 서비스 구현체에다가 각각의 타입을 반환하도록 한다.

```java
@Service
public class PackageOrderServiceImpl implements OrderService{
    @Override
    int getOrderAmount(OrderInfo order){
        return order.getAmount() - 2000;
    }
    
    @Override
    OrderType getType(){
        return OrderType.PACKAGE;
    }
}
```
```java
@Service
public class DeliveryOrderServiceImpl implements OrderService{
    @Override
    int getOrderAmount(OrderInfo order){
        return order.getAmount() + 3000;
    }
    
    @Override
    OrderType getType(){
        return OrderType.DELIVER;
    }
}
```
```java
@Service
public class StoreOrderServiceImpl implements OrderService{
    @Override
    int getOrderAmount(OrderInfo order){
        return order.getAmount() + 1000;
    }
    
    @Override
    OrderType getType(){
        return OrderType.STORE;
    }
}
```

이렇게 생성된 서비스들을 활용하려면 `Factory Pattern`을 활용하여 `ServiceFactory`를 만들어 주입한다.

우선 먼저 `ServiceFactory`를 생성한다.

#### **<span style="color:#ef5369">주문 서비스 팩토리</span>**

```java
@Component
public class OrderServiceFactory{
    private final Map<OrderType, OrderService> orderServiceMap = new HashMap<>();
    
    //생성자 주입
    public OrderServiceFactory(List<OrderService> orderServices){
        orderServices.forEach(service -> orderServiceMap.put(service.getOrderType(), service));
    }
    
    public getOrderService(OrderType orderType){
        return orderServiceMap.get(orderType);
    }
}
```

위와 같이 특정 값에 따라 서비스를 등록해준다. 서비스는 생성자 주입을 통해 등록한다.

단 이때 동일한 `OrderService`를 상속받은 서비스들이 `OrderServiceFactory`에 등록될 것이다.

그러면 이 팩토리를 이제 `OrderController`에서 다음과 같이 사용하면 된다.

#### **<span style="color:#ef5369">주문 컨트롤러</span>**
```java
@RestController
@RequestMapping("/order")
@RequiredArgsConstructor
public class OrderController{
    private final OrderServiceFactory orderServiceFactory;
    
    @PostMapping(value = "/amount", produces = { MediaType.APPLICATION_JSON_VALUE })
    public ResponseEntity<?> order(@RequestBody OrderInfo orderInfo){
        Map<String,Object> response = new HashMap<>();
        
        response.put("amount", orderServiceFactory.getOrderService(orderInfo.getOrderType()).getOrderAmount(orderInfo));
        return new ResponseEntity<>(response, HttpStatus.OK);
    }
}
```

---

이제는 장황하게 `if`나 `switch`를 작성할 필요없고 이해하기 쉽고 유지보수가 용이하게 사용할 수 있게 되었다.

앞으로 `OrderType`이 늘어나더라도 `OrderService`를 상속받은 서비스만 생성하면 해당 서비스를 사용할 수 있다.
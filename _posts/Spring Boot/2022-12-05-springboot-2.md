---
title:  "Controller에서 파라미터 받는 방법(PathVariable, RequestParam, RequestBody)"
date:   2022-12-05 17:33:23
categories: [Spring Boot]
tags: [SpringBoot, Controller]
---
<br>

Controller에서 API를 호출할 때 파라미터를 넘겨서 호출할 수 있다.

3 가지 대표적인 방식이 존재하는데, 동일한 점은 데이터를 전달하기 위해 사용된다는 점이다.

HTTP에서는 필요할때 마다 요청하여 새로운 데이터를 받아오게 된다.(이전 데이터는 가지고 있지 않음)

하지만 HTTP에서는 데이터를 보장하지 않아서 보관할 곳이 필요한데 해당하는 방법이 데이터 보관을 보장해준다.

---

#### **<span style="color:#ef5369">@PathVariable</span>**

URI의 { }로 들어가는 변수를 받는다.

```java
@RestController
@RequestMapping("/users/")
public class UserController{

    @RequestMapping("detail/{userId}")
    public String getUserDetails(@PathVariable(value="userId") String userId){
        ...
        return "userDetailView.do" ; 
    }
}
```

위와 같이 userId 라는 값을 `PathVariable`로 받아오고 있다. 

userId 하나만 설정하여 이것을 인지한 상태에서 작업을 진행해야한다.

이렇게 파라미터를 URL 경로에 포함시키게되면 이해하기 쉽고 보기좋게 만들 수 있다.

<br>

파라미터의 타입은 URI의 내용이 적절히 변환될 수 있는 것을 사용해야한다.

만약에 `String`이 아닌 `int` 타입을 썻을 경우에 해당 변수 자리에 숫자 값이 들어 있어야 한다. 

그렇지 않은 경우 **HTTP 400- Bad Request** 가 발생한다.

---

#### **<span style="color:#ef5369">@RequestParam</span>**

GET 방식으로 넘어온 URI의 `queryString` 변수를 받는다.

예로 들면 `http://test-api.com/users/info?userId=oliver&code=123123` 이렇게 호출했을 때

물음표의 뒷부분에 존재하는 `queryString`을 분석하여 파라미터를 받아오며, `&`기준으로 파라미터를 구분한다.

```java
@RestController
@RequestMapping(path = "/users/")
public class UserController{

    @GetMapping("info")
    public ResponseEntity getUserInfo(@RequestParam(value="userId") String userId,
                                      @RequestParam(value="code", required=false) String code ){
    	Map<String, Object> result = new HashMap<String, Object>();
        ...                          
        return ResponseEntity.ok(result);         
    }
}
```

위와 같이 value에는 넘어오는 파라미터 명을 적으면 된다.

`required`는 필수는 아니지만 기본값은 `true`이다. 따라서 작성하지 않으면 선언한 파라미터가 반드시 존재해야하며,

없을 경우 **HTTP 400- Bad Request** 가 발생한다.

그리고 `Default Value` 속성이 존재하는데, 해당 속성은 파라미터가 존재하지 않을 경우 매핑될 기본값을 나타낸다.

---

#### **<span style="color:#ef5369">@RequestBody</span>**

HTTP 요청 시 body 부분을 객체로 받게 해주는 어노테이션이다. 주로 `json type`을 많이 활용한다.

```json
{
    "userId" : "oliver",
    "code" : "123",
    "role" : "04",
    "name" : "올리버"
}
```

다음과 같은 파라미터가 존재한다고 가정해본다.

데이터를 받아올 때 **DTO 방식**으로 받아오거나 **Map 방식**으로 받아올 수 있다.

<br>

#### **<span style="color:#ef5369">DTO 방식</span>**

먼저 다음과 같은 `DTO` 객체를 먼저 작성한다.

```java
//Data Annotation을 활용하여 Getter, Setter 자동 생성
@Data
public class UserDTO {
    private String userNo;
    private String code;
    private String role;
    private String name;
}
```

`DTO` 작성 후, `Controller`의 파라미터에 해당 객체를 사용하여 받는다.

```java
@RestController
@RequestMapping(path = "/users/")
@Slf4j
public class UserController{
    @Autowired
    UserService userService;

    @PostMapping("info")
    public ResponseEntity setUserInfo(@RequestBody UserDTO userDTO ){      
        log.info("userId :: {} name :: {}",userDTO.getUserId(),userDTO.getName());
        
        return ResponseEntity.ok(userService.insertUser(userDTO));         
    }
}
```

`DTO`를 사용하면 객체를 직접 작성해야하고, 유동적인 파라미터 수정이 조금 귀찮아진다.

조금 더 유동적인 파라미터를 받기 원한다면 `MAP`으로 받을 수 있다.

<br>

#### **<span style="color:#ef5369">MAP 방식</span>**

```java
@RestController
@RequestMapping(path = "/users/")
@Slf4j
public class UserController{
    @Autowired
    UserService userService;

    @PostMapping("info")
    public ResponseEntity setUserInfo(@RequestBody Map<String,Object> body ){  
    	
        String userId   = body.get("userId");
        String name     = body.get("name");
        String phoneNum = body.get("phoneNum");
        
        log.info("userId :: {} name :: {} phoneNum :: {}",userId,name,phoneNum);
        
        return ResponseEntity.ok(userService.insertUser(body));         
    }
}
```

`MAP`을 사용하여 데이터를 받아오면 파라미터를 유동적이게 받아올 수 있다. 

데이터는 `Key:Value` 형태로 받아오기 때문에 간단한 데이터에 대해서는 쉽게 추가할 수 있다.

하지만 데이터를 가져올때 정확한 `Key` 명을 써야 올바른 값을 불러올 수 있기때문에 주의 해야한다.
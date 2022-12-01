---
title:  "WITH절 사용하기"
date:   2022-12-01 11:18:50
categories: [PostgreSQL]
tags: [PostgreSQL, sql]
---
<br>

### **<span style="color:#ef5369">WITH 절이란?</span>**

반복되는 구문이거나, 동일한 구문 또는 복잡한 구문을 작성할 때 매크로처럼 미리 선언하여 사용하는 서브쿼리라고 생각하면 된다.

해당 구문을 통해 임시 테이블이 생성된다.

<br>

`VIEW`는 한번 만들어 놓으면 `DROP` 전 까지 없어지지 않지만,

**WITH 절**로 생성된 임시 테이블은 한번 실행되는 쿼리 내에만 실행되는 생명주기를 가지고 있다. 

**WITH 절**로 생성한 임시 테이블은 한번 생성되면 동일한 블록을 재사용할 수 있는 장점이 있으며, Planning 할때 속도 향상에 많은 도움을 준다.

---

### **<span style="color:#ef5369">사용 방법</span>**

```sql
/*
 * [명칭] 에다가 alias 할 이름을 작성하면 된다.
 * 그리고 해당 결과를 불러올 땐 alias한 이름을 불러오면 된다.
 */
with [명칭] as(
    select ...
    from ...
    where ...
)
select ...
from [명칭]
where ...
```

---

### **<span style="color:#ef5369">예시</span>**

다음 쿼리는 employee(직원 상세 정보), employees(직원 정보), department(부서 정보) 테이블을 통해 부서명과 매니저 번호 기준으로 직원이 몇명있는지 파악하는 쿼리이다.

현재 쿼리를 보면 반복되는 구문을 조인하여 결과를 도출해내고 있다.

```sql
select coalesce(B.department_name,'empty') as department_name
     , A.manager_id
     , count(A.employee_id) as employee_cnt
from employee A 
left outer join (
                  select e.employee_id
                       , e.employee_name 
                       , e.department_id
                       , d.department_name 
                  from employees e 
                  left outer join departments d on (e.department_id = d.department_id)
                ) B on (A.employee_id  = B.employee_id)
group by B.department_name,A.manager_id 
union all 
select 'total' as depart_name
     , null as manager_id
     , count(A.employee_id) as employee_cnt
from employee A 
left outer join (
                  select e.employee_id
                       , e.employee_name 
                       , e.department_id
                       , d.department_name 
                  from employees e 
                  left outer join departments d on (e.department_id = d.department_id)
                ) B on (A.employee_id  = B.employee_id)
```

위의 쿼리를 WITH 절로 변환을 하면 다음과 같다.

WITH 절을 사용하여 반복되는 구문을 재사용하여 작성한 방법이다.

```sql
with B as(
       select e.employee_id
            , e.employee_name 
            , e.department_id
            , d.department_name 
       from employees e 
       left outer join departments d on (e.department_id = d.department_id)         
)
select coalesce(B.department_name,'empty') as department_name
     , A.manager_id
     , count(A.employee_id) as employee_cnt
from employee A 
left outer join B on (A.employee_id  = B.employee_id)
group by B.department_name,A.manager_id 
union all 
select 'total' as depart_name
     , null as manager_id
     , count(A.employee_id) as employee_cnt
from employee A 
left outer join B on (A.employee_id  = B.employee_id)
```

---

#### **<span style="color:#ef5369">결론</span>**

WITH 절을 사용함으로써 간결하고, 직관적인 쿼리를 만들 수있어 좋다고 생각한다. 

특히나 동일한 구문을 반복해서 작성하다보면 실수가 생기기 마련인데, 블록을 재사용함으로써 실수나 오류를 줄일 수 있다는 것이 큰 장점인 것 같다.
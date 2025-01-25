---
title: "[Sql튜닝] [oracle][MySql] 임시로 생성된 가상테이블 WITH절과 UNION ALL의 조합"
date: 2024-09-28 20:39 +0900
lastmod: 2024-11-27 19:51 +0900
categories: Sql튜닝
tags: [WITH, subquery, VIEW, join, UNION, UNION ALL, hint, materialize, inline]
---

## Oracle9, WITH절

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 WITH절은 이름이 부여된 서브쿼리이다 (서브쿼리 정의)
</div>

WITH절 서브쿼리의 결과를 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">임시로 생성된 가상테이블</span>로 사용할 수 있다.  
이렇게 함으로써 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">중복쿼리를 줄일 수 있다.</span>  
즉, 반복적으로 사용하는 쿼리를 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">WITH절</span>로 감싸고 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;font-weight:bold;">QUERY BLOCK</span> 이름을 부여하여 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">재사용할 수 있다.</span>

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">🪭 WITH절 안에서 다른 WITH절을 참조할 수 있다.</span>

뷰와 쓰임새가 비슷하지만 뷰와 다른 점은, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">VIEW는 한번 만들어 놓으면 DROP할 때까지 없어지지 않는다.</span>  
하지만, <span style="padding:0 3px;border-radius:5px;background-color:#BCD4E6;color:#34343c;">WITH절은 쿼리문에서만 실행된다.</span>

또한, WITH절이 자주 실행되는 경우 <span style='color:#652DC1;'>한번만 파싱</span>되고, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#462679;font-weight:bold;">PLAN계획</span>이 수립되므로 쿼리의 성능향상에 도움이 된다.  
<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;color:#34343c;">즉, WITH절을 여러번 참조하는 쿼리를 만들수록 효과가 증대된다.</span>

```sql
WITH TABLE_NAME(COLUM1, COLUM2, ...) AS
(
    -- SUB_QUERY
)
```

```sql
WITH EX AS
(
    SELECT *
    FROM PRODUCTS
    WHERE STANDARD_COST > 1000
)
SELECT * FROM EX WHERE EX.CATEGORY_ID = '1'
UNION ALL
SELECT * FROM EX WHERE EX.CATEGORY_ID = '2'
UNION ALL
SELECT * FROM EX WHERE EX.CATEGORY_ID = '3'
```

<span style="font-weight:700;font-size:1.03rem;">🌩️ Oracle 11g R2 ver.까지는 WITH절에 SELECT 구문을 반드시 사용해야 한다.</span>

### WITH절 동작방식

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;font-weight:bold;">
    🥯 Materialize &nbsp;/&nbsp; Inline
</div>

- Materialize 방식  
  : 🔅 오라클 공유 메모리에 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">GLOBAL\_임시테이블</span> 생성  
  임시테이블을 생성 후, WITH절 결과를 저장하며,  
  반복호출 시, <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">쿼리를 실행하지 않고 임시테이블에 저장된 결과를 사용한다.</span>  
  🌩️ 임시테이블을 Create, Drop하는 행위를 반복하기 때문에, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 자주 수행되는 쿼리문에 사용하면 시스템부하가 생길 수 있다.</span>

- Inline 방식
  : 🔅 임시테이블을 생성하지 않고, 메인쿼리에서 바로 사용하는 방식  
  <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(193,151,210,0.7);color:#34343c;">참조된 횟수만큼 반복적으로 쿼리를 실행한다.</span>  
  <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">그러므로 호출하는 쿼리에서 딱 한번만 실행하는 경우 유리하다...</span>

```sql
WITH EX2 AS
(
    SELECT /*+ materialize */ *
      FROM PRODUCTS
      WHERE STANDARD_COST > 1000
)
SELECT * FROM EX2 WHERE EX2.CATEGORY_ID = '1'
UNION ALL
SELECT * FROM EX2 WHERE EX2.CATEGORY_ID = '2'
UNION ALL
SELECT * FROM EX2 WHERE EX2.CATEGORY_ID = '3'
```

<span style="font-weight:700;font-size:1.03rem;">⚠️ Oracle 11g부터 직접 힌트를 사용하여 동작방식을 바꿀 수 있다.</span>

### UNION / UNION ALL

<div style="margin-bottom:15px;font-size:17px;background-color:#DEDEDE;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;font-weight:bold;">
    🥯 UNION &nbsp;/&nbsp; UNION ALL
</div>

<span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">두개의 SELECT문을 서로 합치고 싶은데</span>, 중복데이터를 한번만 출력하고 싶다면 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">UNION</span>을 사용한다.  
결과값을 자동으로 정렬한 후 반환한다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">단, SELECT문의 컬럼명, 컬럼위치, 컬럼수가 동일해야 한다..</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">그렇지 않으면 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 ORA-01789: 질의 블록은 부정확한 수의 결과 열을 가지고 있습니다.</span> 라는 에러메세지를 출력한다.</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">컬럼명이 같지 않다면 alias를 사용하여 억지로라도 같게 만들어줘야 한다.</span>

서로 합치고 싶으나, <span style="margin-bottom:15px;padding:0 3px;border-radius:5px;background-color:#ffdce0;color:#34343c;">중복데이터를 모두 조회</span>하고 싶다면 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);font-weight:bold;">UNION ALL</span>을 사용한다.  
자동정렬을 안되니 정렬을 해줘야하는데, 마지막에 연결한 SELECT문에만 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#6A9487;font-weight:bold;">ORDER BY(정렬)</span> 가능하다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#34343c;">그렇지 않으면 <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:rgb(196,58,26);">🐖 ORA-00933: SQL 명령어가 올바르게 종료되지 않았습니다.</span> 라는 에러메세지를 출력한다.</span>

<span style="font-weight:700;font-size:1.03rem;">🍪 JOIN / UNION</span>  
`JOIN`은 키를 기준으로 테이블을 연결하여, <span style='color:#652DC1;'>컬럼을 확장</span>하여 보여주고  
`UNION`은 다른 테이블의 <span style='color:#652DC1;'>ROW를 합쳐서</span> 보여준다.

## 예제(MySql): WITH문으로 계층쿼리 작성하기

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(45,204,112);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    ⚔️ 예제(MySql): WITH문으로 계층쿼리 작성하기
</div>

재귀적인 쿼리를 이용하면, 쿼리의 결과를 반복적으로 조회하고 처리할 수 있다.

```sql
WITH RECURSIVE cte AS
(
    -- FOUNDATION CASE (Non-Recursive 문장) (첫번째 루프에서만 실행)
    -- 재귀적 프로세스를 시작하기 위해 반환할 초기 데이터 집합 정의
    SELECT level, name
      FROM employees
     WHERE id = 1
     UNION ALL
    -- RECURSIVE CASE (Recursive 문장) (읽어올 때마다 행의 위치가 기억)
    -- FOUNDATION CASE 반환 데이터로부터 실행되며, 각 재귀적 단계에서 결과집합에 새로운 행 추가
    SELECT cte.level + 1, e.name
      FROM employees e
      JOIN cte
        ON e.parent_id = cte.id
)
SELECT *
  FROM cte;
```

`WITH RECURSIVE` 구문은 가상 테이블을 생성하면서 <span style="padding:0 3px;border-radius:5px;background-color:#ffff9e;color:#624a3d;">가상 테이블 자신의 값을 참조</span>하여 값을 결정할 때 사용된다.

---

```sql
-- 예제(1)
WITH cte_count(num) AS
(
    SELECT 1 AS num
      FROM DUAL
     UNION ALL
    SELECT c.num + 1
      FROM cte_count c
     WHERE c.num < 10
)
SELECT num
  FROM cte_count;

-- 예제(2)
WITH employee_hierarchy(eid, ename, edept) AS
(
    SELECT emp_id AS eid
         , emp_name AS ename
         , department AS edept
      FROM employees
     WHERE emp_name = 'John Doe'
     UNION ALL
    SELECT e.emp_id -- 두번째 쿼리부터는 별칭 생략 가능, 단 컬럼의 개수와 데이터 타입이 일치해야한다.
         , e.emp_name
         , e.department
      FROM employees e
      JOIN employee_hierachy eh
        ON e.emp_id = eh.manager_id
     ORDER BY e.emp_name -- ORDER BY절은 마지막 SELECT문에만 사용가능
)
SELECT eid, ename, edept
  FROM employee_hierachy
```

재귀적인 쿼리를 이용하지 않고도, <span style="padding:3px 6px;font-size:16px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">WITH절</span>만으로도 계층형 쿼리를 구현할 수 있다.

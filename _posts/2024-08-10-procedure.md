---
title: "[Sql튜닝] 재사용 가능한 일련의 쿼리의 집합, 프로시저와 EXCEPTION 처리 루틴 모듈화 "
date: 2024-08-10 10:20 +0900
lastmod: 2024-08-11 10:20 +0900
categories: Sql튜닝
tags:
  [
    Procedure,
    PL/SQL,
    Oracle,
    Sequence,
    MySQL
  ]
---

## 트랜잭션 관리 및 보안상 이점, 프로시저

<div style="margin-bottom:15px;font-size:20px;background-color:#462679;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🍝 프로시저란 일련의 쿼리를 마치 하나의 함수처럼 실행하기 위한 일종의 쿼리의 집합을 말한다
</div>

정의한 프로시저를 <span style='color:rgb(196,58,26);'>호출</span>해서 사용하므로서 코드의 `재사용성 🏆`이 증가한다.  
프로시저를 정의할 때 여러 개의 sql문을 하나의 트랜잭션으로 묶어 `일관성 🏆`을 유지할 수 있다.  
또한, 사용자에게 직접적인 database 접근을 허용하지 않음으로 보안상에도 이점이 있다.  

## PL/SQL

<div style="margin-bottom: 15px;font-size:20px;background-color:#FFD24D;color:black;font-weight:normal;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐀 Oracle's Procedural Language extension to SQL
</div>

Oracle 자체에 내장되어 있는 절차적 언어로서, <span style='color:rgb(196,58,26);'>변수정의</span>, <span style='color:rgb(196,58,26);'>분기</span>, <span style='color:rgb(196,58,26);'>반복 처리</span> 등을 지원한다.  
※ PL/SQL문은 <span style="margin-bottom:15px;padding:0 6px;border-radius:5px;color:white;background-color:#0072BB;">BLOCK 구조</span>로 되어있고, PL/SQL 자신이 <span style="margin-bottom:15px;padding:0 6px;border-radius:5px;color:white;background-color:#0072BB;">컴파일 엔진</span>을 가지고 있다.  

BLOCK 구조안에서 다수의 SQL 문을 <span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;">한번에 Oracle DB로 보내서 처리하므로 성능을 향상시킬 수 있다.</span>  
또한, 이 BLOCK 구조로 인해 모듈화가 가능하다.  

테이블의 <span style="padding:1px 3px;border: 1px solid #DEDEDE;border-radius:5px;">데이터 구조</span>와 <span style="padding:1px 3px;border: 1px solid #DEDEDE;border-radius:5px;">컬럼명</span>에 준하여 <span style='color:#0072BB;'>동적으로 변수를 선언</span>할 수 있다.  

<span style="padding:3px 6px;font-size:17px;color:rgb(196,58,26);background-color:#f6f6f7;">EXCEPTION 처리 루틴</span>을 이용하여 `Oracle Server Error`를 처리하며,  
사용자 정의 에러를 선언하고 EXCEPTION 처리 루틴으로 처리 가능하다.  

## 프로시저 기본문법__Oracle

<div style="margin-bottom:15px;font-size:20px;background-color:#2A52BE;color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🐙 프로시저 기본문법
</div>

`🐫 if_statement_proc`  

```sql
CREATE OR REPLACE PROCEDURE if_statement_proc
IS
    -- 프로시저 내에서 사용할 변수
    start_message VARCHAR2(100) := '-----TEST_START-------';
    end_message VARCHAR2(100) := '-----TEST_END-------';
    err_message VARCHAR2(100) := '-----ERROR-------';
    empno_sys NUMBER(4, 0);
    ename_sys VARCHAR2(10);
    start_state VARCHAR2(1) := '0';

-- DECLARE가 PL/SQ 블럭의 시작을 의미하지만, 생략되면 BEGIN이 블럭의 시작이 됨.
BEGIN
    -- 기능 구현
    DBMS_OUTPUT.PUT_LINE(start_message);

    IF start_state IS NOT NULL AND start_state = '0' THEN
        BEGIN
            SELECT empno, ename
            INTO empno_sys, ename_sys
            FROM emp
            WHERE ename = 'KING';
        END;
    
    ELSEIF start_state IS NOT NULL AND start_state = '1' THEN
        BEGIN
            SELECT empno, ename
            INTO empno_sys, ename_sys
            FROM emp
            WHERE ename = 'JONES';
        END;

    ELSE 
        BEGIN
            SELECT empno, ename
            INTO empno_sys, ename_sys
            FROM emp
            WHERE ename = 'JONES';
        END;
    
    END IF;
    DBMS_OUTPUT.PUT_LINE('empno: ' || empno_sys);
    DBMS_OUTPUT.PUT_LINE('ename: ' || ename_sys);

    DBMS_OUTPUT.PUT_LINE(end_message);
END;
```

🍪 <span style="padding:3px 6px;font-size:17px;color:rgb(196,58,26);background-color:#f6f6f7;">EXECUTE</span> 문을 이용해 <span style='color:rgb(196,58,26);'>프로시저를 호출</span>한다.  

```sql
EXECUTE if_statement_proc;
```

`🐫 loop_statement_proc`  

```sql
CREATE OR REPLACE PROCEDURE loop_statement_proc
IS
    start_message VARCHAR2(100) := '-----TEST_START-------';
    end_message VARCHAR2(100) := '-----TEST_END-------';
    test_num NUMBER := 1;

BEGIN
    DMBS_OUTPUT.PUT_LINE(start_message);

    -- 조건 for문
    LOOP
    EXIT WHEN test_num > 10;
        DBMS_OUTPU.PUT_LINE(test_num);
        test_num := test_num + 1;
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE(end_message);

    -- 기본 for문
    FOR num IN 1..10 LOOP
    DBMS_OUTPUT.PUT_LINE(test_num);
    test_num := test_num + 1;
    END LOOP;

    DBMS_OUTPUT.PUT_LINE(end_message);
END;
```
`🐫 table_for_loop_statement_proc`  

```sql
CREATE OR REPLACE PROCEDURE table_for_loop_statement_proc
IS
    start_message VARCHAR2(100) := '-----TEST_START-------';
    end_message VARCHAR2(100) := '-----TEST_END-------';

    -- PL/SQL에서 사용 가능한 사용자 정의 컬렉션 타입 선언
    -- BINARY_INTEGER로 인덱싱, 임의의 정수로 각 요소에 접근 가능
    TYPE ename_table_type IS TABLE OF emp.ename%TYPE
    INDEX BY BINARY_INTEGER;

    i BINARY_INTEGER := 0;
    ename_table ename_table_type;

BEGIN
    DBMS_OUTPUT.PUT_LINE(start_message);

    FOR table_data IN (SELECT * FROM emp) LOOP
        i := i+1;
        ename_table(i) := table_data.ename;
        DBMS_OUTPUT.PUT_LINE(ename_table(i));
    END LOOP;
END;
```

`🐫 row_type_proc`  

```sql
CREATE OR REPLACE PROCEDURE row_type_proc
IS
    test_emp emp%ROWTYPE;

BEGIN
    SELECT empno, ename, sal
    INTO test_emp.empno, test_emp.ename, test_emp.sal
    FROM emp
    WHERE empno = 7469;

    DBMS_OUTPUT.PUT_LINE('empno: ' || test_emp.empno);
    DBMS_OUTPUT.PUT_LINE('ename: ' || test_emp.ename);
    DBMS_OUTPUT.PUT_LINE('sal: ' || test_emp.sal);
END;
```

## EXCEPTION 처리 루틴 모듈화__Oracle

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🦧 EXCEPTION 로그 남기기
</div>

`🥖 error_log`
: 에러 로그 테이블 생성  

```sql
CREATE TABLE error_log (
    seq NUMBER,
    proc_name VARCHAR2(80),
    error_code NUMBER,
    error_message VARCHAR2(300),
    error_line VARCHAR2(100),
    error_date DATE DEFAULT SYSDATE
);
COMMIT;
```

`🍢 error_seq`
: 시퀀스 테이블 생성  

```sql
CREATE SEQUENCE error_seq
INCREMENT BY 1
START WITH 1
MINVALUE 1
MAXVALUE 99999
NOCYCLE  -- 최대값 도달시 순환 여부 (N)
NOCACHE; -- 캐시(원하는 숫자만큼 미리 만들어 Shared Pool의 Libray Cache에 상주) 여부 (N)
COMMIT;
```

`🐫 error_log_proc`  
: 에러 로그 삽입 프로시저  

```sql
CREATE OR REPLACE PROCEDURE error_log_proc (
    p_proc_name error_log.proc_name%TYPE,
    p_error_code error_log.error_code%TYPE,
    p_error_message error_log.error_message%TYPE,
    p_error_line error_log.error_line%TYPE)
IS

BEGIN
    INSERT INTO error_log(seq, proc_name, error_code, error_message, error_line)
        VALUES(error_seq.NEXTVAL, p_proc_name, p_error_code, p_error_message, p_error_line);
    COMMIT;
END;
```

`🐫 inst_emp_proc`  
: 회원 삽입 프로시저  

```sql
CREATE OR REPLACE PROCEDURE inst_emp_proc (
    p_employee IN employees.first_name%TYPE,
    p_department_id IN employees.department_id%TYPE,
    p_hire_date VARCHAR2)
IS
    vn_employee_id employees.employee_id%TYPE;

    vn_curr_date DATE := SYSDATE;
    vn_cnt NUMBER := 0;

    ex_invalid_deptid EXCEPTION;
    -- PRAGMA란 컴파일러가 실행되기 전에 전처리기 역할을 하는 키워드이다
    -- PRAGMA EXCEPTION_INIT 사용자 정의 예외 처리를 할 때, 특정 예외번호를 명시해서
    -- 컴파일러에 이 예외를 사용한다는 것을 알린다.
    PRAGMA EXCEPTION_INIT(ex_invalid_deptid, -20000);
    ex_invalid_month EXCEPTION;
    PRAGMA EXCEPTION_INIT(ex_invalid_month, -1843);

    v_error_code error_log.error_code%TYPE;
    v_error_message error_log.error_message%TYPE;
    v_error_line error_log.error_line%TYPE;

BEGIN
    SELECT COUNT(*)
    INTO vn_cnt
    FROM departments
    WHERE department_id = p_department_id;

    IF vn_cnt = 0 THEN
        RAISE ex_invalid_deptid;
    END IF;

    IF SUBSTR(p_hire_date, 5, 2) NOT BETWEEN '01' AND '12' THEN
    RAISE ex_invalid_month;
    END IF;

    INSERT INTO EMPLOYEES(EMPLOYEE_ID, FIRST_NAME, HIRE_DATE, DEPARTMENT_ID)
    VALUES(VN_EMPLOYEE_ID, P_EMPLOYEE_NAME, TO_DATE(P_HIRE_DATE || '01'), P_DEPARTMENT_ID);

    COMMIT;

    EXCEPTION
    WHEN ex_invalid_deptid THEN
        v_error_code := SQLCODE;
        v_error_message := '해당 부서가 없습니다.';
        v_error_line := DBMS_UTILITY.FORMAT_ERROR_BACKTRACE;
        ROLLBACK;
        error_log_proc('inst_emp_proc', v_error_code, v_error_message, v_error_line);
    WHEN ex_invalid_month THEN
        v_error_code := SQLCODE;
        v_error_message := SQLERRM;
        v_error_line := DBMS_UTILITY.FORMAT_ERROR_BACKTRACE;
        ROLLBACK;
        error_log_proc('inst_emp_proc', v_error_code, v_error_message, v_error_line);
    WHEN OTHERS THEN
        V_ERROR_CODE := SQLCODE;
        V_ERROR_MESSAGE := SQLERRM;
        V_ERROR_LINE := DBMS_UTILITY.FORMAT_ERROR_BACKTRACE;
        ROLLBACK;
        error_log_proc('inst_emp_proc', v_error_code, v_error_message, v_error_line);
END inst_emp_proc;
```

<div style="margin-bottom:15px;font-size:15px;background-color:rgba(0,0,0,0.03);border-radius:5px;padding:7px;"><span style="font-weight:bold;">📘 시퀀스(Sequence)</span><br>
<span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">순차적으로 증가하는 컬럼(UNIQUE)</span>을 자동적으로 생성해주는 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">Oracle 객체</span>로써, <span style='color:#0072BB;'>테이블과 독립적으로 저장되고 생성</span>된다.<br>
보통 <span style='color:rgb(196,58,26);color:rgb(196,58,26);'>PRIMARY KEY</span> 값을 생성하기 위해 사용한다.<br>
또한, 메모리에 캐시되었을 때 시퀀스 값의 Access 효율이 증가한다.<br><br>

Oracle에서는 Mysql에 있는 AUTO_INCREMENT 기능이 없다.<br>
따라서 <span style='color:#9E0ADD;'>인덱스 값을 증가시켜주기 위해서 시퀀스를 생성</span>해서 사용해야 한다.<br>
하지만 MAX(seq) + 1 같은 방법이 있음에도, 굳이 시퀀스 테이블을 만들어서 사용하는 이유는<br>
<span style="font-size:16px;background-color:#ffdce0;color:rgb(196,58,26);border-radius:5px;padding:2px 3px;">중복 허용 여부</span>에 있다.<br><br>

즉, 값을 하나 증가시키기 위해 여러명의 사용자가 동시에 조회할 때, <span style="font-size:16px;background-color:#ffdce0;color:rgb(196,58,26);border-radius:5px;padding:2px 3px;">동시성 이슈가 발생</span>할 수 있다.<br>
중복이 될 수 있기 때문에, <B>중복이 없는 시퀀스를 권장</B>한다.
</div>

```sql
-- 다음 번호 조회
SELECT error_seq.NEXTVAL
FROM DUAL;

-- 현재 번호 조회
SELECT error_seq.CURRVAL
FROM DUAL;

-- 시퀀스 값 삽입
INSERT INTO error_log(seq, error_code)
VALUES (error_seq.NEXTVAL, '512');

-- 시퀀스 삭제
DROP SEQUENCE error_seq;

-- 동시성 이슈 발생 가능
SELECT NVL(MAX(seq) + 1, 1)
FROM error_log;
```

`MAX(seq) + 1`로 조회해서 사용하는 경우 동시성 이슈가 발생할 수 있다.  
예를 들어 3명의 사용자가 동시에 해당 쿼리를 실행하고, PRIMARY KEY로 설정되어 있다면,  
1명을 제외한 <span style='color:rgb(196,58,26);'>나머지 2명은 EXCEPTION 처리</span>가 된다.  

<span style="padding:0 3px;border-radius:5px;background-color:#E1FEE5;">시퀀스는 중복을 허용하지 않는다.</span>  

## EXCEPTION 처리 루틴 모듈화__MySQL

<div style="margin-bottom:15px;font-size:20px;background-color:rgb(196,58,26);color:white;border-top-left-radius:5px;border-top-right-radius:5px;padding:2px;overflow-x:auto;white-space:nowrap;">
    🦧 EXCEPTION 로그 남기기__MySQL
</div>

`🥖 error_log`  

```sql
CREATE TABLE IF NOT EXISTS error_log (
    seq NUMBER NOT NULL AUTO_INCREMENT PRIMARY KEY,
    proc_name VARCHAR(80),
    error_code INT,
    error_message VARCHAR(300),
    error_line VARCHAR(100),
    error_date DATE DEFAULT DATE_FORMAT(now(), '%Y%m%d %H%i%s')
)
```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F6F3;border-radius:5px;padding:7px;"><span style="font-weight:bold;">📕 VARCHAR와 VARCHAR2</span><br>
기본적으로 <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">VARCHAR</span>는 <span style='color:rgb(196,58,26);'>가변형 길이</span>를 말한다.<br><br>

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">VARCHAR</span>는 MYSQL(MariaDB)에서 사용하는 형식이고,<br>
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#e1d1b8;">VARCHAR2</span>는 Oracle에서 사용하는 형식이다.<br><br>

※ CHAR는 고정형 길이를 말한다.<br>
ex. CHAR(50)으로 설정하고 12345와 같은 문자열을 해당 필드에 넣었을 때, 45개의 공간이 공백으로 차지된다.<br>
<span style='color:rgb(196,58,26);'>가변길이가 공간 절약면에서는 효율적</span>이지만 고정길이에 비해 검색속도가 월등히 떨어진다.
</div>

`🐫 error_log_proc`  
: MySQL에서는 `Stored Procedure`라고 한다.  

```sql
DELIMITER // -- 구분문자, 기본적으로 SQL문을 끝낼 때(;)을 쓰는데, 프로시저 정의문의 끝과 오해하지 않도록, 구분자 변경
DROP PROCEDURE IF EXISTS error_log_proc //
CREATE PROCEDURE error_log_proc (
    IN p_proc_name VARCHAR(255),
    IN p_error_code INT,
    IN p_error_message TEXT,
    IN p_error_line VARCHAR(255))
BEGIN
    INSERT INTO error_log(proc_name, error_code, error_message, error_line)
        VALUES(p_proc_name, p_error_code, p_error_message, p_error_line);
    
    -- SEQ Column은 AUTO_INCREMENT 설정되어 있음으로
    -- 명시적으로 처리하지 않아도 자동 증가

    -- 기본적으로 자동커밋 설정
    -- COMMIT;
END // -- 프로시저 정의문 끝

DELIMITER ; -- 구분자 원래대로 돌려놓음
```

`🐫 inst_emp_proc`  

```sql
CREATE PROCEDURE inst_emp_proc (
    IN p_employee_name VARCHAR(255),
    IN p_department_id INT,
    IN p_hire_date VARCHAR(10))

BEGIN
    DECLARE vn_employee_id INT DEFAULT NULL;
    DECLARE vn_cnt INT DEFAULT 0;
    DECLARE v_error_code INT;
    DECLARE v_error_message VARCHAR(255);
    -- EXIT HADLER 코드가 실행된 후 종료
    -- FOR SQLEXCEPTION 모든 예외를 처리하기 위한 조건
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        -- GET DIAGNOSTICS 명령을 사용해, 오류메세지와 오류코드를 가져온다
        -- CONDITION 1은 가장 최근에 발생한 오류(경고)를 의미한다
        GET DIAGNOSTICS CONDITION 1
            v_error_code = MYSQL_ERRNO;
            v_error_message = MESSAGE_TEXT,
            

        IF v_error_code = 20000 THEN
            SET v_error_message = '해당 부서가 없습니다.';
        ELSEIF v_error_code = 1843 THEN
            SET v_error_message = 'Invalid month.';
        END IF;

        ROLLBACK;
        CALL ERROR_LOG_PROC('inst_emp_proc', v_error_code, v_error_message, NULL);
    END;

    SELECT COUNT(*)
    INTO vn_cnt
    FROM departments
    WHERE department_id = p_department_id;

    IF vn_cnt = 0 THEN
        -- SIGNAL SQLSTATE 사용자 정의 예외 발생
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'EX_INVALID_DEPTID', MYSQL_ERRNO = 20000;
    END IF;

    IF SUBSTRING(p_hire_date, 5, 2) NOT BETWEEN '01' AND '12' THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'EX_INVALID_MONTH', MYSQL_ERRNO = 1843;
    END IF;

    INSERT INTO employees(first_name, hire_date, department_id)
    VALUES(p_employee_name, STR_TO_DATE(CONCAT(p_hire_date, '01'), '%Y%m%d'), p_department_id);

END //

DELIMITER ;
```

---

`🐫 case_when_statement_proc`  

```sql
DELIMITER //

CREATE PROCEDURE case_when_statement_proc (
    IN p_customer_no int,
    OUT p_shipping VARCHAR(255))
BEGIN
    DECLARE customer_country VARCHAR(255);

    SELECT country 
    INTO customer_country
    FROM customers
    WHERE customer_no = p_customer_no;

    CASE customer_country
        WHEN 'USA' THEN
            SET p_shipping = '2-day Shipping';
        WHEN 'CANADA' THEN
            SET p_shipping = '3-day Shipping';
        ELSE
            SET p_shipping = '5-day Shipping';
    END CASE;
END //

DELIMITER ;
```

`🐫 loop_statement_test`  

```sql
CREATE PROCEDURE loop_statement_test()
BEGIN
    DECLARE x INT;
    DECLARE str VARCHAR(255);

    SET x = 1;
    SET str = "";

    loop_label:LOOP
    IF x > 10 THEN
    LEAVE loop_label;
    END IF;

    SET x = x + 1;
    IF(x mod 2) THEN
        ITERATE loop_label;
    ELSE
        SET str = CONCAT(str, x, ',');
    END IF;
    END LOOP;

    SELECT str;
END;
```
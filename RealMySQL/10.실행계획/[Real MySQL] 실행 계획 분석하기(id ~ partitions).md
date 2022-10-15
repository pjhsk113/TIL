# [Real MySQL 8.0] 실행 계획 분석하기(id ~ partitions)

# 실행 계획 분석

## id 컬럼

SELECT 쿼리 별로 부여되는 식별자 값이다.
예를 들어 하나의 SELECT 문장에서 여러 개의 테이블을 조인하면 조인되는 테이블 개수만큼 실행 계획 레코드가 출력되지만 같은 id 값을 가지게 된다. 반면에 하나의 SELECT 문장에 여러개의 단위 SELECT 쿼리가 포함되어 있는 경우 각기 다른 id 값을 가지게 된다.

한 가지 주의할 점은 실행 계획의 id 컬럼이 테이블의 접근 순서를 의미하지 않는다는 것이다. 명확한 테이블 접근 순서를 알고싶다면 TREE 형태의 포맷으로 실행 계획을 출력해 확인할 수 있다.

## select_type 컬럼

select_type은 단위 SELECT 쿼리가 어떤 타입의 쿼리인지 표시되는 컬럼이다.
해당 컬럼에 표시될 수 있는 값은 다음과 같다.

### SIMPLE

UNION이나 서브쿼리를 사용하지 않는 단순한 SELECT 쿼리인 경우 SIMPLE로 표시된다. 쿼리 문장이 아무리 복잡해도 실행 계획에서 select_type이 SIMPLE인 단위 쿼리는 하나만 존재한다. 일반적으로 제일 바깥 SELECT 쿼리의 select_type이 SIMPLE로 표시된다.

### PRIMARY

UNION이나 서브쿼리를 가지는 SELECT 쿼리의 실행 계획에서 가장 바깥쪽에 있는 단위 쿼리는 select_type이 PRIMARY로 표시된다. SIMPLE과 마찬가지로 하나만 존재한다.

### UNION

UNION으로 결합하는 단위 SELECT 쿼리 중 **첫 번째를 제외한 두 번째 이후 단위 SELECT 쿼리의 select_type은 UNION으로 표시된다.**

첫 번째 단위 SELECT는 쿼리 결과를 모아서 저장하는 **임시 테이블(DERIVED)로 표시**된다.

```sql
EXPLAIN
SELECT * FROM (
	(SELECT emp_no FROM employees e1 LIMIT 10) UNION ALL
  (SELECT emp_no FROM employees e2 LIMIT 10) UNION ALL
  (SELECT emp_no FROM employees e3 LIMIT 10)) tb;

+----+------------+------------+-------+-------------+------+-------+-------------+
|id  |select_type | table      | type  |key          |ref   |rows   |Extra        |
+----+------------+------------+-------+-------------+------+-------+-------------+
| 1  |PRIMARY     | <derived2> | ALL   |NULL         |NULL  |   30  | NULL        |
| 2  |DERIVED     | e1         | index |ix_hiredate  |NULL  |300252 | Using index |
| 3  |UNION       | e2         | index |ix_hiredate  |NULL  |300252 | Using index |
| 4  |UNION       | e3         | index |ix_hiredate  |NULL  |300252 | Using index |
+----+------------+------------+-------+-------------+------+-------+-------------+
```

UNION이 되는 단위 SELECT 쿼리 3개 중 첫 번째(e1 테이블)만 UNION이 아니고 나머지 2개는 UNION으로 표시돼 있다. 세 개의 서브쿼리로 조회된 결과를 UNION ALL로 결합해 임시 테이블을 만들어서 사용하고 있기 때문에 첫 번째 쿼리는 DERIVED라는 select_type을 갖는다.

### DEPENDENT UNION

DEPENDENT는 UNION이나 UNION ALL로 결합된 **단위 쿼리가 외부 쿼리에 의해 영향을 받는 것을 의미한다**. 다음 예제 쿼리를 살펴보자.

```sql
EXPLAIN SELECT *
FROM employees e1 WHERE e1.emp_no IN (
  SELECT e2.emp_no FROM employees e2 WHERE e2.first_name='Matt' 
  UNION
  SELECT e3.emp_no FROM employees e3 WHERE e3.last_name='Matt'
);

+------+-------------------+------------+--------+----------+------+--------+------------------+
|id    |select_type        | table      | type   |key       |ref   |rows    |Extra             |
+------+-------------------+------------+--------+----------+------+--------+------------------+
| 1    |PRIMARY            | e1         | ALL    | NULL     | NULL | 300252 | Using where      |
| 2    |DEPENDENT SUBQUERY | e1         | eq_ref | PRIMARY  | func |  1     | Using where      |
| 3    |DEPENDENT UNION    | e2         | eq_ref | PRIMARY  | func |  1     | Using where      |
| NULL |UNION RESULT       | <union2, 3>| ALL    | NULL     | NULL | NULL   | Using temporary  |
+------+-------------------+------------+--------+----------+------+--------+------------------+
```

두 개의 SELECT 쿼리가 UNION으로 결합됐으므로 select_type이 UNION으로 표시됐다.
MySQL 옵티마이저는 IN 절 내부의 서브 쿼리를 먼저 처리하지 않고 외부의 employees 테이블을 먼저 읽은 다음 서브쿼리를 실행한다. employees 테이블의 컬럼 값이 서브쿼리에 영향을 주기 때문에 DEPENDENT 키워드가 표시된다.

내부적으로 UNION에 사용된 SELECT 쿼리의 WHERE 조건에 외부에 정의된 employees 테이블의 emp_no 컬럼이 사용되기 때문에 DEPENDENT UNION이 표시된 것이다.

### UNION RESULT

UNION RESULT는 UNION 결과를 담아두는 테이블을 의미한다.
MySQL 8.0 이전 버전에서는 UNION의 결과를 임시 테이블로 생성했는데, MySQL 8.0부터는 UNION ALL인 경우 임시 테이블을 사용하지 않도록 기능이 개선됐다. 하지만 UNION은 여전히 임시 테이블에 결과를 버퍼링한다.

실행 계획상에서 임시 테이블을 가리키는 라인의 select_type이 UNION RESULT이며, 단위 쿼리가 아니기 때문에 별도의 id 값은 부여되지 않는다.

```sql
EXPLAIN
SELECT emp_no FROM salaries WHERE salary > 100000
UNION DISTINCT
SELECT emp_no FROM dept_emp WHERE from_date > '2001-01-01';

+------+-------------+-------------+-------+-------------+---------+---------------------------+
|id    |select_type  | table       | type  |key          | rows    | Extra                     |
+------+-------------+-------------+-------+-------------+---------+---------------------------+
| 1    |PRIMARY      | salaries    | range | ix_salary   | 191348  | Using where; Using index  |
| 2    |UNION        | dept_emp    | range | ix_fromdate |      1  | Using where; Using index  |
| NULL |UNION RESULT | <union1, 2> | ALL   | NULL        |    NULL | Using temporary           |
+------+-------------+-------------+-------+-------------+---------+---------------------------+
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37a44b18-ffed-4320-8ce7-d81d068b8f26/Untitled.png)

<union1, 2>는 id 값이 1인 단위 쿼리와 2인 단위 쿼리의 조회 결과를 UNION 했다는 것을 의미한다.

위와 동일한 쿼리에서 UNION DISTINCT를 UNION ALL로 변경하면 임시 테이블에 버퍼링을 하지 않으므로 UNION RESULT 표시 라인이 사라진다.

### SUBQUERY

select_type의 SUBQUERY는 FROM 절 이외에서 사용되는 서브쿼리를 의미한다.
FROM 절에 사용된 서브쿼리는 DERIVED(파생 테이블)로 표시되고, 그 밖의 위치에서 사용된 서브쿼리는 전부 SUBQUERY로 표시된다.

### DEPENDENT SUBQUERY

서브쿼리가 바깥쪽 SELECT 쿼리에서 정의된 컬럼에 의존적인 경우 DEPENDENT SUBQUERY가 표시된다.

```sql
EXPLAIN
SELECT e.first_name,
  (SELECT COUNT(*)
   FROM dept_emp de, dept_manager dm
   WHERE dm.dept_no = de.dept_no AND de.emp_no = e.emp_no) AS cnt
FROM employees e
WHERE e.first_name='Matt' ;
```

안쪽 서브쿼리 결과가 바깥쪽 SELECT 쿼리의 컬럼에 의존적이므로 DEPENDENT 키워드가 붙는다. 또한 DEPENDENT SUBQUERY는 외부 쿼리가 먼저 수행된 후 내부 쿼리가 실행돼야 하므로 일반 서브쿼리보다 처리 속도가 느릴 떄가 많다.

### DERIVED

**DERIVED는 단위 SELECT 쿼리의 실행 결과로 메모리나 디스크에 임시 테이블을 생성하는 것을 의미한다.** select_type이 DERIVED 인 경우에 생성되는 임시 테이블을 파생 테이블이라고도 한다.

MySQL 5.5 버전까지는 서브쿼리가 FROM 절에 사용된 경우 항상 DERIVED(파생 테이블)을 만들었기 때문에 성능상 불리한 점이 많았지만, MySQL 5.6 버전부터는 옵티마이저 옵션에 따라 쿼리 특성에 맞게 임시 테이블에도 인덱스를 추가해서 만들수 있게 최적화됐다.

```sql
EXPLAIN SELECT *
FROM (SELECT de.emp_no FROM dept_emp de GROUP BY de.emp_no) tb, 
  employees e
WHERE e.emp_no=tb.emp_no;
```

위 쿼리는 FROM 절의 서브쿼리를 제거하고 조인으로 처리할 수 있다. **가능하면 DERIVED 형태의 실행 계획을 조인으로 해결할 수 있게 쿼리를 변경하는 것이 좋다.** MySQL 8.0부터는 FROM 절의 서브쿼리 최적화도 많이 개선되어 **불필요한 서브쿼리를 조인으로 재작성하는 최적화를 지원하지만**, **옵티마이저의 처리 능력에도 한계가 있으므로 최적화된 쿼리를 작성하는 것은 매우 중요하다.**

> 쿼리 튜닝을 위해 실행 계획을 분석한다면 select_type 컬럼이 DERIVED인 것을 먼저 확인하고 제거하자.
또한, 서브쿼리를 조인으로 해결할 수 있는 경우 조인을 사용하는 것을 강력히 권장한다.
>

### DEPENDENT DERIVED

MySQL 8.0 이전 버전에서는 FROM 절의 서브쿼리는 외부 컬럼을 사용할 수 없었는데, MySQL 8.0 부터는 래터럴 조인 기능이 추가되면서 FROM 절의 서브쿼리에서도 외부 컬럼을 참조할 수 있게됐다. **래터럴 조인을 사용하면 select_type 컬럼이 DEPENDENT DERIVED 키워드가 표시 된다.**

> 래터럴 조인이란?
특정 그룹별로 서브쿼리를 실행해서 그 결과와 조인하는 기능이다.
>

### UNCACHEABLE SUBQUERY

조건이 똑같은 서브쿼리가 실행될 때는 쿼리를 다시 실행하지 않고 이전의 실행 결과를 내부적인 캐시 공간에 담아두고 재사용하게 된다. 하지만 **서브쿼리에 포함된 요소에 의해 캐시 자체가 불가능한 경우 select_type이 UNCACHEABLE SUBQUERY로 표시된다.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1dd321dd-678f-4c3a-a45f-d1e545ce41b4/Untitled.png)

위 그림은 select_type이 SUBQUERY인 경우 캐시를 사용하는 방법을 나타내며 캐시가 처음 한번만 생성된 것을 알 수 있다. 다만, select_type이 DEPENDENT SUBQUERY인 경우 캐시 방식은 똑같지만 한 번만 캐시되는 것이 아니라 외부 쿼리의 값 단위로 캐시가 만들어진다.

- SUBQUERY
    - 한 번만 실행해서 그 결과를 캐시하고 필요할 때 캐시된 결과를 이용한다.
- DEPENDENT SUBQUERY
    - 의존하는 바깥쪽 쿼리의 컬럼 값 단위로 캐시해두고 사용한다.

캐시를 사용하지 못하게 하는 요소로는 다음과 같은 것들이 있다.

- 사용자 변수가 서브쿼리에 사용된 경우
- NOT-DETERMINISTIC 속성의 스토어드 루틴이 서브쿼리내에 사용된 경우
- UUID(), RAND() 같은 결과값이 호출할 때마다 달라지는 함수가 서브쿼리에 사용된 경우

### UNCACHEABLE UNION

UNION 수행 시 포함된 요소에 의해 캐시가 불가능한 경우 select_type이 UNCACHEABLE UNION로 표시된다.

### MATERIALIZED

MySQL 5.6 버전부터 도입된 select_type으로 주로 FROM 절이나 IN(subquery) 형태의 쿼리에 사용된 서브쿼리의 최적화를 위해 사용된다.

```sql
EXPLAIN 
SELECT *
FROM employees e
WHERE e.emp_no IN (SELECT emp_no FROM salaries WHERE salary BETWEEN 100 AND 1000);

+------+-------------+-------------+-------+-----------+------+--------------------------+
|id    |select_type  | table       | type  |key        | rows | Extra                    |
+------+-------------+-------------+-------+-----------+------+--------------------------+
| 1    |SIMPLE       | <subquery2> | ALL   | NULL      | NULL | NULL                     |
| 1    |SIMPLE       | e           | eq_ref| PRIMARY   |    1 | NULL                     |
| 2    |MATERIALIZED | salaries    | range | ix_salary |    1 | Using where; Using index |
+------+-------------+-------------+-------+-----------+------+--------------------------+
```

위 예제에서 MySQL 5.6 버전까지는 employees 테이블을 읽어서 레코드마다 salaries 테이블을 읽는 서브쿼리가 실행되는 형태로 처리됐다. 하지만 MySQL 5.7 버전부터는 서브쿼리의 내용을 임시 테이블로 구체화(Materialization)한 후, 임시 테이블과 employees 테이블을 조인하는 형태로 최적화되어 처리된다.

## table 컬럼

MySQL 서버의 실행 계획은 단위 SELECT 쿼리 기준이 아니라 테이블 기준으로 표시된다. 테이블의 이름에 별칭이 부여된 경우 별칭이 표시된다.

- 별도의 테이블을 사용하지 않는 SELECT 쿼리의 경우 table 컬럼이 NULL로 표시된다.
- <derived N> 또는 <union M, N> 으로 표시되는 것은 임시 테이블을 의미한다.
    - M, N은 단위 SELECT 쿼리의 id 값을 가리킨다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76f6f1bb-7097-4dd6-94f0-cbba0a7ec14c/Untitled.png)

지금까지 살펴본 **id 컬럼, select_type 컬럼, table 컬럼은 실행 계획의 각 라인에 명시된 테이블이 어떤 순서로 실행되는지 판단하는 근거를 표시해주는 역할을 한다.**

다음 예시를 통해 앞서 살펴본 3개의 컬럼을 이용해 테이블이 어떤 순서로 실행되는지 실행 계획을 분석해보자.

```sql
EXPLAIN
SELECT * 
FROM
  (SELECT de.empjio FROM dept_emp de GROUP BY de.emp_no) tb,
   employees e
WHERE e.emp_no = tb.emp_no;

+------+------------+------------+-------+--------------------+--------+-------------+
|id    |select_type | table      | type   |key                | rows   | Extra       |
+------+------------+------------+-------+--------------------+--------+-------------+
| 1    |PRIMARY     | <derived2> | ALL    | NULL              | 331143 | NULL        |
| 1    |PRIMARY     | e          | eq_ref | PRIMARY           |      1 | NULL        |
| 2    |DERIVED     | de         | index  | ix_empno_fromdate | 331143 | Using index |
+------+------------+------------+-------+--------------------+--------+-------------+
```

1. 첫 번째 테이블이 <derived2>이므로 id 값이 2인 라인이 먼저 실행되고 그 결과가 파생 테이블로 준비된다.
2. 세 번째 라인(id 값이 2인) select_type이 DERIVED 이므로 dept_emp 테이블을 읽어 파생 테이블을 생성한다.
3. 세 번째 라인의 분석이 끝났으니 다시 첫 번째 라인으로 돌아간다.
4. 첫 번째와 두 번째 id 값이 같은 것으로 보아 조인되는 쿼리라는 것을 알 수 있다.
   표시 순서에 따라 **첫 번째 라인이 드라이빙 테이블**이 되고 **두 번째 라인이 드리븐 테이블**이 된다는 것을 알 수 있다. 즉, <derived2> 테이블을 먼저 읽어서 e 테이블로 조인을 실행한 것으로 분석할 수 있다.

## partitions 컬럼

파티션을 참조하는 쿼리(파티션 키 컬럼을 WHERE 조건으로 가진)의 경우 옵티마이저가 쿼리 처리를 위해 필요한 파티션의 목록만 모아서 실행 계획의 partitions 컬럼에 표시해준다.

이해를 돕기위한 예제를 살펴보자. 우선, hire_date 컬럼값을 기준으로 5년 단위로 나누어진 파티션을 생성한다.

```sql
CREATE TABLE employees_2 (
  emp_no int NOT NULL,
  birth_date DATE NOT NULL, 
  first_name VARCHAR(14) NOT NULL, 
  last_name VARCHAR(16) NOT NULL, 
  gender ENUM('M','F') NOT NULL, 
  hire_date DATE NOT NULL, 
  PRIMARY KEY (emp_no, hire_date)
) PARTITION BY RANGE COLUMNS(hire_date) 
(PARTITION p1986_1990 VALUES LESS THAN ('1990-01-01'), 
 PARTITION p1991_1995 VALUES LESS THAN ('1996-01-01'), 
 PARTITION p1996_2000 VALUES LESS THAN ('2000-01-01'), 
 PARTITION p2O01_2005 VALUES LESS THAN ('2006-01-01'));

INSERT INTO employees_2 SELECT * FROM employees;
```

그리고 hire_date 컬럼 값이 1999-11-15 ~ 2000-01-15 사이인 레코드를 검색하는 쿼리를 살펴보자.

```sql
EXPLAIN 
SELECT *
FROM employees_2
WHERE hire_date BETWEEN '1999-11-15' AND '2000-01-15' ;
```

해당 쿼리에 필요한 데이터는 p1996_2000과 p2000_2006 파티션에 저장돼 있다. 실제로 옵티마이저는 hire_date 컬럼의 조건을 확인해 해당 파티션에 필요한 데이터가 들어있다는 것을 알아낸다. 따라서 실행 계획에서도 나머지 파티션에 대해서는 데이터 분포 등의 분석을 수행하지 않는다.

**파티션이 여러 개인 테이블에서 불필요한 파티션을 뺴고 접근해야 할 것으로 판단되는 테이블만 골라내는 과정을 파티션 프루닝(Partition pruning)이라고 한다.**

이제 위 쿼리의 실제 실행 계획을 살펴보자.

```sql
+------+------------+------------+------------------------+------+-------+
|id    |select_type | table      | partitions             | type | rows  | 
+------+------------+------------+------------------------+------+-------+
| 1    |SIMPLE     | employees_2 | p1996_2000, p2000_2006 | ALL  | 21743 |
+------+------------+------------+------------------------+------+-------+
```

실행 계획을 살펴보면 쿼리 처리를 위해 필요한 파티션 목록이 partitions 컬럼에 표시된 것을 확인할 수 있다. 한 가지 특이한 점은 type의 값이 ALL(풀 테이블 스캔)이라는 것이다.

어떻게 풀 테이블 스캔으로 테이블의 일부만 읽을 수 있는 것일까?

그 이유는 대부분의 RDBMS에서 지원하는 파티션은 물리적으로 개별 테이블처럼 별도의 저장 공간을 가지기 때문이다. 따라서 해당 쿼리의 경우 employees_2 테이블의 모든 파티션이 아니라 p1996_2000 파티션과 p2000_2006 파티션만 풀 스캔을 실행한 것이다.
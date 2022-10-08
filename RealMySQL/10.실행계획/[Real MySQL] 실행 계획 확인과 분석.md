# [Real MySQL 8.0] 실행 계획 확인과 분석

# 실행 계획 확인

MySQL 서버의 실행 계획은 DESC 또는 EXPLAIN 명령으로 확인할 수 있다.
MySQL 8.0부터는 실행 계획의 출력 포맷과 실제 쿼리의 실행 결과까지 확인할 수 있는 기능이 추가됐다.

## 실행 계획 출력 포맷

MySQL 8.0 이전 버전에서는 EXPLAIN EXTENDED 또는 EXPLAIN PARTITIONS 명령이 구분돼 있었지만, 해당 옵션은 현재(MySQL 8.0) 문법에서 제거됐다.

MySQL 8.0부터는 **FORMAT 옵션을 사용해** 실행 계획의 표시 방법을 **JSON이나 TREE, 단순 테이블 형태**로 선택할 수 있다.

EXPLAIN의 기본값은 테이블 형태 출력이다.

```sql
// 테이블 형태로 출력
EXPLAIN 
SELECT *
FROM employees e
	INNER JOIN salaries s ON s.emp_no=e.emp_no
WHERE first_name='ABC';

+----+------------+--------+----------+-----+---------------------+-------------+--------+------+-----+---------+------+
|id  |select_type | table |partitions |type |possible_keys        |key          |key_len |ref   |rows |filtered |Extra |
+----+------------+--------+----------+-----+---------------------+-------------+--------+------+-----+---------+------+
| 1  |SIMPLE      | e     | NULL      |ref  |PRIMARY,ix_firstname |ix_firstname |58      |const | 1   | 100.00  | NULL |
| 1  |SIMPLE      | s     | NULL      |ref  |PRIMARY              |PRIMARY      |4       |const | 10  | 100.00  | NULL |
+----+------------+--------+----------+-----+---------------------+-------------+--------+------+-----+---------+------+
```

```sql
// 트리 형태로 출력
EXPLAIN FORMAT=TREE 
SELECT *
FROM employees e
	INNER JOIN salaries s ON s.emp_no=e.emp_no
WHERE first_name='ABC'\G

*************************** 1. row ***************************
EXPLAIN:-> Nested loop inner join (cost=2.40 rows=10)
-> Index lookup on e using ix_firstname (first_name='ABC') (cost=0.35 rows=1) 
-> Index lookup on s using PRIMARY(emp_no=e.emp_no) (cost=2.05 rows=10)
```

```sql
// JSON 형태로 출력
EXPLAIN FORMAT=JSON 
SELECT *
FROM employees e
	INNER JOIN salaries s ON s.emp_no=e.emp_no
WHERE first_name='ABC'\G

*************************** 1, row ***************************
EXPLAIN:{ 
    "query_block" : {
        "select_id": 1, 
        "cost_info" : {
            "query_cost" :"2.40"
        }, 
        "nested_loop":[
        {
            "table": {
                "table.name":"e", 
                "access_type":"ref", 
                "possible_keys":[
                    "PRIMARY",
                    "ix_firstname"
                ],
                "key":"ix_firstnam e", 
                "used_key_parts":[
                    "first_name"
                ],
......
```

## 쿼리의 실행 시간 확인

MySQL 8.0.18 버전부터 EXPLAIN ANALYZE 기능이 추가돼 쿼리의 실행 계획과 단계별 소요된 시간 정보를 확인할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0cc6636-535a-47d1-ade7-e9e9fea6b288/Untitled.png)

실제 실행 순서는 다음 기준으로 읽으면 된다.

- 들여쓰기가 같은 레벨에서는 상단에 위치한 라인이 먼저 실행
- 들여쓰기가 다른 레벨에서는 가장 안쪽에 위치한 라인이 먼저 실행

따라서 위 쿼리는 다음과 같은 실행 순서를 가진다.

```sql
1. D) Index lookup on e using ix_firstname 
2. F) Index lookup on s using PRIMARY
3. E) Filter
4. C) Nested loop inner join
5. B) Aggregate using temporary table 
6. A) Table scan on <temporary>

SELECT e.emp_no, avg(s.salary) 
FROM employees e
	INNER JOIN salaries s ON s.emp_no=e.emp_no 
						AND s.salary>50000
						AND s.from_date<= '1990-01-01'
						AND s.to_date>'1990-01-01' 
WHERE e.first_name='Matt'
GROUP BY e.hire_date \G

1. employee 테이블에서 ix_firstname 인덱스를 이용해 first_name = 'Matt' 인 레코드를 찾는다.
2. salaries 테이블의 PRIMARY 키를 통해 emp_no가 1번 결과와 동일한 레코드를 찾는다.
3. INNER JOIN의 조건에 일치하는 건만 가져온다.
4. 1번과 3번의 결과를 조인한다.
5. 임시 테이블에 결과를 저장하면서 GROUP BY 집계를 실행한다.
6. 임시 테이블의 결과를 읽어서 결과를 반환한다.
```

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
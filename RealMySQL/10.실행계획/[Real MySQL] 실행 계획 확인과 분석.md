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
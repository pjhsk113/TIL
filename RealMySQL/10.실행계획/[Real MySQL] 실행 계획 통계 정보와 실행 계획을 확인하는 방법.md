# [Real MySQL 8.0] 실행 계획 통계 정보와 실행 계획을 확인하는 방법

사용자의 쿼리를 최적으로 처리될 수 있게 하려면 쿼리의 **실행 계획을 수립**할 수 있어야 한다. 이에 가장 큰 영향을 미치는 정보는 바로 통계 정보이다.

# 통계 정보

MySQL 5.7 버전까지 **테이블과 인덱스에 대한 개략적인 정보를 가지고 실행 계획을 수립**했다. 하지만 테이블 컬럼의 값들이 어떻게 분포돼 있는지에 대한 정보가 없기 떄문에 실행 계획의 정확도가 떨어지는 경우가 많았다.

이러한 문제를 해결하기 위해 MySQL8.0부터는 데이터 분포도를 수집해 저장하는 **히스토그램 정보**가 도입됐다.

## 테이블 및 인덱스 통계 정보

비용 기반 최적화에서 가장 중요한 것은 통계 정보다.
통계 정보가 정확하지 않다면 엉뚱한 방향으로 쿼리를 실행할 수 있기 때문이다.

예를 들어, 1억 건의 레코드가 저장된 테이블의 통계 정보가 갱신되지 않아서 10건 미만인 것처럼 통계 정보가 구성되어 있다면 부정확한 통계 정보 때문에 0.1초에 끝날 쿼리가 1시간이 소요될 수도 있다.

### MySQL 서버의 통계 정보

MySQL 5.5 버전까지는 통계 정보가 메모리에만 관리되었기 때문에 MySQL 서버가 재시작된 경우 통계 정보가 모두 사라지고 모든 테이블의 통계 정보는 다시 수집돼야 했다. 따라서 MySQL 5.6 버전부터는 테이블에 대한 통계 정보를 영구적으로 관리할 수 있도록 개선되었다.

테이블을 생성할 때 STATS_PERSISTENT 옵션을 설정해 테이블 단위로 영구적인 통계 정보를 보관할지 결정할 수 있다.

- STATS_PERSISTENT = 0
  - 통계 정보를 메모리에만 관리(MySQL 5.5버전과 동일한 방식)
- STATS_PERSISTENT = 1
  - 통계 정보를 innodb_index_stats와 innodb_table_stats 테이블에 저장
- STATS_PERSISTENT = DEFAULT
  - 테이블 통계 정보 영구적 관리 여부를 innodb_stats_persistent 시스템 변수 값으로 결정


만약 영구적 통계 정보를 사용하고자 한다면 **innodb_stats_auto_recalc** 시스템 설정 변수의 기본 값을 OFF로 설정해 통계 정보 자동 갱신을 막는 것이 좋다.

영구적인 통계 정보를 사용할 때 더 정확한 통계 정보를 수집하고자 한다면 **innodb_stats_persistent_sample_pages** 시스템 변수에 높은 값을 설정해 쿼리 성능을 끌어올릴 수 있다. 하지만 이 값이 너무 높으면 정보 수집 시간이 길어지므로 주의해야 한다.

## 히스토그램

MySQL 5.7 버전까지는 단순히 인덱스 컬럼의 유니크한 값의 개수 정도만 통계 정보로 가지고 있었다. 하지만 옵티마이저가 최적의 실행 계획을 수립하기에는 이 정보만으로는 많이 부족했다. 이러한 부족한 정보를 보완하기 위해 MySQL 8.0부터는 데이터 분포도를 참조할 수 있는 히스토그램 정보를 활용할 수 있게 됐다.

### 히스토그램 정보 수집 및 삭제

히스토그램 정보는 컬럼 단위로 관리되는데, 이 정보는 자동으로 수집되지 않고 ANALZE TABLE … UPDATE HISTOGRAM 명령으로 수동으로 수집 및 관리된다. 수집된 히스토그램 정보는 시스템 딕셔너리에 함께 저장되고 MySQL 서버가 시작될 때 딕셔너리의 히스토그램 정보를 information_schema 데이터베이스의 column_statistics 테이블로 로드한다. 히스토그램 정보 수집 과정을 정리하면 다음과 같다.

1. ANALZE TABLE … UPDATE HISTOGRAM 명령으로 히스토그램 수동 수집 및 관리
2. 수집된 히스토그램 정보는 시스템 딕셔너리에 저장
3. MySQL 서버 시작 시 딕셔너리 히스토그램 정보를 information_schema 데이터베이스의 column_statistics 테이블로 로드
4. 실제 히스토그램 정보 조회는 column_statistics 테이블 조회

```sql
> ANALYZE TABLE employees.employees UPDATE HISTOGRAM ON gender, hire_date;

> SELECT * FROM COLUMN_STATISTICS 
WHERE SCHEMA_NAME='employees' AND TABLE_NAME='employees'\G

```

MySQL 8.0 버전에서는 2종류의 히스토그램 타입이 지원된다.

- 싱글턴 히스토그램(Singleton)
  - 컬럼값 개별로 레코드 건수를 관리하는 히스토그램
  - 컬럼이 가지는 값별로 버킷 할당
  - 각 버킷이 **컬럼의 값과 발생 빈도의 비율**, 2개 값을 가짐
  - Value-Based 히스토그램 또는 도수 분포라고 불린다.

    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/207960ef-4a20-4389-bff8-1b7d593386fd/Untitled.png)


- 높이 균형 히스토그램(Equi-Height)
  - 컬럼값 범위를 균등한 개수로 구분해 관리하는 히스토그램
  - 개수가 균등한 컬럼값의 범위별로 버킷 할당
  - 각 버킷이 **범위 시작 값, 마지막 값, 발생 빈도율, 버킷에 포함된 유니크 값의 개수** 등 4개 값을 가짐
  - Height-Balanced 히스토그램이라고 불린다.

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76137f18-f49d-4a01-93ca-3d018fc1654a/Untitled.png)


### 히스토그램의 용도

히스토그램이 도입되기 이전 MySQL 서버에도 테이블과 인덱스에 대한 통계 정보는 존재했다. 하지만 테이블의 전체 레코드 건수와 인덱스된 컬럼이 가지는 유니크 값의 개수 정도만 통계 정보로 가지고 있었고, 실제 응용 프로그램의 데이터는 항상 균등한 분포를 가지지 않는다는 점을 기존 MySQL 서버는 고려하지 못했다.

히스토그램은 범위 별로 레코드의 건수와 유니크한 값의 개수 정보를 가져 훨씬 정확한 예측을 할 수 있으므로 이러한 단점을 보완하기 위해 MySQL 8.0 버전부터 히스토그램이 도입됐다.

예를 들어, 성이 Zita이고 1950년대 출생인 사람이 143명인 테이블 데이터에서 다음과 같은 쿼리가 있다고 생각해보자.

```sql

// 히스토그램이 없을 때 예측치
> EXPLAIN SELECT *
FROM employees
WHERE first_name='Zita'
AND birth_date BETWEEN '1950-01-01' AND '1960-01-01';

+----+------------+----------+------+--------------+------+---------+
| id |select_type | table    | type | key          | rows | filtered|                      |
+----+------------+----------+------+--------------+------+---------+
| 1  | SIMPLE     | employees| ref  | ix_firstname | 224  | 11.11   |
+----+------------+----------+--------------------------------------+
```

히스토그램이 없을 때 실행 계획에서는 first_name=’Zita’ 조건 일치 224건중 11.11%(24.8명)만 1950년대 출생일 것으로 예측했다.

```sql
// 히스토그램 정보 수집
> ANALYZE TABLE employees
UPDATE histogram ON first_name, birth_date;

> EXPLAIN SELECT *
FROM employees
WHERE first_name='Zita'
AND birth_date BETWEEN '1950-01-01' AND '1960-01-01';

+----+------------+----------+------+--------------+------+---------+
| id |select_type | table    | type | key          | rows | filtered|                      |
+----+------------+----------+------+--------------+------+---------+
| 1  | SIMPLE     | employees| ref  | ix_firstname | 224  | 60.82   |
+----+------------+----------+--------------------------------------+
```

반면에 동일한 쿼리임에도 히스토그램 정보를 수집한 후 예측치는 60.82%(136.2명)로 상승했고, 실제 조회 결과(143명)와 예측치(136.2명)가 크게 차이가 안나는 것을 확인할 수 있다. 이렇듯 단순 통계 정보만 이용한 경우와 히스토그램을 이용한 경우 차이가 매우 큰 것을 알 수 있다.

이뿐만아니라 히스토그램 정보는 조인 쿼리에도 성능에 상당한 영향을 미칠 수 있다. 조인 시 드라이빙 테이블을 선택할 때 옵티마이저가 통계 정보를 활용해 결정하기 때문이다.

### 히스토그램과 인덱스

히스토그램과 인덱스는 완전히 다른 객체이기 때문에 비교 대상은 아니지만, 인덱스는 부족한 통계 정보를 수집하기 위해 사용된다는 측면에서 어느정도 공통점을 가진다고 볼 수 있다.

MySQL 서버는 쿼리의 실행 계획을 수립할 때 사용 가능한 인덱스들로부터 조건절에 일치하는 레코드 건수를 대략 파악하고 최종적으로 가장 나은 실행 계획을 선택한다. 이때, 조건에 일치하는 레코드 건수를 예측하기 위해 옵티마이저는 실제 인덱스의 B-Tree를 샘플링해서 살펴보는데 이를 **인덱스 다이브(Index Dive)**라고 표현한다.

## 코스트 모델

MySQL 서버가 쿼리를 처리하려면 다음과 같은 작업을 필요로 한다.

- 디스크로부터 데이터 페이지 읽기
- 메모리로부터 데이터 페이지 읽기
- 인덱스 키 비교
- 레코드 평가
- 메모리 임시 테이블 작업
- 디스크 임시 테이블 작업

MySQL 서버는 쿼리에 대해 이러한 작업이 얼마나 필요한지 예측하고 전체 작업 비용을 계산한 결과를 바탕으로 실행 계획을 수립한다. 이렇게 **전체 쿼리 비용을 계산하는데 필요한 단위 작업들의 비용을 코스트 모델**이라고 한다.

**MySQL 5.7 이전 버전**까지는 이런 작업들의 비용을 MySQL **서버 소스 코드에 상수화해서 사용**했다. 하지만 사용하는 하드웨어에 따라 비용은 달라질 수 있기 때문에 비용을 상수화해 적용하는 것은 최적의 실행 계획 수립에 방해 요소였다.

MySQL 5.7 버전부터 각 단위 작업의 비용을 DMBS 관리자가 조정할 수 있게 개선하여 이러한 단점을 개선했지만, 인덱스되지 않은 컬럼의 데이터 분포나 메모리에 올라가있는 페이지의 비율 등 비용 계산과 연관된 부분의 정보는 여전히 부족한 상태였다. 이러한 정보는 MySQL 8.0 버전에서야 비로소 히스토그램(데이터 분포)과 인덱스별 메모리의 페이지 비율이 관리되므로써 옵티마이저의 실행 계획 수립에 사용되기 시작했다.

- row_evaluate_cost
  - 스토리지 엔진이 반환한 레코드가 쿼리의 조건에 일치하는지 평가하는 단위 작업
  - 값이 증가할수록 풀 테이블 스캔과 같이 많은 레코드를 처리하는 쿼리의 비용이 높아짐
  - 값이 증가할수록 레인지 스캔과 같이 상대적으로 적은 수의 레코드를 처리하는 쿼리의 비용은 낮아짐
- key_compare_cost
  - 키 값의 비교 작업에 필요한 비용
  - 값이 증가할수록 레코드 정렬과 같이 키 값 비교 처리가 많은 경우 쿼리 비용이 높아짐


```sql
EXPLAIN FORMAT = TREE
SELECT *
FROM employees WHERE first_name='Matt' \G
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d2df86ea-7ec4-4f0c-b9fa-b5fee8b30611/Untitled.png)

코스트 모델에서 중요한 것은 각 단위 작업에 설정되는 비용 값이 커지면 어떤 실행 계획들이 고비용으로 바뀌고 어떤 실행 계획들이 저비용으로 바뀌는지를 파악하는 것이다. 아래의 예제는 개략적으로 코스트 모델을 이해하고 각 단위 작업의 비용 조절을 연습해볼 수 있는 기준이다.

- key_compare_cost
  - 해당 비용을 높이면 MySQL 서버 옵티마이저가 가능하면 정렬을 수행하지 않는 방향의 실행 계획을 선택할 가능성이 높아짐
- row_evaluate_cost
  - 해당 비용을 높이면 풀 스캔을 실행하는 쿼리들의 비용이 높아지고 MySQL 서버 옵티마이저는 가능하면 인덱스 레인지 스캔을 사용하는 실행 계획을 선택할 가능성이 높아짐
- disk_temptable_create_cost와 disk_temptable_row_cost
  - 해당 비용을 높이면 MySQL 옵티마이저는 디스크에 임 시 테이블을 만들지 않는 방향의 실행 계획을 선택할 가능성이 높아짐
- memory_temptable_create_cost와 memory_temptable_row_cost
  - 해당 비용을 높이면 MySQL 서버 옵티마이저는 메모리 임시 테이블을 만들지 않는 방향의 실행 계획을 선택할 가능성이 높아짐
- io_block_read_cost
  - 해당 비용이 높아지면 MySQL 서버 옵티마이저는 가능하면 InnoDB 버퍼 풀에 데이터 페이지가 많이 적재돼 있는 인덱스를 사용하는 실행 계획을 선택할 가능성이 높아짐
- memory_block_read_cost
  - 해당 비용이 높아지면 MySQL 서버는 InnoDB 버퍼 풀에 적재된 데이터 페이지가 상대적으로 적다고 하더라도 그 인덱스를 시용할 가능성이 높아짐

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
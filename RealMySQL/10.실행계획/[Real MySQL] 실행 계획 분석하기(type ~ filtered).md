# [Real MySQL 8.0] 실행 계획 분석하기(type ~ filtered)

## type 컬럼

쿼리의 실행 계획에서 type 이후의 컬럼은 테이블 레코드를 어떤 방식으로 읽었는지를 나타낸다. 일반적으로 **쿼리 튜닝에서 인덱스를 효율적으로 사용하는지 확인하는 것이 중요하므로 type 컬럼은 반드시 체크해야 하는 중요한 정보다.**

아래의 12개 접근 방법은 type 컬럼에 표시될 수 있는 값들 중 성능이 빠른 순서대로 나열한 것이다.

- system
- const
- eq_ref
- ref
- fulltext
- ref_or_null
- unique_subquery
- index_subquery
- range
- index_merge
- index
- ALL

마지막 ALL을 뺀 나머지는 모두 index를 사용한 접근 방법이며, index_merge를 제외한 나머지 접근 방법은 하나의 인덱스만 사용한다. 따라서 **type 컬럼에도 라인 별로 하나의 인덱스 이름만 표시된다.**

### system

레코드가 1건만 존재하는 테이블 또는 한 건도 존재하지 않는 테이블을 참조하는 형태의 접근 방법이다. InnoDB 스토리지 엔진에서는 나타나지 않고 MyISAM이나 MEMORY에서만 사용되는 접근 방법이다.

### const

테이블 레코드 건수와 관계없이 **쿼리가 프라이머리 키나 유니크 키 컬럼을 이용하는 where 조건절을 가지고있으며, 반드시 1건을 반환하는 쿼리의 처리 방식**을 const라고 한다.

```sql
EXPLAIN
SELECT * FROM employees WHERE emp_no = 10001;

+------+------------+------------+------+---------+---------+
|id    |select_type | table      | type | key     | key_len |
+------+------------+------------+------+---------+---------+
| 1    |SIMPLE      | employees  | const| PRIMARY |       4 |
+------+------------+------------+------+---------+---------+
```

이러한 방식을 다른 DBMS에서는 **유니크 인덱스 스캔**이라고 표현한다.

```sql
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no='d005' ;

+------+------------+------------+------+---------+---------+
|id    |select_type | table      | type | key     | rows    |
+------+------------+------------+------+---------+---------+
| 1    |SIMPLE      | dept_emp   | ref  | PRIMARY | 165571  |
+------+------------+------------+------+---------+---------+
```

다중 컬럼으로 구성된 프라이머리 키나 유니크 키 중에서 인덱스의 일부 컬럼만 조건으로 사용할 때는 const 타입의 접근 방법을 사용할 수 없다. MySQL 엔진이 데이터를 읽어보지 않고서는 레코드가 1건이라는 것을 확신할 수 없기 때문이다. 따라서 프라이머리 키의 일부만 조건으로 사용할 때는 type 컬럼에 ref로 표시된다.

하지만 다음과 같이 **프라이머리 키나 유니크 인덱스의 모든 컬럼을 동등 조건으로 WEHRE 절에 명시하면 const 접근 방법을 사용**한다.

```sql
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no='d005' and emp_no=10001;

+------+------------+------------+------+---------+---------+------+
|id    |select_type | table      | type | key     | key_len | rows |
+------+------------+------------+------+---------+---------+------+
| 1    |SIMPLE      | dept_emp   | const| PRIMARY | 20      |    1 |
+------+------------+------------+------+---------+---------+------+
```

실행 계획의 type 컬럼이 const(상수)로 표시되는 이유는 MySQL의 옵티마이저가 쿼리를 최적화하는 단계에서 쿼리를 먼저 실행해 통째로 상수화 하기 때문이다.

### eq_ref

eq_ref 접근 방법은 여러 테이블이 조인되는 쿼리의 실행 계획에서만 표시된다.
**드라이빙 테이블의 컬럼 값을 드리븐 테이블의 프라이머리 키나 유니크 키 컬럼의 검색 조건에 사용할 때를 가리켜 eq_ref라고 한다.** 이때 두 번째 이후에 읽는 테이블의 type 컬럼에 eq_ref가 표시된다.

- eq_ref 접근 방법의 제약 조건
  - 두 번째 이후에 읽히는 테이블을 유니크 키로 검색할 때, 그 유니크 인덱스는 NOT NULL이어야 한다.
  - 다중 컬럼으로 만들어진 PK나 유니크 인덱스라면 인덱스의 모든 컬럼이 비교 조건에 사용돼야만 eq_ref 접근 방법이 사용될 수 있다.
  - 두 번째 이후에 읽는 테이블에서 반드시 1건의 레코드만 반환한다는 보장이 있어야 한다.

이해를 돕기위해 다음 예제를 살펴보자.

```sql
EXPLAIN
SELECT * FROM dept_emp de, employees e
WHERE e.emp_no=de.emp_no AND de.dept_no='d005';

+------+------------+-------+-------+---------+---------+--------+
|id    |select_type | table | type  | key     | key_len | rows   |
+------+------------+-------+-------+---------+---------+--------+
| 1    |SIMPLE      | de    | ref   | PRIMARY | 16      | 165571 |
| 1    |SIMPLE      | e     | eq_ref| PRIMARY | 4       |      1 |
+------+------------+-------+-------+---------+---------+--------+
```

id 값이 같은 것을 보아 조인으로 실행된다는 것을 알 수 있고, 실행 계획에서 de 테이블이 위쪽에 있으므로 드라이빙 테이블이 된다는 것을 알 수 있다. 이는 dept_emp 테이블을 먼저 읽고 `e.emp_no = de.emp_no` 조건을 이용해 employees 테이블을 검색한다는 것을 나타낸다. 이때, emp_no는 employees 테이블의 PK이므로 두 번째 라인에서 eq_ref가 표시된 것을 확인할 수 있다.

### ref

eq_ref와는 다르게 조인의 순서와 관계없이 사용되며, PK나 유니크 키 등의 제약 조건이 없다. 인덱스의 종류와 관계없이 동등 조건으로 검색할 때는 ref 접근 방법이 사용된다.

반환되는 레코드가 반드시 1건이라는 보장이 없으므로 const나 eq_ref보다는 느리지만, 동등 조건으로만 비교되므로 매우 빠른 레코드 조회 방법 중 하나다.

```sql
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no='d005' ;

+------+------------+----------+-----+---------+---------+-------+
|id    |select_type | table    | type| key     | key_len | ref   |
+------+------------+----------+-----+---------+---------+-------+
| 1    |SIMPLE      | dept_emp | ref | PRIMARY | 16      | const |
+------+------------+----------+-----+---------+---------+-------+
```

위 예제에서는 dept_emp의 PK를 구성하는 컬럼(dept_no, emp_no)의 일부만 동등 조건으로 사용됐기 때문에 일치 레코드가 1건이라는 보장이 없다. 따라서 const가 아닌 ref 접근 방법이 사용되었다.

지금까지 살펴본 실행 계획의 type에 대해 다시 한번 정리해보면 다음과 같다.

- const
  - 조인 순서에 상관없이 PK나 유니크 키의 모든 컬럼에 대해 동등 조건으로 검색(1건의 레코드만 반환)
- eq_ref
  - 드라이빙 테이블의 컬럼 값을 이용해 드리븐 테이블의 PK나 유니크 키로 동등 조건 검색(1건의 레코드만 반환)
- ref
  - 조인 순서와 인덱스의 종류에 상관없이 동등 조건으로 검색(1건의 레코드만 반환된다는 보장이 없어도 됨)

이 세 가지 접근 방법 모두 WHERE 조건절에 **동등 비교 연산자(=, <=>)**를 사용해야 한다는 공통점이 있다. 또한, 세 가지 모두 매우 좋은 접근 방법으로 성능상의 문제를 일으키지 않는 접근 방법이다.

### fulltext

MySQL 서버의 전문 검색 인덱스를 사용해 레코드를 읽는 접근 방법을 의미한다.
전문 검색은 `MACTH (…) AGAINST (…)` 구문을 사용해 실행하는데, 이때 반드시 전문 검색용 인덱스가 준비돼 있어야 한다. 테이블에 전문 검색 인덱스가 없다면 쿼리는 오류가 발생하고 중지된다.

### ref_or_null

ref 접근 방법과 같지만, NULL 비교가 추가된 형태다. ref 접근 방법 또는 NULL 비교(IS NULL) 접근 방법을 의미한다.

```sql
EXPLAIN
SELECT * FROM titles
WHERE to_date='1985-03-01' OR to_date IS NULL;
```

### unique_subquery

WHERE 조건절에서 사용될 수 있는 IN(subquery) 형태의 쿼리를 위한 접근 방법이다.
의미 그대로 서브쿼리에서 중복되지 않는 유니크한 값만 반환할 때 이 접근 방법을 사용한다.

```sql
EXPLAIN
SELECT * FROM departments
WHERE dept_no IN (SELECT dept_no FROM dept_emp WHERE emp_no=10001) ;
```

### index_subquery

IN 연산자 특성상 중복된 값이 먼저 제거돼야 한다.
앞서 살펴본 unique_subquery의 경우 서브쿼리가 중복된 값을 반환하지 않는다는 보장이 있으므로 중복을 제거할 필요가 없었지만, 일반적으로 IN(subquery) 형태의 쿼리는 서브쿼리가 중복된 값을 반환할 수 있기 때문에 인덱스를 이용해 중복을 할 수 있는 경우 별도의 중복 제거를 수행한다.

**IN(subquery) 형태의 쿼리에서 인덱스를 이용해 서브쿼리의 중복된 값을 제거할 수 있을 때 index_subquery가 표시된다.**

- unique_subquery
  - IN(subquery) 형태의 조건에서 서브쿼리의 반환 값에는 중복이 없으므로 중복 제거 작업이 필요없다.
- index_subquery
  - IN(subquery) 형태의 조건에서 서브쿼리의 반환 값에 중복이 있을 수 있지만 인덱스를 이용해 중복된 값을 제거할 수 있다.


### range

인덱스 레인지 스캔 형태의 접근 방법이다.
인데스를 하나의 값이 아니라 범위로 검색하는 경우를 의미한다.
주로 `<, >, IS NULL, BETWEEN, IN, LIKE` 등의 연산자를 이용해 인덱스를 검색할 때 이용된다.

MySQL 서버가 가지고 있는 접근 방법의 우선순위는 낮지만, range 접근 방법도 상당히 빠른 접근 방법이며 해당 접근 방법만 사용해도 최적의 성능이 보장된다고 볼 수 있다.

```sql
EXPLAIN
SELECT * FROM employees WHERE emp_no BETWEEN 10002 AND 10004;

+------+------------+----------+------+---------+---------+------+
|id    |select_type | table    | type | key     | key_len | rows |
+------+------------+----------+------+---------+---------+------+
| 1    |SIMPLE      | employees| range| PRIMARY | 4       |    3 |
+------+------------+----------+------+---------+---------+------+
```

### index_merge

index_merge 접근 방법은 2개 이상의 인덱스를 이용해 각각의 검색 결과를 만들어낸 후, 그 결과를 병합해 처리하는 방식이다. 하지만 다음과 같은 특징 때문에 이름만큼 그렇게 효율적으로 작동하는 것은 아니다.

- 여러 인덱스를 읽어야 하므로 range 접근 방법보다 효율성이 떨어진다.
- 전문 검색 인덱스를 사용하는 쿼리에는 적용되지 않는다.
- 항상 2개이상의 집합이 되기 때문에 교집합이나 합집합, 또는 중복 제거와 같은 부가적인 작업이 더 필요하다.

```sql
EXPLAIN
SELECT * FROM employees
WHERE emp_no BETWEEN 10001 AND 11000 OR first_name='Smith' ;

+------+------------+----------------------+---------+-------------------------------------------------+
|id    |select_type | key                  | key_len | Extra                                           |
+------+------------+----------------------+---------+-------------------------------------------------+
| 1    |index_merge | PRIMARY, ix_firstname| 4, 58   | Using union(PRIMARY, ix_firstname); Using where |
+------+------------+----------------------+---------+-------------------------------------------------+
```

위 쿼리에서 `emp_no BETWEEN 10001 AND 11000` 조건은 employees 테이블의 PK를 이용해 조회하고 `first_name='Smith'` 조건은 ix_firstname 인덱스를 이용해 조회한 후, 두 결과를 병합하는 형태로 실행 계획을 수립한다.

### index

index 접근 방법은 인덱스를 처음부터 끝까지 읽는 인덱스 풀 스캔을 의미한다.
**접근 방법의 이름 때문에 효율적으로 인덱스를 사용하는 것으로 자주 오해하는 접근 방법이다.**

range 접근 방법과 같이 인덱스의 필요 부분만 읽는 것이 아니라 **인덱스를 풀 스캔하는 접근 방법이라는 것을 잊지 말자.** 인덱스는 데이터 파일보다 크기가 작기 때문에 빠르게 처리되고 정렬이라는 인덱스의 장점을 활용할 수 있기 때문에 테이블을 풀 스캔하는 것보다 훨씬 효율적이라 할 수 있다.

index 접근 방법은 다음 조건 중 `1번 + 2번` 조건을 충족하거나 `1번 + 3번` 조건을 충족하는 쿼리에서 사용된다.

1. range나 const, ref 같은 접근 방법으로 인덱스를 사용할 수 없는 경우
2. 인덱스에 포함된 컬럼만으로 처리할 수 있는 경우(커버링 인덱스)
3. 인덱스를 이용해 정렬이나 그루핑 작업이 가능한 경우(별도의 정렬이 필요 없는 경우)

### ALL

풀 테이블 스캔을 의미하는 접근 방법이다.
테이블을 처음부터 끝까지 읽어 불필요한 레코드를 제거한 후 반환한다.
앞서 살펴본 접근 방법으로 처리할 수 없을 때 가장 마지막에 선택되는 가장 비효율적인 방법이다.

다만, 데이터 웨어하우스나 배치 프로그램처럼 대량의 레코드를 처리하는 쿼리에서는 잘못 튜닝된 쿼리보다 더 나은 접근 방법이 될 수 있다. 하지만 일반적으로 빠른 응답을 사용자에게 보내야하는 웹 서비스와 같은 온라인 트랜잭션 처리 환경에서는 적합하지 않다.

> 리드 어헤드(Read Ahead)
InnoDB에서 제공하는 대량의 디스크 I/O를 유발하는 작업을 위해 한번에 많은 페이지를 읽어 들이는 기능이다.
MySQL 서버에서는 백그라운드 읽기 스레드가 최대 64개의 페이지씩 한번에 디스크로 읽어 들이므로 빠르게 레코드를 읽을 수 있다.
>

## possible_keys 컬럼

possible_keys 컬럼은 옵티마이저가 **최적의 실행 계획을 만들기 위해 후보로 선정했던 접근 방법에서 사용되는 인덱스의 목록**을 나타낸다. 즉, **사용될 법한 인덱스의 목록**인 것이다.

따라서 possible_keys 컬럼은 쿼리를 튜닝하는데 큰 도움이 되지 않는다. 특별한 경우를 제외하고는 그냥 무시해도 된다. **possible_keys 컬럼에 인덱스가 나열됐다고 해서 그 인덱스를 사용한다고 판단하는 일이 없도록 주의하자.**

## key 컬럼

key 컬럼에 표시되는 인덱스는 최종 선택된 실행 계획에서 사용하는 인덱스를 의미한다.
즉, 실제 사용된 인덱스를 나타낸다.

따라서 쿼리 튜닝을 할 때 의도했던 인덱스가 key 컬럼에 표시되는지 확인하는 것이 중요하다.
type 컬럼이 index_merge인 경우를 제외한 나머지 경우에는 테이블당 하나의 인덱스만 이용할 수 있다.(index_merge는 2개 이상의 인덱스가 사용됨)

만약 인덱스를 사용하지 못한다면 key 컬럼은 NULL로 표시된다.

## key_len 컬럼

key_len 컬럼은 쉽게 무시하는 정보지만 실제로는 매우 중요한 정보를 나타낸다.
key_len 컬럼 값은 쿼리를 처리하기 위해 다중 컬럼으로 구성된 인덱스에서 몇 개의 컬럼까지 사용했는지 알려준다. 더 **정확하게는 인덱스의 각 레코드에서 몇 바이트까지 사용했는지 알려준다**.

다음 예제를 살펴보자. PK는 dept_no, emp_no로 구성됐고 dept_emp 테이블을 조회하는 쿼리다.

```sql
EXPLAIN
SELECT * FROM dept_nmp
WHERE dept_no='d005';

+------+------------+----------+------+---------+---------+
|id    |select_type | table    | tpye | key     | key_len |
+------+------------+----------+------+---------+---------+
| 1    |SIMPLE      | dept_emp | ref  | PRIMARY | 16      |
+------+------------+----------+------+---------+---------+
```

위 쿼리는 PK 중 dept_no만 비교에 사용한다. key_len이 16으로 표시된 이유는 dept_no의 컬럼 타입은 CHAR(4)이므로 PK에서 앞쪽 16바이트만 유효하게 사용했다는 것을 나타내기 때문이다.

문자를 위해 메모리 공간에 할당해야하는 4바이트와 실제 dept_no 값의 길이 4바이트를 곱해 16이라는 값이 표시된 것이다.

추가로 emp_no 조건을 추가하면 key_len 컬럼 값은 다음과 같이 변경된다.

```sql
EXPLAIN
SELECT * FROM dept_nmp
WHERE dept_no='d005' AND emp_no=10001;

+------+------------+----------+------+---------+---------+
|id    |select_type | table    | tpye | key     | key_len |
+------+------------+----------+------+---------+---------+
| 1    |SIMPLE      | dept_emp | const| PRIMARY | 20      |
+------+------------+----------+------+---------+---------+
```

emp_no는 Integer 타입으로 4바이트 메모리 공간을 차지한다.
따라서 detp_no 컬럼의 길이와 emp_no 컬럼의 길이를 합친 결과인 20이 표시되는 것이다

## ref 컬럼

접근 방법이 ref면 동등 비교 조건으로 어떤 값이 제공됐는지 보여준다.
상수를 지정하면 const로 표시되고, 다른 테이블의 컬럼 값이면 테이블명과 컬럼명이 표시된다.

**ref 컬럼에 출력되는 내용은 크게 신경쓰지 않아도 무방**하지만, 콜레이션 변환이나 값 자체의 연산을 거쳐 참조되는 경우(func으로 표시된다.) 조금 주의해서 볼 필요가 있다. 사용자가 명시적으로 값을 변환하거나 문자집합이 일치하지 않는 두 문자열 컬럼을 조인할 때, 숫자 타입의 컬럼과 문자열 타입의 컬럼으로 조인할 때가 대표적인 예다.

다음 예제에서 emp_no 값에서 1을 뺀 값으로 테이블과 조인한다.

```sql
EXPLAIN 
SELECT *
FROM employees e, dept_emp de WHERE e.emp_no=(de.emp_no-1);

+------+------------+-------+-------+---------+------+
|id    |select_type | table | tpye  | key     | ref  |
+------+------------+-------+-------+---------+------+
| 1    |SIMPLE      | de    | ALL   | NULL    | NULL |
| 1    |SIMPLE      | e     | eq_ref| PRIMARY | func |
+------+------------+-------+-------+---------+------+
```

ref 값이 func으로 표시되는 것을 확인할 수 있다. 조인 조건에 산술 표현식을 넣어 쿼리를 만들었기 때문이다.

## rows 컬럼

MySQL 옵티마이저는 가능한 처리 방식을 나열하고, 최종적으로 가장 비용이 적은 하나의 실행 계획을 수립한다. 이때 비용 비교 기준은 얼마나 많은 레코드를 읽고 비교해야 하는지에 대한 것이다.

**rows 컬럼은 이렇게 실행 계획의 효율성 판단을 위해 예측했던 레코드 건수**를 보여준다. 이는 쿼리를 처리하기 위해 얼마나 많은 레코드를 읽고 체크해야 하는지를 의미한다.

옵티마이저는 이 값을 참조해 가장 효율적인 실행 계획을 수립하게 된다.
이해를 돕기 위해 다음 예제를 살펴보자.

```sql
EXPLAIN
SELECT * FROM dept_emp WHERE from_date>='1985-01-01'

+------+------------+----------+------+------+---------+-------+
|id    |select_type | table    | tpye | key  | key_len | rows  |
+------+------------+----------+------+------+---------+-------+
| 1    |SIMPLE      | dept_emp | ALL  | NULL | NULL    | 331143|
+------+------------+----------+------+------+---------+-------+
```

dept_emp 테이블에는 약 33만 건의 데이터가 들어있고 from_date로 생성된 ix_fromdate 인덱스가 있다.

실행 계획 살펴보면 해당 쿼리를 처리하는데 인덱스 레인지 스캔이 아닌 풀 테이블 스캔을 선택했다.
그 이유는 옵티마이저가 331,143 건의 레코드를 읽어야 할 것으로 예측했고 실제 테이블의 데이터 수는 약 33만건이므로 풀 테이블 스캔이 더 효율적이라 판단한 것이다.

그렇다면 범위를 조금 더 줄인 쿼리를 실행해보면 어떻게 될까?

```sql
EXPLAIN
SELECT * FROM dept_emp WHERE from_date>='2012-07-01'

+------+------------+----------+-------+-------------+---------+------+
|id    |select_type | table    | tpye  | key         | key_len | rows |
+------+------------+----------+-------+-------------+---------+------+
| 1    |SIMPLE      | dept_emp | range | ix_fromdate | 3       |  292 |
+------+------------+----------+-------+-------------+---------+------+
```

옵티마이저는 읽어야 할 레코드를 292건으로 예측했고, 따라서 실행 계획도 인덱스 레인지 스캔으로 변경된 것을 볼 수 있다.

rows 컬럼에 표시되는 값은 스토리지 엔진별로 가지고 있는 통계 정보를 참조해 산출해낸 예측값이라 정확하지는 않다. 하지만 이 값이 어느 정도 근접해야만 옵티마이저가 제대로 된 실행 계획을 수립할 수 있다.

## filtered 컬럼

옵티마이저는 일치하는 레코드 개수를 가능한 정확히 파악해야 효율적인 실행 계획을 수립할 수 있다. **앞서 살펴본 rows 컬럼은 인덱스를 사용하는 조건에만 일치하는 레코드 건수를 예측한 것이지만, 모두 인덱스를 사용할 수 있는 것은 아니다.**

특히, 조인이 사용되는 경우 WHERE 절에서 인덱스를 사용할 수 있는 조건도 중요하지만 **인덱스를 사용하지 못하는 조건에 일치하는 레코드 건수를 파악하는 것도 매우 중요하다.**

**filetered 컬럼의 값은 필터링이 되고 남은 레코드의 비율을 의미한다.**
예를 들어, rows 컬럼 값이 233건이고 filtered 값이 16.03%라고 하면 233건 중 16.03%의 레코드만 남았다는 뜻이다. 즉, 233건 중 약 37건(233 * 0.1603)만 모든 조건을 만족한다는 것을 나타낸다.

예제를 통해 더 자세히 알아보자.

- 선행 조건
  - employees 테이블의 `e.first_name='Matt'` 조건은 인덱스 사용이 가능하다.
  - salaries 테이블의 `s.salary BETWEEN 50000 AND 60000` 조건은 인덱스 사용이 가능하다.
  - 그 외 나머지 조건은 인덱스를 사용하지 못한다.

```sql
EXPLAIN 
SELECT *
FROM employees e,
     salaries s
WHERE e.first_name='Matt' // employees 인덱스 사용 가능
   AND e.hire_date BETWEEN '1990-01-01' AND '1991-01-01' // employees 인덱스 사용 불가능
   AND s.emp_no=e.emp_no 
   AND s.from_date BETWEEN '1990-01-01' AND '1991-01-01' // salaries 인덱스 사용 불가능
   AND s.salary BETWEEN 50000 AND 60000 // salaries 인덱스 사용 가능
```

위 쿼리에 대한 실행 계획은 다음과 같다.

```sql
+------+------------+-------+------+--------------+------+----------+
|id    |select_type | table | tpye | key          | rows | filtered |
+------+------------+-------+------+--------------+------+----------+
| 1    |SIMPLE      | e     | ref  | ix_firstname | 233  |   16.03  |
| 1    |SIMPLE      | s     | ref  | PRIMARY      | 10   |   0.48   |
+------+------------+-------+------+--------------+------+----------+
```

employees 테이블에서 `e.first_name='Matt'` 인덱스 조건에 일치하는 레코드는 233건이고 `AND e.hire_date BETWEEN '1990-01-01' AND '1991-01-01'` 조건까지 만족하는 레코드는 37건(233 * 0.1603)이다.

먼저 인덱스 조건에 일치하는 모든 레코드(233건)를 찾은 후, 인덱스를 사용하지 못하는 조건에 대해 필터링을 수행한 것이다. 이렇게 filtered 컬럼 값을 이용해 인덱스를 사용하지 못하는 조건에 일치하는 레코드 건수를 파악할 수 있으며, 모든 조건이 필터링된 후 남은 레코드의 비율이 filtered 컬럼의 값이 된다.

옵티마이저는 메모리 사용량을 낮추기 위해 대상 건수가 적은 테이블을 드라이빙 테이블로 선택할 가능성이 높다. 따라서 filtered 컬럼에 표시되는 값이 얼마나 정확히 예측되느냐에 따라 조인 성능이 달라질 수 있다.

앞서 인덱스를 사용하지 못하는 조건에 일치하는 레코드 건수를 파악하는 것이 중요하다고 설명한 것도 이러한 이유 때문이다.
[Real MySQL 8.0] 쿼리 힌트

MySQL의 버전이 업그레이드 되면서 옵티마이저의 쿼리 실행 계획 최적화 방법도 다양해지고 있다. 하지만 옵티마이저가 개발자나 DBA의 요구 조건을 100% 예측할 수는 없다. 따라서 옵티마이저가 부족한 실행 계획을 수립한 경우에는 우리가 옵티마이저에게 어떻게 실행계획을 수립해야 할지 알려줄 수 있는 방법이 필요하다.

MySQL에서 사용 가능한 쿼리 힌트는 다음과 같이 2가지로 구분할 수 있다.

- 인덱스 힌트
- 옵티마이저 힌트

# 인덱스 힌트

인덱스 힌트는 예전 버전의 MySQL 서버에서 사용되어 오던 STRAIGHT_JOIN과 USE INDEX 같은 힌트를 의미한다. 이 기능들은 각 SQL문법에 영향을 받기 때문에 ANSI-SQL 표준 문법을 준수하지 못하며, 가능하면 MySQL 5.6 버전에 도입된 옵티마이저 힌트를 사용하는 것을 권장한다.

## STRAIGHT_JOIN

STRAIGHT_JOIN은 여러 개의 테이블이 조인되는 경우 조인 순서를 고정하는 역할을 한다. 즉, 옵티마이저에게 드라이빙 테이블과 드리븐 테이블에 대한 힌트를 주는 기능이다.

옵티마이저는 각 테이블의 통계 정보와 쿼리의 조건을 기반으로 최적의 순서로 조인을 수행한다. 일반적으로 조인을 하기 위한 컬럼들의 인덱스 여부로 조인의 순서가 결정되면, 조인 컬럼의 인덱스에 아무런 문제가 없는 경우 레코드가 적은 테이블을 드라이빙으로 선택한다.

이런 쿼리의 조인 순서를 변경하려는 경우에는 다음과 같이 STRAIGHT_JOIN 힌트를 사용해 테이블의 조인 순서를 유도할 수 있다.

```sql
SELECT /*! STRAIGHT_]OIN */ 
	e.first_name, e.last_name, d.dept_name
FROM employees e, dept_emp de, departments d 
WHERE e.emp_no=de.emp_no
	AND d.dept_no=de.dept_no;
```

이 쿼리의 실행 계획을 보면 FROM 절에 명시된 테이블의 순서대로(employees → dept_emp → departments) 조인이 수행된다.

다음 기준에 맞게 조인 순서가 결정되지 앟는 경우에만 STRAIGHT_JOIN 힌트로 조인 순서를 조정하는 것이 좋다.

- 임시 테이블과 일반 테이블의 조인
  - 일반 테이블 조인 컬럼에 인덱스가 없는 경우 옵티마이저가 실행 계획을 제대로 수립하지 못해 성능 저하가 심할 때 레코드 건수가 작을 쪽을 드라이빙으로 선택하는게 좋다.
- 임시 테이블끼리 조인
  - 임시 테이블은 항상 인덱스가 없으므로 어느쪽이 드라이빙이 되어도 상관없다. 따라서 크기가 작은 테이블을 드라이빙으로 선택하는게 좋다.
- 일반 테이블끼리 조인
  - 양 테이블의 인덱스 상황이 같은 경우(양쪽 모두 인덱스가 있거나 없거나) 레코드 건수가 적은 테이블을 드라이빙으로 선택하는 것이 좋다. 이 외의 경우 조인 컬럼에 인덱스가 없는 테이블을 드라이빙으로 선택하는 것이 좋다.

위에서 언급한 레코드 건수는 테이블 전체의 레코드 건수가 아니라 WHERE 조건에 포함된 레코드 건수를 나타내니 혼동하지 않도록 주의하자.

## USE INDEX / FORCE INDEX / IGNORE INDEX

옵티마이저는 테이블에 3~4개 이상의 컬럼을 포함하는 비슷한 인덱스가 여러개 존재하는 경우 어떤 인덱스를 선택해야하는지 혼동할 수 있다. 이런 경우 강제로 특정 인덱스를 사용하도록 힌트를 추가할 수 있다.

- USE INDEX
  - 옵티마이저에게 특정 테이블의 인덱스를 사용하도록 **권장**하는 힌트
  - 옵티마이저가 항상 그 힌트를 선택하는건 아니다.
- FORCE INDEX
  - USE INDEX보다 영향력이 강한 힌트
  - USE INDEX와 비교해 차이가 없으므로 거의 사용할 필요가 없다.
- IGNORE INDEX
  - 특정 인덱스를 사용하지 못하도록 하는 힌트
  - 때로는 풀 테이블 스캔을 유도하기 위해 사용

만약 좋은 실행 계획이 어떤 것인지 판단하기 힘든 상황이라면 힌트를 사용해 옵티마이저의 실행 계획에 영향을 미치는 것은 피하는 것이 좋다. 최적의 실행 계획은 데이터의 성격에 따라 시시각각 변하므로 옵티마이저가 당시 통계 정보를 가지고 선택하게 하는 것이 가장 좋은 방법이며, 가장 훌륭한 최적화는 튜닝할 필요가 없게 데이터를 최소화하는 것이다.

# 옵티마이저 힌트

MySQL 8.0에서는 사용 가능한 힌트 종류가 매우 다양하고 미치는 영향 범위도 매우 다양하다.

## 옵티마이저 힌트의 종류

- 인덱스
  - 특정 인덱스의 이름을 사용할 수 있는 옵티마이저 힌트
- 테이블
  - 특정 테이블의 이름을 사용할 수 있는 옵티마이저 힌트
- 쿼리 블록
  - 특정 쿼리 블록에 사용할 수 있는 옵티마이저 힌트
  - 힌트가 명시된 쿼리 블록에 대해서만 영향을 미침
- 글로벌(쿼리 전체)
  - 전체 쿼리에 대해 영향을 미치는 힌트

### MAX_EXECUTION_TIME

옵티마이저 힌트 중 유일하게 실행 계획에 영향을 미치지 않는 힌트로, 단순히 쿼리의 최대 실행 시간을 설정하는 힌트다. 지정된 시간을 초과하게되면 쿼리는 실패하게 된다.

```sql
SELECT /*+ MAX_EXECUTION_TIME(100) */ * 
FROM employees
ORDER BY last_name LIMIT 1;
```

### SET_VAR

SET_VAR 힌트는 실행 계획을 바꾸는 용도뿐만 아니라 조인 버퍼나 정렬용 버퍼의 크기를 일시적으로 증가시켜 대용량 처리 쿼리의 성능을 향상시키는 용도로 사용할 수 있다.

```sql
EXPLAIN
SELECT /★+ SET_VAR(optimizer_switch='index_merge_intersection=off') */ ★ 
FROM employees
WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

### SEMIJOIN & NO_SEMIJOIN

SEMIJOIN 힌트는 세미 조인 최적화의 어떤 세부 전략을 사용할지 제어하는데 사용할 수 있다. 세미조인의 최적화 전략은 다음과 같다.

- Duplicate Weed-out
- First Match
- Loose Scan
- Materialization
- Table Pull-out

이때 SEMIJOIN 힌트의 경우 SEMIJOIN(최적화 전략)의 형태로 사용할 수 있다.
ex) SEMIJOIN(FIRSTMATCH)

Table Pull-out의 경우 별도의 힌트를 사용할 수 없는데, Table Pull-out 전략을 사용하는 경우 항상 더 나은 성능을 보장하기 때문이다.

세미조인 힌트는 서브쿼리에 명시하거나 서브쿼리에 쿼리 블록 이름을 정의하고 외부 쿼리 블록에 명시해야 한다.

```sql
-- 서브쿼리에 세미조인 힌트 명시
EXPLAIN
SELECT *
FROM departments d 
WHERE d.dept.no IN
		(SELECT /*+ SEMIJOIN(MATERIALIZATION) */ de.dept_no 
		 FROM dept_emp de);

// 쿼리 블록 이름 정의 및 외부 쿼리 블록에 세미조인 힌트 명시
EXPLAIN
SELECT /*+ SEMIJOIN(@subq1 MATERIALIZATION) */ *
FROM departments d
WHERE d.dept_no IN
	(SELECT /*+ QB_NAME(subq1) */ de.dept_no 
   FROM dept_emp de);
```

### JOIN_FIXED_ORDER & JOIN_ORDER & JOIN_PREFIX & JOIN_SUFFIX

MySQL 서버는 조인의 순서를 결정하기 위해 STRAIGHT_JOIN 힌트를 사용해왔다. 하지만 이는 쿼리 FROM 절에 사용된 테이블의 순서를 조인 순서에 맞게 변경해야하는 번거로움이 있었다. 또한 일부 조인 순서를 강제하고 나머지는 옵티마이저에게 맞기는 것도 불가능했다.

이 단점을 보완하기 위해 옵티마이저 힌트에서는 다음과 같이 4개의 힌트를 제공한다.

- JOIN_FIXED_ORDER
  - FROM 절의 테이블 순서대로 조인을 실행
- JOIN_ORDER
  - FROM 절에 사용된 테이블 순서가 아니라 힌트에 명시된 테이블 순서대로 조인 실행
- JOIN_PREFIX
  - 조인에서 드라이빙 테이블만 강제
- JOIN_SUFFIX
  - 조인에서 드리븐 테이블만 강제


```sql
// FROM 절에 나열된 테이블의 순서대로 조인 실행
SELECT /*+ JOIN_FIXED_ORDER() */ *
FROM employees e
	INNER JOIN dept_emp de ON de.emp_no = e.emp_no 
	INNER JOIN departments d ON d.dept_no = de.dept_no;

// 일부 테이블에 대해서만 조인 순서를 나열
SELECT /*+ JOIN_ORDER(d, de) */ *
FROM employees e
	INNER JOIN dept_emp de ON de.emp_no = e.emp_no 
	INNER JOIN departments d ON d.dept_no = de.dept_no;

// 조인의 드라이빙 테이블에 대해서만 조인 순서를 나열 
SELECT /*+ JOIN_PREFIX(e, de) */ *
FROM employees e
	INNER JOIN dept_emp de ON de.emp_no = e.emp_no
	INNER JOIN departments d ON d.dept_no = de.dept_no;

// 조인의 드리븐 테이블에 대해서만 조인 순서를 나열 
SELECT /*+ JOIN_SUFFIX(de, e) */ *
FROM employees e
	INNER JOIN dept_emp de ON de.emp_no = e.emp_no
	INNER JOIN departments d ON d.dept_no = de.dept_no;
```

### MERGE & NO_MERGE

이전 MySQL 서버에서는 FROM 절에 사용된 서브쿼리를 항상 내부 임시 테이블로 생성해 불필요한 자원을 소모했었다. 따라서 MySQL 5.7 이상 버전에서는 임시 테이블을 사용하지 않게 FROM 절의 서브쿼리를 외부 쿼리와 병합하는 최적화를 도입했다.

때로는 임시 테이블을 생성하는 것이 나은 선택이 될 수도 있기 때문에 옵티마이저가 최적의 방법을 선택하지 못했을 때는 MERGE 또는 NO_MERGE 옵티마이저 힌트를 사용하면 된다.

```sql
// 외부 쿼리와 병합
EXPLAIN
SELECT /*+ MERGE(sub)*/ *
FROM (SELECT *
			FROM employees
			WHERE first_name='Matt1') sub LIMIT 10;

// 임시 테이블 사용
EXPLAIN
SELECT /*+ N0_MERGE(sub)*/ *
FROM (SELECT * 
			FROM employees
			WHERE first_name='Matt') sub LIMIT 10;
```

### INDEX_MERGE & NO_INDEX_MERGE

MySQL 서버는 가능하면 테이블 당 하나의 인덱스만을 이용해 쿼리를 처리하려고 한다. 이때 인덱스를 통해 검색된 레코드의 교집합 또는 합집합만을 구해 결과를 반환하고 **하나의 테이블에 대해 여러 개의 인덱스를 동시에 사용하는 것을 인덱스 머지**라고 한다.

INDEX_MERGE와 NO_INDEX_MERGE는 인덱스 머지 실행 계획 사용 여부를 제어하고자 할 때 사용된다.

```sql
EXPLAIN 
SELECT * /*+ INDEX_MERGE(employees ix_firstname, PRIMARY) */ *
FROM employees
WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000

EXPLAIN
SELECT /*+ NO_INDEX_MERGE(employees PRIMARY) */ *
FROM employees
WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

### NO_ICP

인덱스 컨디션 푸시다운 최적화는 항상 성능 향상에 도움이 되므로 옵티마이저는 최대한 인덱스 컨디션 푸시다운 기능을 사용하는 방향으로 실행 계획을 수립한다. 따라서 MySQL 옵티마이저에서는 ICP(Index Condition Pushdown) 힌트는 제공하지 않고 인덱스 컨디션 푸시다운으로 인한 잘못된 실행 계획 수립시 인덱스 컨디션 푸시다운 최적화를 비활성화하는 힌트인 NO_ICP만을 제공한다.

```sql
EXPLAIN
SELECT /*+ NO_ICP(employees ix_lastname_firstname) * / * 
FROM employees
WHERE last_name='Acton' AND first_name LIKE '%sar';
```

### SKIP_SCAN & NO_SKIP_SCAN

인덱스 스킵 스캔은 인덱스의 선행 칼럼에 대한 조건이 없어도 옵티마이저가 해당 인덱스를 사용할 수 있게 해주는 최적화 기능이다. 하지만 조건이 누락된 선행 컬럼의 유니크 값의 개수가 많아진다면 오히려 성능이 떨어지므로 옵티마이저가 비효율적인 인덱스 스킵 스캔을 선택한 경우 NO_SKIP_SCAN 힌트로 이를 제어할 수 있다.

```sql
// 인덱스 스킵 스캔 비활성화
EXPLAIN
SELECT /*+ NO_SKIP_SCAN(employees ix_gender_birthdate) ★/ gender, birth_date 
FROM employees
WHERE birth_date>='1965-02-01•;
```

### INDEX & NO_INDEX

INDEX와 NO_INDEX 옵티마이저 힌트는 예전 MySQL 서버에서 사용되던 인덱스 힌트를 대체하는 용도로 사용된다. 대체된 인덱스 힌트는 다음과 같다.

- USE INDEX
  - → INDEX
- USE INDEX FOR GROUP BY
  - → GROUP_INDEX
- USE INDEX FOR ORDER BY
  - → ORDER_INDEX
- IGNORE INDEX
  - → NO_INDEX
- IGNORE INDEX FOR GROUP BY
  - → NO_GROUP_INDEX
- IGNORE INDEX FOR ORDER BY
  - → NO_ORDER_INDEX

```sql
// 인덱스 힌트 사용
EXPLAIN
SELECT *
FROM employees USE INDEX(ix_firstname) 
WHERE first_name='Matt';

// 옵티마이저 힌트 사용
EXPLAIN
SELECT /*+ INDEX(employees ix_firstname) ★/ * 
FROM employees
WHERE first_name='Matt';
```
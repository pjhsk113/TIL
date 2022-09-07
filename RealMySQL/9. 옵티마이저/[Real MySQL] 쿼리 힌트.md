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
# [Real MySQL 8.0] 옵티마이저의 기본 데이터 처리 2 / 2

## GROUP BY 처리

GROUP BY도 ORDER BY처럼 스트리밍된 처리를 할 수 없는 작업 중 하나다.
GROUP BY 절에는 그루핑 결과에 필터링 역할을 수행하는 HAVING 절을 사용할 수 있는데, GROUP BY에 사용된 조건은 인덱스를 사용해 처리될 수 없어 **HAVING 절을 튜닝하려고 인덱스를 생성하거나 다른 방법을 고민할 필요는 없다.**

GROUP BY 작업은 인덱스를 사용하는 경우와 그렇지 못한 경우로 나눌 수 있다.

- 인덱스를 이용하는 경우
  - 인덱스를 차례대로 읽는 타이트 인덱스 스캔
  - 인덱스를 건너뛰면서 읽는 루스 인덱스 스캔
- 인덱스를 사용하지 못하는 경우
  - 임시 테이블 사용


### 인덱스 스캔을 이용하는 GROUP BY(타이트 인덱스 스캔)

드라이빙 테이블에 속한 컬럼만 이용해 그루핑할 때 이미 인덱스가 있다면 해당 인덱스를 차례대로 읽으면서 그루핑 작업을 수행하고 그 결과로 조인을 수행한다. 이미 정렬된 인덱스를 읽는 것이므로 쿼리 실행 시점에 추가 정렬 작업이나 임시 테이블을 필요로하지 않는다.

### 루스 인덱스 스캔을 이용하는 GROUP BY

루스 인덱스 스캔 방식은 인덱스의 레코드를 건너뛰면서 필요한 부분만 읽어 가져오는 것을 의미한다. 루스 인덱스 스캔을 사용할 때는 실행 계획의 Extra 컬럼에 **Using index for group-by** 코멘트가 표시된다.

다음 쿼리를 실행하면 루스 인덱스 스캔을 사용한다.

```sql
EXPLAIN
	SELECT emp_nᄋ
	FROM salaries
	WHERE from_date='1985-03-01' 
	GROUP BY emp_no;

+----+----------+-------+---------+---------------------------------------+
| id | table    | type  | key     | Extra                                 |
+----+----------+-------+---------+---------------------------------------+
|  1 | salaries | range | PRIMARY | Using where; Using index for group-by |
+----+----------+-------+---------+---------------------------------------+
```

**쿼리 실행 순서**

1. (emp_no, from_date) 인덱스를 차례로 스캔하면서 emp_no의 첫 번째 유일한 값 10001을 찾아낸다.
2. emp_no가 10001인 것 중에서 from_date가 ‘1985-03-01’인 레코드만 가져온다.
3. emp_no의 그 다음 유니크한 값을 가져온다.
4. 3번의 결과가 없으면 처리를 종료하고, 결과가 있다면 2번 과정을 반복 수행한다.

루스 인덱스 스캔 방식은 단일 테이블의 GROUP BY 처리에만 사용할 수 있다. 또한 컬럼값의 앞쪽 일부만 생성된 인덱스인 프리픽스 인덱스는 루스 인덱스 스캔을 사용할 수 없다.

일반적인 인덱스 레인지 스캔과는 다르게 루스 인덱스 스캔에서는 유니크한 값의 수가 적을수록 성능이 향상된다는 것을 생각해두자.

**루스 인덱스 스캔을 사용할 수 없는 쿼리 패턴**

```sql
// MIN()과 MAX() 이외의 집합 함수가 사용됐기 때문에 루스 인덱스 스캔은 사용 불가
SELECT col1, SUM(col2) FROM tb_test GROUP BY col1;

// GROUP BY에 사용된 칼럼이 인덱스 구성 칼럼의 왼쪽부터일치하지 않기 때문에 사용 불가
SELECT col1, col2 FROM tb_test GROUP BY col2, col3; 

// SELECT 절의 칼럼이 GROUP BY와 일치하지 않기 때문에 사용 불가
SELECT col1, col3 FROM tb_test GROUP BY col1, col2;
```

### 임시 테이블을 사용하는 GROUP BY

GROUP BY의 기준 컬럼이 드라이빙 테이블에 있든 드리븐 테이블에 있든 관계 없이 인덱스를 전혀 사용하지 못할 때 임시 테이블을 사용하여 처리된다.

MySQL 8.0에서는 GROUP BY가 필요한 경우 내부적으로 GROUP BY 절의 컬럼들로 구성된 유니크 인덱스를 가진 임시 테이블을 만들어 중복 제거와 집합 함수 연산을 수행한다.

실행 계획의 Extra 컬럼에 Using temporary 코멘트가 표시되며, MySQL 8.0 부터는 묵시적 정렬을 수행하지 않으므로 Using filesort 코멘트는 표시되지 않는다.

## DISTINCT 처리

DISTINCT는 특정 컬럼의 유니크한 값만 조회하기 위해 사용한다. 이 작업은 집합 함수와 함께 사용하는 경우와 집합 함수가 없는 경우에 따라 영향을 미치는 범위가 달라지기 떄문에 구분해 살펴봐야 한다.

### 집합 함수가 없는 경우 ( SELECT DISTINCT … )

SELECT DISTINCT 형태의 쿼리는 GROUP BY와 동일한 방식으로 처리된다.
예를 들어 아래의 두 쿼리는 내부적으로 같은 작업을 수행한다.

```sql
SELECT DISTINCT emp_no FROM salaries; 

SELECT emp_no FROM salaries GROUP BY emp_no;
```

DISTINCT는 특정 컬럼만 유니크하게 조회하는 것이 아니라 유니크한 레코드(튜플)를 조회한다.

```sql
SELECT DISTINCT first_name, last_name FROM employees;
```

위 쿼리에서는 first_name이 유니크한 값을 조회하는 것이 아니라 (first_name, last_name) 조합 전체가 유니크한 레코드를 가져오는 것이다. 쿼리만 봤을 때 쉽게 착각할 수 있는 부분이기 때문에 실수하지 않도록 주의하자.

### 집합 함수와 함께 사용된 DISTINCT

COUNT(), MIN(), MAX()와 같은 집합 함수 내에서 DISTINCT 키워드가 사용된 경우로, 앞서 살펴본 일반적인 DISTINCT와는 다른 형태로 해석된다. **집합 함수 내에서 사용된 DISTINCT는 인자로 전달된 컬럼값이 유니크한 것들을 가져온다.**

```sql
SELECT COUNT(DISTINCT s.salary)
FROM employees e, salaries s 
WHERE e.emp_no=s.emp_no
AND e.emp_no BETWEEN 100001 AND 100100;
```

위 쿼리는 `COUNT(DISTINCT s.salary)`를 처리하기 위해 임시 테이블을 사용한다. 테이블 조인 결과에서 드리븐 테이블의 컬럼 값(s.salary)만 저장해야하기 때문이다. 이때 salary 컬럼에 유니크 인덱스가 생성되므로 레코드 수가 많아진다면 상당이 느려질 수 있는 형태의 쿼리다.

집합 함수와 함께 사용된 DISTINCT를 최적화하려면 인덱스된 컬럼에 대해 DISTINCT 처리를 수행하도록 쿼리를 작성해야 한다.

```sql
SELECT COUNT(DISTINCT emp_no) FROM employees;
SELECT COUNT(DISTINCT emp_no) FROM dept_emp GROUP BY dept_no;
```

인덱스 컬럼에 대해 DISTINCT 처리를 수행할 때 인덱스 풀 스캔이나 레인지 스캔하면서 임시 테이블 없이 최적화된 처리를 수행할 수 있다.

## 내부 임시 테이블 활용
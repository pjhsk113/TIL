# [Real MySQL 8.0] 실행 계획 분석하기(Extra)

## Extra 컬럼

실행 계획에서 성능과 과련된 중요한 내용이 Extra 컬럼에 자주 표시된다.
주로 내부적인 처리 알고리즘에 대해 깊이 있는 내용을 보여주는 경우가 많다.

이제 Extra 컬럼에 표시될 수 있는 문장을 하나씩 자세히 살펴보자.

### const row not found

실행 계획에서 const 접근 방법으로 테이블을 읽었지만 실제로 해당 테이블에 레코드가 1건도 존재하지 않으면 나타나는 문장이다.

### Deleting all rows

스토리지 엔진의 핸들러 차원에서 테이블의 모든 레코드를 삭제하는 기능을 제공하는 경우 해당 문구가 표시된다. WHERE 조건절이 없는 DELETE 문장의 실행 계획에서 자주 표시되며, 모든 레코드를 삭제하는 핸들러 API를 호출함으로써 처리됐다는 것을 의미한다.

MySQL 8.0 버전에서는 Deleting all rows 최적화는 표시되지 않는다.
테이블의 모든 레코드를 삭제하는 경우 WEHERE 조건절이 없는 DELETE 보다 TRUNCATE TABLE 명령을 사용할 것을 권장하고 있다.

### Distinct

조회하려는 값을 중복없이 유니크하게 가져오기 위해 DISTINCT 키워드를 사용하면 Extra 컬럼에 Distinct가 표시된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/441a4dec-a8b0-4ed4-ab8a-2438b97c8a2d/Untitled.png)

위 예시처럼 두 테이블을 조인해서 dept_no만 중복없이 유니크하게 가져오기 위해 DISTINCT 키워드를 사용한다. 쿼리의 DISTINCT를 처리하기 위해 조인하지 않아도 되는 항목은 모두 무시하고 dept_emp 테이블에서는 필요한 레코드만 읽은 것을 볼 수 있다.

### FirstMatch

세미 조인의 여러 최적화 중에서 FirstMatch 전략이 사용되면 FirstMatch(table_name)이 표시된다.

### Full scan on NULL key

`col1 IN (SELECT col2 FROM …)` 과 같은 조건을 가진 쿼리에서 자주 발생하는 표시 값이다. 만약 col1이 NULL이면 서브쿼리에 사용된 테이블에 대해 풀 테이블 스캔을 해야만 결과를 알아낼 수 있다.

즉, Full scan on NULL key는 쿼리 실행 중 col1이 NULL을 만나면 풀 테이블 스캔을 사용할 것이라는 사실을 알려주는 키워드다.

만약 col1이 NOT NULL로 정의된 컬럼이라면 해당 키워드는 아예 표시되지 않는다. 또한, col1이 절대 NULL이 아님을 옵티마이저에게 알려주면(IS NOT NULL) 옵티마이저는 이러한 NULL 비교 규칙을 무시하게 된다.

### Impossible HAVING

쿼리에 사용된 HAVING 절의 조건을 만족하는 레코드가 없을 때 나타나는 키워드다. 실행 계획에 이 키워드가 출력된다면 쿼리가 제대로 작성되지 못한 경우가 대부분이므로 쿼리를 다시 점검하는 것이 좋다.

### Impossible WHERE

위와 비슷하게 WHERE 조건이 항상 False인 경우 나타나는 키워드다.

```sql
SELECT * FROM employees WHERE emp_no IS NULL:
```

위 쿼리에서 emp_no는 PK이므로 NULL이 될 수 없다. 따라서 emp_no IS NULL이라는 조건은 불가능한 WHERE 조건이므로 Extra 컬럼에 Impposible WHERE 키워드가 표시된다.

### LooseScan

세미조인 최적화 중 LooseScan 최적화 전략이 사용된 경우 LooseScan 키워드가 표시된다.

### No mathcing min / max row

MIN()이나 MAX() 같은 집합 함수가 있는 쿼리의 조건절에 일치하는 레코드가 없는 경우 No machting min/max row 메시지가 출력된다.

### no matching row in const table

조인에 사용된 테이블에서 const 방법으로 접근할 때 일치하는 레코드가 없다면 나타나는 메시지다.

```sql
SELECT *
FROM dept_emp de,
  (SELECT emp_no FROM employees WHERE emp_no = 0) tb1
WHERE tb1.emp_no = de.emp_no AND de.dept_no='d0O5' ;
```

조인에 사용된 테이블의 조건이 const(상수) 방법을 사용하고 있지만 일치하는 레코드가 없는 경우 실행 계획을 만들기 위한 기초 자료가 없기 때문에 no matching row in const table 메시지가 표시된다.
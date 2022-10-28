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

### No matching rows after partition pruning

해당 메시지는 파티션된 테이블에 대한 UPDATE나 DELETE할 대상 레코드가 없는 경우 표시된다. 단순히 삭제할 레코드가 없음을 의미하는 것이 아니라 대상 파티션이 없다는 것을 의미한다.

따라서 대상 파티션이 존재하고 레코드가 비어있는 경우 이 메시지는 표시되지 않는다.

### No tables used

메시지 그대로 FROM 절이 없는 쿼리나 FROM DUAL 형태의 쿼리 실행 계획에서 출력되는 메시지다.

### Not exists

아우터 조인(LEFT OUTER JOIN)을 이용해 안티-조인을 수행하는 쿼리에서는 Not exists 메시지가 표시된다.

> 안티-조인이란?
A 테이블에는 존재하지만 B 테이블에는 없는 값을조회 하는 형태의 쿼리를 말한다. 즉, 일반 조인을 했을 때 나오지 않는 결과만 가져오는 방법이다. 일반적으로 NOT IN이나 NOT EXISTS 연산자를 주로 이용한다. 레코드가 많은 경우 아우터 조인을 이용해 구현하면 성능을 더 끌어올릴 수 있다.
>

Not exists 메시지는 옵티마이저가 테이블 조인 조건에 일치하는 레코드의 존재 유무를 딱 1건만 조회해보고 처리를 완료하는 최적화를 말한다.

### Plan isn’t ready yet

MySQL 8.0 부터는 다른 커넥션에서 실행 중인 쿼리의 실행 계획을 `EXPLAIN FOR CONNECTION` 명령을 통해 살펴볼 수 있다. Plan isn’t ready yet 메시지는 이름 그대로 조회한 커넥션이 쿼리 실행 계획을 수립하지 못한 상태에서 `EXPLAIN FOR CONNECTION` 명령이 수행된 경우 표시된다.

### Range checked for each record(index map:N)

조인 조건에 상수가 없고 둘 다 변수를 사용하는 경우 인덱스 레인지 스캔과 풀 테이블 스캔 중 어느 것이 더 효율적인지 옵티마이저는 판단할 수 없다.

이해를 돕기 위해 아래 예제를 살펴보자.

```sql
EXPLAIN 
SELECT *
FROM employees e1, employees e2 
WHERE e2.emp_no >= e1.emp_no;

+------+------------+-------+-------+-----------------------------------------------+
|id    |select_type | table | type  | Extra                                         |
+------+------------+-------+-------+-----------------------------------------------+
| 1    |SIMPLE      | e1    | ALL   | NULL                                          |
| 1    |SIMPLE      | e2    | ALL   | Range checked for each record (index map: 0x1 |
+------+------------+-------+-------+-----------------------------------------------+
```

e1 테이블의 레코드를 하나씩 읽을 때마다 e1.emp_no 값이 계속 바뀌므로 쿼리 비용 계산을 위한 기준 값이 계속 변하게 된다. 따라서 옵티마이저는 어떤 접근 방법으로 e2 테이블을 읽는 것이 효율적인지 판단할 수 없게 되는 것이다.

Range checked for each record는 “레코드마다 인덱스 레인지 스캔을 체크한다.” 라는 의미를 가지고 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e17df757-2ee3-4365-90dd-b1265d773074/Untitled.png)

실행 계획을 살펴보면 `index map: 0x1` 이라는 메시지도 함께 표시되어 있다. 이는 후보 인덱스의 순번을 나타내며, 해석하기 위해선 이진수로 변환을 해야한다.

예를 들어, 다음과 같은 테이블의 실행 계획에서 `index map: 0x19`라는 값이 표시됐다고 가정해보자.

```sql
CREATE TABLE tb_member( 
 mem_id INTEGER NOT NULL,
 mem_name VARCHAR(100) NOT NULL, 
 mem_nickname VARCHAR(100) NOT NULL, 
 mem_region TINYINT,
 mem_gender TINYINT,
 mem_phone VARCHAR(25),
 PRIMARY KEY (mem_id),
 INDEX ix_nick_name (mem_nickname, mem_name), 
 INDEX ix_nick_regiᄋn (mem_nickname, mem_region), 
 INDEX ix_nick_gender (mem_nickname, mem_gender), 
 INDEX ix_nick_phone (mem_nickname, mem_phone)
}
```

생성된 인덱스는 총 5개이고 0x19를 이진수로 변환하면 11001이다.
이 비트 배열을 해석하는 방법은 다음과 같다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33cc4ab7-ba5f-41bd-9a50-9f07bd2fcf09/Untitled.png)

여기서 값이 1인 인덱스를 사용 가능한 인덱스 후보로 선정했음을 의미한다.

### Recursive

MySQL 8.0 부터는 CTE(Common Table Expression)을 이용해 재귀 쿼리를 작성할 수 있게 됐다.

```sql
WITH RECURSIVE cte (n) AS 
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 5
)
SELECT * FROM cte;
```

위와 같이 WITH 구문을 이용해 CTE를 사용하면 된다.

위 쿼리의 WITH 절의 동작을 살펴보면 다음과 같다.

1. n이라는 컬럼 하나를 가진 cte라는 이름의 임시 테이블을 생성
2. n 컬럼의 값이 1부터 5까지 1씩 증가해서 레코드 5건을 만들어 cte 내부 임시 테이블에 저장

그리고 WITH 절 다음의 SELECT 쿼리에서는 생성된 임시 내부 테이블을 풀 스캔해 결과를 반환한다. 이때 실행계획에 Recursive 구문이 표시된다.

### Rematerialize
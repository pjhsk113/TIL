# [Real MySQL 8.0] 옵티마이저의 기본 데이터 처리 1 / 2

요청된 쿼리는 같은 결과를 반환하지만, 내부적으로 그 결과를 어떻게 만들어낼 것인지에 대한 방법은 매우 다양하다. 따라서 어떤 방법이 최적이고 최소의 비용이 소모되는지 결정해야 한다.

MySQL에서는 테이블의 데이터가 어떤 분포로 저장돼 있는지 통계 정보를 참조해 최적의 **실행 계획**을 수립한다. 대부분의 DBMS에서도 옵티마이저가 이러한 기능을 담당하고 있다.

# 쿼리 실행 절차

쿼리가 실행되는 과정은 크게 세 단계로 나눌 수 있다.

1. 요청된 SQL 문장을 쪼개서 MySQL 서버가 이해할 수 있는 수준으로 분리(파스 트리)
  - SQL 파싱 단계로 SQL 파서 모듈로 처리
  - SQL 문법 오류(Syntax Error)가 이 단계에서 걸러짐
  - SQL 파스 트리 생성
2. SQL의 파싱 정보(파스 트리)를 확인해 어떤 테이블을 읽을지, 어떤 인덱스를 이용할지 결정
  - 불필요한 조건 제거 및 복잡한 연산 단순화
  - 테이블 조인이 있는 경우 어떤 순서로 테이블을 읽을지 결정
  - 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스 결정
  - 임시 테이블 사용 여부 결정
  - 최적화 및 실행 계획 수립 단계로 위 과정들이 완료되면 실행 계획이 수립됌
3. 결정된 테이블의 읽기 순서나 인덱스를 이용해 스토리지 엔진으로부터 데이터를 가져옴
  - 2번에서 만들어진 실행 계획대로 스토리지 엔진에 레코드를 읽어오도록 요청
  - 스토리지 엔진으로부터 받은 레코드를 조인하거나 정렬하는 작업 수행

# 옵티마이저 종류

- 비용 기반 최적화
  - 쿼리를 처리하기 위한 여러 방법을 만들고, 각 단위 작업의 비용과 예측된 통계 정보를 이용해 실행 계획별 비용을 산출
  - 최소 비용 처리 방법을 선택해 쿼리를 실행
  - 현재 대부분의 DBMS가 해당 최적화 방법을 사용중이다.

- 규칙 기반 최적화
  - 대상 테이블의 레코드 건수나 선택도 등을 고려하지 않고 옵티마이저에 내장된 우선순위에 따라 실행 계획을 수립
  - 같은 쿼리에서는 항상 같은 실행 방법을 선택하는 단점이 존재한다.
  - 예전 초기 버전으로 최근에는 거의 사용되지 않는다.

# 기본 데이터 처리

RDBMS는 데이터를 정렬하거나 그루핑하는 기본 데이터 가공 기능을 가지고 있다.
MySQL 서버가 어떤 알고리즘을 이용해 이러한 기본 데이터 가공을 처리하는지 알아보자.

## 풀 테이블 스캔과 풀 인덱스 스캔

풀 테이블 스캔은 말 그대로 테이블의 데이터를 처음부터 끝까지 읽어 처리하는 작업을 의미한다. MySQL 옵티마이저는 다음과 같은 조건일 때 주로 풀 테이블 스캔을 선택한다.

- 테이블 레코드 건수가 너무 작은 경우(테이브블 페이지 1개로 구성된 경우)
- WHERE 절이나 ON 절에 인덱스를 이용할 수 있는 적절한 조건이 없는 경우
- 인덱스가 있더라도 조건 일치 레코드 건수가 너무 많은 경우

대부분의 DBMS는 풀 테이블 스캔 시 한꺼번에 여러 개의 블록이나 페이지를 읽어오는 기능을 내장하고 있다. InnoDB 스토리지 엔진의 경우 **백그라운드 스레드에 의해 시작되는 리드 어헤드 작업**에 의해 이러한 기능이 지원된다.

**리드 어헤드(Read ahead)란** **어떤 영역의 데이터가 앞으로 필요해질 것을 예측해서 미리 디스크로부터 읽어 InnoDB 버퍼풀에 담아두는 것을 의미한다.**

풀 테이블 스캔이 일어나면 **포그라운드 스레드가 페이지 읽기를 실행**하고 **특정 시점부터는 해당 읽기 작업을 백그라운드 스레드로 넘긴다**. 백그라운드 스레드로 넘겨받는 시점부터 한 번에 최대 64개의 데이터 페이지까지 읽어 버퍼풀에 저장해둔다. 이렇게 하면 포그라운드 스레드는 버퍼풀에 미리 준비된 데이터를 가져다 사용하면되므로 쿼리를 빠르게 처리할 수 있게 되는것이다.

- 풀 인덱스 스캔은 [인덱스 살펴보기 2/2](https://www.notion.so/Real-MySQL-8-0-2-2-6c1ace9839b44ca299cadcfc07934fe9)를 참조

## 병렬 처리

MySQL 8.0부터는 용도가 한정돼 있긴 하지만 하나의 쿼리를 여러 스레드가 나누어 동시에 처리할 수 있는 병렬 처리가 가능해졌다.

innodb_parallel_read_threads라는 시스템 변수를 이용해 **아무런 WHERE 조건 없이 단순히 테이블 전체 건수를 가져오는 쿼리만 병렬로 처리할 수 있다.**

```sql
-- 4개의 스레드를 사용해 쿼리를 병렬 처리
SET SESSION innodb_parallel_read_threads=4;
SELECT COUNT(*) FROM salaries;
```

병렬 처리용 스레드 수가 늘어날 수록 쿼리 처리 속도가 빨라지는걸 화인할 수 있지만, 서버에 장착된 CPU 코어 개수를 넘어서면 오히려 성능이 떨어질 수 있으니 주의하자.

## ORDER BY 처리(Using filesort)

정렬을 처리하는 방법은 **인덱스를 이용하는 방법**과 **Filesort 처리 방법**으로 나눌 수 있다.

- 인덱스
  - 장점
    - Insert, Update, Delete 쿼리가 실행될 때 이미 인덱스가 정렬되어 있으므로 읽을 때(Insert, Update, Delete 쿼리의 조건절을 검색할 때) 매우 빠르다.
  - 단점
    - 부가적인 인덱스 추가/삭제 작업이 필요하므로 실제 Insert, Update, Delete 작업 시 느리다.
    - 인덱스 때문에 디스크 공간이 더 많이 필요하다.
    - 버퍼 풀을 위한 메모리가 많이 필요하다.

- Filesort
  - 장점
    - 인덱스가 필요없으므로 인덱스의 단점이 장점으로 바뀐다.
    - 레코드가 적을 경우 충분히 빠르다
  - 단점
    - 레코드 건수가 많아질수록 쿼리 응답 속도가 느려진다.

정렬 처리를 수행할 때 Filesort보다 인덱스를 이용하도록 튜닝하면 좋지만 모든 정렬에 인덱스를 이용하도록 튜닝하기란 불가능하다. 그 이유는 다음과 같다.

- 정렬 기준이 너무 많아 기준 별 인덱스를 생성할 수 없는 경우
- Group By의 결과 또는 DISTINCT 같은 처리 결과를 정렬해야 하는 경우
- 임시 테이블의 결과를 재정렬 하는 경우
- 랜덤하게 결과 레코드를 가져오는 경우

MySQL 서버가 인덱스를 이용하지 않고 별도의 정렬 처리를 수행하게되면 실행 계획의 Extra 컬럼에 Using filesort라는 메시지가 표시된다. 이를 통해 MySQL 서버가 어떤 정렬 처리를 수행했는지 알 수 있다.

### 소트 버퍼

**소트 버퍼란 MySQL이 정렬을 수행하기 위해 할당받은 별도의 메모리 공간이다.**
소트 버퍼의 공간은 한정적이므로 정렬해야 할 **레코드의 건수가 소트 버퍼 공간보다 큰 경우**가 있다. 이런 경우 소트 버퍼에서 정렬을 수행하고 **디스크에 임시 저장**하게 된다. 각 버퍼 크기만큼 정렬된 레코드를 다시 병합하면서 정렬을 수행하는데,  이를 **멀티 머지(Multi-merge)**라고 한다.

이러한 작업은 모두 디스크 읽기/쓰기를 유발하며, 레코드가 많을수록 반복 작업 횟수도 많아진다.

![](https://blog.kakaocdn.net/dn/DeZRe/btrH1CpjvLK/0LkIfuHj50gXOv5dXONsf1/img.png)

소트 버퍼를 크게 잡아서 디스크 작업을 줄이고 메모리 작업을 늘려도 실제 성능에는 큰 차이가 나지 않는다. 오히려 너무 큰 메모리 공간 할당 때문에 성능이 떨어질 수도 있다. 하지만 디스크 I/O를 줄일 수 있으므로 성능이 낮은 장비에는 충분히 도움이 될 수 있다.

### 정렬 알고리즘

- 싱글 패스 정렬 방식
  - 정렬 키와 레코드 전체를 가져와 정렬하는 방식
  - 정렬 대상 레코드의 크기나 건수가 작은 경우 빠른 성능
  - 레코드 전체를 가져오므로 더 많은 소트 버퍼 공간이 필요하다.
  - additional_field : 레코드의 컬럼들은 고정 사이즈로 메모리 저장
  - pack_additional_field :  레코드의 컬럼들은 가변 사이즈로 메모리 저장

![](https://blog.kakaocdn.net/dn/bHovcH/btrHZVRgUUY/sq3lLEQyJemhpkC0Mi2o90/img.png)

- 투 패스 정렬 방식
  - 정렬 키와 RowID만 가져와 정렬하는 방식
  - 정렬 대상 컬럼, 프라이머리 키 값만 소트 버퍼에 담아 정렬하고 정렬된 프라이머리 키로 테이블을 다시 조회해 컬럼을 가져온다.
  - 테이블을 두번 조회하므로 비효율적이다.
  - 다만, 정렬 대상 레코드의 크기나 건수가 많은 경우 효율적이다.


### 정렬 처리 방법

1. 인덱스를 사용한 정렬
2. 조인에서 **드라이빙 테이블만 정렬** - Using filesort
3. 조인에서 조인 결과를 **임시 테이블로 저장 후 정렬** - Using temporary; Using filesort

1번을 기준으로 밑으로 갈 수록 정렬 처리 속도는 느려진다.

옵티마이저는 정렬 대상 레코드를 최소화하기 위해 다음 2가지 방법 중 하나를 선택한다.

- 조인의 드라이빙 테이블만 정렬한 다음 조인을 수행
- 조인이 끝나고 일치하는 레코드를 모두 가져온 후 정렬을 수행

당연하지만 드라이빙 테이블만 정렬한 다음 조인을 수행하는게 가장 효율적이다.

1. **인덱스를 이용한 정렬** (스트리밍 처리)

- 반드시 ORDER BY에 명시된 컬럼이 제일 먼저 읽는 테이블(조인의 경우 드라이빙 테이블)에 속해야 한다.
- ORDER BY의 순서대로 생성된 인덱스가 있어야 한다.
- WHERE절에 첫 번째로 읽는 테이블의 컬럼에 대한 조건이 있다면 ORDER BY와 같은 인덱스를 사용할 수 있어야 한다.
- 해시 인덱스나 전문 검색 인덱스, R-Tree 등에서는 인덱스를 이용한 정렬을 사용할 수 없다.

다음 쿼리는 드라이빙 테이블의 PK를 기준으로 ORDER BY를 수행하므로 인덱스를 이용한 정렬을 사용하게 된다.

```sql
SELECT *
FROM employees e, salaries s 
WHERE s.emp_no=e.emp_no
	AND e.emp_no BETWEEN 100002 AND 100020 
ORDER BY e.emp_no;
```

인덱스는 이미 정렬돼 있기 때문에 순서대로 읽기만하면 된다.

![](https://blog.kakaocdn.net/dn/vcrMh/btrH2nyvk4v/08C0poqdFJnJST10aO5sG0/img.png)

2. **조인의 드라이빙 테이블만 정렬** (버퍼링 처리)

- 조인을 실행하기 전 첫 번째로 읽히는 테이블(드라이빙 테이블)의 레코드를 먼저 정렬한 다음 조인을 실행한다.
- 드라이빙 테이블의 컬럼만으로 ORDER BY절을 작성한 경우 사용 가능하다.

다음 쿼리에서 ORDER BY에 명시된 필드는 드라이빙 테이블의 PK와 아무 연관이 없으므로 인덱스를 이용한 정렬이 불가능하다.

```sql
SELECT *
FROM employees e, salaries s
WHERE s.emp_no=e.emp_no
	AND e . emp_no BETWEEN 100002 AND 100010
ORDER BY e.last_name;
```

하지만 ORDER BY에 명시된 필드는 드라이빙 테이블에 속하므로 옵티마이저는 드라이빙 테이블을 먼저 검색해 정렬을 수행한 후 salaries와의 조인 작업을 실행한다.

![](https://blog.kakaocdn.net/dn/bGprhJ/btrH3ccBYJH/Ezr19Fo0tH6NhNRPTsBxM0/img.png)

**3. 임시 테이블을 이용한 정렬** (버퍼링 처리)

- 위의 경우를 뺀 나머지 패턴의 쿼리에서는 항상 조인 결과를 임시 테이블에 저장하고, 다시 정렬하는 과정을 거친다.
- 실행 계획의 Extra에 Using temporary; Using filesort로 표시되며 앞선 정렬 방법 중 가장 느리다.

다음 쿼리의 정렬 기준(ORDER BY)은 드리븐 테이블의 컬럼이므로 조인된 데이터를 가지고 정렬할 수 밖에 없다.

```sql
SELECT *
FROM employees e, salaries s 
WHERE s.emp_no=e.emp_no
	AND e.emp.no BETWEEN 100002 AND 100010 
ORDER BY s.salary;
```

![](https://blog.kakaocdn.net/dn/5nwRW/btrH1bevi8S/aNMp2dUgzGGo9hqUDkhKM0/img.png)

이렇듯 어떤 테이블이 먼저 드라이빙되어 조인되는지도 중요하지만, 어떤 정렬 방식으로 처리되느냐에 따라 더 큰 성능 차이를 만들어낸다. 가**능하다면 인덱스를 사용한 정렬로 유도하고 그렇지 못하면 최소한 드라이빙 테이블만 정렬해도 되는 수준으로 유도하는 것이 좋은 쿼리 튜닝 방법**이라고 할 수 있다.

### 정렬 처리 방법의 성능 비교

ORDER BY나 GROUP BY 같은 작업은 WHERE 조건을 만족하는 레코드를 모두 가져와서 정렬을 수행하거나 그루핑 작업을 실행해야만 LIMIT로 건수를 제한할 수 있다. 즉, **잘못된 ORDER BY나 GROUP BY 작업은** MySQL 서버가 처리해야할 작업량을 줄이지 못하고 **슬로우 쿼리를 자주 발생 시킨다.**

인덱스를 사용하지 못하는 정렬이나 그루핑 작업이 왜 느리게 작동하는지 이해하려면 쿼리가 처리되는 방법을 이해해야 한다. 쿼리가 처리되는 방법에는 **스트리밍 처리**와 **버퍼링 처리**, 2가지 방식으로 구분할 수 있다.

**스트리밍 처리**

서버쪽에서 처리할 데이터가 얼마인지에 관계없이 조건에 일치하는 레코드가 검색될 때마다 바로 클라이언트로 전송해주는 방식을 의미한다. 따라서 쿼리가 얼마나 많은 레코드를 조회하냐에 상관없이 **빠른 응답 시간을 보장**해준다. 이때, LIMIT 조건을 활용하면 가져오는 레코드 건수가 줄어들어 마지막 레코드를 가져오기 까지의 시간을 상당히 줄일 수 있다.

**버퍼링 방식**

ORDER BY나 GROUP BY 같은 작업은 쿼리 결과가 스트리밍 되는 것을 불가능하게 만든다. WHERE 조건에 일치하는 레코드를 모두 가져온 후, 정렬하거나 그루핑해야 하기 때문이다.

![](https://blog.kakaocdn.net/dn/0bfNh/btrH0SzqWmR/lJFKDIAzrqZEw0HpXau8x0/img.png)

1. 버퍼링 방식으로 처리되는 쿼리는 먼저 결과를 모은다.
2. MySQL 서버에서 일괄 가공해야 하므로 모든 결과를 스토리지 엔진으로부터 가져올 때까지 기다린다.
3. 정렬 작업을 하는 동안 클라이언트는 결과를 기다려야하므로 응답 속도가 느려진다.

위와 같은 방식 때문에 LIMIT로 결과를 제한하더라도 성능 향상에 별로 도움이 되지 않는다.
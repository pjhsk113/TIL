# [Real MySQL 8.0] MySQL의 격리 수준

**트랜잭션의 격리 수준이란** 여러 **트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정**하는 것이다.

격리 수준은 크게 4가지로 나뉜다.

- READ-UNCOMMITTED
- READ-COMMITTED
- REAPEATABLE-READ
- SERIALIZABLE

4개의 격리 수준에서 순서대로 뒤로 갈수록 각 트랜잭션 간의 데이터 고립 정도가 높아지며, 동시 처리 성능도 떨어진다. 일반적인 온라인 서비스 용도의 데이터베이스는 READ-COMMITTED나 REAPEATABLE-READ 중 하나를 사용한다. 이 중 MySQL의 기본 격리 수준은 REAPEATABLE-READ이다.

## READ-UNCOMMITTED 격리 수준

READ-UNCOMMITTED 격리 수준에서는 트랜잭션의 변경 내용이 Commit이나 Rollback 여부에 관계 없이 다른 트랜잭션에서 보인다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07f4495c-6949-43e6-b845-29b409a2999c/Untitled.png)

사용자 A가 INSERT를 수행한 후 Commit 하지 않은 상태에서 사용자 B가 해당 컬럼을 조회하는 경우, 사용자 B는 정상적으로 해당 컬럼을 읽어 올 수 있다.

### READ-UNCOMMITTED의 문제점

READ-UNCOMMITTED 격리 수준에서는 트랜잭션에서 처리 작업이 완료되지 않았음에도 다른 트랜잭션에서 볼 수 있다는 특징 때문에 데이터 정합성에 많은 문제가 발생한다. 사용자 A가 수행한 INSERT문이 롤백되어도 사용자 B의 조회 요청이 정상적으로 동작한다는 뜻이다.

이와 같은 현상을 더티 리드(Dirty read)라고 하며, READ-UNCOMMITTED 격리 수준은 이러한 더티 리드가 발생하므로 일반적인 온라인 서비스에서는 READ-COMMITTED 이상의 격리 수준을 사용할 것을 권장한다.

## READ-COMMITTED 격리 수준

READ-COMMITTED 격리 수준은 오라클 DBMS에서 기본으로 사용하는 격리 수준이며, 온라인 서비스에서 가장 많이 선택되는 격리 수준이다. 어떤 트랜잭션에서 데이터를 변경했더라도 Commit이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있으며 더티 리드같은 현상은 발생하지 않는다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/de4b706a-592d-45a8-adc3-9ab1f3b38ee1/Untitled.png)

READ-COMMITTED 격리 수준에서 부터 앞서 공부했던 언두 로그가 등장한다.
데이터 변경 발생 시 언두 로그 영역에 이전 레코드 정보가 백업된다. 따라서 사용자 A가 데이터를 변경한 후 사용자 B가 해당 컬럼을 조회하면 언두 로그에 백업된 레코드를 조회하게 된다. 따라서 READ-COMMITTED 격리 수준에서는 어떤 트랜잭션에서 변경한 내용이 커밋되기 전까지는 다른 트랜잭션에서 그러한 변경 내역을 조회할 수 없다.

### READ-COMMITTED 격리 수준의 문제점

READ-COMMITTED 격리 수준에서도 NON-REPEATABLE READ라는 부정합의 문제가 존재한다. 레코드가 반복되어 읽어지는 상황에서 발생하는 데이터 부정합 문제이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db21b636-034b-42f3-897f-59b1911f9589/Untitled.png)

**사용자 B가 처음 트랜잭션을 시작하고 특정 조건으로 데이터를 조회했을 때 결과가 없없다**. 하지만 사용자 A가 트랜잭션 중간에 조건에 부합하도록 해당 레코드를 업데이트하고 Commit을 한 후, **사용자 B가 같은 조건으로 레코드를 다시 검색하면 일치하는 결과가 검색된다**.

따라서 하나의 트랜잭션 내에서 똑같은 Select 쿼리를 실행했을 때, 항상 같은 결과를 가져와야한다는 REPEATABLE READ 정합성에 어긋나게 된다. 이러한 현상을 NON-REPEATABLE READ 라고 부른다.

NON-REPEATABLE READ는 일반적인 웹 서비스에서는 큰 문제가 되지 않을 수 있지만, 금전적인 처리와 연결되었을 때 큰 문제점이 될 수 도 있다.

- 출금 처리가 진행되는 와중에 다른 트랜잭션에서 출금 총액을 조회한다면 조회할 때마다 항상 다른 결과를 반환하는 현상 발생

## REPEATABLE-READ 격리 수준

REPEATABLE READ 격리 수준은 InnoDB에서 기본으로 사용되는 격리 수준이다.
이 격리 수준부터 NON-REPEATABLE READ 현상이 발생하지 않는다.

트랜잭션 내부에서 실행되는 SELECT와 외부에서 실행되는 SELECT의 차이가 없는 READ-COMMITTED 격리 수준과는 다르게 REPEATABLE-READ 격리 수준은 기본적으로 SELECT 쿼리 문장도 트랜잭션 범위 내에서만 작동한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58c09502-64c9-4e1c-a2ea-024d29c5e507/Untitled.png)
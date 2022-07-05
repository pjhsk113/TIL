# [Real MySQL 8.0] 그 외에 다양한 인덱스

## 멀티 밸류 인덱스

전문 검색 인덱스를 제외한 모든 인덱스는 인덱스 키와 데이터 레코드가 1:1 관계를 가진다. 하지만 멀티 밸류 인덱스는 하나의 데이터 레코드가 여러 개의 키 값을 가질 수 있는 형태의 인덱스다.

JSON의 배열 타입의 필드에 저장된 원소들에 대한 인덱스를 지원하기 위해 MySQL 8.0 버전에서 업그레이드 됐다.

```sql
// 멀티 밸류 인덱스 생성
CREATE TABLE user (
			user_id BIGINT AUTO_INCREMENT PRIMARY KEY, 
			first_name VARCHAR(10),
			last_name VARCHAR(10),
			credit_info JSON,
			INDEX mx_creditscores (
				(CAST(credit_info->'$.credit_scores' AS UNSIGNED ARRAY))
			)
);

// INSERT
INSERT INTO user VALUES (1, 'Matt', 'Lee1', '{"credit.scores":[360, 353, 351]}');
```

위와 같은 멀티 밸류 인덱스를 활용하려면 반드시 다음 함수들을 활용해서 검색해야만 옵티마이저가 실행 계획을 수립할 수 있다.

- MEMBER OF()
- JSON_CONTAINS()
- JSON_OVERLAPS()

```sql
SELECT * FROM user WHERE 360 MEMBER OF(credit.info->'$.credit_scores');

+----------+------------+----------+--------------------------------------+
| user_id  | first_name |last_name | credit_info                          |
+----------+------------+----------+--------------------------------------+
|       1  | Matt       | Lee      | {"credit_scores": [360, 353, 351]}   |
+----------+------------+----------+--------------------------------------+
```
# 4-2 InnoDB 스토리지 엔진 아키텍처

## InnoDB 스토리지 엔진

MySQL의 스토리지 엔진 중에서 유일하게 레코드 기반의 잠금을 제공하며, 높은 동시성 처리가 가능하고 안정적이며 성능이 뛰어난 스토리지 엔진이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/542f4ab3-514e-4e55-ba8a-fd126a7f79ca/Untitled.png)

### 프라이머리 키에 의한 클러스터링
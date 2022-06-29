# [Real MySQL 8.0] R-Tree와 전문 검색 인덱스

## R-Tree 인덱스

R-Tree 인덱스는 2차원의 데이터를 저장하는 인덱스이다. R-Tree 인덱스를 구성하는 컬럼의 값은 2차원의 공간 개념 값으로 MySQL 공간 인덱스에 이용된다. 공간 인덱스는 위치 기반의 서비스를 구현할 때 주로 사용되며 MySQL의 공간 확장을 이용해 간단하게 구현할 수 있다.

MySQL의 공간 확장에는 크게 **세 가지 기능이 포함**돼 있다.

1. 공간 데이터를 저장할 수 있는 데이터 타입 (POINT, LINE, POLYGON, GEOMETRY)
2. 공간 데이터 검색을 위한 공간 인덱스 (R-Tree 알고리즘)
3. 공간 데이터의 연산 함수 (ST_Contains(), ST_Within() ....)

### 구조 및 특성

MySQL은 공간 정보의 저장 및 검색을 위해 **POINT, LINE, POLYGON, GEOMETRY라는 여러 가지 기하학적 도형 정보를 관리할 수 있는 데이터 타입을 제공한다**.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29e82b60-25cc-42c5-bc22-ed11bafdeb3e/Untitled.png)

GEOMETRY 타입은 나머지 3개 타입의 슈퍼 타입으로, Java의 Object처럼 나머지 모든 객체를 저장할 수 있다. 그리고 R-Tree 알고리즘을 이해하기 위해서 데이터 타입만큼 중요한 개념이 있는데, 바로 **MBR이라는 개념이다.**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6d2c8b2f-b92b-4d21-b545-1a6c5059c8cb/Untitled.png)

MBR(Minimum Bounding Rectangle)은 도형을 감싸는 최소 크기의 사각형을 의미한다. 이러한 MBR들의 포함 관계를 B-Tree 형태로 구현한 인덱스가 R-Tree 인덱스다.

이러한 도형이 저장됐을 때 만들어지는 인덱스 구조를 이해하려면 MBR이 어떻게 되는지 알아야 한다. 예를 들어, 다음과 같은 공간 데이터가 존재한다고 해보자.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f22d9449-ced9-4c00-9d79-da10fa8a7c3a/Untitled.png)

이제 해당 공간 데이터들을 효과적으로 검색하기 위해 인덱스를 만들어야할 차례다. 이때 사용되는게 MBR이라는 개념이다. 공간 데이터를 MBR로 그룹화한 모습을 나타내는 그림을 통해 더 자세히 알아보자.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fef23d46-7357-44e4-9c81-91654cc24dfd/Untitled.png)

- 최상위 레벨: R1 ~ R2
- 차상위 레벨: R3 ~ R6
- 최하위 레벨: R7 ~ R14

**각 공간 데이터들은 MBR로 둘러쌓여있고** 이 MBR들을 그룹화한 **차상위 레벨의 MBR이 존재**한다. 그리고 차상위 레벨 그룹을 둘러싼 MBR이 **최상위 레벨의 MBR**이 된다.

이렇듯 공간을 MBR 그룹으로 나눔으로써 해당 공간 데이터를 찾아 들어갈 수 있는 인덱스의 형태가 만들어진다. 최상위 MBR은 루트 노드에 저장되는 정보이며, 차상위 그룹 MBR은 브랜치 노드가 된다. 마지막으로 각 도형의 객체는 리프 노드에 저장되므로 R-Tree 인덱스의 내부를 표현할 수 있게 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6fd02337-0087-427a-8483-897b32b9002e/Untitled.png)

### R-Tree 인덱스의 용도

R-Tree 인덱스는 위도, 경도 좌표 저장에 주로 사용된다. 좌표 시스템에 기반을 둔 정보에 대해 모두 적용할 수 있는 인덱스이다.
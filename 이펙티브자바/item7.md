# [이펙티브 자바] Item7- 다 쓴 객체 참조를 해제하라

---

자바에서는 메모리 관리를 GC가 해주기 때문에 메모리 관리를 더 이상 신경 쓰지 않아도 된다고 생각할 수 있다. 하지만 GC의 영역에 벗어난 자원이 생성될 수 있으며, 이러한 자원이 생기지 않도록 주의 해야한다.

# 메모리 누수 케이스

## 1. 자기 자신의 메모리를 관리하는 클래스

이펙티브 자바에서는 Stack을 예로 든다. Stack은 객체 참조를 담는 **저장소 풀**을 만들어 원소들을 관리한다. 즉, 자기 자신의 메모리를 직접 관리한다.

문제는 저장소 풀에 등록된 원소들을 pop할 때 생긴다. 일반적인 구현하는 pop의 로직은 Stack의 size를 -1해주는 식으로 구현된다. 이때 Stack은 비활성화 영역에 **다 쓴 참조**를 가지게 되고, 메모리 누수의 원인이 된다.

> 다 쓴 참조(obselete reference)란? 
더 이상 사용하지 않지만 참조되어 GC의 대상이 되지 않는 객체를 말한다. 이러한 객체가 참조하고 있는 또 다른 객체까지 수거의 대상이 되지 않기 때문에 잠재적으로 성능에 악영향을 줄 수 있다.

이를 해결하는 방법은 간단하다.

1. 해당 참조를 다 썼을 때 null 처리(참조 해제) 한다.
2. 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어낸다.

pop 메서드를 살펴보면, 

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1c1e4b5f-58ed-4e08-bead-5ceb55674c00/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210106%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210106T133441Z&X-Amz-Expires=86400&X-Amz-Signature=7f2d37876d5f7bc91392da656b0fd03491203742b603085fa1c03b37d6a20613&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

 removeElementAt 메서드를 사용하고 있다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0d8a2e0d-2aeb-4cf0-b937-a3c2f88243d6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210106%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210106T133443Z&X-Amz-Expires=86400&X-Amz-Signature=24ef8848a50c0177afba89115c5c89ae1a2e4c94dd39adfcdd06a4398a0ac284&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

removeElementAt에서도 원소를 삭제할 때 참조 해제해주고 있는 것을 볼 수 있다.

## 2. 캐시

빠른 속도와 자원을 아끼기 위해 사용하는 캐시가 계속해서 자원에 쌓이게 된다면? 이는 자신의 역할을 제대로 하지 못하게 되고 메모리 누수의 원인이 된다. 따라서 캐시도 정리를 해주어야 한다.

### 해**결 방법!**

- WeakHashMap

WeakHashMap을 통해 특정 Key가 더 이상 사용되지 않다고 판단되면 해당 Key - Value 쌍을 삭제할 수 있다. 

### 시간이 지날수록 캐시의 가치를 떨어트리는 방식

LinkedHashMap.removeEldestEntry을 사용해 가장 오래된 엔트리를 삭제하여 캐시를 관리할 수 있다.

## 3. 리스너, 콜백

리스너와 콜백을 등록만하고 해지하지 않는다면, 계속해서 쌓이고 이는 메모리 낭비가 된다. 

이를 해결하는 방법은 콜백을 WeakHashMap의 key로 등록하고, 사용 후 null을 할당해 gc의 대상이 될 수 있도록 하면 된다.
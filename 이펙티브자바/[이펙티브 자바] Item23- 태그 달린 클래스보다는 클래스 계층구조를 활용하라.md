# [이펙티브 자바] Item23- 태그 달린 클래스보다는 클래스 계층구조를 활용하라

두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스를 태그 달린 클래스라고 한다.

> 태그란?

클래스가 어떠한 타입인지에 대한 정보를 담고있는 멤버 변수(필드)를 의미

태그 달린 클래스는 많은 단점을 가지고 있다.

## 태그 달린 클래스의 단점

### 1. 쓸대없는 코드들이 많다.

태그에 대한 선언이 필요하기 때문에 열거형 타입 선언, 태그 필드, switch 문 등 쓸대없는 코드들이 많아 진다. 태그에 따른 메서드의 행동이 달라져야 하기 때문에 switch 문이 많아져 가독성이 떨어진다.

### 2. 메모리를 많이 사용한다.

다른 행동을 위한 코드도 정의되기 때문에 메모리도 많이 사용하게 된다.

### 3. 불필요한 초기화가 필요하다.

필드를 final로 선언하려면 해당 태그에서 사용되지 않는 필드를 위해 생성자에서 불필요한 초기화가 필요해진다. 생성자가 태그 필드를 설정하고 해당 태그에 쓰이는 데이터 필드들을 초기화하는데 컴파일러가 도와줄 수 있는 부분이 별로 없다. 엉뚱한 필드를 초기화해도 런타임에야 문제가 드러난다.

### 4. 새로운 코드(행동)를 추가하려면 태그를 추가해야한다.

새로운 태그가 추가될 때마다 모든 switch 문에 새로운 태그에 대한 행동을 처리하는 코드를 추가해야한다. 따라서 새로운 행동에 대한 switch 문이 하나라도 빼먹고 작성되면, 런타임에러가 터진다.

다시 말해 태그 달린 클래스는 읽기 어렵기만하고 오류를 내기 쉬운 비효율적인 클래스이다.

## 개선 방안

태그 달린 클래스를 클래스 계층구조로 변경하면 위와 같은 많은 단점들을 개선할 수 있다.

### 1. 루트(root)가 될 추상 클래스를 정의한다.

### 2. 태그에 따라 행동이 달라지던 메서드는 추상 메서드로 구현한다. 그렇지 않다면 일반 메서드로 구현한다.

### 3. 공통으로 사용되는 필드들은 루트에 올린다.

### 4. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.

 → 각 의미에 해당하는 필드를 구체 클래스에 정의한다.

### 5. 구체 클래스에서 추상 메서드를 구현한다.

코드로 살펴보면 다음과 같다.

```java
// 태그 달린 클래스
class Figure {
    enum Shape {RECTANGLE, CIRCLE};

    // 어떤 모양인지 나타내는 태그 필드
    final Shape shape;

    // 태그가 RECTANGLE일 때만 사용되는 필드들
    double length;
    double width;

    // 태그가 CIRCLE일 때만 사용되는 필드들
    double radius;

    // 원을 만드는 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형을 만드는 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;

            case CIRCLE:
                return Math.PI * (radius * radius);

            default:
                throw new AssertionError(shape);
        }
    }
}
```

위와 같은 태그 달린 클래스를 **개선 방안의 규칙**을 적용하여 변경하면, 다음과 같이 개선할 수 있다.

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

위와같이 변경하면 컴파일 시에 형 검사(type checking)를 하기 용이해진다. 이제 컴파일러의 도움을 받을 수 있게된 것이다. 또한, 각 구체 클래스의 멤버변수는 final로 선언할 수 있게 된다.

# 핵심 정리

태그 달린 클래스를 사용하지말고 계층 구조로 대체하는 방법을 생각해라. 기존에 태그 달린 클래스가 사용되어지고 있다면, 계층구조로의 리팩토링을 고려해보자.
# 예외는 진짜 예외 상황에만 사용하라

### 예외는 예외 상황에 적합하게 설계되어있다.
```java
try {
    int i = 0;
    while (true) {
        range[i++].climb();
    } catch (ArrayIndexOutOfBoundsException e) {
    }
}
```
위의 코드의 가장 큰 문제점은 **전혀 직관적이지 않다.** 라는 것 하나만으로 이렇게 코드를 사용하면 안된다.
그럼에도 이 코드를 잠시 살펴보면, 위의 코드는 무한루프를 돌다가 배열의 끝에 도달하면 `ArrayIndexOutOfBoundsException`이 발생하면서 끝이나게 되는 형태이다.

이와 같은 실수는 잘못된 추론으로 성능을 높여보려고 한 것에서 비롯되는데, 
JVM은 배열에 접근할 때마다 경계를 넘지 않는지 검사를 하는데 반복문도 배열 경계에 도달하면 종료를 하게 되는데 이 과정이 중복된다고 판단하고 생략하기 위해서 위와 같은 코드를 작성했다고 한다.

하지만 이 검사에는 세가지 면에서 잘못된 추론이다.
1. 예외는 예외 상황에 쓸 용도로 설계되었기 때문에 배열의 끝을 검사하는 것만큼 최적화를 할 동기가 부족하다. 즉, 예외에 관련해서 최적화를 하지 않았을 가능성이 크다.
2. 위의 코드를 try-catch 블론 안에 넣으면 JVM이 적용할 수 있는 최적화가 제한된다.
3. 표준 관용구는 위에서 걱정한 중복 검사를 수행하지 않고 최적화한다.

실제로 위의 잘못된 코드는 표준 광용구를 사용한 반복문보다 2배정도 속도가 느리다.

### 디버깅을 어렵게 한다.
만약 위의 코드에서 반복문에 버그가 숨어 있다면 반복문을 종료하기 위해서 사용한 예외가 반복문에 존재하는 버그를 숨겨서 디버깅을 더 어렵게 한다.

예를 들어 위의 코드의 반복문에서 다른 배열을 참조하다가 실제로 `ArrayIndexOutOfBoundsException`가 발생했다고 가정해보겠다.
만약 표준 관용구를 사용했다면 예외를 잡지않고 호출스택을 출력하고 즉시 쓰레드를 정지시켰을 것 이다. (문제 상황을 인지할 수 있음)
하지만 위의 코드의 경우엔 `ArrayIndexOutOfBoundsException`를 정상적인 상황으로 판단하고 넘어가게 된다.

위의 이야기를 요약하자면, 예외는 진짜 예외 상황에 사용하고, 일상적인 코드 흐름제어의 용도로는 사용해선 안된다.
(성능의 목적으로 자바에서 의도하지 않는 방향으로 사용해서 효과를 보더라도, 자바 플랫폼이 계속해서 최적화해가고 있기 때문에 그 이점이 오래가지 않을 수 있고, 오히려 유지보수 문제점이 발생함)

### 상태 의존적 메서드가 존재하다면 상태 검사 메서드를 함께 제공한다.
이러한 원칙은 API를 설계할때도 동일하게 적용이 됩니다. 클라이언트에서 흐름제어를 위해 설계된 API의 예외를 사용하는 일이 없어야한다.
만약 상태 의존적 메서드를 제공하는 클래스라면 상태 검사 메서드도 함께 제공하도록 한다.

대표적인 예로 `Iterator` 인터페이스의 상태 의존적 메서드 `next`와 상태 검사 메서드 `hasNext`가 있다.
(만약 `Iterator`가 `hasNext`를 제공하지 않았다면 그 일을 클라이언트에서 대신해야함)

`hasNext`를 사용한 for은 아래와 같다. (for-each문도 내부적으로 `hasNext`를 사용함)
```java
for (Iterator<Foo> i = collection.iterator(); i.hasNext();) {
    Foo foo = i.next();
    ...
}
```

### 상태 검사 메서드 대신 사용할 수 있는 선택지
만약 상태 검사 메서드를 사용하지 않는다면 `Optional`이나 `null`과 같은 특정값 반환하는 방법도 있다.
어떤 상황에 어떤 방법을 선택해야하는가?라는 생각이 들 수 있다.

1. 외부 상황에 의해 상태가 변할 수 있다면 `Optional` 혹은 특정 값을 사용한다.
(외부 동기화 없이 여러 쓰레드가 동시 접근해서 상태 의존적 메서드가 호출 되는 사이에 객체의 상태가 변환될 수 있기 때문에.)
2. 성능이 매우 중요한 상황인데, 상태 검사 메서드가 사애 의존적 메서드의 작업 일부를 **중복 수행** 한다면 `Optional`이나 특정 값을 사용한다.
3. 다른 모든 경우엔 상태 검사 메서드 방식이 더 낫다. (가독성이나 디버깅이 쉬움)
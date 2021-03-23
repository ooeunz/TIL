# 예외의 상세 메세지에 실패 관련 정보를 담으라

예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 스택 추적 정보를 자동으로 출력한다. 스택 추적은 예외 객체의 toString 메서드를 호출해 얻는 문자열이다.

예외의 toString 메서드에 실패 원인에 관한 정보를 가능한 한 많이 담아 반환하는 일은 아주 중요하다.

### 실패의 순간 포착하기
실패의 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 메세지에 담아야 한다.
예컨대 `IndexOutOfBoundsException`의 상세 메세지는 범위의 최소, 최대, 그리고 그 범위를 벗어난 인덱스 값을 모두 담아야 한다. 그래야 이 메세지를 보고 무엇을 고쳐야 할지를 분석하는 데 도움이 된다.

### 실패 메세지
예외의 상세 메세지와 최종 사용자에게 보여줄 오류 메세지를 혼동해서는 안된다.
최종 사용자는 친절한 안내메세지를 받아야 하는 한편, 예외 메세지는 가독성보다는 담긴 내용이 훨신 중요하다. 이를 적절히 구분하여 오류 메세지를 만들자.

실패를 적절히 포착하려면 필요한 정보를 예외 생성자에서 모두 받아서 상세 메시지까지 미리 작성해놓는 방법도 좋다.

```java
public class IndexOutOfBoundsException extends RuntimeException {
    private static final long serialVersionUID = 234122996006267687L;
    public IndexOutOfBoundsException() {
        super();
    }
    public IndexOutOfBoundsException(String s) {
        super(s);
    }
    public IndexOutOfBoundsException(int index) {
        super("Index out of range: " + index);
    }
}
```
현재 IndexOutOfBoundsException 생성자는 String을 받고 있고, 자바 9에서 드디어 IndexOutOfBoundsException 에 정수 인덱스 값을 받는 생성자가 추가되었다. 

하지만 아쉽게도 최솟값과 최댓값까지는 받지 않는다. 

이처럼 자바 라이브러리에서는 이조언을 적극 수용하지는 않지만, 저자는 다음과 같이 구현하는 것을 강력히 권장한다. 

```java
/**
* IndexOutOfBoundsException을 생성한다
* @param lowerBound 인덱스의 최솟값
* @param upperBound 인덱스의 최댓값 + 1
* @param index 인덱스의 실젯값
*/
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index){
    // 실패를 포착하는 상세 메시지를 생성한다.
    super(String.format("최솟값: %d , 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
    
    // 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
    this.lowerBound = lowerBound;
    this.upperBound = upperBound;
    this.index = index;
}
```
# 아이템 7. 다 쓴 객체를 참조 해제하라

- C, C++과 달리 자바는 GC가 존재하기 때문에 개발자가 상당 부분 메모리 관리에 신경쓸 필요가 없다.
- 하지만 메모리를 전혀 신경쓰지 않아도 되는 것은 아니다.

## 메모리 누수가 발생하는 스택 코드 예제

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAILT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAILT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        encureCapacity(); // 배열 크기를 늘려야할 때 늘려주는 메서드
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size]; // 참조가 여전히 남아있음
    }
}
```

- 위 스택을 오래 사용하면 메모리 누수가 발생하여 점차 GC와 메모리 사용량이 늘어나 성능이 저하될 것이다.
    - 심한 경우 디스크 페이징이나 `OutOfMemoryError`를 일으킨다.
- `pop()` 메서드에서 꺼내진 객체들을 더 이상 사용하지 않아도 참조를 여전히 가지고 있기에 GC가 회수하지 않는다.
- 가비지 컬렉션 언어에서 메모리 누수를 찾기 아주 까다롭다.
    - 회수되지 않은 가비지는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수하지 못하게 한다.

## 참조 해제하기 (null 처리)

- 이를 해결하기 위해선 참조를 다 썼을 때 `null` 처리하면 된다.

    ```java
        public Object pop() {
            if (size == 0) {
                throw new EmptyStackException();
            }
            Object result = elements[--size];
            elements[size] = null;
            return result;
        }
    ```

    - 다 쓴 참조를 `null` 처리하게 되면 실수로 이 참조를 사용하려 하면 NPE가 발생하기에 프로그램 오류를 조기에 발견할 수도 있다.
    - 하지만 모든 객체를 일일이 `null` 처리하게 되면 코드가 지저분해지기 때문에 `null` 처리는 예외적인 경우여야 한다.
    - 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다. (메서드의 지역 변수로만 사용)

## null 처리를 해야하는 경우

- 일반적으로 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의해야 한다.
- `Stack` 코드에서 `null` 처리가 필요했던 이유는 스택이 ‘자기 메모리를 직접 관리’하기 때문이다.
    - `Stack`에서 `elements` 배열로 원소들을 관리하고 `size` 이하의 활성화 영역과 `size` 밖의 비활성화 영역이  존재한다.
    - 비활성화 영역이 가비지라는 것을 GC가 알턱이 없다. (GC 입장에선 똑같은 참조가 있어 유효한 객체)
    - 그러므로 개발자가 참조를 해제하는 작업을 추가적으로 해줘야 한다.
- 캐시 역시 메모리 누수를 일으키는 주범이다.
    - 캐시에 객체를 넣고 다 쓴 뒤에도 참조를 놔두는 일이 자주 일어난다.
    - 캐시 외부에서 key를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요하다면 `WeakHashMap`을 사용해 캐시를 만들면 된다.
    - `WeakHashMap`을 사용하면 다 쓴 엔트리는 그 즉시 자동으로 제거된다.
- 리스너 혹은 콜백을 사용하는 경우에도 메모리 누수가 발생할 수 있다.
    - 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면 콜백은 계속 쌓여갈 것이다.
    - 이 경우도 `WeakHashMap`에 콜백을 키로 저장하면 해결할 수 있다.

## Capture List

기본적으로 클로저는 주변 범위에서 변수와 상수를 strong reference로 캡쳐한다. capture list를 사용하여 클로저에서 값이 캡처되는 방식을 명시적으로 제어할 수 있다. capture list를 사용할 때는 파라미터 이름, 타입, 반환 타입을 제외하더라도 `in` 키워드를 반드시 써야한다.

캡처 리스트는 클로저가 **생성될 때 초기화** 된다. 각 항목은 주변 범위에서 이름이 같은 상수 혹은 변수로 초기화 된다.

```swift
var a = 0
var b = 0
let closure = { [a] in
    print(a, b)
}

a = 10
b = 10
closure()
// Prints "0 10"
```

위 코드에서 a는 클로저 주변의 변수와 클로저 내부의 상수가 있다. 클로저 내부의 a는 클로저가 생성될 때 초기화 된다. 그러나 연결은 되어있지 않기 때문에 클로저를 실행 했을 때, 처음 캡처한 0값이 출력된다. 그러나 b 변수는 하나이기 때문에 변화가 모두 적용된다.



+) 내용 추가

위 코드의 b를 보면 알 수 있듯이 클로저에서 명시적 capture list를 작성하지 않으면 value type 변수일지라도 **reference capture**가 일어난다. (그래서 b 값이 10으로 출력됨) 따라서 value copy를 위해서는 [b] 와 같이 captrue list를 명시해 주어야 한다.

또한 value capture된 값은 closure 안에서 변경이 불가능하다. b를 value capture 하지 않은 상태에서 closure 내부에서 변경하게 되면 reference capture이기 때문에 기존 b 역시 변경된다.





```swift
class SimpleClass {
    var value: Int = 0
}
var x = SimpleClass()
var y = SimpleClass()
let closure = { [x] in
    print(x.value, y.value)
}

x.value = 10
y.value = 10
closure()
// Prints "10 10"
```

그러나 이러한 차이는 참조 의미론을 가진 변수에서는 적용되지 않는다. 그리고 표현 값의 타입이 class면 weak나 unowned로 값의 참조를 명시할 수 있다.

```swift
myFunction { [weak parent = self.parent] in print(parent!.title) }
```

또한 임의의 식을 capture list에서 명명된 값에 바인딩 할 수도 있다.
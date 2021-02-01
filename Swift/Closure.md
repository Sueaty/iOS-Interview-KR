## Closure

https://docs.swift.org/swift-book/LanguageGuide/Closures.html

클로저는 코드 블럭. 일급시민으로 전달인자, 변수, 상수 등으로 저장 및 전달이 가능하다. 함수는 클로저의 일종으로 이름이 있는 클로저라고 생각하면 된다.

값으로써 함수를 인수로 지정하고, 다른 함수에 전달하여 사용하는 것을 클로저 라고한다. Swift의 클로저는 C 나 Objective-C에서의 blocks와 유사하고, 다른 프로그래밍 언어의 lambdas와 비슷하다.

클로저는 어떤 상수나 변수의 참조를 캡쳐해 저장할 수 있다. 이 캡쳐와 관련된 모든 메모리 관리는 Swift가 알아서 처리한다.

전역 함수나 중첩 함수는 클로저의 특별한 케이스라고 볼수 있다. 클로저는 다음 세 가지 형태 중 하나로 나타난다.

- **전역 함수**: 이름을 가지고 있고, 값을 캡쳐하지 않는 클로저
- **중첩 함수**: 이름을 가지고 있고, 그들의 폐쇄함수(감싸고 있는 함수)에 의해 값이 캡쳐될 수 있음
- **클로저 표현**: 이름을 가지고 있지 않고, 주변 컨텍스트로부터 값을 캡쳐할 수 있는 간략화된 구문

Swift의 클로저 표현은 깔끔하고, 명확한 스타일로, 일반적 상황에서 간단하면서도 군더더기 없는 최적화 기능을 제공한다. 다음은 최적화의 일종이다.

- 인자와 반환 값의 타입을 추론
- 단일 표현식 클로저로부터 암시적 반환
- 축약된 인자 이름
- 후행 클로저

## Capturing Values

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}

let incrementByTen = makeIncrementer(forIncrement: 10)

print(incrementByTen()) // 10
print(incrementByTen()) // 20
print(incrementByTen()) // 30
```

**다음과 같은 nested function 에서 incrementByTen 함수가 각각 실행되지만, `runningTotal` , `amount` 가 캡쳐링 되어 계산이 누적된 결과를 갖게 된다.**

### Autoclosures

인자 값이 없으며, 특정 표현을 감싸서 다른 함수에 전달인자로 사용할 수 있는 클로저. 자동클로저는 클로저를 실행하기 전까지 실제 실행이 되지 않는다.

```swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)

// 자동 클로저 (인자가 없고, 특정 표현을 감싸서 다른 함수에 인자로 사용할 수 있음)
let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)

// 이 때 지연 실행된다
print("Now serving \\(customerProvider())!")
print(customersInLine.count)

func serve(customer customerProvider: () -> String) {
    print("Now serving \\(customerProvider())")
}
serve(customer: { customersInLine.remove(at: 0) })

// @autoclosure 키워드 붙이기
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \\(customerProvider())")
}

// 클로저 형식({})으로 넣지 않아도 됨
serve(customer: customersInLine.remove(at: 0) )
```
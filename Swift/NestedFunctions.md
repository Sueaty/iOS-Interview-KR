## Nested Functions

https://docs.swift.org/swift-book/LanguageGuide/Functions.html

함수 내부 안에 함수를 정의할 수 있는데, 이를 `nested functions` 라고 한다. (이하 중첩 함수)

중첩 함수는 기본적으로 바깥 세계에서 숨겨져 있지만, 그들의 폐쇄 함수(감싸고 있는 함수를 말하는 듯)에 의해 호출되고, 사용될 수 있다. 폐쇄 함수는 그 들의 중첩 함수 중 하나를 반환하여 다른 범위에서 사용가능하게 할 수도 있다.

```swift
// 내부에 함수 정의를 가진 중첩 함수
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    func stepForward(input: Int) -> Int { return input + 1 }
    func stepBackward(input: Int) -> Int { return input - 1 }
    return backward ? stepBackward : stepForward
}

var currentValue = -4
// 함수를 변수에 저장!
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)

while currentValue != 0 {
    print("\\(currentValue)")
    currentValue = moveNearerToZero(currentValue)
}

print("zero!")
```

chooseStepFunction() 함수 내부에 새로운 함수 정의가 되어있고, 중첩 함수를 반환시켜 함수 외부에 저장해서 사용하고 있다.


## Equatable, Comparable, Hashable

https://jcsoohwancho.github.io/2019-10-27-Equatable,-Comparable,-Hashable/

### Equatable

==, != 연산 가능, sequence나 collection에서 contains 가능

구조체의 경우 모든 저장 프로퍼티가 equatable을 채택해야 함.

열거형의 경우, 모든 연관 값들이 equatable을 채택해야 함. 연관 값이 없을 때는 자동으로 컴파일러가 채택해 줌

swift의 기본 자료형들은 대부분 equatable을 채택하고 있음

위 경우가 아닐 경우 == 연산을 직접 구현해주면 됨

> Comparable과 Hashable은 Equatable 을 채택한다.

### Comparable

부등호 연산(>, <, ≥, ≤)이 가능하고, sequence나 collection에서 정렬 가능.

equatable을 채택하기 때문에 == 연산과 < 연산을 구현해줘야 한다.

### Hashable

Hasher를 통해 Int형의 해쉬 값을 만들어 낼 수 있도록 한다. Set과 Dictionary에서 키로 사용 가능. hash() 함수를 구현해줘야 함.

hasher의 combine() 메소드로 값을 제공해 주도록 구현된다. 컴파일러가 기본 구현을 자동으로 해줄 수도 있다.

```swift
func hash(into hasher: inout Hasher) {
    hasher.combine("변수")
}
```
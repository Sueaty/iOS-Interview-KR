## Map, FlatMap, CompactMap

https://jinshine.github.io/2018/12/14/Swift/22.고차함수(2) - map, flatMap, compactMap/

https://zeddios.tistory.com/448

### map

map은 배열 내부의 각 값을 mapping 해준다. 각 요소의 값을 변경하여 배열로 반환

```swift
let array = [1, 2, 3, 4, 5]
array.map { $0 + 1 }

// [2, 3, 4, 5, 6]
```

### compactMap

Swift 4.1에서 기존 flatMap의 일부 기능을 compactMap의 기능으로 바뀌고 서로 쓰임이 다르게 되었다.

```swift
let array = [1, nil, 3, nil, 5]
let compactMapArray = array.compactMap { $0 }

// [1, 3, 5]

let str: String? = "hi"
let strArray = [str, nil, "hi"]
let compactMapStrArray = strArray.compactMap { $0 }

// ["hi", "hi"]
```

다음과 같이 1차원 배열에서 compactMap은 nil값을 제거하고, 옵셔널 바인딩 해준다.

2차원 배열에서는 ?

```swift
let array2: [[Int?]] = [[1, 2, 3], [nil, 5]]
let compactMapArray2 = array2.compactMap { $0 }

// [[Optional(1), Optional(2), Optional(3)], [nil, Optional(5)]]
```

compactMap은 2차원 배열에서 아무 소용 없어진다..

### flatMap

위와 같은 1차원 배열에 flatMap을 적용해보면

```swift
let flatMapArray = array.flatMap { $0 }
let flatMapStrArray = strArray.flatMap { $0 }

// [1, 3, 5]
// ["hi", "hi"]
```

결과는 compactMap과 동일하게 나오지만

<img width="734" alt="스크린샷 2021-01-27 오후 11 37 03" src="https://user-images.githubusercontent.com/62557093/106006440-91405380-60f8-11eb-8562-95ba09b21ebf.png">

compactMap으로 쓰라고 나온다. 언제까지 1차원 배열에서 flatMap 결과를 보장할지 모르니 이런 경우에는 compactMap을 쓰도록 하자.

2차원 배열에서도 해보면

```swift
let array2: [[Int?]] = [[1, 2, 3], [nil, 5]]
let flatMapArray2 = array2.flatMap { $0 }

// [Optional(1), Optional(2), Optional(3), nil, Optional(5)]
```

2차원이었던 배열을 1차원으로 flat하게 만들어주지만, nil을 제거해 주지 않고 옵셔널 바인딩도 수행하지 않는다.

즉, 단순히 2차원 배열을 1차원으로만! 만들어줌

```swift
let arrayNonOptional: [[Int]] = [[1, 2, 3]]
let flatMapArrayNonOptional = arrayNonOptional.flatMap { $0 }

// [1, 2, 3]
```

optional이 아닐 때에도 2차원을 1차원 배열로만 바꿔주는 역할만 한다.
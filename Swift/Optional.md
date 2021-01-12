### Optional과 관련된 면접 질문

* Optional이 무엇인지 설명하시오.
* Optional의 값 추출 방식에 대해 설명하시오.
  * guard 구문을 활용한 optional binding에 대해 설명하시오.
  * Optional Chaining이 무엇인지 설명하시오.


# Optional

## Optional의 기본

```swift
public enum Optional<Wrapped>: ExpressibleByNilLiteral {
     case none	// abscence of value
     case some(Wrapped)	// presence of a value, stored as `Wrapped`
}
```

Optional은 **변수나 상수 등에 값이 있음이 보장되지 않을 때 사용하는 기능**으로 Swift 언어의 특징 중 하나인 안전성(safe)이 보장되는 방법 중 하나다. <br>
열거형으로 정의되어 있는 optional은 none과 some 케이스를 갖는데 이 때 none은 값이 없음을, some은 값이 있음을 나타낸다. <br>
하지만 유효한 값이 존재하더라도 some의 연관값 Wrapped에 할당되어있기 때문에 이 값을 사용하기 위해서는 **3가지 방법** 중 하나를 이용하여 값을 추출해내야 한다.


> < **Optional... 조금 더** 😀 > <br>
> Optional을 사용하며 반환할 수 있는 nil은 Objective-C에서 반환되던 nil과는 다르다. <br>
> Objective-C의 경우 유효한 **객체** 가 없을 때 함수에서 nil 포인터를 반환할 수 있다. 참조형식에 대해서만 적용이 가능한 개념이다. <br>
> 반면 Swift의 optional은 모든 타입에 적용이 가능해 Objective-C보다 표현성이 더 뛰어나다.

## Optional의 값 추출 방식

Optional을 추출하는 방법은 3가지가 있다.

* 강제 추출 (Forced Unwrapping / Unconditional Unwrapping)
* 옵셔널 바인딩 (Optional Binding)
* 암시적 추출 옵셔널 (Implicitly Unwrapped Optional)

### 강제 추출

Optional로 선언한 상수/변수에 값이 있음이 **확실**할 때 !를 통해 강제 추출을 할 수 있다. 그러나 강제 추출을 시도했는데 반환 값이 nil이라면 **런타임 오류**가 발생한다.

```swift
let a: Int? // 자동으로 nil 할당
print(a!) // Fatal error: Unexpectedly found nil while unwrapping an Optional value
```

### 옵셔널 바인딩

Optional binding은 optional에 값이 있는지 확인하기 위해 사용하며 if 또는 while 구문 등과 결합하여 사용한다. <br>
값이 있다면 추출한 값을 scope 내에서만 사용 가능한 상수/변수로 할당하여 사용할 수 있게끔 만들 수 있다. 만약 값이 없다면 블록 내부 명령문은 실행되지 않는다. <br>

```swift
var address: String? = "Seoul, Korea"
if let addr = address {
   print(addr)
}
```
Optional binding은 guard 문을 이용할 수도 있다. if 문은 if scope 내부에서 쓰기 위한 변/상수에 값을 넣었다면 guard 문에서는 optional binding된 상수를 guard 구문 실행 후 지역 상수처럼 쓸 수 있다. 하지만 guard 문은 언제나 else와 함께 쓰여 guard 문보다 상위 코드 블록을 종료하는 코드를 작성해야 하므로 예외사항을 처리해야 할 때 사용하면 좋다.

```swift
// URLSession dataTask를 통해 data, response, error 받아서 data 처리
guard let data = data else {
   completion(.failure(.APIInvalidResponse))
   return
}
```

### 암시적 추출 옵셔널

프로젝트 구조/프로그램의 로직 상 optional이 무조건 값이 있다면 optional binding을 통해 추출하는 방식이 번거로울 것이다. <br>
이 때 optional로 만들고자 하는 변수나 상수의 타입 뒤에 ? 대신 !를 붙여 implicitly unwrapped optionals를 사용하면 된다. <br>
암시적 추출 옵셔널을 주로 볼 수 있는 경우는 바로 참조 순환을 피하기 위한 class를 초기화할 때 이다.([바로가기](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID55)) 

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

## Optional Chaining

Optional chaining은 현재 nil일 수도 있는 properties, methods 와 subscripts를 호출하거나 값을 가져올 때 사용하는 방법으로 optional을 반복적으로 사용하여 중첩시킨다. Optional chaining을 통한 접근 역시 optional을 반환하므로 추출 방식 중 하나를 선택해서 진행해야한다.

```swift
let person: Person = Person(name: "Sueaty")
if let roomNumber: Int = person.address?.room?.number {
   print("Sueaty stays in room: #\(roomNumber)")
} else {
   print("Cannot find room number")
}
```

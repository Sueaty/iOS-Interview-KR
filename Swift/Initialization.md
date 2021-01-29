## Two-Phase Initialization

https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID220

Swift에서 클래스는 2단계로 초기화 된다. 첫 번째는 각 저장 프로퍼티들이 **그 클래스에 의해 초기 값을 할당** 받는다. 모든 저장 프로퍼티의 초기 값이 결정되면, 두 번째 단계가 시작된다. 각 클래스는 새로운 인스턴스의 사용 준비가 끝나기 전에 **각 저장 프로퍼티를 커스터마이즈 할 기회를 가진다.**

2단계의 초기화 과정은 클래스 계층에서의 각 클래스의 완전한 유연함을 제공하면서 안전하게 초기화 할 수 있도록 해준다. 또한 프로퍼티 값이 초기화되기 전에 접근하는 것을 막아주고, 다른 예기치 않은 이니셜라이저에 의해 다른 값으로 설정되는 것을 방지한다.

Swift의 two-phase initialization은 Objective-C에서의 초기화와 비슷하다. 가장 큰 차이점은 첫 번째 단계에서 Objective-C는 모든 프로퍼티에 0이나 nil 값을 할당하지만, Swift는 사용자 지정 초기 값을 지정할 수 있어 유연하고 0이나 nil이 기본 값으로 적합하지 않는 타입들을 다룰 수 있다.

Swift 컴파일러는 two-phase initialization이 에러없이 완료되는지 확인하기 위해 4개의 safety-checks를 수행한다.

### Safety check1

*지정된 이니셜라이저는 모든 프로퍼티가 superclass 이니셜라이저까지 위임되기 전에 초기화 되어야 한다.*

객체의 메모리는 모든 프로퍼티의 초기 상태가 있어야만 완전히 초기화 된 걸로 간주된다. 이 조건을 충족시키기 위해서는 현재 이니셜라이저가 체인을 넘겨주기 전에 (superClass의 initializer를 호출하기 전에) 모든 프로퍼티의 초기화를 진행해야 한다.

```swift
class SuperClass {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

class SubClass: SuperClass {
    var type: String
    
    init(type: String) {
        super.init(name: "hi", age: 13)
        self.type = type // error 
    }
}
```

다음과 같이 SubClass의 self.type이 초기화 되지 않은 상태에서 super class이 생성자를 호출할 수 없다.

### Safety check2

*지정된 이니셜라이저는 상속된 프로퍼티에 값을 할당하기 전에 super class의 이니셜라이즈를 호출해야한다. 그렇지 않으면 지정된 이니셜라이저가 할당한 새 값은 super class의 이니셜 라이저에서 다시 쓰여질 것이다.(overwritten)*

```swift
class SubClass: SuperClass {
    var type: String
    
    init(type: String) {
        self.type = type
        self.age = 11 // error
        super.init(name: "hi", age: 13)
    }
}
```

### Safety check3

*convenience 이니셜라이저는 어떤 프로퍼티(같은 클래스에 의해 정의된 프로퍼티 포함)에 값을 할당하기 전에 다른 이니셜라이저를 호출해야한다. 그렇지 않으면 convenience 이니셜라이저가 할당한 새 값은 지정된 이니셜라이저에 의해 다시 쓰여질 것이다.*

```swift
convenience init(age: Int) {
	self.name = "welly" // error
	self.init(name: "welly", age: age)
}
```

### Safety check4

*이니셜라이저는 첫 번째 초기화가 완료되기 전까지 어떠한 인스턴스 메서드 호출, 인스턴스 프로퍼티 값 읽기, self 참조 등을 할 수 없다.*

클래스 인스턴스는 첫 번째 초기화가 끝나기 전까지는 유효하지 않은 상태이다. 첫 번째 초기화가 끝나고 유효한 상태가 되어야지만 프로퍼티에 접근하거나 메소드를 호출할 수 있다.



### 위 4 가지 규칙들에 의한 two-phase 초기화 방법

**[Phase 1]**

- 지정 이니셜라이저 (그냥 init)이나 convenience 이니셜라이저가 class 에서 호출
- 해당 클래스의 새로운 인스턴스를 위한 메모리 할당. 메모리는 아직 초기화 되지 않은 상태
- 지정 이니셜라이저는 모든 저장 프로퍼티가 해당 클래스에 의해 초기 값을 가지고 있는지 확인. 저장 프로퍼티에 대한 메모리가 현재 초기화 됨
- 지정 이니셜라이저는 super class의 이니셜라이저를 호출하여 저장 프로퍼티에 대해 같은 작업을 수행
- 체인의 맨 위에 도달할 때까지 클래스 상속 체인을 거슬러 올라감
- 가장 상위에 도달하여 마지막 class가 모든 저장 프로퍼티가 값을 가짐을 보장하면, 인스턴스의 메모리가 완전히 초기화되었다고 간주하고 phase 1 초기화 종료

**[Phase 2]**

- 체인의 상단에서부터 다시 내려오면서 각 지정 이니셜라이저는 인스턴스를 추가로 커스터마이징 할 수 있음. 이니셜라이저는 self에 접근 가능하고 프로퍼티 수정, 인스턴스 메소드 호출 등의 작업이 가능
- 마지막으로, 체인의 어떠한 convenience 이니셜라이저도 인스턴스를 커스터마이징 가능하고 self를 호출 가능





<img width="521" alt="스크린샷 2021-01-29 오후 10 05 44" src="https://user-images.githubusercontent.com/62557093/106278496-23259900-627e-11eb-9b53-3458ebb8d308.png">

예를 들어, 위의 서브클래스의 convenience 이니셜라이저 호출로 시작되었다고 보자. convenience 이니셜라이저는 현재 어떤 프로퍼티도 변경할 수 없다. 같은 클래스의 지정 이니셜라이저로 간다.

지정 이니셜라이저는 현재 서브클래스의 모든 프로퍼티 값이 초기화 되었는지 확인한다. **(safety check1)**  그 후 슈퍼클래스의 지정 이니셜라이저로 체인을 타고 올라간다.

슈퍼클래스의 지정 이니셜라이저도 모든 슈퍼클래스의 프로퍼티 값이 초기화되었는지 확인하고 더이상 올라갈 체인이 없으면 (슈퍼클래스의 슈퍼클래스가 없으면) 모든 프로퍼티 값이 초기화되고, 메모리도 완전히 초기화 되었다. - Phase1 끝



<img width="533" alt="스크린샷 2021-01-29 오후 10 05 58" src="https://user-images.githubusercontent.com/62557093/106278516-2b7dd400-627e-11eb-80e0-edee7f4f0318.png">

이제 슈퍼클래스의 지정 이니셜라이저는 인스턴스를 추가로 커스터마이징 할 수 있다. 슈퍼 클래스의 커스터마이징이 끝나면, 서브 클래스의 지정 이니셜라이저가 커스터마이징 할 수 있다. (안해도 됨) 마지막으로 서브클래스의 지정 이니셜라이저의 커스터마이징이 끝나면 원래 호출되었던 convenience 이니셜라이저가 추가적인 커스터마이징을 할 수 있게 된다. - Phase2 끝



## Failable Initializers

https://wlaxhrl.tistory.com/49

초기화를 하다가 실패할 경우가 있는 경우는 `init?()` 과 같은 형식으로 failable init을 할 수 있다.

failable init과 designed init을 동일한 파라미터 타입/이름으로 둘 다 정의는 안된다.

failable initializer는 초기화하는 타입의 **옵셔널 값을 생성**한다.

```swift
init?(species: String) {
	if species.isEmpty { return nil }
	self.species = species
}
```

다음과 같이 실패할 경우에는 nil을 반환하도록 한다.

```swift
enum TemperatureUnit {
    case kelvin, celsius, fahrenheit
    init?(symbol: Character) {
        switch symbol {
        case "K":
            self = .kelvin
        case "C":
            self = .celsius
        case "F":
            self = .fahrenheit
        default:
            return nil
        }
    }
}
```

enum에서는 해당하는 경우가 없을 경우 초기화에 실패하고 싶을 때 사용



## Requierd Initializers

클래스의 이니셜라이저에 required 수식어를 붙이면 해당 클래스를 상속받는 모든 서브클래스는 해당 이니셜라이저를 구현해야 한다. override 키워드는 안붙여도 된다.
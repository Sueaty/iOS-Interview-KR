## Delegate/ Notification/ KVO

https://medium.com/@Alpaca_iOSStudy/delegation-notification-그리고-kvo-82de909bd29

특정 이벤트가 일어나면 원하는 객체에 알려주어 해당되는 처리를 하는 방법

객체가 다른 객체와 소통은 하지만 묶이기는 (종속되기는) 싫을 때 사용

### **Delegate**

프로토콜을 사용하여 delegate로 지정된 객체가 해야하는  메소드의 원형을 명시하고, delegate 역할의 객체는 이 프로토콜을 채택하여 구현한다.

```swift
// 프로토콜 정의 
protocol SomeDelegate: class {
    func someFunction(someProperty: Int)
}

// delegate를 가지는 class 선언 
class SomeView {
    weak var delegate: SomeDelegate?
    
    func someTappped(num: Int) {
        delegate?.someFunction(someProperty: num)
    }
}

// view의 delegate로 지정 및 함수 실행
class SomeController {
    private var view: SomeView?
    
    init() {
        view = SomeView()
        view?.delegate = self
        view?.someTappped(num: 1)
    }
}

// delegate 프로토콜 채택 및 구현 
extension SomeController: SomeDelegate {
    func someFunction(someProperty: Int) {
        print("tap \\(someProperty)")
    }
}

let controller = SomeController()
```

<img width="427" alt="스크린샷 2021-02-04 오후 10 35 24" src="https://user-images.githubusercontent.com/62557093/106899915-46968b00-6739-11eb-8e3e-06ffa327358d.png">

위의 코드를 실행하면 controller init() 함수에서 view의 someTapped() 함수를 실행하고, view의 delegate로 지정된 controller가 구현한 someFunction이 실행되게 된다

즉, view의 행동을 controller에게 위임함으로서 역할을 떠넘기기(?)함.

- 특징

  엄격한 문법으로 프로토콜 구현이 제대로 되어있지 않을 때 컴파일 시에 오류를 확인할 수 있다. 소통 과정을 유지하고 모니터링 할 객체가 필요 없다. 하지만 많은 줄의 코드가 필요하고, 많은 객체들에게 이벤트를 알려주는 것이 어렵고 비효율적이다.

### **Notification**

Notification Center라는 싱글턴 객체로 원하는 이벤트의 observer로 등록한 객체들에게 이벤트 발생 시에 notification을 post하여 알려준다. Notification name이라는 key값 이용.

- 특징

  코드 구현이 쉬움. 여러 객체들에게 동시에 이벤트 발생을 알려줄 수 있다. 관련 정보를 userInfo로 전달 가능. key값으로 notification name을 이용하여 컴파일 타임에 오류를 잡아내기 어렵다.

### **KVO(Key-Value Observing)**

https://www.hackingwithswift.com/example-code/language/what-is-key-value-observing

프로퍼티의 상태를 추적하여 두 객체 사이의 소통을 하는 방식. key path로 추적하여 nested objects도 추적이 가능. NSObject를 상속받는 객체에서만 사용 가능. 사용하지 않을 때에는 observer를 지워줘야 한다.

willSet, didSet과의 차이는 정의부에서의 관찰이 아닌 정의부 외부에서의 관찰 행위라는 것. 또한, Objective 런타임에 의존하기 때문에 swift에서 사용하기 위해서는 `@objc` 키워드를 사용해야 한다.

```swift
// @objcMembers를 사용하면 프로퍼티 앞에 @objc 생략 가능 (class에서만 사용)
@objcMembers class Person: NSObject {
    dynamic var name = "Taylor Swift"
}

let taylor = Person()

let observation = taylor.observe(\\.name) { person, change in
    print("I'm not called \\(person.name)")
}

taylor.name = "Justin Bieber"
```

위 코드를 실행하면 taylor의 name에 새로운 값을 할당하는 순간 observation에 저장한 closure가 실행된다.

### 언제 어떤 것을 💬

(개인 의견 반영)

KVO는 프로퍼티의 상태를 추적할 때 사용하지만, didSet이나 willSet으로 대체 가능하다. 프로토콜을 사용하는 delegate 패턴이 일반적으로 안전하기 때문에 권장되긴 하지만, 여러 객체에 한번에 이벤트를 전달해야할 경우에는 Notification을 사용하는게 효율적일 수도 있다.
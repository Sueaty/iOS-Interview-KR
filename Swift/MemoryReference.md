# Q. ARC가 뭔가요?

ARC는

- **컴파일 시 코드를 분석해서 자동으로 retain, release 코드를 생성해주는 것**
- **참조된 횟수를 추적해 더 이상 참조되지 않는 인스턴스를 메모리에서 해제해주는 것**

입니다.



### 메모리 영역

1. **코드 영역**

   작성한 소스코드가 기계어 형태로 저장

   컴파일 타임에 결정되고 , 중간에 코드가 변경되지 않도록 read-only 형태로 저장

2. **데이터 영역**

   전역변수, static 변수 저장

   프로그램 시작과 동시에 할당되고, 프로그램이 종료 되어야 메모리가 해제

   실행 도중 변수 값이 변경될 수 있으니 read-write로 저장

3. **힙 영역**

   프로그래머가 할당/해제 하는 영역 (동적 할당)

   사용하고 난 후에는 반드시 메모리 해제를 해줘야한다. 그렇지 않으면 memory leak이 발생

   → swift는 `ARC`가 자동으로 해제해줌

   런타임 시 결정되기 때문에 데이터 크기가 확실하지 않을 때 사용

   class, 클로저 같은 참조 타입 + String 할당

4. **스택 영역**

   함수 호출 시 함수의 지역변수, 매개변수, 리턴 값 저장 함수가 종료되면 메모리도 해제

   컴파일 타임에 결정되기 때문에 무한히 할당할 수 없다

struct 내에 둘 이상의 참조타입이 존재하면 그 레퍼런스 카운팅을 유지하는 비용이 class로 만들 때보다 증가할 수 있다.

### 앱 메모리

<img width="310" alt="스크린샷 2021-01-26 오후 10 12 25" src="https://user-images.githubusercontent.com/62557093/105849328-92ec1780-6023-11eb-9249-d3d261db59c5.png">

Clean 메모리는 이미지, 프레임워크 데이터 등을 포함하며, Dirty 메모리는 객체 등 앱에서 수정한 데이터들과, 프레임워크 dirty 메모리 등을 포함합니다.

Compressed memory는 문자 그대로 압축된 메모리를 의미하는데, 일정 기간 동안 특정 메모리 영역에 접근하지 않으면, 시스템이 해당 메모리 페이지들을 압축하고, 다시 접근할 때 압축 해제합니다. 이런 압축된 영역을 Compressed memory라고 합니다. iOS는 Memory compressor를 이용하여 이런 압축 또는 압축 해제 작업들을 수행합니다.

또한 앱이 할당 받을 수 있는 메모리 Footprint에는 제한 한도가 존재하며, 이 한도치를 넘어가면 `EXC_RESOURCE_EXCEPTION` 익셉션이 발생합니다.

footprint? 특정 하드웨어나 소프트웨어 단위가 차지하고 있는 공간의 크기



## 메모리 릭

앱의 메모리 릭을 캐치하는 좋은 접근 방법은, 앱의 특정 flow를 여러번 반복하면서 반복할 때마다 메모리 그래프 디버거를 이용해 메모리 스냅샷을 찍어 비교하는 것 입니다.

retain cycle? 메모리가 해제되지 않고 유지되어 누수가 발생하는 현상

> delegate는 weak으로 선언하여 메모리 누수를 예방하여야 한다

→ 이럴 경우 delegate와 관련된 프로토콜은 class 전용이어야 한다!!! 왜? unowned나 weak는 reference count 관리(즉 class)를 위한 개념이기 때문에!







# Weak And Unowned References

https://krakendev.io/blog/weak-and-unowned-references-in-swift

## Strong

일반적인 참조(포인터와 같은 것들)이지만 ARC에 의해 해제되지 않도록 reference count을 1 증가시킨다는 점에서 특별하다. 본질적으로, 어떠한 것이 객체를 strong 참조하는 동안 객체는 절대 해제되지 않을 것이다.

property 선언에서 default로 strong 참조를 가진다. 일반적으로 객체의 계층이 **선형적**으로 이루어져 있을  때 안전하게 사용할 수 있다. 부모 → 자식으로 강한 참조 관계일 때 항상 ok이다.

## Weak

weak 참조는 ARC로 부터 객체가 해제되는 것을 막지 않는다. 또한 객체가 할당 해제되면 포인터는 nil이 된다. 이렇게 하면 weak 참조는 항상 객체에 접근할 때 유효한 객체이거나 nil이거의 상태가 된다.

모든 weak 참조 변수는 let이 아닌 var여야 한다.

→ 참조되고 있지 않은 객체는 nil로 변경되기 때문 (mutable)

weak 참조는 retain cycle이 일어날 가능성이 있을 때 사용하는 것이 중요하다. 두 객체가 서로를 strong하게 참조하고 있으면, ARC는 두 객체에 적절한 release message를 전달할 수 없다.

예를 들어, 클로져 범위 밖에 변수가 선언되어 있고, 클로져 범위 내에서 그 변수를 참조하면 또 다른 strong 참조가 생성되게 된다. 단, 값의 의미(value semantics)를 가지는 것들은 예외.

```swift
class Kraken {
    var notificationObserver: ((Notification) -> Void)?
    init() {
        notificationObserver = NotificationCenter.default.addObserver(forName: "humanEnteredKrakensLair", object: nil, queue: .main) { notification in
            self.eatHuman()
        }
    }

    deinit {            
        NotificationCenter.default.removeObserver(notificationObserver)
    }
}
```

다음과 같은 경우에서 NotificationCenter는 eatHuman()을 호출할 때 self를 강하게 참조하는 클로져를 유지한다. deinit을 할 때 observer를 지우는 것이 적절하겠지만, 해당 observer는 kraken에 의해 strong 참조 관계를 가지기 때문에 ARC에서 deinit을 하지 않는다. `NSTimers` 와 `NSThread` 에서도 이러한 일이 일어난다.

해결방법은 클로저의 캡쳐 리스트에서 self를 weak 참조하는 것이다. 이는 강한 순환 참조를 끊는다. weak 참조일 때는 ARC에 의해 reference count가 증가되지 않는다.

```swift
//Look at that sweet, sweet Array of capture values.
let closure = { [weak self, unowned krakenInstance] in
    self?.doSomething() //weak variables are Optionals!
    krakenInstance.eatMoreHumans() //unowned variables are not.
}
```

캡쳐 리스트에서 여러 캡쳐 값을 지정할 수 있다.

또 한가지 경우는 클래스에서 delegation을 지정하기 위해 프로토콜을 사용할 경우이다. struct나 enum도 프로토콜을 채택할 수 있지만 클래스와 달리 value semantics이다.

delegate property를 weak로 선언하여 retain cycle을 방지할 수 있지만, 이 경우의 protocol은 class를 상속받아 reference semantics임을 밝혀야 한다.

### : class

해당 프로토콜의 요구조건에 의해 정의된 동작이 값 의미론보다는 참조 의미론을 가지고 있다고 가정하거나 요구될 때 클래스 전용 프로토콜을 사용한다.

### value semantics?

The only exceptions to this are variables that use value semantics such as Ints, Strings, Arrays, and Dictionaries in Swift.

https://oaksong.github.io/2017/12/29/value-semantics-vs-reference-semantics/

- 추후 참고할 자료

  [Value SEMANTICS (not value types!)](https://academy.realm.io/posts/swift-gallagher-value-semantics/)

## Unowned

unowned는 weak와 유사하게 reference count를 증가시키지는 않지만, optional 값이 아니라는 이점을 가지고 있다. 이는 optional binding의 수고를 덜어준다.

또한 객체가 해제되고 나면, dagling pointer가 된다.

weak  reference는 객체의 lifetime에 nil이 될 수 있는 경우, 반대로 참조가 nil이 되지 않을 때 즉, 참조하고 있는 객체와 같거나 더 긴 lifetime을 가지고 있을 때는 unowned reference를 쓰라고 apple 문서에 적혀있다.

- unowned를 사용해도 되는 경우의 예시

```swift
class RetainCycle {
    var closure: (() -> Void)!
    var string = "Hello"

    init() {
        closure = { [unowned self]
            self.string = "Hello, World!"
        }
    }
}

//Initialize the class and activate the retain cycle.
let retainCycleInstance = RetainCycle()
retainCycleInstance.closure() 
//At this point we can guarantee the captured self inside the closure will not be nil. Any further code after this (especially code that alters self's reference) needs to be judged on whether or not unowned still works here.
```

클로저가 서로 상호의존적일 때 ( 하나가 다른 하나 없이는 live할 수 없을 때), weak 보다는 unowned를 쓰는 것이 프로그램이 불필요하게 nil 참조를 유지하는 overhead를 막아준다.

```swift
class Kraken {
    let petName = "Krakey-poo"
    lazy var businessCardName: (Void) -> String = { [unowned self] in
        return "Mr. Kraken AKA " + self.petName
    }
}
```

위의 경우에서 `businessCardName` 클로저와 클로저 내부의 self는 상호 의존적이기 때문에 unowned reference를 사용한다.

```swift
class Kraken {
    let petName = "Krakey-poo"
    lazy var businessCardName: String = {
        return "Mr. Kraken AKA " + self.petName
    }()
}
```

그러나 다음과 같이 클로저가 아닌 lazy 변수를 사용할 때와 혼동을 피해야한다.

lazy 변수를 호출하는 클로저를 실제로 유지하는 것은 없기 때문에 unowned self는 필요하지 않다. 변수는 오직 클로저의 결과 값 string만 자신에게 할당하고, 클로저는 사용 후 바로 해제되기 때문.

## Unowned Optional References

https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID57

unowned 객체를 optional로 지정할 수 있다. weak reference와 동일한 문맥에서 사용가능하다. 차이는 개발자가 유효한 객체를 참조하는지 보장하거나 nil 값으로 설정해 주어야한다는 점이다.

```swift
class Department {
    var name: String
    var courses: [Course]
    init(name: String) {
        self.name = name
        self.courses = []
    }
}

class Course {
    var name: String
    unowned var department: Department
    unowned var nextCourse: Course?
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextCourse = nil
    }
}
```

다음 상황에서 course는 두 개의 unowned 변수를 가진다. 각 course는 department에 속하므로 department는 optional 값이 아니지만 nextCourse는 있을 수도 없을 수도 있기 때문에 optional 값이다.

```swift
let department = Department(name: "Horticulture")

let intro = Course(name: "Survey of Plants", in: department)
let intermediate = Course(name: "Growing Common Herbs", in: department)
let advanced = Course(name: "Caring for Tropical Plants", in: department)

intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced]
```

<img width="517" alt="스크린샷 2021-01-26 오후 10 14 44" src="https://user-images.githubusercontent.com/62557093/105849566-e5c5cf00-6023-11eb-9e56-405cde2b6d9a.png">

intro, intermediate 코스는 다음 코스를 가지고 있다. unowned optional reference는 ARC에서 reference count를 증가시키지 않기 때문에 nil이 될 수 있다는 점을 제외하고는 unowend reference와 같은 방식으로 동작한다.

unowend reference와 같이 nextCourse는 항상 해제되지 않은 객체를 참조하고 있어야 하지 때문에 course를 삭제할 경우에는 그 course를 nextCourse로 참조하는 관계 역시 삭제해주어야 한다.

optional은 enum이며, 값 타입은 unowned로 취급될 수 없다의 예외적 상황이다.

optional로 포장된 class는 reference counting을 사용하지 않기 때문에 strong reference로 유지할 필요가 없다.

## Unowned References and Implicitly Unwrapped Optional Properties

어느 객체의 변수도 nil이 될 수 없고, 값을 가져야 할 때가 있다.

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

다음 상황에서 contry는 항상 capitalCity를 가지고 City도 Country 변수를 참조한다. City는 Country가 initialize될 때 생성되는데, Country가 완전히 초기화 되기 전까지 City의 생성자에 self를 넘길 수 없다.

이에 대처하기 위해 Country의 capitalCity의 변수를 암시적 추출 옵셔널로 선언한다. 이것은 capitalCity의 default는 nil이지만, 암묵적으로 이 값을 unwrap할 필요없이 접근할 수 있다는 것을 의미한다. 따라서 Country 객체는 name 변수가 설정되자마자 완전히 초기화 된 것으로 간주된다. 따라서 Country는 capitalCity의 init에 자기 자신 self를 넘겨줄 수 있다.

```swift
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\\(country.name)'s capital city is called \\(country.capitalCity.name)")
// Prints "Canada's capital city is called Ottawa"
```

이것은 strong reference 없이도 단일 구문으로 Country, City 인스턴스를 생성할 수 있으며 optional 값을 느낌표로 unwrap 할 필요없이 capitalCity에 바로 접근 가능하게 한다.

따라서 capitalCity는 초기화 된 후 non-optional 타입처럼 사용가능하고, 강한 순환 참조 역시 피할 수 있다!

## Discover Side Table

https://maximeremenko.com/swift-arc-weak-references

Side Table은 추가적인 객체의 정보를 저장하는 별도의 메모리 공간이다. 객체는 초기에 side Table을 가지고 있지 않는다. 객체가 weak 참조 될 때, strong 이나 unowned 카운터가 overflow 될 때 (32-bit 시스템에서 inline count는 작다) side Table이 생성된다.

<img width="512" alt="스크린샷 2021-01-26 오후 10 16 09" src="https://user-images.githubusercontent.com/62557093/105849709-186fc780-6024-11eb-8141-1dbe29a70f31.png">

side Table과 객체는 서로 가리키는 포인터가 있다. 주목할 점은 weak 참조는 side Table을 가리키고 strong과 unowned는 객체를 직접 가리킨다는 것이다. 이것은 객체를 완전히 해제되게 만든다. 따라서 언제 쓰일지 모르는 weak 참조를 위해 메모리에서 좀비 상태로 존재하는 공간을 비워낼 수 있다.

<img width="511" alt="스크린샷 2021-01-26 오후 10 15 05" src="https://user-images.githubusercontent.com/62557093/105849611-f37b5480-6023-11eb-86c2-cbae633119cc.png">

### Swift Object Life Cycle

https://devsday.ru/blog/details/1781

<img width="357" alt="스크린샷 2021-01-26 오후 10 15 55" src="https://user-images.githubusercontent.com/62557093/105849688-10178c80-6024-11eb-97e4-368755f19812.png">
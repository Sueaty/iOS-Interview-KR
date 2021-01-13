## Optional과 관련된 면접 질문

* 접근 제어자의 종류엔 어떤게 있는지 설명하시오 [출처](https://github.com/JeaSungLEE/iOSInterviewquestions#swift)

# Access Control

Access control은 **파일 또는 모듈 내의 코드에 접근을 제한할 수 있는 방법**이다. <br>
구현한 코드에 접근 제한을 두면 개발자로 하여금 내가 의도한대로 코드를 작성하도록 유도할 수 있다. 

OOP 패러다임에서 **Encapsulation**(캡슐화), **Hiding**(은닉화)이 중요한 이유는 외부로부터 데이터/코드가 보호되어야 하기 때문이다.<br>
외부 접근으로 인해 의도치 않은 결과가 초래될 수 있으므로 적절히 접근제어를 통해 제한할 필요가 있다.

Access control은 다음에 지정할 수 있다.
* Type: class, struct, enumeration
* (Type 내의) properties, methods, initializers, subscripts (단, enumeration의 case마다 접근 지정자는 붙이지 못함)

## Access Level

Access level keyword를 통해 access control 을 조절할 수 있는데 Swift에는 총 5가지의 access level이 존재한다. <br>
`open`, `public`, `internal`, `fileprivate`, `private` <br>
위 순서는 접근도를 높은 순서대로 나열한 것이며, 모든 타입에 적용되는 access level은 **상위 요소보다 하위 요소가 더 높은 접근 수준을 가질 수 없다**.

### open

* 외부 모듈에서 접근 가능

|          | open       | 그 외의 access level      |
| ----     | --------   | --------                |
| 클래스 상속 | 제한 없음    | 해당 클래스가 정의된 모듈 내   |
| 클래스 멤버 재정의 | 제한 없음    | 해당 클래스 멤버가 정의된 모듈 내   |

### public

* 외부 모듈에서 접근 가능

### internal

* default로 지정되는 access level
* 소스 파일이 속해있는 모듈 내에서 사용 가능(= 해당 모듈을 외부에서 import 했다면 접근 불가)

### fileprivate

* 요소가 구현된 소스파일 내부에서만 사용 가능
* 해당 소스파일 외부에서 값이 변경되거나 함수를 호출하면 부작용이 생길 수 있을 때 사용하면 좋음

### private

* 그 기능을 정의하고 구현한 범위 내에서만 사용 가능
* 같은 소스파일 안에 구현되어 있더라도 구현한 타입이 다르면 사용 불가

```swift
// 파일명 : example.swift
class A {
   fileprivate var fpProperty: Int = 3
   private var pProperty: Int = 5
}

class B {
   let a = A()
   
   func printAProperties() {
      print(a.fpProperty) // 가능
      print(a.pProperty) // 불가능
   }
}
```

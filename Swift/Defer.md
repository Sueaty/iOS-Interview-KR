## defer 관련 면접 질문 

* defer란 무엇인지 설명하시오. [출처](https://github.com/JeaSungLEE/iOSInterviewquestions)
* defer가 호출되는 순서는 어떻게 되고, defer가 호출되지 않는 경우를 설명하시오. [출처](https://github.com/JeaSungLEE/iOSInterviewquestions)

# defer

## defer 란

defer 구문이 하는 역할에 대한 궁금증은 단어 자체의 뜻을 알면 많은 부분 해소된다.

![](https://i.imgur.com/BbhWe9C.png)

> A defer statement is used for executing code **just before transferring program control**
> outside of the scope that the defer statement appears in.

defer 구문은 자신이 속해있는 **코드 블럭의 scope에서 빠져나가기 '직전'** 에 실행되는 구문이다. <br>
이 때 scope는 함수, 조건문, 반복문, do-try-catch문 등의 scope을 모두 뜻하고 빠져나간다는 것은 정상 종료를 뜻할수도 있고 에러를 만나 비정상 종료가 될 수도 있다는 뜻이다.
하지만 **어떤 종료를 하던 무관하게 빠져나가기 직전에 defer문 내부의 코드를 실행하고 종료** 한다.

어떤 형태로 빠져나가도 반드시 실행이 보장되어 있는 구문이라서 오류처리를 할 때 유용하게 쓰인다.

## defer 사용 예시

### 파일 handling

코드 설명 : Network의 상태 메세지를 가져와서 정상("Success")이 아닐 경우 상태 메세지를 파일에 write

```swift
func logErrorMessage() {
   let file = openFile(...)
   defer { file.close() }
   
   guard let statusMessage = fetchNetworkStatus() where statusMessage != "Success" { 
      return 
   }
   
   file.write(statusMessage)
   
}
```

파일에 읽고 쓰는 작업을 하려면 파일을 열어야하고, 종료 시 안전하게 닫아야 한다.<br>
파일을 닫는 코드가 defer block 안에 있기 때문에 해당 함수가 종료되기 직전에 실행이 되는데, 불리는 시점은 else 문이 종료될 때 일수도 있고, 정상 실행 후 마지막에 종료 직전에 불릴 수 있다. 하지만 어떤 종료를 하건 무조건 파일은 닫아야 하니 defer block안에 넣은 것이다. <br>

## defer의 특징

코드 설명 : do-try-catch 문에서 무조건 catch 문으로 넘어가게 함

```swift
do {
    defer { print("1") }
    defer { print("2") }
    try fail()
    print("do-statement finished")
} catch  {
    print("catch statement")
} 

// 무조건 Error가 발생
func fail() throws -> () {
    throw NSError(domain: "U", code: 1, userInfo: nil)
}
```

fail 함수를 실행 위쪽을 보면 2개의 defer block이 있다.<br>
즉, try fail()을 통해 error가 발생한다면 catch문으로 넘어갈텐데, 그 전에 defer block을 실행하고 만약 try가 무탈하게 지나간다면 "do-statement finished"가 불리고 defer가 불릴 것이다. 그래서 정리를 해보면 아래와 같다.

1. try 실패
```
2
1
catch statement
```

2. try 성공 (물론 위 코드에서는 이렇게 나올 수 없음)
```
do-statement finished
2
1
```

우리는 결과를 보고 defer의 특징을 알 수 있다.

1. (당연) scope를 빠져나가기 직전에 실행 됨
2. 정의 된 역순으로 실행 됨

### Reference

* [Swift Programming Language Guid](https://docs.swift.org/swift-book/ReferenceManual/Statements.html)
* [How Defer Operator in Swift Actually Works](https://medium.com/@sergeysmagleev/how-defer-operator-in-swift-actually-works-30dbacb3477b)
* [defer문 이해하기](https://soooprmx.com/archives/6118)

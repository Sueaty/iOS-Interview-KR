## GCD란 ?

ios에서 쉽고 편한 멀티 스레딩 처리를 위해 제공되는 api

개발자가 작업 큐에 작업을 넣어주기만 하면 gcd가 알아서 스레드 관리를 해주는 느낌?

스레드를 생성하고 할당하는 관리를 시스템에게 맡겨준다.

## Dispatch Queues

[iOS ) Concurrency Programming Guide - Dispatch Queues](https://zeddios.tistory.com/513)

[iOS ) GCD - Dispatch Queue사용법 (2) / DispatchWorkItem, DispatchGroup](https://zeddios.tistory.com/520)

사용하기 쉽고, 해당 쓰레드 코드 보다 해당 task를 실행 할 때 훨씬 효율적이다(?)

task를 비동기적으로 동시에 수행할 수 있는 손쉬운 방법이다. task는 앱이 수행해야하는 작업. dispatch queues는 제출한 (넘겨 받은?) 모든 task를 관리하는 객체와 유사한 구조이다. 모든 dispatch queues는 FIFO 구조로 항상 순서대로 시작한다.



### Serial(private dispatch queue)

큐에 추가된 순서대로 **한 번에 하나의 task**를 실행. 현재 실행 중인 task는 dispatch queues에서 관리하는 고유한 쓰레드**(task마다 다를 수 있다)**에서 실행된다. 

> Serial queues를 4개 작성하면 각 큐는 한 번에 하나의 task만 실행하지만, 최대 4개의 task가 각 큐에서 동시에 실행될 수 있다!



### Concurrent(global dispatch queue)

**동시에 하나 이상의 task를 실행**하지만 task는 큐에 추가된 순서대로 시작. 현재 실행 중인 task는 dispatch queues에서 관리하는 고유 쓰레드에서 실행된다. 특정 시점에서 실행되는 정확한 task 수는 가변적. iOS5 이상에서는 큐 타입으로 DISPATCH_QUEUE_CONCURRENT를 지정하여 사용자가 동시에 dispatch queue를 생성할 수 있다. 앱에 사용할 사전에 정의된 global concurrent queues가 4개 존재함.



### Main Dispatch Queue

앱의 **main 쓰레드**에서 task를 실행하는, 전역적으로 사용 가능한 **serial queue**.

앱의 실행 루프와 함께 작동하여 큐에 있는 **task 실행이 실행 루프에 연결된 다른 이벤트 소스의 실행과 얽힌다**. 종종 앱의 주요 동기화 지점으로 사용된다.



### Dispatch Queue의 장점

쓰레드에 비한 장점. work-queue(작업 큐) 프로그래밍의 단순성.

쓰레드 생성 및 관리에 대한 작업을 dispatch queue에서 알아서 해준다. 시스템은 사용가능한 자원 및 현재 시스템 조건에 따라 동적으로 쓰레드 수를 조절한다. 일반적으로 쓰레드를 직접 작성한 것 보다 빨리 task를 수행한다.

쓰레드 코드 작성하는 것 보다 쉽다. 핵심은 독립적이고, 비동기적으로 실행되는 task를 설계하는 것. 쓰레드에서 공유 리소스에 접근 할 때에는 동시에 수정을 하지 못하도록 lock하는 작업을 해주어야 한다.

**dispatch queues는 한 번에 하나의 task만 자원을 수정하도록 할 수 있다.** lock을 사용할 때 선점식의 경우 항상 고가의 커널 트랩을 필요로 하지만, dispatch queues는 주로 앱의 프로세스 공간에서 작동하고, 필요한 경우에만 커널을 호출한다.

쓰레드 모델은 커널과 사용자 공간 메모리를 모두 차지하는 두 개의 쓰레드를 생성해야한다. dispatch queues는 쓰레드에 대해 동일한 메모리 페널티를 지불하지 않으며, 사용하는 쓰레드는 사용 중이기 때문에 차단되지 않는다.

**[특징]**

- 다른 dispatch queues와 동시에 task를 실행한다. task의 직렬화는 single dispatch queue일 때만 나타난다.
- 시스템은 한 번에 실행되는 총 task 수를 결정한다. 100개의 다른 큐에서 100개의 task를 가진 앱은 100개 이상의 유효코어가 없는 한 모든 task를 동시에 실행할 수 없다.
- 새 task를 선택할 때 큐 priority level이 고려된다.
- 큐의 task는 큐에 추가될 때, 실행할 준비가 되어 있어야 한다.
- private dispatch queues는 reference-counted 객체이다. **retain count를 증가 시키기 때문에, 모든 dispatch sources가 취소 되었는지 확인하고, retain-release를 고려해야한다**.



### QoS(Quality of Service)

priority가 높은 작업은 우선순위가 낮은 작업보다 더 빨리 수행되고 리소스가 많으므로 일반적으로 priority가 낮은 작업보다 더 많은 에너지가 필요하다. 적절한 QoS클래스를 지정하면, 앱이 에너지 효율적일 뿐만 아니라 반응적임을 보장할 수 있다.

- **User-interactive**: ~~main thread에서 작업~~(아님). 사용자 인터페이스 새로고침 또는 애니메이션 수행처럼 사용자와 상호작용하는 작업. 작업이 신속하게 수행되지 않으면, UI가 중단된 상태로 표시될 수 있다. 반응성과 성능에 중점을 둔다.
- **User-initiated**: 사용자가 시작한 작업. 저장된 문서를 열거나 사용자 인터페이스에서 무언가를 클릭할 때 작업을 수행하는 것과 같은 즉각적인 결과가 필요하다. 사용자 상호작용을 계속하려면 작업이 필요하다. 반응성과 성능에 중점. 거의 순식간이며, 몇 초 혹은 그 이하
- **Utility**: 작업을 완료하는데 약간의 시간이 걸릴 수 있으며, 데이터 다운로드와 같은 즉각적인 결과가 필요하지 않음. 일반적으로 사용자가 볼 수 있는 progress bar가 있다. 반응성, 성능, 에너지 효율성 간의 균형을 유지하는데 중점. 몇 초에서 몇 분정도
- **Background**: 백그라운드에서 작동. indexing, 동기화, 백업같이 사용자가 볼 수 없는 작업. 에너지 효율에 중점. 분 혹은 시간과 같은 상당한 시간이 걸림

>  ‼️ 사용자 작업이 발생하지 않는 시간의 90% 이상을 **Utility** QoS level에서 실행하는 것이 좋다

default는 User-initiated와 Utility 사이



### 🚨QoS Promotion Rule

추후 꼭 공부하기



---



### Dispatch WorkItem

수행할 수 있는 task를 캡슐화 한 것.

```swift
let wellyWorkItem = DispatchWorkItem {
    for i in 1...5 {
        print("workItem \\(i)")
    }
}

DispatchQueue.global().async(execute: wellyWorkItem)
```

다음과 같이 작업 단위를 workItem으로 나타내고 수행할 수 있다.

**workItem에도 QoS를 지정할 수 있다.** 따라서 같은 queue 내에서의 작업의 우선순위를 지정해 줄 수 있다. **그치만 보장은 안됨!!** 그냥 대략 그렇다는 것 ,,

### Dispatch Groups

완료(completion)를 위해 블록 객체 집합을 모니터링 하는 방법. 동기적/비동기적으로 블록을 모니터링 할 수 있다. 다른 task의 완료 여부에 따라 코드에 유용한 동기화 매커니즘을 제공한다.

```swift
let myGroup = DispatchGroup()

myQueue.async(group: myGroup) {
    for i in 100...105 {
        print("🏓 \\(i)")
    }
}

myQueue.async(group: myGroup) {
    for i in 1...5 {
        print("🚩 \\(i)")
    }
}

myGroup.notify(queue: myQueue) {
    print("End!")
}
```

다음과 같이 해당 그룹의 해당 큐의 모든 작업이 완료되었을 때 End를 출력해준다.

```swift
myGroup.notify(queue: myQueue) {
    print("End!")
    myQueue.async(group: myGroup) {
        for i in 200...205 {
            print("hi \\(i)")
        }
    }
    
    myQueue.async(group: myGroup) {
        for i in 300...305 {
            print("bye \\(i)")
        }
    }
    
    myGroup.notify(queue: myQueue) {
        print("Real End!!!!!")
    }
    
}
```

<img width="338" alt="스크린샷 2021-01-29 오후 5 05 42" src="https://user-images.githubusercontent.com/62557093/106248365-391e6400-6254-11eb-9815-dc527bb462fb.png">

notify 안에 notify를 호출할 수도 있음

비동기 작업에서의 순서를 보장하고 싶을 때 유용하게 사용가능



## 그 외의 기능들

### Dispatch Semaphores(?)

세마포어를 사용할 수 없어서 호출 쓰레드를 차단해야하는 경우에만 세마포어가 커널로 호출된다. 세마포어를 사용할 수 있으면, 커널 호출이 수행되지 않는다



### Dispatch Sources

특정 타입 시스템 이벤트에 대한 응답으로 notifications을 생성한다. dispatch sources를 사용하여 프로세스 notifications, signal, descriptor events 등을 모니터링 할 수 있다. 이벤트가 발생하면 dispatch sources는 처리를 위해 지정된 dispatch queue에 비동기적으로 task 코드를 제출한다.





---



### +) 왜 main.sync를 하면 안될까?

[iOS ) 왜 main.sync를 하면 안될까](https://zeddios.tistory.com/519)

UI 업데이트는 반드시 main에서 해야한다

→ UIApplication 인스턴스가 main thread에 붙고, UIApplication은 앱의 run loop를 포함해서 main event loop를 설정한다. main event loop는 모든 UI event를 수신.

UI event는 **UIApplication → UIWindow → UIViewController → UIView → subViews..** 와 같이 responder chain을 따라 UIResponder로 전달된다.

따라서 모든 이벤트는 main thread의 일부가 된다.

만약 background thread에서 view를 지우고 그 thread의 run loop가  끝나지 않았을 때 사용자가 그 view를 탭하게 되면 touch event와 충돌하게 될 것이다.

sync는 큐에 있는 작업이 끝날 때까지 block하고 wait 상태가 된다.

만약 main queue에서 sync를 호출하여 큐를 block하게 되면 deadlock 현상이 발생한다.

**main thread는 thread-safe하지 않다!**
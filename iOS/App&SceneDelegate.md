
## SceneDelegate과 관련된 면접 질문

* SceneDelegate에 대해 설명하시오. [출처](https://github.com/JeaSungLEE/iOSInterviewquestions#ios)

# SceneDelegate

## iOS 12 이하 버전의 AppDelegate

Application은 **1개의 process** 와 **1개의 user interface 인스턴스** 를 가졌다. <br>
AppDelegate은 2가지 역할을 했는데, application에게

1. process level의 이벤트 발생을 알려주었고
2. UI의 상태변화를 알려주었다.

<img width = 800, src = "https://i.imgur.com/sUu5zrc.png">

process level의 이벤트의 예시로는 application이 launch 되고 있는지 또는 terminate 되기 직전 등이 있고, <br>
UI의 상태변화는 willEnterForeground, willResignActive와 같은 메소드들을 통해 알릴 수 있었다.

## iOS 13 이후 버전의 AppDelegate

그러나 iOS 13으로 넘어오면서 **1 process & multiple user interface(= scene sessions)** 를 지원하게 되었다. <br>
실제로 plist의 Application Scene Manifest를 보면 기존에는 없던 Enable Multiple Windows가 포함되어 있다.<br>
기존에는 AppDelegate에서 UIWindow 객체에 대한 configuration도 진행했었는데 이제는 하나의 window 객체만 관리하지 못한다.<br>
AppDelegate의 일부 역할을 SceneDelegate에게 넘겨주었고, AppDelegate은 새로운 역할을 하나 더 맡게 되었다.

1. process level의 이벤트 발생을 알려주고 (그대로)
2. session life-cycle을 application에게 알려주게 되었다. (신규)

<img width = 800, src = "https://i.imgur.com/rytWU1q.png">

기존의 UI 상태 변화를 알려주는 것은 SceneDelegate이 책임지게 되었고 AppDelegate은 새로운 Scene Session이 생성되거나 버려질 때 알리는 역할을 새로 맡게 되었다.

## SceneDelegate

**UI의 상태변화를 메소드들을 통해 application에게 알리는 역할**을 한다. 기존 AppDelegate들이 갖고 있던 메소드들과 거의 1:1 mapping이 가능하다.<br>
SceneDelegate은 아래의 메소드를 갖고 있다.
* ```scene(_: willConnectTo: options:)```
* ```sceneWillEnterForeground(_ :)```
* ```sceneDidBecomeActive(_ :)```
* ```sceneWillResignActive(_ :)```
* ```sceneDidEnterBackground(_ :)```
* ```sceneDidDisconnect(_ :)``` 

### 실제 call stack 따라가보기

1. AppDelegate: didFinishLaunching → UI와 무관한 일회성 setup을 이곳에서 해도 무방하다
2. system이 scene session을 생성했다. (UI Scene 아님)
3. AppDelegate: UISceneConfiguration
4. SceneDelegate: scene willConnectTo

**User가 Home 화면으로 돌아갔다** <br>

1. SceneDelegate: willResignActive
2. SceneDelegate: didEnterBackground
3. SceneDelegate: didDisconnect

> **didDisconnect** 살펴보기 <br>
> system은 한정 된 자원을 가지고 있기 때문에 자원을 언젠가는 회수를 해야 한다. 따라서 scene이 background에 들어간 후 어느 시점에 scene을 memory로부터 해제시킨다. 즉, <br>
> 1. scene delegate이 메모리에서 해제됨<br>
> 2. scene delegate이 관리하던 window 및 view 계층 메모리에서 해제됨<br>
> 따라서 didDisconnect코드 내부에서는 scene과 관련된 불필요한 자원을 돌려주는 작업을 해주면 된다. 가령 디스크나 네트워크를 통해 쉽게 데이터를 다시 불러올 수 있다거나
재생성이 쉬운 데이터는 돌려주는게 좋지만 **scene이 다시 reconnect** 될 수 있으므로 사용자의 input과 같이 재생성이 어려운 데이터는 가지고 있어야 하니 데이터를 무조건적으로 영구삭제하면 안된다.

**User가 app switcher틑 통해 app 종료**

1. AppDelegate: didDiscardSceneSessions → 이 곳에서 user data, state 등 scene과 관련된 모든 데이터를 지우는 작업을 해주면 된다.

## Reference

* [WWDC - Architecting Your App for Multiple Windows](https://developer.apple.com/videos/play/wwdc2019/258/)
* [공식문서](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)
* [DW 블로그](https://www.donnywals.com/understanding-the-ios-13-scene-delegate/)

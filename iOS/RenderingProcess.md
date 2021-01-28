## iOS Rendering Process

https://medium.com/@duwei199714/ios-why-the-ui-need-to-be-updated-on-main-thread-fd0fef070e7f

위 글을 번역 및 해석

### Rendering Framework

<img width="331" alt="스크린샷 2021-01-28 오후 9 10 01" src="https://user-images.githubusercontent.com/62557093/106136840-30268780-61ad-11eb-889c-a9bd2ed46c34.png">

- UIKit: 모든 종류의 컴포넌트를 포함하며, 사용자 이벤트를 관리한다. 렌더링 코드를 포함하지 않음
- CoreAnimation: 모든 뷰의 그리기, 보여주기, 애니메이션을 담당
- OpenGL ES: 2D와 3D 렌더링 서버를 제공
- Core Graphics: 2D 렌더링 서버 제공
- Graphics Hardware: GPU

따라서 모든 뷰는 UIKit이 아니라 **Core Animation Framework에서 보여지고 애니메이트 된다.**



### Core Animation Pipeline

<img width="511" alt="스크린샷 2021-01-28 오후 9 10 14" src="https://user-images.githubusercontent.com/62557093/106136855-37e62c00-61ad-11eb-8c54-5f4461ee0b13.png">

Core Animation은 렌더링 시 4단계로 나뉘어진 파이프라인을 사용한다.

- **Commit Transaction**: 뷰 레이아웃, 이미지 디코딩, 포맷 변환, 뷰 레이어 패킹 및 렌더링 서버로 전송
- **Render Server**: 렌더링, commit transaction에서 전송된 패키지를 분석하고 렌더링 트리로 deserialization. 그 후 뷰 레이어 프로퍼티에 따라 drawing 지시사항을 생성하여 다음 VSync Signal 때 OpenGL이 스크린을 렌더링 하도록 호출한다.
- **GPU**: 스크린의 VSync Signal을 기다렸다가 렌더링을 위해 OpenGL을 사용한다. 렌더링 후 결과를 버퍼에 전송한다
- **Display**: 버퍼로부터 데이터를 가져와 화면에 보여주기 위해 전송한다.

우리는 이 작업의 준비가 1/60초 안에 끝나 렌더 서버로 전송하고 렌더링 하는데에 1/60초 안에 완료되어 앱이 고착되지 않는 것을 기대한다. (not stuck)

> 하지만 만약 background thread가 UI를 업데이트하게 되면 runloop가 종료되고 렌더링을 하려할 때 문제가 발생한다. 각 쓰레드가 다른 렌더링 정보를 commit하고 Core Animation Pipeline에서는 CPU에 계속해서 렌더링 정보를 commit하게 된다. 그러나 **렌더링 작업은 시스템 리소스에 비용이 많이 들고, thread 간의 컨텍스트 전환과 다량의 트랜잭션으로 인해 GPU에서 처리가 불가능하게 된다**. 결국 1/60초 내에 레이어 트리를 전송할 수 없어 성능에 영향을 미친다.



#### 그래도 background threads에서 UI를 업데이트 하고 싶다면,,

thread-safe한 객체를 사용하는 `Texture`와 `ComponentKit` 과 같은 프레임워크들이 있다.



#### 더 자세한 내용은

https://developer.apple.com/videos/play/wwdc2011/121/

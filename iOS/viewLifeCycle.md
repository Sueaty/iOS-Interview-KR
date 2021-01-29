

## 뷰의 상태 변화 감지 메서드

<img src="/Users/alimelon/Library/Application Support/typora-user-images/스크린샷 2021-01-13 오후 4.36.04.png" width=400>



**뷰의 상태 변화 메서드**

- ```func viewDidLoad()```

- - **뷰 계층이 메모리에 로드된 직후 호출되는 메서드**
  - 뷰의 추가적인 초기화 작업을 하기 좋은 시점
  - 메모리에 처음 로딩 될때 1회 호출되는 메서드로, 메모리 경고로 뷰가 사라지지 않는 이상 다시 호출되지 않음

- ```func viewWillAppear(_ animated: Bool) ```

- - **뷰가 뷰 계층에 추가되고 화면이 표시되기 직전에 호출되는 메서드**
  - 뷰 표시와 관련하여 추가 작업을 할 수 있다
  - 다른 뷰로 이동했다가 되돌아오면 재호출되는 메서드로, 화면이 나타날때마다 수행해야하는 작업을 하기 좋은 시점

- ```func viewDidAppear(_ animated: Bool) ```

- - **뷰가 뷰 계층에 추가되어 화면이 표시되면 호출되는 메서드**
  - 뷰를 나타내는 것과 관련된 추가적인 작업을 하기 좋은 시점

- ```func viewWillDisappear(_ animated: Bool) ```

- - **뷰가 뷰 계층에서 사라지기 직전에 호출되는 메서드**
  - 뷰가 생성된 뒤 발생한 변화를 이전상태로 되돌리기 좋은 시점

- ```func viewDidDisappear(_ animated: Bool)```

  - **뷰가 뷰 계층에서 사라진 후 호출되는 메서드**
  - 뷰를 숨기는 것과 관련된 추가적인 작업을 하기 좋은 시점
  - 시간이 오래 걸리는 작업은 하지 않는 것이 좋음



**뷰의 레이아웃 변화 메서드**

뷰가 생성된 후 바운드 및 위치 등의 레이아웃에 변화가 발생했을 때 호출되는 메서드입니다. 

- ```func viewWillLayoutSubviews()```

- - **뷰가 서브뷰의 레이아웃을 변경하기 직전에 호출되는 메서드**
  - 서브뷰의 레이아웃을 변경하기 전에 수행할 작업을 하기 좋은 시점

- ```func viewDidLayoutSubviews()```

- - **서브뷰의 레이아웃이 변경된 후 호출되는 메서드**
  - 서브뷰의 레이아웃을 변경 한 후 추가적인 작업을 수행하기 좋은 시점





### Q. 오토레이아웃에서 특정한 상황에서 제약사항을 바꾸고 싶을 때 할 수 있는 방법들을 설명해주세요



* **레이아웃 변경을 위한 함수들** 

  \- **setNeedsLayout()**
  \- **layoutIfNeeded()**

  setNeedsLayout은 다음 업데이트 사이클을 기다린 뒤 업데이트 시켜준다. 레이아웃 업데이트를 한 번에 통합해서 업데이트 해줘서 일반적으로 성능이 좋다. 바뀌는 UI가 많던가 바로 볼 필요가 없는 급한 UI가 아니면 쓰면 좋다

  이 함수를 실행하면 viewwillLayoutSubviews와 viewDidLayoutSubViews도 호출
  viewDidLayoutSubviews에서는 스토리보드상의 레이아웃이 아니라 폰에 적용된 크기로 컴포넌트의 크기를 받아올 수 있어서 레이아웃 크기에 관련한 작업을 할 때 편함

  layoutIfNeeded는 즉시 업데이트 시키고 레이아웃 관련 콜백을 호출하지 않고 종료. 바로바로 변화를 보고 싶은 경우 쓰면 좋다 애니메이션 같은거에 많이 씀



## Layout Cycle

https://zeddios.tistory.com/1202?category=682195

Auto Layout은 view의 프레임을 즉시 업데이트 하는 대신에 가가운 미래에 layout 변경을 예약한다.

layout 변경은 **layout constraints 업데이트 후, view 계층의 모든 view에 대한 프레임을 계산** 한다.

`setNeedsLayout()` 혹은 `setNeedsUpdateConstraints()`를 호출하여 예약가능

<img width="511" alt="스크린샷 2021-01-29 오후 5 11 16" src="https://user-images.githubusercontent.com/62557093/106248905-0032bf00-6255-11eb-94dd-680fce134810.png">

위의 그림에서 layout의 constraints가 변경 될 때까지 Application Run Loop가 반복되고, 변경이 생겼을 때 Deferred Layout Pass가 예약된다.



- **Constraints에 영향을 주는 요인**
  - constraint 활성화 혹은 비활성화
  - constraint 상수 값 변경
  - constraint의 priority 변경
  - view 계층에서 view 제거

constarint 변경이 일어남 → 레이아웃 엔진이 레이아웃을 다시 계산 → superview에 다시 레이아웃이 필요하다고 표시 ( == `superview.setNeedsLayout()`)



> Deferred Layout Pass = Update pass + Layout Pass

### Update Constraints (= Update Pass)

<img width="510" alt="스크린샷 2021-01-29 오후 5 11 24" src="https://user-images.githubusercontent.com/62557093/106248917-04f77300-6255-11eb-9212-f1c0f7eb2397.png">

**아래에서 위로**

setNeedsUpdateConstraints() 호출 → updateViewConstraints() 호출

### Reassign View Frames (= Layout Pass)

<img width="509" alt="스크린샷 2021-01-29 오후 5 13 36" src="https://user-images.githubusercontent.com/62557093/106249157-54d63a00-6255-11eb-91d1-99aa05b05213.png">

**위에서 아래로**

탐색하면서 레이아웃이 필요하다고 mark된 모든 view에서 layoutSubviews() 호출

- Constraints를 수정 할 때 view 프레임이 즉시 변경될 것으로 기대하지 말자
- layoutSubviews를 override 해야하는 경우, 디버깅이 어려울 수 있으므로 레이아웃 피드백 루프를 피하도록 주의 할 것

https://medium.com/ios-expert-series-or-interview-series/auto-layout-cycle-in-ios-the-layout-cycle-18704d5a4ff7

### Avoiding FeedBack Loop

- Do not call `setNeedsUpdateConstraints` inside your `updateConstraints` method. Calling `setNeedsUpdateConstraints` schedules another update pass, creating a feedback loop.
- You must call the superclass’s implementation somewhere in your method.
- You can safely invalidate the layout of views in your subtree; however, you must do this before you call the superclass’s implementation.
- Don’t invalidate the layout of any views outside your subtree. This could create a feedback loop.
- Don’t call `[setNeedsLayout](<https://developer.apple.com/documentation/uikit/uiview/1622601-setneedslayout>)` inside layoutSubviews,Calling this method creates a feedback loop.
- Be careful about changing constraints. You don’t want to accidentally invalidate the layout of any views outside your subtree.

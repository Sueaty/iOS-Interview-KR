## Main run Loop

### UIApplication

UIApplication.shared 형태의 싱글턴으로 접근 가능하다. 유저의 이벤트에 반응하여 앱의 초기 설정을 하는 역할. Background에 진입한 상태에서 추가적인 작업을 할 수 있도록 만들어주거나, 푸시알람을 받았을 때 어떤 방식으로 이를 처리할 지 등에 관한 것을 다룬다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/79905096-ce2f-4df1-b854-c27ea4ccea9f/_2021-01-15__4.50.52.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/79905096-ce2f-4df1-b854-c27ea4ccea9f/_2021-01-15__4.50.52.png)

### Main Run Loop

UIApplication 객체는 앱이 실행될 때, Main Run Loop를 실행한다. Main Run Loop는 유저가 이벤트를 일으키는 이벤트들을 처리하는 프로세스이다. view와 관련된 이벤트나 view의 업데이트에 활용, view와 관련되어 있기 때문에 main 쓰레드에서 실행된다.

- 이벤트 발생 과정
  1. 이벤트 발생 (터치, 줌)
  2. 시스템을 통해 이벤트 생성
  3. UIKit 프레임워크를 통해 생성된 port로 해당 이벤트가 앱으로 전달
  4. 이벤트는 앱 내부적으로 queue의 형태로 정리되고, Main Run Loop에하나씩 mapping
  5. UIApplication 객체는 가장 먼저 이벤트를 받는 객체로 어떤 것이 실행되야 하는지 결정
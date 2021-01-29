# Http 메시지 분석

https://sabarada.tistory.com/141

## http 통신의 구성

요청 - 응답 세트

시작줄, 헤더, 바디로 구성되어 있다

## 시작 줄(Start Line)

요청 시작줄은 서버에 무엇을 해야할지, 응답 시작줄은 그 결과가 어땠는지 알려줌

### 요청

```swift
// 요청 시작줄 
GET / HTTP/1.1
```

- 메서드

  제일 처음 나오는 부분으로 서버에 어떤 행동을 하라는 것을 요청하는지 명시하는 부분. GET: 읽기, POST: 쓰기, PUT: 업데이트, DELETE: 삭제

- 요청 URL

  요청하는 URL path를 명시. 전체 url, 상대 url을 사용할 수 있으며 상대 url을 사용할 때는 base URL을 host 헤더에 명시해 주어야한다

- Http 버전 번호

  요청과 응답에 모두 명시. http 프로토콜 버전을 알려줌

### 응답

```swift
HTTP/1.1 200 OK
```

- Http 버전 번호

- 상태 코드(status code)

  클라이언트에게 서버로 요청한 일이 어떻게 마무리되었는지 알려주는 부분.

  ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33fda033-a400-4ade-8f1f-f2c8424df19a/_2021-01-30__12.25.43.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33fda033-a400-4ade-8f1f-f2c8424df19a/_2021-01-30__12.25.43.png)

  404 Error : 사용자가 요청한 페이지나 파일을 찾을 수 없을 때. 보통 페이지가 이동되거나 삭제된 경우

- 사유 구절(reason-phrase)

  숫자로 된 상태 코드를 설명해주는 짧은 문구.

## 헤더(headers)

헤더 필드는 메세지에 추가적인 정보를 더해준다.

**[일반 헤더]**

요청 메시지와 응답 메시지 모두 사용되는 헤더

- connectioin

  클라이언트와 서버의 연결을 지속할 것인지 마무리 할 것인지에 관한 정보

- date

  메시지 생성 날짜와 시간

- via

  메시지가 어떤 proxy, gateway를 통해 전달됐는지 명시

  proxy: 클라이언트와 서버 사이의 중계기. 일부는 서버에 요청된 내용을 캐시해두기도 한다. gateway: 서로 다른 통신망, 프로토콜을 사용하는 네트워크 간의 통신을 가능하게 한다. 다른 네트워크로 들어가는 네트워크 포인트. 서로 다른 네트워크 상의 통신 프로토콜을 적절히 변환해주는 역할을 한다.

## 본문(body)

http 메시지가 가지고 있는 실제 구체적 내용. text, json 메시지 뿐만 아니라 이미지, 비디오, HTML 문서 등 다양한 형식이 들어올 수 있다.


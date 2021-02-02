## Image Cache (NSCache/ FileManager)

https://nsios.tistory.com/58

이미지 캐시에는 두 가지 방법이 있다.

### 1. NSCache

메모리 캐시, 즉 기기를 끄면 사라진다

### 2. FileManager

디스크 캐시, 기기 안에 저장되어있고, 껐다 켜도 남아있음.



### 이미지 캐시 과정

1. 이미지가 memory cache에 있는지 확인
2. 이미지가 disk cache 확인, 이미지가 존재하면 memory cache에 올려둘 수 있음
3. 다 없으면 서버 통신을 통해 이미지 다운로드

------

## URLSessionDataTask vs URLSessionDownloadTask

https://stackoverflow.com/questions/20604910/what-is-difference-between-nsurlsessiondatatask-vs-nsurlsessiondownloadtask

- dataTask

  메모리에 올라감, NSData 객체를 사용하여 데이터를 보내고 받는다. 데이터를 파일에 저장하지 않으므로, 백그라운드 세션에서는 지원되지 않는다.

- downloadTask

  응답 받은 데이터를 임시 파일에 직접 쓴다. 앱이 실행되고 있지 않을 때 즉, 백그라운드에서 다운로드를 지원한다. 데이터를 파일 형식으로 얻음.



### +) 

<img width="652" alt="스크린샷 2021-02-02 오후 9 56 00" src="https://user-images.githubusercontent.com/62557093/106603278-714ddb80-65a1-11eb-8a2e-2d8820a1bf40.png">

​					다음 함수는 동기 함수이기 때문에 되도록 사용을 지양하자!! 

 
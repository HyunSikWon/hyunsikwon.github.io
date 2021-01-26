---
title: POST, 파일 업로드 요청을 URLSession 사용하여 구현해보자.
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Networking
last_modified_at: 2021-01-26 T19:10:00+08:00
---

## Performing POST and file upload requests using URLSession

`URLSession`은 iOS에서 네트워킹 작업을 위한 아주 유용한 API 이다. `URLSession`은 많은 경우에 데이터를 가져오는데(fetch) 사용되는 `GET` 요청에 초점을 맞추지만, 이번에는 `POST` 요청과 관련한 다른 HTTP 메소드를 살펴보자.

## Data and upload tasks

`URLSession`을 사용하여 `POST` 요청을 수행하는 가장 간단한 방법은 다양한 `dataTask` API를 사용하는 것이다. (이는 델리게이트와 클로저 기반 콜백을 모두 지원한다.) 무엇보다도 `URLRequest`를 사용하면 특정 네트워크 호출이 사용해야하는 `httpMethod`는 물론 보낼 `httpBody` 데이터 및 사용할 `cachePolicy`와 같은 기타 유용한 매개 변수를 커스텀 할 수 있다. 예를 들면 다음과 같다.

```swift
struct Networking {
    var urlSession = URLSession.shared

    func sendPostRequest(
        to url: URL,
        body: Data,
        then handler: @escaping (Result<Data, Error>) -> Void
    ) {
        // To ensure that our request is always sent, we tell
        // the system to ignore all local cache data:
        var request = URLRequest(
            url: url,
            cachePolicy: .reloadIgnoringLocalCacheData
        )
        
        request.httpMethod = "POST"
request.httpBody = body

        let task = urlSession.dataTask(
            with: request,
            completionHandler: { data, response, error in
                // Validate response and call handler
                ...
            }
        )

        task.resume()
    }
}
```

또는,  `uploadTask` API를 사용하여 요청 작업을 생성할 수도 있다. 앱이 백그라운드에있는 동안 데이터를 업로드 할 수 있고 작업 자체에 본문 데이터(body data)를 직접 첨부(attaching)하기위한 빌트인 지원을 제공하기도 한다.

```swift
struct Networking {
    var urlSession = URLSession.shared

    func sendPostRequest(
        to url: URL,
        body: Data,
        then handler: @escaping (Result<Data, Error>) -> Void
    ) {
        var request = URLRequest(
            url: url,
            cachePolicy: .reloadIgnoringLocalCacheData
        )
        
        request.httpMethod = "POST"

        let task = urlSession.uploadTask(
    with: request,
    from: body,
    completionHandler: { data, response, error in
        // Validate response and call handler
        ...
    }
)

        task.resume()
    }
}
```

## Observing progress updates

`POST` 요청으로 적은 양의 데이터를 보낼 때는 위의 두 가지 접근 방식은 완벽하게 작동하지만 때로는 잠재적으로 매우 큰 파일을 업로드하고 싶을 수 있다. 그렇게하면 앱의 UI가 느리거나 응답하지 않는 것처럼 보일 수 있으므로 사용자가 실시간 진행을 확인할 수 있도록 다. 이는 델리게이트 패턴을 사용하여 매우 쉽게 구현할 수 있다.

이를 구현하기 위해 `FileUploader` 클래스 (Objective-C의 `NSObject`의 하위 클래스 여야 함)를 만들어 보자. 그런 다음 `shared`가 아닌 커스텀 `URLSession` 인스턴스를 사용하면 해당 세션의 위임자(delegate)가 될 수 있다. 지정된 로컬 URL에 파일을 업로드 할 수 있는 API를 정의하고 해당 API의 호출자가 진행(progress) 이벤트 처리와 컴플리션 핸들러를 위한 두 개의 클로저를 전달한다. 마지막으로, 각 업로드 딕셔너리에 작업의 ID를 기반으로 모든 진행 이벤트 핸들러를 미리 저장하면, 델리게이트 프로토콜 구현 내에서 이러한 클로저를 호출 할 수 있다.

```swift
class FileUploader: NSObject {
    // We'll define a few type aliases to make our code easier to read:
    typealias Percentage = Double
    typealias ProgressHandler = (Percentage) -> Void
    typealias CompletionHandler = (Result<Void, Error>) -> Void

    // Creating our custom URLSession instance. We'll do it lazily
    // to enable 'self' to be passed as the session's delegate:
    private lazy var urlSession = URLSession(
        configuration: .default,
        delegate: self,
        delegateQueue: .main
    )

    private var progressHandlersByTaskID = [Int : ProgressHandler]()

    func uploadFile(
        at fileURL: URL,
        to targetURL: URL,
        progressHandler: @escaping ProgressHandler,
completionHandler: @escaping CompletionHandler
    ) {
        var request = URLRequest(
            url: targetURL,
            cachePolicy: .reloadIgnoringLocalCacheData
        )
        
        request.httpMethod = "POST"

        let task = urlSession.uploadTask(
            with: request,
            fromFile: fileURL,
            completionHandler: { data, response, error in
                // Validate response and call handler
                ...
            }
        )

        progressHandlersByTaskID[task.taskIdentifier] = progressHandler
        task.resume()
    }
}
```

업로드 예정..

## A stream of progress over time

## Conclusion

## Reference

[Swift by Sundell](https://www.swiftbysundell.com/articles/http-post-and-file-upload-requests-using-urlsession/)

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

`URLSession`은 iOS에서 네트워킹 작업을 위한 아주 유용한 API 이다. `URLSession`은 대부분 데이터를 가져오는데(fetch) 사용되는 `GET` 요청(request)에 초점을 맞추지만, 이번에는 `POST` 요청과 관련한 HTTP 메소드를 살펴보자.

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

또는, `uploadTask` API를 사용하여 요청 작업을 생성할 수도 있다. 앱이 백그라운드에있는 동안 데이터를 업로드 할 수 있고 작업 자체에 본문 데이터(body data)를 직접 첨부(attaching)하기 위한 빌트인 지원을 제공하기도 한다.

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

`POST` 요청으로 적은 양의 데이터를 보낼 때는 위의 두 가지 접근 방식은 완벽하게 작동하지만 때로는 잠재적으로 매우 큰 파일을 업로드하고 싶을 수 있다. 이 상황에선 앱의 UI가 느리거나 응답하지 않는 것처럼 보일 수 있으므로 사용자가 실시간 진행을 확인할 수 있도록 해야한다. 이는 델리게이트 패턴을 사용하여 매우 쉽게 구현할 수 있다.

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

이제, `URLSessionTaskDelegate` 프로토콜을 구현해보자. 이 프로토콜은 작업 단위 이벤트를 관찰(observe) 할 수 있는 몇가지 메소드를 정의한다. 주어진 `URLSessionTask`의 진행 사항이 업데이트 될 때 사용자에게 알리는 메소드는 다음과 같이 구현할 수 있다. 

```swift
extension FileUploader: URLSessionTaskDelegate {
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didSendBodyData bytesSent: Int64,
        totalBytesSent: Int64,
        totalBytesExpectedToSend: Int64
    ) {
        let progress = Double(totalBytesSent) / Double(totalBytesExpectedToSend)
        let handler = progressHandlersByTaskID[task.taskIdentifier]
        handler?(progress)
    }
}
```

위와 같이 이제 각 `progressHandler` 클로저에 백분율 값을 전달하여 `ProgressView`, `UIProgressView` 또는 `NSProgressIndicato`와 같은 업로드 진행률을 시각화하는데 사용하는 모든 UI 구성 요소를 구현 할 수 있다.

## A stream of progress over time

마지막으로 위에서 본 `FileUploader`을 클로저를 사용하는 것에서 Combine을 사용하는 방법으로 바꿔보자. Combine의 "시간 경과에 따른 값"에 초점을 맞춘 디자인은 진행률 업데이트 모델링에 매우 적합하다. 시간이 지남에 따라 여러 백분율 값을 전송 한 다음 완료 이벤트로 끝내고 싶기 때문이다. 이는 바로 Combine 퍼블리셔(publisher)가 하는 역할과 같다.

사용자 정의(custom) 퍼블리셔를 사용하여 이 기능을 구현할 수도 있지만 이 경우에는 값을 캐시하고 모든 새 구독자에게 전송 하는 방식인 `CurrentValueSubject`를 사용하자. 이렇게하면 각 업로드 작업을 주어진 subject와 연결할 수 있고 (이전에 각 `progressHandler` 클로저를 저장 한 방식과 마찬가지로),  `eraseToAnyPublisher` API를 사용하여 해당 subject를 퍼블리셔로 반환 할 수도 있다.

```swift
class FileUploader: NSObject {
    typealias Percentage = Double
    typealias Publisher = AnyPublisher<Percentage, Error>
    
    private typealias Subject = CurrentValueSubject<Percentage, Error>

    private lazy var urlSession = URLSession(
        configuration: .default,
        delegate: self,
        delegateQueue: .main
    )

    private var subjectsByTaskID = [Int : Subject]()

    func uploadFile(at fileURL: URL,
                    to targetURL: URL) -> Publisher {
        var request = URLRequest(
            url: targetURL,
            cachePolicy: .reloadIgnoringLocalCacheData
        )
        
        request.httpMethod = "POST"

        let subject = Subject(0)
        var removeSubject: (() -> Void)?

        let task = urlSession.uploadTask(
            with: request,
            fromFile: fileURL,
            completionHandler: { data, response, error in
                // Validate response and send completion
                ...
                subject.send(completion: .finished)
                removeSubject?()
            }
        )

        subjectsByTaskID[task.taskIdentifier] = subject
        removeSubject = { [weak self] in
            self?.subjectsByTaskID.removeValue(forKey: task.taskIdentifier)
        }
        
        task.resume()
        
        return subject.eraseToAnyPublisher()
    }
}
```

이제 남은 작업은 `URLSessionTaskDelegate`를 갱신하여 각 진행 값을 업로드 작업과 관련된 subject에게 전송하도록 구현하는 것이다.

```swift
extension FileUploader: URLSessionTaskDelegate {
    func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        didSendBodyData bytesSent: Int64,
        totalBytesSent: Int64,
        totalBytesExpectedToSend: Int64
    ) {
        let progress = Double(totalBytesSent) / Double(totalBytesExpectedToSend)
        let subject = subjectsByTaskID[task.taskIdentifier]
        subject?.send(progress)
    }
}
```

## Conclusion
위에서 본 것처럼 우리가 구현한 작업이 완전한 네트워킹 라이브러리는 아니다. 하지만, `URLSession`이 제공하는 내장 기능이 데이터 게시 또는 파일 업로드를 포함한 다양한 종류의 네트워크 요청을 수행하는 데 필요한 것임을 알 수 있다.

## Reference

[Swift by Sundell](https://www.swiftbysundell.com/articles/http-post-and-file-upload-requests-using-urlsession/)

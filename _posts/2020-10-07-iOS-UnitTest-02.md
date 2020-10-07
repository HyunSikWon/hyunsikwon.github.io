---
title: iOS - Unit Test2
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Unit Test
last_modified_at: 2020-10-07 T23:40:00+08:00

---

[iOS-Unit Test](https://hyunsikwon.github.io/ios/iOS-UnitTest-01/)에 이어지는 글 입니다.


## Using XCTestExpectation to Test Asynchronous Operations

모델을 테스트하고, 테스트에서 발견된 오류를 수정하는 것을 배웠으니 이제 비동기 코드를 테스트 해보자.

`HalfTunes` 프로젝트를 통해 테스트를 진행하자. 이 프로젝트는 `URLSession`을 통해 iTunes API를 조회하여 노래 샘플을 다운로드 받는다. 우리는 이 네트워크 작업을 `URLSession` 에서  `Alamofire` 를 이용한 방식으로 변경하길 원한다고 가정하자. 문제점이 있는지 찾기 위해서 네트워크 작업에 대한 테스트를 작성하고 테스트를 수행할 것이다. 

`URLSession` 메소드는 비동기적이다. 비동기 메소드를 테스트하기 위해서 테스트가 비동기 작업이 완전히 끝날 때까지 기다리게 해주는  `XCTestExpectation` 을 사용할 것이다. 

비동기 테스트는 매우 느리기 때문에, 속도가 빠른 단위 테스트와 분리해서 실행해야 한다.

`HalfTunesTests`라는 새로운 unit test target을 생성하자. `HalfTunesTests` 클래스를 열고 `import`문 밑에 HalfTunes 앱 모듈을 import 하자.

```swift
@testable import HalfTunes
```

이 클래스의 모든 테스트는 애플 서버에 요청을 전달하기 위해서 default `URLSession` 을 사용한다.  `sut` 객체를 선언하여 `setUp()` 에 생성하고, `tearDown()` 에서 반환하자.

`HalfTunesTests` 클래스의 내용을 다음과 같이 수정하자:

```swift
var sut: URLSession!

override func setUp()  {
  super.setUp()
  sut = URLSession(configuration: .default)
}

override func tearDown()  {
  sut = nil
  super.tearDown()
}
```

다음으로 비동기 테스트를 추가하자. 

```swift
// Asynchronous test: success fast, failure slow
func testValidCallToiTunesGetsHTTPStatusCode200() {
  // given
  let url = 
    URL(string: "https://itunes.apple.com/search?media=music&entity=song&term=abba")
  // 1
  let promise = expectation(description: "Status code: 200")

  // when
  let dataTask = sut.dataTask(with: url!) { data, response, error in
    // then
    if let error = error {
      XCTFail("Error: \(error.localizedDescription)")
      return
    } else if let statusCode = (response as? HTTPURLResponse)?.statusCode {
      if statusCode == 200 {
        // 2
        promise.fulfill()
      } else {
        XCTFail("Status code: \(statusCode)")
      }
    }
  }
  dataTask.resume()
  // 3
  wait(for: [promise], timeout: 5)
}
```

이 테스트는 유효한 질의(query)를 iTunes에 보냈을 때 상태 코드 200(성공)이 반환 되는지 검사한다. 대부분이 코드가 앱의 코드와 동일하지만 아래와 같은 코드가 추가됐다:

1. `expectation(description:)` : `priomise` 에 저장된  `XCTestExpectation` 객체를 반환한다. `description` 파라미터는 기대 결과를 설명한다.
2. `promise.fullfill()` :  예상이 맞았다는 것을 표시하기 위해서 비동기 메소드의 completion handler 클로저의 성공 조건에서 호출한다. 
3. `wait(for:timeout:)` :  모든 예상이 이행되거나, `time` 이 끝날 때까지 테스트가 수행되도록 한다.

테스트를 진행해보자. 만약 인터넷에 연결되어 있다면, 앱이 시뮬레이터에 로드되면 테스트는 성공할 것이다. 

## Failing Fast

이 테스트를 실패하도록 하려면 URL의 "itunes"에서 's' 를 제거하면 된다.

```swift
let url = 
  URL(string: "https://itune.apple.com/search?media=music&entity=song&term=abba")
```

테스트를 진행 해보자. 테스트는 실패하지만, 모든 timeout interval를 소진한다. 네트워크 요청이 항상 성공할 것이라고 가정하였기 때문에 `promise.fullfill()` 은 네트워크 요청이 성공해야만 호출된다. 따라서 요청이 실패하면 `timeout` 이 소진돼야만 테스트가 종료된다. 

가정을 바꾸면 이를 개선하고 더 빠른 테스트 실패 결과를 만들수 있다: 네트워크 요청이 성공될 때까지 기다리는 대신, 비동기 메소드의 completion handler 클로저가 수행될 때까지만 기다리면 된다. 이는 앱이 서버로부터 OK 혹은 error 응답을 받는 즉시 일어날 수 있고, 따라서 `promise.fullfill()` 가 수행된다. 이를 통해서 테스트는 요청의 성공 유무를 바로 검사할 수 있다.

위와 같은 동작 방식을 보기위해서 새로운 테스트 코드를 작성하자.

```swift
func testCallToiTunesCompletes() {
  // given
  let url = 
    URL(string: "https://itune.apple.com/search?media=music&entity=song&term=abba")
  let promise = expectation(description: "Completion handler invoked")
  var statusCode: Int?
  var responseError: Error?

  // when
  let dataTask = sut.dataTask(with: url!) { data, response, error in
    statusCode = (response as? HTTPURLResponse)?.statusCode
    responseError = error
    promise.fulfill()
  }
  dataTask.resume()
  wait(for: [promise], timeout: 5)

  // then
  XCTAssertNil(responseError)
  XCTAssertEqual(statusCode, 200)
}
```

중요한 차이는 단순히 completion handler가 expectation을 이행(fullfill)하도록 추가한 것 뿐이다. 만약 네트워크 요청을 실패하면 `then` 은 실패를 선언한다. 

테스트를 실행해보자. 실패까지 몇 초 정도 걸릴 것이다. 이 실패는 네트워크 요청을 실패해서 발생한 것이지 테스트의 `timeout` 이 소진돼서가 아니다.

url을 올바르게 고쳐서 테스트가 성공적으로 수행되는지 확인해보자.

# Faking Objects and Interactions

비동기 테스트는 작성한 코드가 비동기 API에 정확한 입력을 생성한다는 신뢰를 줄 수 있다. 우리는 우리의 코드가  `URLSession`으로부터 입력을 받을 때 잘 동작 하는지, `UserDefault` 혹은 `iCloud` 를 적절하게 업데이트 하는지 검사하고 싶을 것이다.

대부분의 앱은 개발자가 통제하지 않는 시스템, 라이브러리 객체와 상호작용 한다. 그리고 이러한 객체와의 상호작용을 테스트 하는 것은 매우 느리고 반복 가능하지 않아서 `FIRST` 이 두가지 원칙에 위배된다. 대신에 `stub` 입력을 사용하거나 `mock` 객체를 업데이트하여 상호작용을 조작할 수 있다. 

코드에 시스템 혹은 라이브러리 객체와의 의존성이 존재하면 가짜 데이터를 만들어라. 의존성이 존재하는 부분에서 수행될 가짜 객체를 만들고 코드에 삽입하면 된다.

## Fake Input From Stub

이 테스트에서는 `searchResults.count` 가 정확한지 확인하여  `updateSearchResults(_:)` 가 session에 의해 다운되는 데이터를 정확하게 파싱하는지 검사할 것이다. SUT는 view controller이고 `stubs` 과 미리 다운로드된 데이터를 이용해 session을 조작할 것이다. 

Test Navigator에서 새로운 unit test target을 생성하여 `HalfTunesFakeTests` 라는 이름을 붙이자. `HalfTunesFakeTests.swift` 를 열어 다음을 추가하자:

```swift
@testable import HalfTunes
```

`HalfTunesFakeTests` 의 내용들을 다음으로 변경하자:

```swift
var sut: SearchViewController!

override func setUp() {
  super.setUp()
  sut = UIStoryboard(name: "Main", bundle: nil)
    .instantiateInitialViewController() as? SearchViewController
}

override func tearDown() {
  sut = nil
  super.tearDown()
}
```

> NOTE: 이 프로젝트는 `SearchViewController` 에서 모든 작업이 이루어지기 때문에 - massive view controller -  SUT가 view controller다.

다음으로는, fake session이 테스트에 제공할 샘플 JSON 데이터가 필요하다. 몇개의 데이터만 있으면 되기 때문에   url에 `&limit=3` 을 추가하여 다운로드 결과에 제한을 주자.

```swift
https://itunes.apple.com/search?media=music&entity=song&term=abba&limit=3 
```

브라우저에 이 URL을 입력하여 파일을 다운받은 후 `abbaData.json` 으로 이름을 바꾸고 Xcode의 project navigator로 가서 `HalfTunesFakeTests` 그룹에 파일을 추가해라.

`HalfTunes` 프로젝트는 테스트를 도와주기 위한 `DHURLSession.swift` 를 포함한다. `DHURLSession.swift`는 `HRULSession` 이라는 간단한 프로토콜을 정의하고, 여기에는 `URL` 혹은 `URLSession` 을 통한 작업을 생성하기 위한 메소드(stubs)가 있다. 또한, 이 프로토콜을 채택하는 `URLSessionMock` 을 정의하고 `URLSessionMock`에는 mock `URLSession` 객체를 생성하는 initializer가 있다.

fake를 만들기 위해서 `HalfTunesFakeTest.swift` 로 가서 `setUP()` 의 SUT를 생성하는 구문 다음에 다음과 같은 코드를 추가하자:

```swift
let testBundle = Bundle(for: type(of: self))
let path = testBundle.path(forResource: "abbaData", ofType: "json")
let data = try? Data(contentsOf: URL(fileURLWithPath: path!), options: .alwaysMapped)

let url = 
  URL(string: "https://itunes.apple.com/search?media=music&entity=song&term=abba")
let urlResponse = HTTPURLResponse(
  url: url!, 
  statusCode: 200, 
  httpVersion: nil, 
  headerFields: nil)

let sessionMock = URLSessionMock(data: data, response: urlResponse, error: nil)
sut.defaultSession = sessionMock
```

fake 데이터와 response를 설정하고 fake session 객체를 생성했다. 그리고 최종적으로 fake session을 sut의 프로퍼티로 앱에 주입하였다.

이제 `updateSearchResults(_:)` 호출하면 fake data를 parsing 하는지 검사하는 테스트를 작성할 준비가 되었다. 다음 테스트를 추가하자:

```swift
func test_UpdateSearchResults_ParsesData() {
  // given
  let promise = expectation(description: "Status code: 200")

  // when
  XCTAssertEqual(
    sut.searchResults.count, 
    0, 
    "searchResults should be empty before the data task runs")
  let url = 
    URL(string: "https://itunes.apple.com/search?media=music&entity=song&term=abba")
  let dataTask = sut.defaultSession.dataTask(with: url!) {
    data, response, error in
    // if HTTP request is successful, call updateSearchResults(_:) 
    // which parses the response data into Tracks
    if let error = error {
      print(error.localizedDescription)
    } else if let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 200 {
      self.sut.updateSearchResults(data)
    }
    promise.fulfill()
  }
  dataTask.resume()
  wait(for: [promise], timeout: 5)

  // then
  XCTAssertEqual(sut.searchResults.count, 3, "Didn't parse 3 items from fake response")
}
```

`stub` 은 비동기 메소드인척 하고있기 때문에 테스트 역시 비동기 테스트로 작성해야 한다.

*when* `searchResults` 가 data 작업을 하기 전에는 비었다는 것을 단언한다. 이는 `setUP()` 에서 완전히 새로운 SUT를 생성했기 때문에 참이 되어야한다.

fake 데이터가 3개의 `Track` 객체를 위한 JSON을 포함하기 때문에 *then*은 view controller의 `searchResults` 배열의 원소가 3개라는 것을 단언한다.

테스트를 실행해보자! 실제로는 어떤 네트워크 작업도 없기 때문에 빠른 속도로 테스트가 성공할 것이다.

## Fake Update to Mock Object

이전 테스트에서는 fake 객체로부터의 입력을 제공하기 위해 stub을 사용했다면, 이제는 코드가 `UserDefault` 를 정확하게 업데이트 했는지 검사하기 이위해서  **mock object** 를 사용해보자.

**BullsEye** 프로젝트를 이용해서 테스트를 진행할 것이다. 이 앱은 두 가지 방식의 게임이 있다. 사용자가 슬라이더를 움직여서 target value를 맞추는 방식의 게임과 slider의 위치 값을 맞추는 방식의 게임이 있다. 우측 아래의 segmented control는 게임 스타일을 바꾸고, 게임 스타일을 `UserDefault` 에 저장한다.

이 테스트는 앱이 `gameStyle` 프로퍼티를 정확하게 저장하는지 검사하는 테스트이다.

Test navigator에서 **New Unit Test Class**를 클릭하고 `BullsEyeMockTests` 라고 이름을 붙여주자.

다음의 코드를 `import` 밑에 추가하자

```swift
@testable import BullsEye

class MockUserDefaults: UserDefaults {
  var gameStyleChanged = 0
  override func set(_ value: Int, forKey defaultName: String) {
    if defaultName == "gameStyle" {
      gameStyleChanged += 1
    }
  }
}
```

 

`MockUserDefaults` 는 `gameStyleChanged` flag를 증가시키기 위해서  `set(_:forKey:)`를 오버라이드 한다. 종종 `Bool` 타입을 써서 비슷한 테스트를 진행하는 것을 볼 수 있는데, `Int` 타입이 값을 증가시키는 것이 테스트에 유연함을 더 줄 수 있다. - 예를들어, 메소드가 호출됐는지를 한번에 검사할 수 있다.

SUT와 mock object를 선언하자

```swift
var sut: ViewController!
var mockUserDefaults: MockUserDefaults!
```

그리고 `setUp()`과 `tearDown()`을 다음과 같이 수정하자:

```swift
override func setUp() {
  super.setUp()

  sut = UIStoryboard(name: "Main", bundle: nil)
    .instantiateInitialViewController() as? ViewController
  mockUserDefaults = MockUserDefaults(suiteName: "testing")
  sut.defaults = mockUserDefaults
}

override func tearDown() {
  sut = nil
  mockUserDefaults = nil
  super.tearDown()
}
```

SUT와 mock object를 생성했고, mock object를 SUT의 프로퍼티로 주입했다.

이제 테스트 코드를 추가하자:

```swift
func testGameStyleCanBeChanged() {
  // given
  let segmentedControl = UISegmentedControl()

  // when
  XCTAssertEqual(
    mockUserDefaults.gameStyleChanged, 
    0, 
    "gameStyleChanged should be 0 before sendActions")
  segmentedControl.addTarget(sut,
    action: #selector(ViewController.chooseGameStyle(_:)), for: .valueChanged)
  segmentedControl.sendActions(for: .valueChanged)

  // then
  XCTAssertEqual(
    mockUserDefaults.gameStyleChanged, 
    1, 
    "gameStyle user default wasn't changed")
}
```

*when*에서는 테스트 메소드가 segmented control을 변경시키지 않았기 때문에 `gameStyleChanged` flag가 0이다. 그렇기 때문에 *then* assertion이 참이라면 `set(_:forKey:)` 가 한번만 호출되었다는 뜻이다.

테스트를 실행 해보면 성공적으로 수행되는 것을 볼 수 있다.


---
공부하고 정리와 번역을하면서 어느정도 이해는 됐지만 완벽하게 이해하기는 힘들었다.

몇번 더 읽어보고 공부해야겠다.

실제 프로젝트를 진행하면서 테스트를 수행해보는 것도 도움이 될듯.
